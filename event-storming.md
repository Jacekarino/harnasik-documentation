# Event Storming: HARNASIK

## 1. Cel dokumentu
Dokument przedstawia wynik sesji **Event Storming** dla systemu zgłaszania i obsługi usterek infrastruktury miejskiej. Model opisuje główne zdarzenia domenowe, komendy, aktorów, agregaty, polityki oraz systemy zewnętrzne zaangażowane w działanie aplikacji.

---

## 2. Legenda kolorów kafelków

Na wszystkich poniższych diagramach zastosowano rzeczywiste kolory karteczek:

| Kolor | Element | Opis |
| :--- | :--- | :--- |
| **Pomarańczowy** (`#FF9800`) | **Domain Event (Zdarzenie)** | Fakt, który wydarzył się w systemie (czas przeszły dokonany). |
| **Niebieski** (`#2196F3`) | **Command (Komenda)** | Akcja wywoływana przez aktora lub politykę (tryb rozkazujący). |
| **Turkusowy** (`#00BCD4`) | **Actor (Aktor)** | Rola użytkownika wykonującego komendę. |
| **Żółty** (`#FFEB3B`) | **Aggregate (Agregat)** | Obiekt domenowy przetwarzający komendy i generujący zdarzenia. |
| **Fioletowy** (`#9C27B0`) | **Policy (Polityka)** | Automatyczna reguła: *„Kiedy [Zdarzenie], wykonaj [Komenda]”*. |
| **Zielony** (`#4CAF50`) | **Read Model (Widok)** | Dane prezentowane użytkownikowi na ekranie przed podjęciem akcji. |
| **Różowy** (`#E91E63`) | **External System (System zewn.)** | Zewnętrzna usługa (OAuth 2.0, GPS, OpenStreetMap). |
| **Czerwony** (`#F44336`) | **Hot Spot (Problem / Pytanie)** | Wąskie gardło lub ryzyko wymagające decyzji. |

---

## 3. Aktorzy i Systemy Zewnętrzne

### Aktorzy:
* **Gość** – niezalogowany użytkownik (przegląda mapę zgłoszeń i publiczne statystyki, może się zarejestrować).
* **Użytkownik (Mieszkaniec)** – zalogowany użytkownik (zgłasza usterki, oznacza „ten problem również mnie dotyczy”, obserwuje zgłoszenia i otrzymuje powiadomienia).
* **Pracownik Urzędu** – obsługuje zgłoszenia (zmienia statusy, dodaje publiczne komentarze, dołącza zdjęcia po naprawie).
* **Administrator** – zarządza użytkownikami i kategoriami, przegląda mapę ciepła (obszary z największą liczbą zgłoszeń) oraz logi aplikacji.

### Systemy zewnętrzne:
* **Dostawca OAuth 2.0 (Google / GitHub)** – zewnętrzne uwierzytelnianie użytkowników.
* **OpenStreetMap + Leaflet** – dostawca map i wizualizacji przestrzennej.
* **Geolocation API (GPS)** – pobieranie bieżącej lokalizacji użytkownika.

---

## 4. Big Picture – Główne zdarzenia domenowe

```mermaid
flowchart LR
    classDef event fill:#FF9800,stroke:#E65100,stroke-width:2px,color:#ffffff;

    subgraph Auth [1. Konto i logowanie]
        E1[Użytkownik zarejestrowany]:::event --> E2[Użytkownik zalogowany]:::event
    end

    subgraph Report [2. Zgłaszanie problemu]
        E3[Lokalizacja usterki wskazana]:::event --> E4[Podobne zgłoszenia w pobliżu wyświetlone]:::event
        E4 --> E5[Istniejące zgłoszenie oznaczone jako 'dotyczy mnie']:::event
        E4 --> E6[Nowe zgłoszenie utworzone]:::event
        E6 --> E7[Zgłoszenie dodane do obserwowanych]:::event
        E5 --> E7
    end

    subgraph Work [3. Obsługa przez urząd]
        E8[Status zgłoszenia zmieniony]:::event --> E9[Publiczny komentarz dodany]:::event
        E9 --> E10[Zdjęcie po naprawie dodane]:::event
        E10 --> E11[Zgłoszenie rozwiązane lub odrzucone]:::event
    end

    subgraph Notify [4. Powiadomienia i Admin]
        E12[Powiadomienie o zmianie statusu wysłane]:::event --> E13[Statystyki i mapa ciepła zaktualizowane]:::event
        E13 --> E14[Log aplikacji zapisany]:::event
    end

    Auth --> Report --> Work --> Notify
```

---

## 5. Przepływy procesów (Process Modeling)

### 5.1. Rejestracja i logowanie (E-mail oraz OAuth)

```mermaid
flowchart LR
    classDef actor fill:#00BCD4,stroke:#006064,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef command fill:#2196F3,stroke:#0D47A1,stroke-width:2px,color:#ffffff;
    classDef aggregate fill:#FFEB3B,stroke:#F57F17,stroke-width:2px,color:#111111,font-weight:bold;
    classDef event fill:#FF9800,stroke:#E65100,stroke-width:2px,color:#ffffff;
    classDef external fill:#E91E63,stroke:#880E4F,stroke-width:2px,color:#ffffff;

    A(Gość):::actor --> C1[Zarejestruj się przez e-mail]:::command
    A --> C2[Zaloguj przez OAuth]:::command
    C2 --> EXT[Dostawca OAuth 2.0]:::external
    EXT --> AG[[Użytkownik]]:::aggregate
    C1 --> AG
    AG --> E1[Użytkownik zarejestrowany]:::event
    AG --> E2[Użytkownik zalogowany]:::event
```

### 5.2. Zgłaszanie usterki i sprawdzanie podobnych zgłoszeń w pobliżu

```mermaid
flowchart TD
    classDef actor fill:#00BCD4,stroke:#006064,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef command fill:#2196F3,stroke:#0D47A1,stroke-width:2px,color:#ffffff;
    classDef aggregate fill:#FFEB3B,stroke:#F57F17,stroke-width:2px,color:#111111,font-weight:bold;
    classDef event fill:#FF9800,stroke:#E65100,stroke-width:2px,color:#ffffff;
    classDef policy fill:#9C27B0,stroke:#4A148C,stroke-width:2px,color:#ffffff;
    classDef readModel fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#ffffff;
    classDef external fill:#E91E63,stroke:#880E4F,stroke-width:2px,color:#ffffff;

    U(Użytkownik):::actor --> C1[Wskaż miejsce na mapie lub GPS]:::command
    C1 --> EXT[Geolokalizacja / Mapa]:::external
    EXT --> E1[Lokalizacja i kategoria wybrane]:::event
    E1 --> P1{{Polityka: Sprawdź zgłoszenia w promieniu 100m}}:::policy
    P1 --> RM1[Widok: Podobne zgłoszenia w pobliżu]:::readModel

    RM1 -->|Problem już istnieje| C2[Oznacz 'Ten problem również mnie dotyczy']:::command
    C2 --> AG[[Zgłoszenie]]:::aggregate
    AG --> E2[Licznik 'dotyczy mnie' zwiększony]:::event

    RM1 -->|Brak duplikatu| C3[Dodaj nowe zgłoszenie z opisem i zdjęciem]:::command
    C3 --> AG
    AG --> E3[Nowe zgłoszenie utworzone]:::event

    E2 --> P2{{Polityka: Automatycznie dodaj do obserwowanych}}:::policy
    E3 --> P2
    P2 --> E4[Zgłoszenie obserwowane przez użytkownika]:::event
```

### 5.3. Obsługa zgłoszenia przez Pracownika Urzędu i powiadomienia

Dopuszczalne statusy zgłoszenia: `zgłoszone`, `przyjęte`, `przekazane do realizacji`, `w realizacji`, `rozwiązane`, `odrzucone`.

```mermaid
flowchart LR
    classDef actor fill:#00BCD4,stroke:#006064,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef command fill:#2196F3,stroke:#0D47A1,stroke-width:2px,color:#ffffff;
    classDef aggregate fill:#FFEB3B,stroke:#F57F17,stroke-width:2px,color:#111111,font-weight:bold;
    classDef event fill:#FF9800,stroke:#E65100,stroke-width:2px,color:#ffffff;
    classDef policy fill:#9C27B0,stroke:#4A148C,stroke-width:2px,color:#ffffff;
    classDef readModel fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#ffffff;

    O(Pracownik Urzędu):::actor --> C1[Zmień status zgłoszenia]:::command
    O --> C2[Dodaj publiczny komentarz]:::command
    O --> C3[Dodaj zdjęcie po naprawie]:::command

    C1 --> AG[[Zgłoszenie]]:::aggregate
    C2 --> AG
    C3 --> AG

    AG --> E1[Status zgłoszenia zmieniony]:::event
    AG --> E2[Komentarz publiczny dodany]:::event
    AG --> E3[Zdjęcie po naprawie dodane]:::event

    E1 --> P1{{Polityka: Powiadom obserwujących}}:::policy
    P1 --> C4[Wyślij powiadomienie]:::command
    C4 --> E4[Powiadomienie dostarczone]:::event
    E4 --> RM[Widok powiadomień użytkownika]:::readModel
```

### 5.4. Statystyki publiczne i Panel Administratora

```mermaid
flowchart LR
    classDef actor fill:#00BCD4,stroke:#006064,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef command fill:#2196F3,stroke:#0D47A1,stroke-width:2px,color:#ffffff;
    classDef aggregate fill:#FFEB3B,stroke:#F57F17,stroke-width:2px,color:#111111,font-weight:bold;
    classDef event fill:#FF9800,stroke:#E65100,stroke-width:2px,color:#ffffff;
    classDef readModel fill:#4CAF50,stroke:#1B5E20,stroke-width:2px,color:#ffffff;

    G(Użytkownik / Gość):::actor --> RM1[Statystyki: wg kategorii, średni czas, rozwiązane]:::readModel

    ADM(Administrator):::actor --> RM2[Mapa obszarów z największą liczbą zgłoszeń]:::readModel
    ADM --> RM3[Widok logów z aplikacji]:::readModel
    ADM --> C1[Zarządzaj kategoriami]:::command
    ADM --> C2[Zarządzaj użytkownikami]:::command

    C1 --> AG1[[Kategoria]]:::aggregate --> E1[Kategoria zaktualizowana]:::event
    C2 --> AG2[[Użytkownik]]:::aggregate --> E2[Konto / rola użytkownika zmieniona]:::event
```

---

## 6. Zestawienie elementów modelu

### Agregaty (Żółte):
1. **Użytkownik (`User`)** – dane konta, metoda logowania (e-mail / OAuth), rola (`user`, `official`, `admin`).
2. **Zgłoszenie (`Issue`)** – współrzędne geograficzne, kategoria, opis, zdjęcia (przed i po naprawie), aktualny status, komentarze urzędowe, lista obserwujących oraz licznik „dotyczy mnie”.
3. **Kategoria (`Category`)** – nazwa (np. *uszkodzona latarnia*, *dziura w drodze*, *uszkodzony znak*, *śmieci*, *zniszczona ławka*), ikona, status aktywności.
4. **Log aplikacji (`SystemLog`)** – zapis zdarzeń systemowych i audytowych dla administratora.

### Modele odczytu / Widoki (Zielone):
* **Mapa zgłoszeń** – publiczna mapa wszystkich zgłoszonych problemów w mieście.
* **Podobne zgłoszenia w pobliżu** – lista otwartych zgłoszeń w promieniu geograficznym wyświetlana przed utworzeniem nowego zgłoszenia.
* **Szczegóły zgłoszenia** – opis, zdjęcia usterki, status, komentarze urzędowe, zdjęcie po naprawie.
* **Publiczne statystyki** – liczba zgłoszeń wg kategorii, średni czas realizacji, liczba rozwiązanych zgłoszeń.
* **Mapa ciepła (Heatmap)** – mapa dla administratora pokazująca obszary z największą liczbą zgłoszeń.
* **Rejestr logów** – tabela logów aplikacji w panelu administratora.

### Hot Spoty / Problemy rozwiązane podczas sesji (Czerwone):
1. **Wyszukiwanie zgłoszeń po odległości w SQLite:** SQLite nie ma domyślnie funkcji geograficznych PostGIS. **Rozwiązanie:** wstępne zawężenie wyników zapytaniem `BETWEEN` dla szerokości i długości geograficznej (*Bounding Box*), a następnie dokładne obliczenie odległości ze wzoru Haversine’a.
2. **Zapobieganie wielokrotnemu klikaniu „Ten problem również mnie dotyczy”:** **Rozwiązanie:** tabela pośrednia `issue_upvotes` z unikalnym kluczem złożonym `(issue_id, user_id)`.
3. **Obliczanie średniego czasu realizacji:** **Rozwiązanie:** zapisywanie daty `resolved_at` w momencie zmiany statusu na `rozwiązane`, co pozwala łatwo wyliczyć różnicę czasu od `created_at`.