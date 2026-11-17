# ADR-009: Wyszukiwanie przestrzenne i wykrywanie duplikatów w oparciu o PostGIS (ST_DWithin, ST_Distance) oraz mechanizm „Ten problem również mnie dotyczy”

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Wymagania funkcjonalne i niefunkcjonalne projektu stawiają szczególny nacisk na zapobieganie duplikatom zgłoszeń:
1. Przed dodaniem nowego zgłoszenia system musi pokazać użytkownikowi **podobne zgłoszenia znajdujące się w pobliżu**.
2. Wyszukiwanie podobnych zgłoszeń **musi uwzględniać rzeczywistą odległość geograficzną** na elipsoidzie WGS84.
3. Użytkownik musi mieć możliwość oznaczenia istniejącego zgłoszenia jako **„ten problem również mnie dotyczy”** zamiast tworzenia nowego zgłoszenia.

W związku z wyborem relacyjnej bazy danych **PostgreSQL z rozszerzeniem PostGIS** (ADR-004), należało zaprojektować optymalną strategię wyszukiwania przestrzennego wykorzystującą natywne możliwości silnika geoprzestrzennego oraz model relacyjny dla głosów poparcia.

---

## 2. Rozważane opcje

### Opcja 1: Pobieranie całej tabeli `issues` do pamięci PHP i filtrowanie w pętli
* **Opis:** Pobranie rekordów `Issue::all()` i wyliczanie odległości w kodzie PHP.
* **Zalety:** Pozorna prostota implementacji.
* **Wady:** Krytyczny spadek wydajności i pamięciożerność wraz z przyrostem bazy danych.

### Opcja 2: Matematyczna aproksymacja prostokątem ograniczającym (Bounding Box) i wzorem Haversine'a w SQL
* **Opis:** Ręczne wyliczanie przedziałów współrzędnych i stosowanie wzorów trygonometrycznych w zapytaniach SQL.
* **Zalety:** Działa na bazach bez rozszerzeń przestrzennych.
* **Wady:** Zbędna złożoność kodu i mniejsza wydajność w środowisku posiadającym natywne rozszerzenie PostGIS.

### Opcja 3: Natywne zapytania przestrzenne PostGIS (`ST_DWithin`, `ST_Distance`) z indeksem GiST oraz tabela pivot `issue_upvotes`
* **Opis:**
  1. **Natywna reprezentacja przestrzenna:** Zgłoszenia przechowują punkt geograficzny w kolumnie `location` typu `geometry(Point, 4326)` lub `geography(Point, 4326)`, z założonym indeksem przestrzennym **GiST**.
  2. **Wyszukiwanie w promieniu (ST_DWithin):** Wyszukiwanie zgłoszeń w zadanym promieniu (domyślnie 100 m) realizowane jest bezpośrednio w zapytaniu SQL przy użyciu funkcji `ST_DWithin`:
     ```sql
     SELECT id, title, category_id, status,
            ST_Distance(location::geography, ST_MakePoint(:lng, :lat)::geography) AS distance_meters
     FROM issues
     WHERE status NOT IN ('resolved', 'rejected')
       AND (:category_id IS NULL OR category_id = :category_id)
       AND ST_DWithin(location::geography, ST_MakePoint(:lng, :lat)::geography, :radius_meters)
     ORDER BY distance_meters ASC;
     ```
  3. **Dokładne sortowanie (ST_Distance):** Odległość zwracana jest w metrach geodezyjnych na elipsoidzie WGS84, a wyniki są natychmiast sortowane od najbliższego.
  4. **Mechanizm „Ten problem również mnie dotyczy”:** Dedykowana tabela relacyjna `issue_upvotes` z unikalnym kluczem złożonym `(issue_id, user_id)` oraz licznikiem `upvotes_count` w tabeli `issues`.

---

## 3. Decyzja

Wybrano **Opcję 3**: Natywne zapytania przestrzenne **PostGIS (`ST_DWithin` i `ST_Distance`)** z indeksem przestrzennym **GiST** w serwisie `IssueSimilarityService` oraz relację `issue_upvotes` z automatycznym dodaniem użytkownika do obserwujących zgłoszenie.

---

## 4. Konsekwencje

### Pozytywne:
* **Maksymalna wydajność dzięki GiST:** Indeks przestrzenny pozwala w ułamkach milisekund zawęzić przestrzeń przeszukiwania bez pełnego skanowania tabeli (`Seq Scan`).
* **Precyzja geodezyjna:** Wykorzystanie typu `geography` w PostGIS uwzględnia krzywiznę Ziemi (elipsoida WGS84), co zapewnia dokładność odległości co do centymetra.
* **Czysty i prosty kod w Laravelu:** Logika wyszukiwania jest zwięzła i realizowana w całości przez zoptymalizowane zapytanie bazy danych, bez konieczności utrzymywania własnych funkcji trygonometrycznych w PHP.
* **Płynny UX prewencji duplikatów:** Formularz zgłoszeniowy odpytuje endpoint `POST /api/v1/issues/check-similar`, prezentując mieszkańcowi odległość do podobnych zgłoszeń i umożliwiając podbicie istniejącego problemu przyciskiem **„Ten problem również mnie dotyczy (+1)”**.
* **Integralność danych:** Unikalny indeks `(issue_id, user_id)` uniemożliwia wielokrotne oddanie głosu przez tego samego użytkownika.

### Negatywne i mitygacja:
* **Wymóg rozszerzenia PostGIS:** Środowisko bazodanowe musi mieć włączone rozszerzenie `CREATE EXTENSION IF NOT EXISTS postgis;`, co zostało zautomatyzowane w pierwszej migracji backendowej Laravela.
