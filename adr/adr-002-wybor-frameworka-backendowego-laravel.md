# ADR-002: Wybór frameworka backendowego - Laravel (API-only + Service-Repository)

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Warstwa serwerowa platformy odpowiada za obsługę uwierzytelniania (klasycznego i OAuth 2.0), kontrolę uprawnień trzech ról użytkowników, walidację i składowanie zdjęć usterek, obliczenia geograficzne dla pobliskich zgłoszeń, obsługę powiadomień po zmianie statusu, agregację statystyk publicznych oraz zbieranie logów aplikacyjnych. Potrzebny jest dojrzały framework backendowy, który zapewni gotowe mechanizmy bezpieczeństwa i pozwoli utrzymać czystą architekturę kodu.

---

## 2. Rozważane opcje

### Opcja 1: Laravel (PHP 8.3+) w trybie API-only
* **Opis:** Wykorzystanie frameworka Laravel z wyłączonym renderowaniem widoków HTML, zorganizowanego wokół wzorca warstwowego (Kontrolery API -> Form Requests -> Serwisy biznesowe -> Modele Eloquent / Repozytoria -> API Resources).
* **Zalety:** Bogaty ekosystem oficjalnych pakietów (Sanctum, Socialite), wbudowany system powiadomień (`Notifications`), kolejki zadań, zaawansowany ORM Eloquent, wbudowana obsługa walidacji i przechowywania plików (`Storage`), doskonała integracja ze Swaggerem (`l5-swagger`) oraz uprawnieniami (`spatie/laravel-permission`).
* **Wady:** Nieco większy narzut zasobów niż w przypadku mikroframeworków.

### Opcja 2: Symfony (PHP)
* **Opis:** Zbudowanie API na bazie komponentów Symfony oraz API Platform.
* **Zalety:** Wysoka modularność i elastyczność konfiguracji.
* **Wady:** Wyższy próg wejścia dla szybkiego prototypowania oraz większy nakład pracy przy konfiguracji OAuth, powiadomień i uploadu mediów w porównaniu z ekosystemem Laravel.

### Opcja 3: Lekki mikroframework (np. Slim PHP / Lumen)
* **Opis:** Minimalistyczny szkielet obsługujący wyłącznie routing i żądania HTTP.
* **Zalety:** Niskie zużycie pamięci.
* **Wady:** Konieczność ręcznego doinstalowywania i integrowania bibliotek do OAuth, powiadomień, autoryzacji ról, walidacji i migracji bazy danych (Lumen jest ponadto oficjalnie wycofywany na rzecz pełnego Laravela).

---

## 3. Decyzja

Wybrano **Opcję 1**: Framework **Laravel (PHP 8.3+)** skonfigurowany jako aplikacja **API-only**, z wydzieleniem logiki biznesowej do warstwy serwisów (**Service Pattern**) oraz transformacją odpowiedzi JSON przez **Laravel API Resources**.

---

## 4. Konsekwencje

### Pozytywne:
* Błyskawiczna implementacja wymaganych funkcji dzięki natywnym modułom: `Laravel Sanctum` (API auth), `Laravel Socialite` (OAuth), `Laravel Notifications` (powiadomienia o zmianie statusu zgłoszenia) oraz `Filesystem Storage` (zdjęcia przed i po naprawie).
* Zastosowanie klas `FormRequest` gwarantuje spójną walidację danych i automatyczne zwracanie kodów HTTP `422 Unprocessable Entity` w formacie JSON.
* Zastosowanie `JsonResource` zapewnia jednolitą strukturę odpowiedzi REST API zgodną ze schematami zdefiniowanymi w OpenAPI.

### Negatywne i mitygacja:
* **Ryzyko umieszczania nadmiernej logiki w kontrolerach lub modelach (tzw. Fat Controllers):** Wprowadzono konwencję projektową przenoszącą złożone operacje (np. wyszukiwanie duplikatów geograficznych, zmiana statusu wraz z wysyłką powiadomień i zapisem logu audytowego) do dedykowanych klas serwisowych (np. `IssueSimilarityService`, `IssueStatusWorkflowService`).
