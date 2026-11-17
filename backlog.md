# Product Backlog: HARNASIK

| Metryczka dokumentu | Wartość |
| :--- | :--- |
| **Projekt** | HARNASIK - Platforma do zgłaszania i obsługi usterek infrastruktury miejskiej |
| **Zespół** | Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk |
| **Stos technologiczny** | Laravel, React, TypeScript, Tailwind CSS, PostgreSQL / PostGIS |
| **Metodyka priorytetyzacji** | MoSCoW (*Must Have*, *Should Have*, *Could Have*) |
| **Skala estymacji** | Story Points (ciąg Fibonacciego: 1, 2, 3, 5, 8) |

---

## 1. Podsumowanie Sprintów i Epików

| Sprint | Cel Sprintu | Zakres zadań (ID) | Suma Story Points |
| :--- | :--- | :--- | :---: |
| **Sprint 1** | Fundamenty architektury, baza PostgreSQL z PostGIS, OpenAPI, uwierzytelnianie (klasyczne + OAuth 2.0) oraz RBAC | `PBI-01` – `PBI-07` | **24 SP** |
| **Sprint 2** | Mapa interaktywna (Leaflet/OSM), zgłaszanie usterek z GPS i zdjęciami, wyszukiwanie duplikatów w pobliżu oraz „dotyczy mnie” | `PBI-08` – `PBI-15` | **34 SP** |
| **Sprint 3** | Panel Pracownika Urzędu (6 statusów, komentarze, zdjęcia po naprawie), obserwowanie zgłoszeń i system powiadomień | `PBI-16` – `PBI-21` | **22 SP** |
| **Sprint 4** | Statystyki publiczne, Panel Administratora (Mapa ciepła / Heatmap, użytkownicy, kategorie, logi aplikacji) oraz testy E2E | `PBI-22` – `PBI-27` | **23 SP** |
| **Razem** | **Pełne wdrożenie wymagań funkcjonalnych i niefunkcjonalnych** | **27 elementów PBI** | **103 SP** |

---

## 2. Rejestr elementów Product Backlogu (Product Backlog Items)

| ID | User Story | Epik | Tytuł zadania / funkcjonalności | Rola | Warstwa | Priorytet | SP | Sprint | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| **PBI-01** | Tech / NFR-01 | Infrastruktura | Inicjalizacja trzech niezależnych repozytoriów (backend Laravel 11 API-only, frontend React 18 TS Vite SPA, documentation) + konfiguracja CORS i połączenia przez REST API | Deweloper | Fullstack | Must Have | 3 | Sprint 1 | Do zrobienia |
| **PBI-02** | Tech / NFR-06 | Infrastruktura | Konfiguracja bazy danych PostgreSQL z rozszerzeniem PostGIS, przygotowanie migracji tabel, indeksu przestrzennego GiST (dla lokalizacji zgłoszeń), indeksów B-Tree oraz seederów | Deweloper | Backend | Must Have | 5 | Sprint 1 | Do zrobienia |
| **PBI-03** | Tech / NFR-02 | Infrastruktura | Konfiguracja `l5-swagger` (OpenAPI 3.0), schematu autoryzacji Bearer oraz generowania dokumentacji Swagger UI (`/api/documentation`) | Deweloper | Backend | Must Have | 3 | Sprint 1 | Do zrobienia |
| **PBI-04** | US-01 | EPIC-01: Auth | Implementacja rejestracji klasycznej (e-mail + hasło) w REST API oraz formularza rejestracji w React z walidacją | Gość | Fullstack | Must Have | 3 | Sprint 1 | Do zrobienia |
| **PBI-05** | US-02 | EPIC-01: Auth | Implementacja logowania i wylogowania opartego o tokeny Laravel Sanctum oraz obsługa stanu autoryzacji w React Context | Użytkownik | Fullstack | Must Have | 2 | Sprint 1 | Do zrobienia |
| **PBI-06** | US-03 | EPIC-01: Auth | Integracja logowania i rejestracji przez zewnętrznego dostawcę OAuth 2.0 (Google / GitHub) za pomocą Laravel Socialite | Gość | Fullstack | Must Have | 5 | Sprint 1 | Do zrobienia |
| **PBI-07** | US-02, US-19 | EPIC-01: Auth | Konfiguracja ról i uprawnień (`spatie/laravel-permission`: `user`, `official`, `admin`), middleware API oraz strażników tras (`ProtectedRoleRoute`) w React | System | Fullstack | Must Have | 3 | Sprint 1 | Do zrobienia |
| **PBI-08** | US-04 | EPIC-02: Mapa | Wdrożenie interaktywnej mapy zgłoszeń (`react-leaflet` + OpenStreetMap) z klastrowaniem znaczników (`leaflet.markercluster`) i kolorami statusów | Gość / Mieszkaniec | Frontend | Must Have | 5 | Sprint 2 | Do zrobienia |
| **PBI-09** | US-05 | EPIC-02: Mapa | Endpoint `GET /api/v1/issues` oraz panel filtrowania zgłoszeń na mapie i liście według kategorii i statusu | Gość / Mieszkaniec | Fullstack | Should Have | 3 | Sprint 2 | Do zrobienia |
| **PBI-10** | US-06 | EPIC-02: Mapa | Widok karty szczegółów zgłoszenia (opis, lokalizacja, galeria zdjęć „przed” i „po naprawie”, oś czasu statusów i komentarze urzędowe) | Gość / Mieszkaniec | Fullstack | Must Have | 3 | Sprint 2 | Do zrobienia |
| **PBI-11** | US-07 | EPIC-03: Zgłaszanie | Obsługa wyboru lokalizacji nowego zgłoszenia: kliknięcie pinezki na mapie lub pobranie pozycji z GPS urządzenia (HTML5 Geolocation API) | Mieszkaniec | Frontend | Must Have | 3 | Sprint 2 | Do zrobienia |
| **PBI-12** | US-08 | EPIC-03: Zgłaszanie | Serwis `IssueSimilarityService` i endpoint `POST /api/v1/issues/check-similar` wyszukujący podobne zgłoszenia w pobliżu z wykorzystaniem natywnych funkcji PostGIS (`ST_DWithin`, `ST_Distance`, indeks GiST) | Mieszkaniec | Backend | Must Have | 5 | Sprint 2 | Do zrobienia |
| **PBI-13** | US-08 | EPIC-03: Zgłaszanie | Komponent ostrzegania o potencjalnych duplikatach w pobliżu przed dodaniem zgłoszenia (prezentacja odległości w metrach i podglądu) | Mieszkaniec | Frontend | Must Have | 3 | Sprint 2 | Do zrobienia |
| **PBI-14** | US-07 | EPIC-03: Zgłaszanie | Endpoint `POST /api/v1/issues` i formularz tworzenia nowego zgłoszenia z wyborem kategorii, opisem i uploadem zdjęć (`multipart/form-data`, `Storage`) | Mieszkaniec | Fullstack | Must Have | 5 | Sprint 2 | Do zrobienia |
| **PBI-15** | US-09, US-10 | EPIC-03: Zgłaszanie | Funkcjonalność „Ten problem również mnie dotyczy” (`POST /api/v1/issues/{id}/affect-me`), licznik poparcia oraz widok „Moje zgłoszenia” | Mieszkaniec | Fullstack | Must Have | 5 | Sprint 2 | Do zrobienia |
| **PBI-16** | US-11 | EPIC-04: Powiadomienia | Mechanizm obserwowania i odobserwowania zgłoszeń (`POST /api/v1/issues/{id}/follow`) wraz z automatyczną subskrypcją dla autora i wspierających | Mieszkaniec | Fullstack | Must Have | 3 | Sprint 3 | Do zrobienia |
| **PBI-17** | US-13 | EPIC-05: Urząd | Serwis przepływu pracy `IssueStatusWorkflowService` i endpoint `PATCH /api/v1/official/issues/{id}/status` obsługujący 6 statusów oraz zapis `resolved_at` | Pracownik Urzędu | Backend | Must Have | 5 | Sprint 3 | Do zrobienia |
| **PBI-18** | US-13 | EPIC-05: Urząd | Panel Pracownika Urzędu w React: tabela zgłoszeń do obsługi, zmiana statusu sprawy oraz podgląd historii zmian | Pracownik Urzędu | Frontend | Must Have | 3 | Sprint 3 | Do zrobienia |
| **PBI-19** | US-14 | EPIC-05: Urząd | Dodawanie publicznych komentarzy dotyczących realizacji przez pracownika urzędu (`POST /api/v1/official/issues/{id}/comments`) | Pracownik Urzędu | Fullstack | Must Have | 3 | Sprint 3 | Do zrobienia |
| **PBI-20** | US-15 | EPIC-05: Urząd | Załączanie zdjęcia wykonanego po naprawie (`type = after`) do rozwiązanego zgłoszenia oraz prezentacja widoku porównawczego „Przed / Po” | Pracownik Urzędu | Fullstack | Must Have | 3 | Sprint 3 | Do zrobienia |
| **PBI-21** | US-12 | EPIC-04: Powiadomienia | System powiadomień (`Laravel Notifications`) wyzwalany po zmianie statusu obserwowanego zgłoszenia oraz komponent dzwonka powiadomień w React | Mieszkaniec | Fullstack | Must Have | 5 | Sprint 3 | Do zrobienia |
| **PBI-22** | US-16 | EPIC-06: Statystyki | Endpoint `GET /api/v1/statistics` i widok Statystyk Publicznych: liczba zgłoszeń wg kategorii, średni czas realizacji oraz liczba rozwiązanych zgłoszeń | Gość / Mieszkaniec | Fullstack | Must Have | 5 | Sprint 4 | Do zrobienia |
| **PBI-23** | US-17 | EPIC-07: Admin | Endpoint `GET /api/v1/admin/heatmap` i widok Mapy Ciepła (`leaflet.heat`) dla administratora prezentujący obszary z największą liczbą zgłoszeń | Administrator | Fullstack | Must Have | 5 | Sprint 4 | Do zrobienia |
| **PBI-24** | US-18 | EPIC-07: Admin | Moduł zarządzania słownikiem kategorii usterek (CRUD kategorii w API oraz w panelu administratora) | Administrator | Fullstack | Must Have | 3 | Sprint 4 | Do zrobienia |
| **PBI-25** | US-19 | EPIC-07: Admin | Moduł zarządzania użytkownikami w panelu administratora (lista kont, zmiana ról `user`/`official`/`admin`, blokowanie kont) | Administrator | Fullstack | Must Have | 3 | Sprint 4 | Do zrobienia |
| **PBI-26** | US-20 | EPIC-07: Admin | Centralny serwis logowania `SystemLoggerService` (tabela `system_logs`) oraz widok przeglądania i filtrowania logów z aplikacji dla administratora | Administrator | Fullstack | Must Have | 5 | Sprint 4 | Do zrobienia |
| **PBI-27** | Tech / NFR-02 | Infrastruktura | Finalna weryfikacja kompletności specyfikacji OpenAPI 3.0, testy integracyjne kluczowych endpointów oraz optymalizacja responsywności UI (Tailwind CSS) | Deweloper | Fullstack | Should Have | 2 | Sprint 4 | Do zrobienia |

---

## 3. Definicja Ukończenia (Definition of Done – DoD)

Każdy element z Product Backlogu uznaje się za ukończony (`Done`), gdy spełnia wszystkie poniższe warunki:
1. Kod w dedykowanych repozytoriach (`harnasik-backend` i `harnasik-frontend`) oraz dokumentacja (w `harnasik-documentation`) zostały zaimplementowane i zaktualizowane zgodnie z kryteriami akceptacji danej *User Story*.
2. Wszystkie nowe lub zmodyfikowane endpointy REST API posiadają aktualne atrybuty OpenAPI (`l5-swagger`), a plik specyfikacji generuje się bez błędów.
3. Dane wejściowe są walidowane po stronie serwera za pomocą klas `FormRequest`, a dostęp jest chroniony odpowiednimi uprawnieniami (`Sanctum` + `Spatie Permission`).
4. Komponenty frontendowe w React są w pełni otypowane w TypeScript (brak użycia typu `any`) i poprawnie wyświetlają się na ekranach mobilnych oraz desktopowych (Tailwind CSS).
5. Kluczowe operacje zmieniające stan systemu zapisują odpowiedni ślad w logach aplikacji (`system_logs`).
