# ADR-004: Wybór silnika bazy danych - SQLite w trybie Write-Ahead Logging (WAL)

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

System wymaga relacyjnej bazy danych przechowującej użytkowników, role, kategorie, zgłoszenia wraz ze współrzędnymi geograficznymi, załączniki, głosy poparcia („dotyczy mnie”), obserwujących, komentarze urzędowe, powiadomienia oraz logi systemowe. Zgodnie z wymaganiami technicznymi projektu docelowym silnikiem bazodanowym ma być **SQLite**, co pozwala na łatwe przenoszenie projektu pomiędzy środowiskami deweloperskimi członków zespołu oraz środowiskiem oceniającym bez konieczności konfiguracji zewnętrznych serwerów bazodanowych.

Należało przeanalizować ograniczenia SQLite (szczególnie w zakresie współbieżnego zapisu oraz funkcji matematycznych potrzebnych do zapytań geograficznych) i określić docelową konfigurację silnika.

---

## 2. Rozważane opcje

### Opcja 1: SQLite w konfiguracji domyślnej (Journal Mode: DELETE)
* **Opis:** Standardowa baza plikowa SQLite bez dodatkowego tuningu.
* **Zalety:** Zerowa konfiguracja.
* **Wady:** Przy jednoczesnym odczycie mapy przez wielu użytkowników i zapisie nowego zgłoszenia lub logu audytowego mogą pojawiać się błędy blokady pliku (`SQLITE_BUSY: database is locked`).

### Opcja 2: SQLite z włączonym trybem WAL (Write-Ahead Logging) i indeksowaniem przestrzennym B-Tree
* **Opis:** Wykorzystanie silnika SQLite (będącego od wersji Laravel 11 domyślnym silnikiem frameworka) z wymuszonym trybem `PRAGMA journal_mode = WAL;`, `PRAGMA synchronous = NORMAL;`, `PRAGMA busy_timeout = 5000;` oraz kompozytowymi indeksami na kolumnach `latitude` i `longitude`.
* **Zalety:**
  * Operacje odczytu nie blokują operacji zapisu (i odwrotnie), co eliminuje wąskie gardło podczas równoczesnego przeglądania mapy i rejestrowania zgłoszeń/logów.
  * Pojedynczy plik bazy danych (`database/database.sqlite`), łatwy w wersjonowaniu struktury, seedowaniu danymi testowymi i uruchamianiu na dowolnej maszynie.
  * Pełna obsługa kluczy obcych (`PRAGMA foreign_keys = ON;`) oraz transakcji ACID.
* **Wady:** Brak natywnych typów `GEOGRAPHY` znanych z PostGIS (wymaga dwuetapowego obliczania odległości opisanego w ADR-009).

### Opcja 3: Zewnętrzny serwer PostgreSQL / MySQL
* **Opis:** Klasyczny serwer RDBMS uruchamiany jako osobna usługa lub kontener.
* **Zalety:** Wysoka skalowalność korporacyjna, rozszerzenie PostGIS.
* **Wady:** Narusza założenie wykorzystania SQLite w projekcie i komplikuje proces szybkiego uruchomienia środowiska przez zespół oraz prowadzącego.

---

## 3. Decyzja

Wybrano **Opcję 2**: Silnik **SQLite** skonfigurowany w trybie **WAL (Write-Ahead Logging)** z aktywną kontrolą kluczy obcych oraz dedykowanymi indeksami B-Tree wspierającymi filtrowanie geograficzne i analityczne.

---

## 4. Konsekwencje

### Pozytywne:
* **Prostota uruchomienia (Zero-Config Server):** Całe środowisko bazodanowe inicjalizuje się jedną komendą `php artisan migrate --seed`, co gwarantuje identyczny stan aplikacji u każdego członka zespołu.
* **Wysoka wydajność odczytu:** Zapytania o pinezki na mapie, publiczne statystyki oraz mapę ciepła wykonują się w ułamkach milisekund dzięki lokalnemu dostępowi do pliku i indeksom na kolumnach `(status, category_id)` oraz `(latitude, longitude)`.
* **Bezpieczna współbieżność:** Tryb WAL wraz z `busy_timeout = 5000` zapobiega błędom blokowania pliku podczas jednoczesnego zapisu zgłoszeń, powiadomień i logów.

### Negatywne i mitygacja:
* **Brak natywnych funkcji sferycznych PostGIS:** Zastosowano dwuetapowy mechanizm wyszukiwania pobliskich zgłoszeń: szybkie zawężenie prostokątem ograniczającym (*Bounding Box*) w SQL z użyciem indeksów na `latitude`/`longitude`, a następnie dokładne przeliczenie dystansu wzorem Haversine’a (szczegóły w `ADR-009`).
