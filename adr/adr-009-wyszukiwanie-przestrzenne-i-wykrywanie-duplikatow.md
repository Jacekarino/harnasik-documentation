# ADR-009: Wyszukiwanie podobnych zgłoszeń w pobliżu (odległość geograficzna) oraz mechanizm „Ten problem również mnie dotyczy”

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Wymagania funkcjonalne i niefunkcjonalne projektu stawiają szczególny nacisk na zapobieganie duplikatom zgłoszeń:
1. Przed dodaniem nowego zgłoszenia system musi pokazać użytkownikowi **podobne zgłoszenia znajdujące się w pobliżu**.
2. Wyszukiwanie podobnych zgłoszeń **musi uwzględniać rzeczywistą odległość geograficzną**.
3. Użytkownik musi mieć możliwość oznaczenia istniejącego zgłoszenia jako **„ten problem również mnie dotyczy”** zamiast tworzenia nowego zgłoszenia.

Ponieważ bazą danych jest **SQLite** (ADR-004), które w standardowej kompilacji nie posiada rozszerzenia SpatiaLite ani wbudowanych funkcji trygonometrycznych w starszych kompilacjach PDO, należało zaprojektować dokładny i wydajny algorytm wyszukiwania przestrzennego oraz strukturę bazodanową dla głosów poparcia.

---

## 2. Rozważane opcje

### Opcja 1: Naiwne pobieranie całej tabeli `issues` do pamięci PHP i filtrowanie w pętli
* **Opis:** Pobranie wszystkich rekordów `Issue::all()` i liczenie odległości w PHP.
* **Zalety:** Prostota kodu.
* **Wady:** Drastyczny spadek wydajności wraz ze wzrostem liczby zgłoszeń w bazie danych.

### Opcja 2: Wymuszenie instalacji rozszerzenia C `SpatiaLite` w środowisku PHP
* **Opis:** Ładowanie binarnego modułu GIS do SQLite.
* **Zalety:** Funkcje przestrzenne bezpośrednio w SQL.
* **Wady:** Domyślna instalacja PHP (na Windows/Linux/macOS) blokuje ze względów bezpieczeństwa ładowanie zewnętrznych rozszerzeń SQLite (`enableExternalExtension`), co uniemożliwiłoby uruchomienie projektu na standardowym środowisku studenckim.

### Opcja 3: Dwuetapowy algorytm hybrydowy - Pre-filtracja SQL Bounding Box + precyzyjny wzór Haversine’a oraz tabela pivot `issue_upvotes`
* **Opis:**
  1. **Etap 1 (Pre-filtracja w SQLite z użyciem indeksu B-Tree):** Dla punktu wskazanego przez użytkownika $(\phi_0, \lambda_0)$ i zadanego promienia $R$ (domyślnie np. $100\text{ m} = 0.1\text{ km}$) serwis wylicza prostokąt ograniczający (*Bounding Box*):
     * $\Delta\phi = \frac{R}{111.32\text{ km}}$
     * $\Delta\lambda = \frac{R}{111.32\text{ km} \cdot \cos(\phi_0)}$
     Następnie zapytanie SQL wybiera wyłącznie aktywne zgłoszenia (status inny niż `rozwiązane` i `odrzucone`, opcjonalnie z tej samej kategorii) mieszczące się w przedziale `latitude BETWEEN minLat AND maxLat` oraz `longitude BETWEEN minLng AND maxLng`.
  2. **Etap 2 (Dokładna odległość sferyczna - wzór Haversine’a):** Dla odfiltrowanego, niewielkiego podzbioru kandydatów obliczana jest dokładna odległość ortodromiczna $d$ na sferze ziemskiej ($R_Z = 6371000\text{ m}$):
     $$d = 2 R_Z \arcsin\left(\sqrt{\sin^2\left(\frac{\Delta\phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta\lambda}{2}\right)}\right)$$
     Odrzucane są punkty z narożników kwadratu, dla których $d > R$, a wynik jest sortowany rosnąco według dokładnej odległości w metrach (np. `24 m stąd`).
  3. **Mechanizm „Ten problem również mnie dotyczy”:** Dedykowana tabela relacyjna `issue_upvotes` z unikalnym kluczem złożonym `(issue_id, user_id)` oraz licznikiem `upvotes_count` w tabeli `issues`.

---

## 3. Decyzja

Wybrano **Opcję 3**: Dwuetapowy algorytm wyszukiwania przestrzennego (**SQLite Bounding Box + formuła Haversine’a**) realizowany w serwisie `IssueSimilarityService` oraz relacja `issue_upvotes` z automatycznym dodaniem użytkownika do obserwujących dane zgłoszenie.

---

## 4. Konsekwencje

### Pozytywne:
* **100% przenośności i wysoka wydajność:** Algorytm korzysta ze standardowych indeksów B-Tree w SQLite na kolumnach `(latitude, longitude)`, a jednocześnie zwraca odległość geograficzną z dokładnością co do metra.
* **Ergonomiczny proces zgłaszania (UX):** Gdy użytkownik w formularzu wybierze punkt na mapie i kategorię, frontend automatycznie odpytuje `POST /api/v1/issues/check-similar`. Jeśli w promieniu np. 100 m istnieją podobne zgłoszenia, użytkownik widzi ich karty z odległością w metrach i przyciskiem **„Ten problem również mnie dotyczy (+1)”**.
* **Brak sztucznego zawyżania licznika:** Unikalny indeks bazodanowy `(issue_id, user_id)` gwarantuje, że jeden zalogowany użytkownik może poprzeć dane zgłoszenie tylko raz.
* **Synergia z powiadomieniami:** Kliknięcie „Ten problem również mnie dotyczy” automatycznie zapisuje użytkownika jako obserwującego to zgłoszenie (`issue_followers`), dzięki czemu otrzyma on powiadomienie, gdy urząd naprawi usterkę.

### Negatywne i mitygacja:
* **Możliwość zignorowania sugestii przez użytkownika:** Użytkownik nadal zachowuje prawo do kliknięcia „Mimo to utwórz nowe zgłoszenie” (np. gdy tuż obok istniejącej dziury w jezdni znajduje się osobna, nowa usterka).
