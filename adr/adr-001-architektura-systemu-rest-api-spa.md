# ADR-001: Architektura systemu - odseparowany Backend REST API oraz Frontend SPA

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Projekt „Platforma do zgłaszania i obsługi usterek infrastruktury miejskiej” wymaga stworzenia nowoczesnego systemu webowego obsługującego interaktywne mapy, dynamiczną geolokalizację, asynchroniczne wczytywanie zgłoszeń w pobliżu oraz panele dla trzech ról (Mieszkaniec, Pracownik Urzędu, Administrator). Jednocześnie formalne wymagania projektowe narzucają, aby:
1. Warstwa backendowa i frontendowa były od siebie wyraźnie odseparowane.
2. Komunikacja pomiędzy nimi odbywała się za pośrednictwem protokołu HTTP w architekturze REST API.
3. Całość API była udokumentowana w standardzie OpenAPI.

Należało podjąć decyzję dotyczącą głównego stylu architektonicznego aplikacji.

---

## 2. Rozważane opcje

### Opcja 1: Monolit wielostronicowy (MPA) - Laravel Blade + Livewire / Alpine.js
* **Opis:** Tradycyjna aplikacja serwerowa, w której widoki HTML są renderowane po stronie backendu w silniku szablonów Blade, a interaktywność zapewniają komponenty Livewire.
* **Zalety:** Szybki start, brak konieczności utrzymywania dwóch osobnych repozytoriów/serwerów budowania, wspólna sesja.
* **Wady:** Silne sprzężenie warstwy widoku z logiką backendu; niespełnienie wymogu pełnej separacji frontendu od backendu; utrudniona obsługa zaawansowanych interakcji na mapie bez pełnego przeładowania stanu.

### Opcja 2: Architektura hybrydowa (Monolit z adapterem) - Laravel + Inertia.js + React
* **Opis:** Wykorzystanie Reacta jako warstwy widoków, ale sterowanej bezpośrednio przez kontrolery Laravela za pomocą protokołu Inertia.js, bez budowy klasycznego REST API.
* **Zalety:** Możliwość pisania komponentów w React i TypeScript bez konieczności ręcznego zarządzania routingiem po stronie klienta i wywołaniami REST.
* **Wady:** Brak jawnego, niezależnego kontraktu REST API udokumentowanego w OpenAPI; ścisłe uzależnienie frontendu od cyklu żądań Inertia.js.

### Opcja 3: Architektura w pełni odseparowana (Headless REST API + Decoupled SPA)
* **Opis:** Podział systemu na dwa niezależne podsystemy:
  1. **Backend (Laravel):** aplikacja działająca w trybie *API-only*, wystawiająca punkty końcowe REST API zwracające dane w formacie JSON oraz udostępniająca specyfikację OpenAPI 3.0.
  2. **Frontend (React + TypeScript + Vite):** niezależna aplikacja kliencka typu *Single Page Application (SPA)*, konsumująca REST API przez klienta HTTP (Axios / Fetch).
* **Zalety:** 100% zgodności z wymaganiami projektowymi; jasny kontrakt komunikacyjny (OpenAPI); możliwość niezależnego rozwijania, testowania i wdrażania obu warstw; wysoka płynność działania interfejsu mapowego po stronie przeglądarki.
* **Wady:** Konieczność konfiguracji polityki CORS, obsługi tokenów autoryzacyjnych po stronie SPA oraz utrzymania spójności typów między obiema warstwami.

---

## 3. Decyzja

Wybrano **Opcję 3**: Architektura w pełni odseparowana (**Headless Laravel REST API + React TypeScript SPA**).

---

## 4. Konsekwencje

### Pozytywne:
* Pełna realizacja wymagania odseparowania frontendu od backendu oraz komunikacji przez REST API.
* Możliwość równoległej pracy zespołu deweloperskiego (podział zadań na wdrożenie endpointów API oraz budowę widoków React na podstawie ustalonego kontraktu OpenAPI).
* Płynne działanie mapy Leaflet - zmiana filtrów, dociąganie zgłoszeń w pobliżu czy aktualizacja statusu odbywają się bez przeładowywania całej strony.
* W przyszłości ten sam backend REST API może bez żadnych zmian zasilać natywną aplikację mobilną (iOS / Android).

### Negatywne i mitygacja:
* **Konieczność obsługi CORS i autoryzacji międzydomenowej:** Rozwiązane poprzez konfigurację middleware CORS w Laravelu oraz zastosowanie pakietu Laravel Sanctum (tokeny Bearer API).
* **Ryzyko rozbieżności struktur danych między PHP a TypeScript:** Zminimalizowane poprzez generowanie specyfikacji OpenAPI (`l5-swagger`) oraz ścisłe typowanie interfejsów DTO w TypeScript.
