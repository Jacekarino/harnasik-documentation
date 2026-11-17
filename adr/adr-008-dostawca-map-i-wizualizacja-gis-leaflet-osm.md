# ADR-008: Wybór silnika mapowego i narzędzi wizualizacji przestrzennej - Leaflet, OpenStreetMap, MarkerCluster oraz Leaflet.heat

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

System musi intensywnie wykorzystywać mapy oraz geolokalizację w kilku kluczowych obszarach:
1. Wyświetlanie użytkownikowi wszystkich zgłoszonych problemów infrastrukturalnych na interaktywnej mapie.
2. Wskazywanie miejsca nowej usterki poprzez kliknięcie na mapie lub automatyczne pobranie współrzędnych z modułu GPS urządzenia użytkownika (wraz z wizualizacją promienia wyszukiwania podobnych zgłoszeń).
3. Wyświetlanie administratorowi dedykowanej **mapy obszarów z największą liczbą zgłoszeń (mapy ciepła / heatmap)**.

Rozwiązanie mapowe musi być bezpłatne, nie może wymagać podpinania karty kredytowej w środowisku akademickim i musi bezproblemowo współpracować z React + TypeScript.

---

## 2. Rozważane opcje

### Opcja 1: Google Maps JavaScript API
* **Opis:** Komercyjna usługa mapowa od Google.
* **Zalety:** Bardzo dokładne podkłady mapowe i gotowe komponenty.
* **Wady:** Wymaga założenia konta rozliczeniowego Google Cloud z kartą płatniczą; restrykcyjne limity darmowych wywołań i klucze API utrudniające uruchomienie aplikacji klienckiej po sklonowaniu repozytorium frontendu.

### Opcja 2: Mapbox GL JS
* **Opis:** Nowoczesna biblioteka wektorowa WebGL.
* **Zalety:** Płynne renderowanie wektorowe 3D.
* **Wady:** Wymaga rejestracji po klucz dostępowy (Access Token) oraz limituje liczbę darmowych inicjalizacji mapy.

### Opcja 3: Leaflet + React-Leaflet + podkłady OpenStreetMap (OSM) + wtyczki `markercluster` i `leaflet.heat`
* **Opis:** Otwartoźródłowa biblioteka mapowa Leaflet z oficjalnym wrapperem `react-leaflet` dla React 18+, korzystająca z darmowych kafelków rastrowych OpenStreetMap oraz natywnego interfejsu przeglądarki `navigator.geolocation`.
* **Zalety:**
  * Całkowicie darmowe rozwiązanie (Open Source, licencja BSD-2-Clause), niewymagające żadnych płatnych kluczy API - aplikacja działa od razu po sklonowaniu repozytorium frontendu.
  * Wtyczka **`leaflet.markercluster`** pozwala estetycznie grupować znaczniki przy oddaleniu mapy miasta.
  * Wtyczka **`leaflet.heat`** umożliwia natychmiastowe wyrenderowanie na płótnie HTML5 Canvas **mapy ciepła (heatmap)** dla administratora na podstawie tablicy punktów `[latitude, longitude, intensity]`.
* **Wady:** Renderowanie rastrowe (w pełni wystarczające dla skali miejskiej).

---

## 3. Decyzja

Wybrano **Opcję 3**: Silnik **Leaflet (`react-leaflet`)** z darmowymi kafelkami **OpenStreetMap**, wzbogacony o wtyczkę **`leaflet.markercluster`** (dla mapy publicznej) oraz **`leaflet.heat`** (dla administracyjnej mapy obszarów z największą liczbą zgłoszeń), w połączeniu z natywnym **HTML5 Geolocation API**.

---

## 4. Konsekwencje

### Pozytywne:
* **Niezależność kosztowa i uruchomieniowa:** Brak kluczy komercyjnych API sprawia, że aplikacja działa natychmiast na każdym komputerze laboratoryjnym.
* **Czytelność przy dużej liczbie zgłoszeń:** Klastrowanie znaczników zapobiega nieczytelności mapy, a kolory markerów jednoznacznie informują o statusie sprawy (np. czerwony - zgłoszone, pomarańczowy - w realizacji, zielony - rozwiązane).
* **Spełnienie wymagania dla Administratora:** Warstwa `leaflet.heat` precyzyjnie wizualizuje obszary miasta o największej koncentracji awarii infrastruktury.

### Negatywne i mitygacja:
* **Wymóg podania atrybucji OpenStreetMap:** W stopce komponentu `TileLayer` na stałe umieszczono wymaganą licencją ODbL notę `© OpenStreetMap contributors`.
* **Odmowa dostępu do geolokalizacji przez użytkownika w przeglądarce:** W przypadku zablokowania uprawnień GPS aplikacja wyświetla przyjazny komunikat i ustawia widok domyślny na centrum miasta, pozwalając wskazać lokalizację kliknięciem na mapie.
