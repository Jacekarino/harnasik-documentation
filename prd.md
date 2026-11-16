# Product Requirements Document (PRD): HARNASIK
Platforma Zgłaszania i Obsługi Usterek Infrastruktury Miejskiej

| Metryczka dokumentu | Wartość |
| :--- | :--- |
| **Nazwa projektu** | Platforma do zgłaszania i obsługi usterek infrastruktury miejskiej |
| **Wersja dokumentu** | 1.0 |
| **Status** | Zatwierdzony |
| **Data opracowania** | 2026-10-31 |
| **Autorzy** | Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk |
| **Stos technologiczny** | Laravel, React, TypeScript, Tailwind CSS, SQLite |

---

## 1. Wstęp i cel projektu

### 1.1. Opis problemu
Mieszkańcy miast i gmin na co dzień napotykają liczne usterki infrastruktury publicznej - takie jak uszkodzone latarnie uliczne, ubytki w jezdni i chodnikach, zniszczone znaki drogowe, przepełnione kosze lub dzikie wysypiska śmieci oraz zdewastowane elementy małej architektury (np. ławki, wiaty przystankowe). Tradycyjne kanały zgłaszania problemów (telefon do urzędu, poczta elektroniczna, wizyta osobista) są nieefektywne, czasochłonne i nie zapewniają transparentnej informacji zwrotnej o postępach naprawy. Z kolei pracownicy urzędu borykają się z problemem wielokrotnego zgłaszania tej samej awarii przez różne osoby oraz brakiem narzędzi do przestrzennej wizualizacji skali problemów.

### 1.2. Cel produktu
Celem projektu jest stworzenie nowoczesnej, responsywnej aplikacji webowej działającej w modelu **Headless REST API + SPA**, która:
1. Umożliwi mieszkańcom błyskawiczne zgłaszanie usterek z wykorzystaniem geolokalizacji GPS lub interaktywnej mapy oraz załączników fotograficznych.
2. Ograniczy liczbę zduplikowanych zgłoszeń dzięki automatycznemu wyszukiwaniu podobnych problemów w pobliżu oraz mechanizmowi poparcia istniejącego zgłoszenia („Ten problem również mnie dotyczy”).
3. Zapewni pełną transparentność procesu obsługi poprzez publiczny podgląd statusów, komentarzy urzędowych, dokumentacji fotograficznej po naprawie oraz statystyk realizacji.
4. Wyposaży pracowników urzędu i administratorów w ergonomiczny panel zarządzania zgłoszeniami, kategoriami, użytkownikami, logami systemowymi oraz analityczną mapą ciepła (heatmap).

---

## 2. Grupy docelowe i role użytkowników (Personas)

W systemie zdefiniowano cztery poziomy dostępu:

1. **Gość (Użytkownik niezalogowany)**
   * Może przeglądać interaktywną mapę zgłoszeń, szczegóły poszczególnych usterek, komentarze urzędowe, zdjęcia przed/po naprawie oraz publiczne statystyki platformy.
   * Może utworzyć konto lub zalogować się w systemie.
2. **Mieszkaniec (Użytkownik zarejestrowany - rola `user`)**
   * Posiada wszystkie uprawnienia Gościa.
   * Może zgłaszać nowe usterki infrastruktury miejskiej.
   * Może oznaczać istniejące zgłoszenia jako „Ten problem również mnie dotyczy”.
   * Może dodawać zgłoszenia do obserwowanych i otrzymuje powiadomienia o każdej zmianie ich statusu.
3. **Pracownik Urzędu (Operator - rola `official`)**
   * Odpowiada za merytoryczną i operacyjną obsługę zgłoszeń.
   * Może zmieniać statusy zgłoszeń zgodnie z ustalonym przepływem pracy (*workflow*).
   * Może dodawać publiczne komentarze urzędowe informujące o przebiegu realizacji.
   * Może załączać zdjęcia potwierdzające wykonanie naprawy przy zamykaniu zgłoszenia (`rozwiązane`).
4. **Administrator (rola `admin`)**
   * Posiada pełne uprawnienia w systemie.
   * Posiada dostęp do analitycznej mapy ciepła (*heatmap*) wskazującej obszary o największym natężeniu zgłoszeń.
   * Zarządza użytkownikami (edycja ról, blokowanie kont) oraz słownikiem kategorii usterek.
   * Posiada wgląd w rejestr logów aplikacyjnych i audytowych.

---

## 3. Wymagania funkcjonalne (Functional Requirements)

### 3.1. Uwierzytelnianie i zarządzanie kontem
* **FR-01 (Rejestracja klasyczna):** Użytkownik może zarejestrować się w systemie podając imię/nazwę, adres e-mail oraz hasło spełniające wymogi bezpieczeństwa.
* **FR-02 (Logowanie klasyczne):** Zarejestrowany użytkownik może zalogować się przy użyciu adresu e-mail oraz hasła (autoryzacja oparta o tokeny Laravel Sanctum).
* **FR-03 (Logowanie i rejestracja OAuth 2.0):** Użytkownik może zarejestrować się oraz zalogować za pomocą zewnętrznego dostawcy tożsamości OAuth 2.0 (np. Google / GitHub) obsługom poprzez Laravel Socialite.

### 3.2. Mapa interaktywna i przeglądanie zgłoszeń
* **FR-04 (Widok mapy zgłoszeń):** Użytkownik widzi na interaktywnej mapie (Leaflet + OpenStreetMap) znaczniki reprezentujące zgłoszone problemy infrastrukturalne.
* **FR-05 (Klastrowanie i filtrowanie):** Przy dużym zagęszczeniu pinezek mapa grupuje je w klastry; użytkownik może filtrować zgłoszenia według kategorii oraz bieżącego statusu.
* **FR-06 (Karta szczegółów zgłoszenia):** Po kliknięciu znacznika lub pozycji na liście użytkownik widzi pełne dane zgłoszenia: kategorię, opis, lokalizację, status, datę utworzenia, liczbę osób dotkniętych problemem, galerię zdjęć usterki, historię komentarzy urzędowych oraz (dla rozwiązanych) zdjęcie po naprawie.

### 3.3. Zgłaszanie usterek i prewencja duplikatów
* **FR-07 (Lokalizacja zgłoszenia):** Podczas tworzenia nowego zgłoszenia użytkownik wskazuje miejsce poprzez kliknięcie na mapie lub automatyczne pobranie swojej aktualnej pozycji GPS (HTML5 Geolocation API).
* **FR-08 (Wybór kategorii):** Użytkownik wybiera kategorię usterki ze zdefiniowanego słownika (obejmującego m.in.: *uszkodzona latarnia*, *dziura w drodze*, *uszkodzony znak*, *śmieci*, *zniszczona ławka*).
* **FR-09 (Opis i dokumentacja zdjęciowa):** Użytkownik wprowadza opis tekstowy usterki oraz może dołączyć jedno lub więcej zdjęć obrazujących problem.
* **FR-10 (Wykrywanie podobnych zgłoszeń w pobliżu):** Przed ostatecznym zapisaniem nowego zgłoszenia system automatycznie wyszukuje aktywne zgłoszenia znajdujące się w określonym promieniu geograficznym (np. 100 metrów) od wskazanego punktu (z opcjonalnym uwzględnieniem tej samej kategorii) i wyświetla je użytkownikowi jako potencjalne duplikaty.
* **FR-11 (Oznaczenie „Ten problem również mnie dotyczy”):** Zamiast tworzyć nowe, zduplikowane zgłoszenie (lub bezpośrednio na karcie istniejącego zgłoszenia), zalogowany użytkownik może kliknąć przycisk „Ten problem również mnie dotyczy”. Zwiększa to licznik dotkniętych mieszkańców przy danym zgłoszeniu (1 użytkownik = maksymalnie 1 głos na dane zgłoszenie) i automatycznie zapisuje użytkownika do obserwowania tego zgłoszenia.

### 3.4. Obserwowanie i powiadomienia
* **FR-12 (Obserwowanie zgłoszeń):** Zalogowany użytkownik może dodać dowolne zgłoszenie do listy obserwowanych (oraz usunąć je z tej listy).
* **FR-13 (Powiadomienia o zmianie statusu):** W momencie gdy pracownik urzędu zmieni status zgłoszenia, każdy użytkownik obserwujący to zgłoszenie (w tym jego autor) otrzymuje powiadomienie w systemie (oraz opcjonalnie powiadomienie e-mail).

### 3.5. Obsługa zgłoszeń przez Pracownika Urzędu
* **FR-14 (Zarządzanie statusem zgłoszenia):** Pracownik urzędu może zmieniać status zgłoszenia na jeden ze zdefiniowanych stanów:
  1. `zgłoszone` (*reported*) - stan początkowy każdego nowego zgłoszenia,
  2. `przyjęte` (*accepted*) - zgłoszenie zweryfikowane pozytywnie pod kątem formalnym,
  3. `przekazane do realizacji` (*forwarded*) - skierowane do odpowiedniej jednostki wykonawczej/służb miejskich,
  4. `w realizacji` (*in_progress*) - trwają prace naprawcze w terenie,
  5. `rozwiązane` (*resolved*) - usterka została usunięta,
  6. `odrzucone` (*rejected*) - zgłoszenie bezzasadne, duplikat lub poza kompetencjami jednostki.
* **FR-15 (Publiczne komentarze urzędowe):** Pracownik urzędu może dodawać do zgłoszenia publiczne komentarze dotyczące postępów realizacji (np. planowany termin naprawy, powód odrzucenia, nazwa brygady technicznej).
* **FR-16 (Zdjęcie po naprawie):** Przy zmianie statusu na `rozwiązane` (lub w ramach edycji rozwiązanego zgłoszenia) pracownik urzędu może załączyć zdjęcie wykonane po naprawie jako dowód zamknięcia sprawy.

### 3.6. Statystyki publiczne
* **FR-17 (Moduł statystyk publicznych):** System udostępnia ogólnodostępny widok analityczny prezentujący w czasie rzeczywistym:
  * Liczbę zgłoszeń w podziale na kategorie (wykres słupkowy/kołowy oraz zestawienie tabelaryczne).
  * Średni czas realizacji zgłoszeń (średnia różnica czasu od utworzenia zgłoszenia w statusie `zgłoszone` do przejścia w status `rozwiązane`, wyrażona w godzinach/dniach).
  * Całkowitą liczbę rozwiązanych zgłoszeń (oraz odsetek skuteczności).

### 3.7. Panel Administratora
* **FR-18 (Mapa ciepła / Heatmap):** Administrator posiada dostęp do specjalnego widoku mapy prezentującego zagęszczenie zgłoszeń (mapa ciepła obszarów z największą liczbą usterek), z możliwością filtrowania po przedziale czasu i kategorii.
* **FR-19 (Zarządzanie użytkownikami):** Administrator może przeglądać listę zarejestrowanych użytkowników, zmieniać ich role (`user`, `official`, `admin`) oraz aktywować/dezaktywować konta.
* **FR-20 (Zarządzanie kategoriami):** Administrator może dodawać nowe kategorie usterek, edytować istniejące (nazwa, ikona/kolor znacznika, opis) oraz dezaktywować kategorie nieużywane.
* **FR-21 (Podgląd logów aplikacji):** Administrator widzi z poziomu interfejsu webowego logi z aplikacji (logi audytowe kluczowych akcji biznesowych oraz logi błędów systemowych) wraz z możliwością ich filtrowania i przeszukiwania.

---

## 4. Wymagania niefunkcjonalne i architektoniczne (Non-Functional Requirements)

* **NFR-01 (Separacja Frontend-Backend i REST API):** Warstwa prezentacji (React + TypeScript) oraz warstwa serwerowa (Laravel) muszą być całkowicie odseparowane i komunikować się wyłącznie przez bezstanowe API REST w formacie JSON.
* **NFR-02 (Dokumentacja OpenAPI):** Backend musi posiadać kompletną, aktualną dokumentację zgodną ze specyfikacją **OpenAPI 3.0** (wygenerowaną przy użyciu `l5-swagger`), udostępniającą interaktywny interfejs Swagger UI dla wszystkich endpointów.
* **NFR-03 (Geolokalizacja i zapytania przestrzenne):** System musi przechowywać współrzędne geograficzne w układzie WGS84 (`latitude`, `longitude`) i realizować wyszukiwanie bliskich zgłoszeń z uwzględnieniem rzeczywistej odległości geograficznej na sferze (pre-filtracja Bounding Box w SQLite + formuła Haversine’a).
* **NFR-04 (Responsywność - Mobile First):** Interfejs użytkownika zbudowany w Tailwind CSS musi być w pełni responsywny i przystosowany do wygodnego zgłaszania usterek w terenie za pomocą smartfona.
* **NFR-05 (Bezpieczeństwo):**
  * Walidacja wszystkich danych wejściowych po stronie serwera (Laravel Form Requests).
  * Ochrona przed atakami CSRF/XSS oraz SQL Injection (Eloquent ORM + bindowanie parametrów).
  * Walidacja typów MIME oraz rozmiaru przesyłanych plików graficznych (maks. 5 MB na zdjęcie, formaty JPEG/PNG/WebP).
* **NFR-06 (Wydajność bazy SQLite):** Baza danych SQLite musi pracować w trybie **WAL (Write-Ahead Logging)** z założonymi indeksami na kolumnach współrzędnych (`latitude`, `longitude`), statusu (`status`) oraz kategorii (`category_id`).

---

## 5. Historyjki użytkownika (User Stories) i kryteria akceptacji

| ID | Rola | Historyjka użytkownika (Jako... chcę... aby...) | Kryteria akceptacji (Acceptance Criteria) |
| :--- | :--- | :--- | :--- |
| **US-01** | Gość | Jako niezalogowany mieszkaniec chcę zarejestrować konto przez e-mail lub konto Google, aby móc zgłaszać usterki. | 1. Formularz waliduje unikalność e-maila i siłę hasła.<br>2. Przycisk „Zaloguj przez Google” przeprowadza proces OAuth 2.0 i tworzy/loguje konto.<br>3. Po zalogowaniu użytkownik otrzymuje token autoryzacyjny. |
| **US-02** | Użytkownik | Jako mieszkaniec chcę zobaczyć na mapie wszystkie zgłoszone problemy w mojej okolicy, aby wiedzieć, nad czym pracują służby miejskie. | 1. Mapa wyświetla pinezki z kolorami/ikonami odpowiadającymi kategoriom lub statusom.<br>2. Kliknięcie pinezki otwiera podgląd ze szczegółami zgłoszenia. |
| **US-03** | Użytkownik | Jako mieszkaniec chcę zgłosić dziurę w drodze wskazując pozycję GPS ze smartfona i dodając zdjęcie, aby urząd szybko zlokalizował problem. | 1. Przycisk „Użyj mojej lokalizacji” ustawia pinezkę na współrzędnych GPS.<br>2. Użytkownik może ręcznie przesunąć pinezkę na mapie.<br>3. Formularz wymaga wyboru kategorii i opisu oraz pozwala dodać zdjęcia. |
| **US-04** | Użytkownik | Jako mieszkaniec chcę przed wysłaniem zgłoszenia zobaczyć podobne problemy w promieniu 100 m, aby nie duplikować pracy urzędników. | 1. Po wybraniu lokalizacji i kategorii API zwraca listę otwartych zgłoszeń w promieniu 100 m posortowaną wg odległości w metrach.<br>2. Użytkownik może kliknąć „Ten problem również mnie dotyczy” przy istniejącym zgłoszeniu lub kontynuować dodawanie nowego. |
| **US-05** | Użytkownik | Jako mieszkaniec chcę kliknąć „Ten problem również mnie dotyczy” oraz obserwować zgłoszenie, aby otrzymywać powiadomienia o jego naprawie. | 1. Licznik poparcia zgłoszenia zwiększa się o 1.<br>2. Użytkownik zostaje dodany do listy obserwujących.<br>3. Przy każdej zmianie statusu użytkownik widzi nowe powiadomienie. |
| **US-06** | Pracownik Urzędu | Jako pracownik urzędu chcę zmieniać status zgłoszenia, pisać komentarze i dodawać zdjęcie po naprawie, aby informować mieszkańców o realizacji. | 1. Pracownik ma wybór statusów: `zgłoszone`, `przyjęte`, `przekazane do realizacji`, `w realizacji`, `rozwiązane`, `odrzucone`.<br>2. Zmiana statusu wyzwala powiadomienia dla obserwujących.<br>3. Przy statusie `rozwiązane` można wgrać zdjęcie „po naprawie”. |
| **US-07** | Gość / Użytkownik | Jako mieszkaniec chcę przeglądać publiczne statystyki usterek, aby ocenić skuteczność działania urzędu. | 1. Widok prezentuje liczbę zgłoszeń wg kategorii.<br>2. Widok wylicza średni czas realizacji (od zgłoszenia do rozwiązania).<br>3. Widok pokazuje łączną liczbę rozwiązanych zgłoszeń. |
| **US-08** | Administrator | Jako administrator chcę widzieć mapę ciepła usterek, zarządzać użytkownikami/kategoriami i przeglądać logi, aby nadzorować system. | 1. Warstwa `leaflet.heat` wizualizuje skupiska usterek.<br>2. Panel pozwala na CRUD kategorii i zmianę ról użytkowników.<br>3. Widok logów wyświetla wpisy audytowe i błędy z filtrowaniem. |

---

## 6. Model danych (Zarys encji bazodanowych)

1. **`users`**: `id`, `name`, `email`, `password` (nullable dla OAuth), `oauth_provider`, `oauth_id`, `email_verified_at`, `is_active`, `timestamps`. (Role obsługiwane przez tabele pakietu `spatie/laravel-permission`: `roles`, `permissions`, `model_has_roles`).
2. **`categories`**: `id`, `name`, `slug`, `description`, `icon`, `color`, `is_active`, `timestamps`.
3. **`issues` (Zgłoszenia)**: `id`, `user_id` (autor), `category_id`, `title`, `description`, `latitude` (DECIMAL/REAL), `longitude` (DECIMAL/REAL), `address_hint`, `status` (ENUM: `reported`, `accepted`, `forwarded`, `in_progress`, `resolved`, `rejected`), `upvotes_count` (cache licznika „dotyczy mnie”), `resolved_at` (nullable timestamp do obliczania średniego czasu realizacji), `timestamps`.
4. **`issue_attachments` (Zdjęcia)**: `id`, `issue_id`, `uploaded_by`, `file_path`, `type` (`before` - zdjęcie usterki / `after` - zdjęcie po naprawie), `mime_type`, `file_size`, `timestamps`.
5. **`issue_upvotes` („Ten problem również mnie dotyczy”)**: `id`, `issue_id`, `user_id`, `created_at` (unikalny klucz złożony `(issue_id, user_id)`).
6. **`issue_followers` (Obserwowane zgłoszenia)**: `id`, `issue_id`, `user_id`, `created_at` (unikalny klucz złożony `(issue_id, user_id)`).
7. **`issue_comments` (Publiczne komentarze urzędowe)**: `id`, `issue_id`, `user_id` (pracownik/admin), `content`, `timestamps`.
8. **`issue_status_histories` (Historia zmian statusów)**: `id`, `issue_id`, `changed_by`, `old_status`, `new_status`, `comment`, `created_at`.
9. **`notifications` (Powiadomienia użytkowników)**: standardowa tabela powiadomień Laravel (`id` UUID, `type`, `notifiable_type`, `notifiable_id`, `data` JSON, `read_at`, `timestamps`).
10. **`system_logs` (Logi aplikacyjne i audytowe)**: `id`, `user_id` (nullable), `level` (`INFO`, `WARNING`, `ERROR`, `AUDIT`), `action`, `message`, `context` (JSON), `ip_address`, `user_agent`, `created_at`.

---

## 7. Specyfikacja głównych endpointów REST API (OpenAPI 3.0)

* **Autoryzacja (`/api/v1/auth`)**
  * `POST /api/v1/auth/register` - Rejestracja klasyczna e-mail + hasło.
  * `POST /api/v1/auth/login` - Logowanie klasyczne, zwrot tokenu Bearer.
  * `POST /api/v1/auth/logout` - Unieważnienie tokenu.
  * `GET /api/v1/auth/oauth/{provider}/redirect` - Przekierowanie do dostawcy OAuth.
  * `GET /api/v1/auth/oauth/{provider}/callback` - Obsługa powrotu z OAuth i wydanie tokenu dla SPA.
  * `GET /api/v1/auth/me` - Pobranie profilu zalogowanego użytkownika wraz z rolami.
* **Kategorie i Statystyki Publiczne (`/api/v1/public`)**
  * `GET /api/v1/categories` - Lista aktywnych kategorii usterek.
  * `GET /api/v1/statistics` - Publiczne statystyki (liczba zgłoszeń wg kategorii, średni czas realizacji, liczba rozwiązanych zgłoszeń).
* **Zgłoszenia (`/api/v1/issues`)**
  * `GET /api/v1/issues` - Lista zgłoszeń na mapę/listę (filtrowanie po kategorii, statusie, obszarze geograficznym).
  * `POST /api/v1/issues/check-similar` - Wyszukiwanie podobnych zgłoszeń w zadanym promieniu geograficznym (`latitude`, `longitude`, `radius_meters`, `category_id`) przed utworzeniem nowego zgłoszenia.
  * `POST /api/v1/issues` - Utworzenie nowego zgłoszenia (wraz z uploadem zdjęć `multipart/form-data`).
  * `GET /api/v1/issues/{id}` - Pobranie pełnych szczegółów zgłoszenia, zdjęć, komentarzy urzędowych i historii statusów.
  * `POST /api/v1/issues/{id}/affect-me` - Oznaczenie „Ten problem również mnie dotyczy” (toggle/add).
  * `POST /api/v1/issues/{id}/follow` - Przełączenie obserwowania zgłoszenia (follow/unfollow).
* **Obsługa urzędowa (`/api/v1/official/issues`)** *(Wymagana rola: `official` lub `admin`)*
  * `PATCH /api/v1/official/issues/{id}/status` - Zmiana statusu zgłoszenia (opcjonalnie ze zdjęciem po naprawie przy statusie `resolved`).
  * `POST /api/v1/official/issues/{id}/comments` - Dodanie publicznego komentarza dotyczącego realizacji.
* **Powiadomienia (`/api/v1/notifications`)**
  * `GET /api/v1/notifications` - Lista powiadomień zalogowanego użytkownika (np. o zmianie statusu obserwowanego zgłoszenia).
  * `PATCH /api/v1/notifications/{id}/read` - Oznaczenie powiadomienia jako przeczytane.
* **Administracja (`/api/v1/admin`)** *(Wymagana rola: `admin`)*
  * `GET /api/v1/admin/heatmap` - Dane zagregowanych punktów i wag do wygenerowania mapy ciepła obszarów z największą liczbą zgłoszeń.
  * `GET|PATCH /api/v1/admin/users` - Zarządzanie użytkownikami i ich rolami.
  * `POST|PUT|DELETE /api/v1/admin/categories` - Zarządzanie kategoriami zgłoszeń.
  * `GET /api/v1/admin/logs` - Przeglądanie i filtrowanie logów aplikacji.
