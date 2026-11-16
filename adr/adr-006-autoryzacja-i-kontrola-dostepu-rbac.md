# ADR-006: Autoryzacja i kontrola dostępu oparta na rolach (RBAC) - Spatie Laravel Permission

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

W systemie występują wyraźnie rozgraniczone uprawnienia zależne od roli użytkownika:
* **Gość (niezalogowany):** odczyt mapy zgłoszeń, szczegółów zgłoszeń oraz publicznych statystyk.
* **Mieszkaniec (`user`):** zgłaszanie nowych usterek, oznaczanie „Ten problem również mnie dotyczy”, obserwowanie zgłoszeń i odbieranie powiadomień.
* **Pracownik Urzędu (`official`):** zmiana statusów zgłoszeń (`zgłoszone`, `przyjęte`, `przekazane do realizacji`, `w realizacji`, `rozwiązane`, `odrzucone`), dodawanie publicznych komentarzy dotyczących realizacji oraz załączanie zdjęć po naprawie.
* **Administrator (`admin`):** dostęp do mapy ciepła obszarów z największą liczbą zgłoszeń, zarządzanie użytkownikami i ich rolami, zarządzanie kategoriami oraz podgląd logów z aplikacji.

Należało wybrać mechanizm implementacji kontroli dostępu po stronie API oraz warstwy klienckiej.

---

## 2. Rozważane opcje

### Opcja 1: Pojedyncza kolumna tekstowa `role` w tabeli `users`
* **Opis:** Przechowywanie nazwy roli jako zwykłego stringa (`user`, `official`, `admin`) bezpośrednio w rekordzie użytkownika i sprawdzanie jej warunkami `if ($user->role === 'admin')`.
* **Zalety:** Bardzo prosta struktura tabeli.
* **Wady:** Brak możliwości przypisania wielu ról jednemu użytkownikowi (np. administratora będącego jednocześnie pracownikiem urzędu) oraz brak granularnych uprawnień (*permissions*).

### Opcja 2: Pakiet `spatie/laravel-permission` + Laravel Policies
* **Opis:** Zastosowanie standardowego w ekosystemie Laravel pakietu realizującego wzorzec RBAC (*Role-Based Access Control*) w powiązaniu z klasami `Policy` oraz middleware’ami tras (`role:official|admin`, `permission:manage-categories`).
* **Zalety:**
  * Gotowa, zoptymalizowana struktura tabel relacyjnych (`roles`, `permissions`, `model_has_roles`, `role_has_permissions`) wraz z wbudowanym cache’owaniem uprawnień.
  * Czytelne zabezpieczanie grup endpointów w `routes/api.php` za pomocą middleware.
  * Łatwa obsługa zarządzania rolami użytkowników z poziomu panelu administratora (`$user->syncRoles(...)`).
* **Wady:** Dodatkowe tabele w bazie danych (generowane automatycznie z migracji pakietu).

---

## 3. Decyzja

Wybrano **Opcję 2**: Wykorzystanie pakietu **`spatie/laravel-permission`** w połączeniu z natywnymi **Laravel Policies** na backendzie oraz komponentami strażników tras (**ProtectedRoleRoute**) po stronie React SPA.

---

## 4. Konsekwencje

### Pozytywne:
* **Wielowarstwowa ochrona:** Każde żądanie do endpointów urzędniczych (`/api/v1/official/*`) oraz administracyjnych (`/api/v1/admin/*`) jest weryfikowane na poziomie middleware w Laravelu. Nawet przy próbie ręcznego wywołania API z pominięciem interfejsu React, nieuprawniony użytkownik otrzyma kod `403 Forbidden`.
* **Elastyczność ról:** Administrator może w panelu administracyjnym w prosty sposób awansować konto na rolę `official` (Pracownik Urzędu) lub `admin`.
* **Spójność UI:** Endpoint `/api/v1/auth/me` zwraca tablicę przypisanych ról i uprawnień, na podstawie których aplikacja React dynamicznie renderuje zakładki „Panel Urzędnika” lub „Panel Administratora (Heatmap, Użytkownicy, Kategorie, Logi)”.

### Negatywne i mitygacja:
* **Konieczność synchronizacji ról przy seedowaniu bazy:** Przygotowano dedykowany `RoleAndPermissionSeeder`, który podczas inicjalizacji bazy tworzy role `user`, `official` oraz `admin` wraz z kontami demonstracyjnymi dla każdej roli.
