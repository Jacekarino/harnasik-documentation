# HARNASIK
Platforma Zgłaszania i Obsługi Usterek Infrastruktury Miejskiej

## Opis projektu

Projekt stanowi kompleksową platformę internetową umożliwiającą mieszkańcom zgłaszanie usterek oraz problemów dotyczących infrastruktury miejskiej (takich jak uszkodzone latarnie, dziury w drodze, uszkodzone znaki drogowe, dzikie wysypiska śmieci czy zniszczona mała architektura), a pracownikom urzędu oraz administratorom - efektywne zarządzanie cyklem życia zgłoszeń i analizę danych przestrzennych.

System został zaprojektowany w architekturze odseparowanej (**Headless REST API + Single Page Application**), zapewniając jasny podział odpowiedzialności pomiędzy warstwą serwerową a kliencką, udokumentowany za pomocą standardu **OpenAPI 3.0**.

---

## Zespół projektowy

* **Jakub Broszko** - Project Manager, Backend Developer
* **Maciej Wróblewski** - DevOps, Backend Developer
* **Jacek Kowalczyk** - Frontend Developer, Dokumentacja

---

## Stos technologiczny

| Warstwa | Technologia | Przeznaczenie |
| :--- | :--- | :--- |
| **Backend** | PHP 8.3+ / Laravel 11 | Logika biznesowa, REST API, obsługa powiadomień i autoryzacji |
| **Baza danych** | SQLite (tryb WAL) | Przechowywanie danych relacyjnych oraz współrzędnych geograficznych |
| **Uwierzytelnianie** | Laravel Sanctum + Socialite | Autoryzacja tokenowa SPA oraz integracja OAuth 2.0 (Google / GitHub) |
| **Uprawnienia** | Spatie Laravel Permission | Kontrola dostępu oparta na rolach (RBAC) |
| **Dokumentacja API** | L5-Swagger (OpenAPI 3.0) | Automatyczna generacja specyfikacji REST API z atrybutów PHP |
| **Frontend** | React 18+ / TypeScript / Vite | Kliencka aplikacja typu Single Page Application (SPA) |
| **Stylizacja UI** | Tailwind CSS | Responsywny interfejs użytkownika w podejściu *utility-first* |
| **Mapy i GIS** | Leaflet (`react-leaflet`) + OpenStreetMap | Wizualizacja zgłoszeń, geolokalizacja, klastrowanie (`markercluster`) i mapa ciepła (`leaflet.heat`) |

---

## Główne funkcjonalności systemu

### 1. Moduł Mieszkańca (Użytkownika)
* Rejestracja i logowanie klasyczne (e-mail oraz hasło) lub przez zewnętrznego dostawcę OAuth 2.0.
* Przeglądanie interaktywnej mapy zgłoszonych problemów w przestrzeni miejskiej wraz z filtrowaniem.
* Dodawanie nowego zgłoszenia:
  * Wskazanie pinezki bezpośrednio na mapie lub pobranie bieżącej lokalizacji GPS urządzenia (Geolocation API).
  * Wybór kategorii usterki (np. uszkodzona latarnia, dziura w drodze, uszkodzony znak, śmieci, zniszczona ławka).
  * Dołączenie opisu tekstowego oraz dokumentacji fotograficznej.
* **Inteligentne wykrywanie duplikatów:** przed finalnym wysłaniem formularza system przeszukuje promień geograficzny wokół wskazanego punktu i prezentuje podobne zgłoszenia znajdujące się w pobliżu.
* **Podbijanie priorytetu („Ten problem również mnie dotyczy”):** możliwość dołączenia się do istniejącego już zgłoszenia zamiast tworzenia duplikatu.
* **Obserwowanie zgłoszeń:** subskrypcja wybranych problemów oraz otrzymywanie powiadomień po każdej zmianie ich statusu.

### 2. Moduł Pracownika Urzędu
* Obsługa zgłoszeń oraz zarządzanie ich pełnym cyklem życia:
  * `zgłoszone` -> `przyjęte` -> `przekazane do realizacji` -> `w realizacji` -> `rozwiązane` / `odrzucone`.
* Publikowanie oficjalnych, publicznych komentarzy urzędowych dotyczących postępów realizacji.
* Dołączanie zdjęć powykonawczych (dokumentujących naprawę) przy zmianie statusu na `rozwiązane`.

### 3. Moduł Statystyk Publicznych
* Ogólnodostępny pulpit analityczny prezentujący:
  * Liczbę zgłoszeń w podziale na poszczególne kategorie.
  * Średni czas realizacji (rozwiązania) zgłoszenia.
  * Całkowitą liczbę rozwiązanych zgłoszeń.

### 4. Moduł Administratora
* Wizualizacja **mapy ciepła (heatmap)** obrazującej obszary miasta z największym zagęszczeniem usterek.
* Zarządzanie kontami użytkowników oraz przypisywanie ról systemowych.
* Zarządzanie słownikiem kategorii zgłoszeń (CRUD).
* Przeglądanie systemowych logów audytowych oraz błędów aplikacji z poziomu panelu administracyjnego.

---

## Struktura dokumentacji

* [`README.md`](./README.md) - Ogólny przegląd projektu, architektury i nawigacja po dokumentacji.
* [`prd.md`](./prd.md) - Dokument Wymagań Produktowych (*Product Requirements Document*).
* **Rejestr Decyzji Architektonicznych (ADR):**
  * [`adr/adr-001-architektura-systemu-rest-api-spa.md`](./adr/adr-001-architektura-systemu-rest-api-spa.md) - Architektura systemu: separacja Backend REST API i Frontend SPA.
  * [`adr/adr-002-wybor-frameworka-backendowego-laravel.md`](./adr/adr-002-wybor-frameworka-backendowego-laravel.md) - Wybór frameworka backendowego (Laravel).
  * [`adr/adr-003-wybor-technologii-frontendowych-react-ts-tailwind.md`](./adr/adr-003-wybor-technologii-frontendowych-react-ts-tailwind.md) - Wybór technologii warstwy klienckiej (React, TypeScript, Vite, Tailwind CSS).
  * [`adr/adr-004-wybor-silnika-bazy-danych-sqlite.md`](./adr/adr-004-wybor-silnika-bazy-danych-sqlite.md) - Wybór silnika bazy danych SQLite z trybem WAL.
  * [`adr/adr-005-uwierzytelnianie-sanctum-oraz-oauth-socialite.md`](./adr/adr-005-uwierzytelnianie-sanctum-oraz-oauth-socialite.md) - Uwierzytelnianie dwutorowe: klasyczne (Laravel Sanctum) oraz OAuth 2.0 (Laravel Socialite).
  * [`adr/adr-006-autoryzacja-i-kontrola-dostepu-rbac.md`](./adr/adr-006-autoryzacja-i-kontrola-dostepu-rbac.md) - Autoryzacja i model ról (RBAC) z wykorzystaniem `spatie/laravel-permission`.
  * [`adr/adr-007-dokumentacja-api-w-standardzie-openapi.md`](./adr/adr-007-dokumentacja-api-w-standardzie-openapi.md) - Dokumentowanie REST API w standardzie OpenAPI 3.0 (`l5-swagger`).
  * [`adr/adr-008-dostawca-map-i-wizualizacja-gis-leaflet-osm.md`](./adr/adr-008-dostawca-map-i-wizualizacja-gis-leaflet-osm.md) - Wybór silnika mapowego (Leaflet + OpenStreetMap) oraz wtyczek klastrowania i mapy ciepła.
  * [`adr/adr-009-wyszukiwanie-przestrzenne-i-wykrywanie-duplikatow.md`](./adr/adr-009-wyszukiwanie-przestrzenne-i-wykrywanie-duplikatow.md) - Algorytm wyszukiwania podobnych zgłoszeń w pobliżu (Bounding Box + wzór Haversine’a) oraz mechanizm „dotyczy mnie”.
  * [`adr/adr-010-cykl-zycia-zgloszenia-i-system-powiadomien.md`](./adr/adr-010-cykl-zycia-zgloszenia-i-system-powiadomien.md) - Maszyna stanów zgłoszenia, obserwowanie oraz system powiadomień o zmianie statusu.
  * [`adr/adr-011-przechowywanie-i-przetwarzanie-zdjec.md`](./adr/adr-011-przechowywanie-i-przetwarzanie-zdjec.md) - Przechowywanie i walidacja załączników fotograficznych (zdjęcia usterek oraz zdjęcia po naprawie).
  * [`adr/adr-012-logowanie-zdarzen-i-audyt-systemowy.md`](./adr/adr-012-logowanie-zdarzen-i-audyt-systemowy.md) - Centralne logowanie zdarzeń aplikacyjnych i audytowych dla panelu administratora.
* [`event-storming.md`](./event-storming.md) - Wynik sesji event storming (2026-11-07).