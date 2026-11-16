# ADR-003: Wybór stosu technologicznego warstwy klienckiej - React, TypeScript, Vite oraz Tailwind CSS

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Część kliencka (SPA) musi oferować responsywny, intuicyjny interfejs zarówno dla mieszkańców zgłaszających usterki na smartfonach w terenie, jak i dla pracowników urzędu oraz administratorów pracujących na komputerach stacjonarnych z wieloma tabelami, wykresami statystyk i mapą ciepła. Aplikacja intensywnie operuje na danych przestrzennych (współrzędne, promienie, klastry znaczników) oraz wieloetapowych formularzach (sprawdzanie podobnych zgłoszeń przed wysłaniem nowego).

---

## 2. Rozważane opcje

### Opcja 1: React 18+ + TypeScript + Vite + Tailwind CSS
* **Opis:** Biblioteka React z silnym typowaniem TypeScript, narzędziem budowania Vite oraz frameworkiem CSS opartym o klasy narzędziowe (Tailwind CSS).
* **Zalety:**
  * Statyczne typowanie TypeScript wyłapuje błędy niezgodności danych z kontraktami REST API już na etapie kompilacji.
  * Bardzo dojrzałe wsparcie dla map poprzez bibliotekę `react-leaflet` wraz z definicjami typów `@types/leaflet`.
  * Tailwind CSS pozwala szybko budować responsywne widoki (*Mobile First*), modale, nakładki na mapę i dymki znaczników bez pisania osobnych plików CSS.
  * Vite zapewnia natychmiastowy start serwera deweloperskiego i szybkie odświeżanie (HMR).
* **Wady:** Konieczność samodzielnego doboru bibliotek pomocniczych (routing: `react-router-dom`, komunikacja HTTP: `axios`).

### Opcja 2: React + JavaScript (ES6) + Bootstrap / Material UI
* **Opis:** Klasyczny React bez statycznego typowania z gotową biblioteką komponentów UI.
* **Zalety:** Brak narzutu pisania interfejsów TypeScript.
* **Wady:** Wysokie ryzyko błędów `undefined is not an object` przy pracy z zagnieżdżonymi strukturami JSON z API (zgłoszenia, załączniki, historia statusów, współrzędne); trudniejsze nadpisywanie domyślnych stylów Bootstrapa/MUI wewnątrz kontenerów mapy Leaflet.

### Opcja 3: Vue.js / Angular
* **Opis:** Alternatywne frameworki SPA.
* **Zalety:** Wbudowane mechanizmy routingu.
* **Wady:** Zespół posiada największe doświadczenie w ekosystemie React + TypeScript, który jest także określony w założeniach technologicznych projektu.

---

## 3. Decyzja

Wybrano **Opcję 1**: **React 18+ z językiem TypeScript**, bundlowany za pomocą **Vite** oraz stylowany przy użyciu **Tailwind CSS**.

---

## 4. Konsekwencje

### Pozytywne:
* **Bezpieczeństwo typów (Type Safety):** Wszystkie modele domenowe (`Issue`, `Category`, `IssueStatus`, `Comment`, `PublicStats`, `SystemLog`) posiadają ścisłe definicje interfejsów TypeScript odpowiadające schematom OpenAPI z backendu.
* **Szybkość tworzenia UI:** Podejście *utility-first* w Tailwind CSS radykalnie przyspiesza tworzenie responsywnych formularzy zgłoszeniowych, odznak statusów (kolorowe badge dla 6 statusów) oraz panelu administracyjnego.
* **Ergonomia pracy z mapą:** Tailwind CSS pozwala łatwo pozycjonować pływające panele filtrów i okno „Podobne zgłoszenia w pobliżu” bezpośrednio nad płótnem mapy (`z-index`, `backdrop-blur`, `absolute`/`fixed`).

### Negatywne i mitygacja:
* **Rozbudowane atrybuty `className` w kodzie JSX:** Zastosowano reużywalne komponenty atomowe (np. `StatusBadge`, `Button`, `Modal`, `IssueCard`) w celu utrzymania czytelności kodu.
