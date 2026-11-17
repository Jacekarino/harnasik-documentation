# ADR-004: Wybór silnika relacyjnej bazy danych - PostgreSQL 16+ z rozszerzeniem PostGIS

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Platforma „HARNASIK” jest systemem o charakterze silnie geoprzestrzennym. Obsługuje zgłoszenia przypisane do punktów na mapie, wyszukiwanie podobnych problemów w promieniu geograficznym (np. 100 m), klastrowanie pinezek, analityczną mapę ciepła (heatmap) oraz relacje użytkowników, ról (RBAC), załączników, komentarzy i logów audytowych.

Baza danych musi zapewnić:
1. Natywną, precyzyjną obsługę typów i zapytań geoprzestrzennych (GIS) w standardzie Open Geospatial Consortium (OGC).
2. Wydajne indeksowanie przestrzenne (drzewa GiST) umożliwiające natychmiastowe wyszukiwanie sąsiedztwa bez narzutu obliczeniowego w warstwie aplikacji.
3. Pełną transakcyjność ACID, integralność kluczy obcych oraz zaawansowaną współbieżność (MVCC), zabezpieczającą system przed blokadami przy jednoczesnym zapisie zgłoszeń, logów i odczycie mapy.

Należało wybrać silnik bazodanowy najlepiej spełniający te kryteria.

---

## 2. Rozważane opcje

### Opcja 1: SQLite (baza plikowa)
* **Opis:** Lekka, plikowa baza danych wbudowana w środowisko PHP/Laravel.
* **Zalety:** Brak konieczności instalowania i uruchamiania osobnego serwera bazy danych.
* **Wady:** Brak natywnego wsparcia dla typów GIS (brak modułu SpatiaLite w standardowych instalacjach PHP PDO); konieczność aproksymacji odległości w kodzie aplikacji; ryzyko blokad pliku (`SQLITE_BUSY: database is locked`) przy współbieżnym zapisie przez wielu użytkowników.

### Opcja 2: MySQL 8+ z funkcjami przestrzennymi
* **Opis:** Popularny serwer relacyjny z podstawowym wsparciem dla funkcji przestrzennych (`Spatial Extensions`).
* **Zalety:** Szerokie wsparcie w ekosystemie PHP.
* **Wady:** Funkcje GIS w MySQL są uboższe niż w standardzie OGC, a obliczenia na elipsoidzie dla układu współrzędnych geograficznych WGS84 posiadają ograniczenia wydajnościowe i mniejszy zestaw operatorów relacji przestrzennych w porównaniu z PostGIS.

### Opcja 3: PostgreSQL 16+ z rozszerzeniem PostGIS
* **Opis:** Zaawansowany system bazodanowy uznawany za globalny standard branżowy dla systemów GIS dzięki rozszerzeniu **PostGIS**.
* **Zalety:**
  * **Natywne typy danych przestrzennych:** Obsługa typów `GEOMETRY` oraz `GEOGRAPHY(Point, 4326)` z pełnym wsparciem dla elipsoidy WGS84.
  * **Indeksowanie GiST (Generalized Search Tree):** Błyskawiczne przeszukiwanie milionów punktów przestrzennych.
  * **Zaawansowane operatory i funkcje GIS:** Natywne funkcje takie jak `ST_DWithin`, `ST_Distance`, `ST_MakePoint`, `ST_SetSRID` wykonywane w jądrze silnika bazy danych.
  * **Wysoka współbieżność (MVCC):** Odczyty nigdy nie blokują zapisów, co jest kluczowe przy dynamicznie odświeżanej mapie miejskiej i jednoczesnym napływie zgłoszeń.
  * Znakomite wsparcie w Laravelu za pośrednictwem dedykowanych bibliotek lub surowych wyrażeń Eloquent.
* **Wady:** Wymaga uruchomienia serwera PostgreSQL (np. lokalnie lub w kontenerze Docker).

---

## 3. Decyzja

Wybrano **Opcję 3**: Silnik **PostgreSQL 16+** z aktywnym rozszerzeniem przestrzennym **PostGIS**, przechowujący lokalizację zgłoszeń jako punkty geograficzne z dedykowanym indeksem przestrzennym **GiST**.

---

## 4. Konsekwencje

### Pozytywne:
* **Natywna wydajność GIS:** Wyszukiwanie zgłoszeń w zadanym promieniu (`ST_DWithin`) oraz sortowanie według odległości (`ST_Distance`) wykonują się bezpośrednio w silniku bazy danych z wykorzystaniem indeksu GiST, bez konieczności manualnej matematyki trygonometrycznej w kodzie backendu.
* **Skalowalność i niezawodność:** Architektura MVCC zapewnia płynną pracę pod obciążeniem – jednoczesne przeglądanie mapy przez setki mieszkańców nie wpływa na czas zapisu nowych zgłoszeń i logów.
* **Zgodność ze standardami OGC:** Baza jest przygotowana na ewentualną rozbudowę o bardziej złożone obiekty przestrzenne (linie dróg, poligony rejonów miejskich, geofencing).

### Negatywne i mitygacja:
* **Konieczność posiadania działającej instancji PostgreSQL z PostGIS:** Zapewniono konfigurację zmiennych w pliku `.env` w repozytorium `harnasik-backend` oraz rekomendację uruchomienia bazy za pomocą dedykowanego kontenera Docker (`postgis/postgis:16-3.4`).
