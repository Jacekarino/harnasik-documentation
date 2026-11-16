# ADR-012: Centralne logowanie zdarzeń aplikacyjnych i audytowych dla panelu Administratora

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Zgodnie z wymaganiami funkcjonalnymi projektu **administrator musi widzieć logi z aplikacji** bezpośrednio w swoim panelu zarządzania. Standardowe logi frameworka Laravel zapisywane wyłącznie do pliku tekstowego `storage/logs/laravel.log` w formie wieloliniowych zrzutów stosu (*stack traces*) są trudne do wydajnego stronicowania, filtrowania po typie zdarzenia lub użytkowniku oraz bezpiecznego prezentowania przez REST API w aplikacji React SPA.

Należało zaprojektować rozwiązanie, które rejestruje zarówno kluczowe zdarzenia biznesowe (audytowe), jak i błędy systemowe, oraz udostępnia je administratorowi w ustrukturyzowanej formie.

---

## 2. Rozważane opcje

### Opcja 1: Parsowanie surowego pliku `storage/logs/laravel.log` wyrażeniami regularnymi przy każdym żądaniu API
* **Opis:** Odczytywanie pliku tekstowego z dysku i dzielenie go na wpisy w kontrolerze.
* **Zalety:** Brak dodatkowej tabeli w bazie danych.
* **Wady:** Niska wydajność przy dużych plikach logów; brak indeksowania po dacie, poziomie ważności (`level`), użytkowniku (`user_id`) czy adresie IP; podatność na błędy parsowania przy wieloliniowych wyjątkach.

### Opcja 2: Zewnętrzny stos ELK (Elasticsearch + Logstash + Kibana) lub Sentry
* **Opis:** Wysyłanie logów do zewnętrznego serwera indeksującego.
* **Zalety:** Zaawansowane możliwości wyszukiwania pełnotekstowego.
* **Wady:** Ogromny narzut infrastrukturalny nieadekwatny do projektu studenckiego i brak bezpośredniej integracji wewnątrz własnego panelu administratora w React.

### Opcja 3: Hybrydowy system logowania - Tabela `system_logs` w bazie danych (kanał audytowy i błędów) + rotacyjne pliki Monolog
* **Opis:**
  * Utworzenie dedykowanej tabeli `system_logs` zawierającej kolumny: `id`, `user_id` (nullable), `level` (`INFO`, `WARNING`, `ERROR`, `AUDIT`), `action` (np. `auth.login`, `issue.created`, `issue.status_changed`, `category.updated`, `user.role_changed`, `system.exception`), `message`, `context` (JSON), `ip_address`, `user_agent`, `created_at`.
  * Zaimplementowanie serwisu `SystemLoggerService` oraz niestandardowego kanału Monolog / integracji z globalnym uchwytem wyjątków (`bootstrap/app.php` -> `withExceptions`), który automatycznie zapisuje w tabeli `system_logs`:
    1. Wszystkie kluczowe akcje użytkowników, urzędników i administratorów (logi audytowe).
    2. Nieobsłużone wyjątki i błędy aplikacyjne (logi błędów `ERROR`).
  * Wystawienie zabezpieczonego endpointu `GET /api/v1/admin/logs` obsługującego stronicowanie (paginację), filtrowanie po poziomie (`level`), typie akcji (`action`), dacie oraz wyszukiwanie tekstowe.

---

## 3. Decyzja

Wybrano **Opcję 3**: Dedykowana tabela **`system_logs`** zasilana przez serwis audytowy oraz globalny handler wyjątków Laravela, udostępniana administratorowi przez stronicowany endpoint REST API.

---

## 4. Konsekwencje

### Pozytywne:
* **Szybki i czytelny interfejs w panelu Administratora:** Administrator widzi w aplikacji React przejrzystą tabelę logów z kolorowymi oznaczeniami poziomów (`AUDIT`, `INFO`, `WARNING`, `ERROR`), informacją kto wykonał akcję, z jakiego adresu IP oraz podglądem szczegółowego kontekstu JSON (np. poprzedni i nowy status zgłoszenia).
* **Błyskawiczne filtrowanie i paginacja:** Dzięki indeksom bazodanowym na kolumnach `(level, created_at)` oraz `action`, przeszukiwanie logów przez REST API jest natychmiastowe.
* **Zachowanie kopii plikowej:** Niezależnie od zapisu w tabeli `system_logs`, standardowy kanał plikowy `daily` w `storage/logs/` pozostaje aktywny jako niskopoziomowe zabezpieczenie na wypadek awarii samej bazy danych.

### Negatywne i mitygacja:
* **Przyrost liczby rekordów w tabeli `system_logs`:** Dodano komendę konsolową `php artisan logs:prune` (lub regułę `Prunable` w modelu Eloquent), która pozwala automatycznie czyścić wpisy starsze niż zadana liczba dni (np. 90 dni).
