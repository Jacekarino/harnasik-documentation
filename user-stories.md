# User Stories: HARNASIK

| Metryczka dokumentu | Wartość |
| :--- | :--- |
| **Projekt** | HARNASIK - Platforma do zgłaszania i obsługi usterek infrastruktury miejskiej |
| **Autorzy** | Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk |
| **Data aktualizacji** | 2026-11-16 |

---

## Podział na Epiki (Epics)

1. **EPIC-01: Uwierzytelnianie i Zarządzanie Kontem** (`US-01` – `US-03`)
2. **EPIC-02: Interaktywna Mapa i Przeglądanie Zgłoszeń** (`US-04` – `US-06`)
3. **EPIC-03: Zgłaszanie Usterek i Prewencja Duplikatów** (`US-07` – `US-10`)
4. **EPIC-04: Obserwowanie Zgłoszeń i System Powiadomień** (`US-11` – `US-12`)
5. **EPIC-05: Obsługa Zgłoszeń przez Pracownika Urzędu** (`US-13` – `US-15`)
6. **EPIC-06: Statystyki Publiczne** (`US-16`)
7. **EPIC-07: Panel Administratora i Analityka Przestrzenna** (`US-17` – `US-20`)

---

## EPIC-01: Uwierzytelnianie i Zarządzanie Kontem

### US-01: Rejestracja klasyczna przez adres e-mail
* **Rola:** Gość (Użytkownik niezalogowany)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-01` (rozinięcie `US-01` z PRD)
* **Historyjka:**
  > **Jako** niezalogowany mieszkaniec,  
  > **chcę** zarejestrować konto w systemie podając imię, adres e-mail oraz hasło,  
  > **aby** móc zgłaszać nowe usterki infrastruktury miejskiej i śledzić postępy ich naprawy.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Formularz rejestracji weryfikuje poprawność formatu adresu e-mail oraz unikalność konta w bazie `users`.
  - [ ] Hasło musi mieć minimum 8 znaków i zostać potwierdzone w drugim polu formularza.
  - [ ] Nowo utworzone konto automatycznie otrzymuje domyślną rolę `user` (Mieszkaniec).
  - [ ] Po poprawnej rejestracji backend zwraca kod `201 Created` wraz z tokenem autoryzacyjnym Laravel Sanctum, a wydarzenie jest zapisywane w logach systemowych.

---

### US-02: Logowanie klasyczne oraz wylogowanie
* **Rola:** Mieszkaniec / Pracownik Urzędu / Administrator
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 2 Story Points
* **Powiązane wymagania PRD:** `FR-02` (rozwinięcie `US-01` z PRD)
* **Historyjka:**
  > **Jako** zarejestrowany użytkownik,  
  > **chcę** zalogować się przy użyciu adresu e-mail i hasła oraz móc bezpiecznie wylogować się z systemu,  
  > **aby** uzyskać dostęp do funkcji przypisanych do mojej roli.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Poprawne dane logowania zwracają token Bearer (Laravel Sanctum) oraz profil użytkownika wraz z listą ról (`user`, `official`, `admin`).
  - [ ] Błędne dane logowania lub próba zalogowania na zablokowane konto (`is_active = false`) zwracają czytelny komunikat błędu (`401 Unauthorized` / `403 Forbidden`).
  - [ ] Kliknięcie „Wyloguj” unieważnia aktualny token po stronie serwera (`POST /api/v1/auth/logout`) i czyści stan sesji w aplikacji React SPA.

---

### US-03: Rejestracja i logowanie przez zewnętrznego dostawcę OAuth 2.0
* **Rola:** Gość / Mieszkaniec
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-03` (rozwinięcie `US-01` z PRD)
* **Historyjka:**
  > **Jako** mieszkaniec,  
  > **chcę** zarejestrować się i zalogować jednym kliknięciem za pomocą zewnętrznego dostawcy OAuth (np. Google / GitHub),  
  > **aby** nie musieć tworzyć i pamiętać kolejnego hasła.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Przycisk „Kontynuuj z Google / GitHub” przekierowuje użytkownika do ekranu autoryzacji dostawcy OAuth 2.0 (`Laravel Socialite`).
  - [ ] Jeśli użytkownik loguje się po raz pierwszy, system automatycznie tworzy konto w bazie `users` z rolą `user` i zapisuje `oauth_provider` oraz `oauth_id`.
  - [ ] Jeśli w bazie istnieje już konto z tym samym adresem e-mail, system bezpiecznie łączy je z identyfikatorem OAuth bez tworzenia duplikatu.
  - [ ] Po powrocie z callbacku OAuth aplikacja React SPA odbiera token Sanctum i loguje użytkownika.

---

## EPIC-02: Interaktywna Mapa i Przeglądanie Zgłoszeń

### US-04: Wyświetlanie zgłoszonych problemów na interaktywnej mapie
* **Rola:** Gość / Mieszkaniec
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-04`, `FR-05` (odpowiada `US-02` z PRD)
* **Historyjka:**
  > **Jako** mieszkaniec,  
  > **chcę** widzieć na interaktywnej mapie wszystkie zgłoszone problemy dotyczące infrastruktury miejskiej,  
  > **aby** mieć orientację, jakie usterki występują w mojej okolicy i na jakim są etapie naprawy.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Mapa (`React-Leaflet` + kafelki `OpenStreetMap`) wyświetla pinezki we współrzędnych GPS zgłoszeń.
  - [ ] Kolor lub ikona pinezki wizualnie odzwierciedla aktualny status lub kategorię usterki.
  - [ ] Przy oddaleniu mapy blisko położone pinezki są automatycznie grupowane w klastry (`leaflet.markercluster`), które rozwijają się po kliknięciu.
  - [ ] Kliknięcie pojedynczej pinezki wyświetla dymek (*popup*) z tytułem, kategorią, statusem i przyciskiem przejścia do pełnych szczegółów.

---

### US-05: Filtrowanie zgłoszeń na mapie i liście według kategorii oraz statusu
* **Rola:** Gość / Mieszkaniec / Pracownik Urzędu
* **Priorytet (MoSCoW):** Should Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-05` (rozwinięcie `US-02` z PRD)
* **Historyjka:**
  > **Jako** użytkownik przeglądający mapę,  
  > **chcę** filtrować wyświetlane zgłoszenia według kategorii (np. tylko „uszkodzona latarnia” lub „dziura w drodze”) oraz statusu realizacji,  
  > **aby** szybko odnaleźć interesujący mnie typ problemów.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Panel filtrów nad mapą pozwala wybrać jedną lub więcej kategorii oraz statusów.
  - [ ] Zmiana filtrów dynamicznie odświeża znaczniki na mapie (zapytanie `GET /api/v1/issues` z parametrami query) bez przeładowania całej strony.
  - [ ] Przycisk „Wyczyść filtry” przywraca domyślny widok wszystkich zgłoszeń.

---

### US-06: Przeglądanie karty szczegółów zgłoszenia wraz z historią i zdjęciami przed/po
* **Rola:** Gość / Mieszkaniec
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-06`, `FR-15`, `FR-16`
* **Historyjka:**
  > **Jako** mieszkaniec,  
  > **chcę** otworzyć kartę szczegółów wybranego zgłoszenia,  
  > **aby** przeczytać jego pełny opis, zobaczyć zdjęcia usterki, sprawdzić oficjalne komentarze urzędu, oś czasu zmian statusów oraz zdjęcie wykonane po naprawie.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Widok szczegółów prezentuje: tytuł, kategorię, dokładną lokalizację na mini-mapie, aktualny status, datę zgłoszenia oraz licznik „Ten problem dotyczy X osób”.
  - [ ] Wyświetlana jest galeria zdjęć zgłoszonych przez mieszkańca (`before`), a dla zgłoszeń o statusie `rozwiązane` — również sekcja ze zdjęciem po naprawie (`after`).
  - [ ] Pod opisem widoczna jest chronologiczna historia zmian statusów oraz publiczne komentarze dodane przez pracowników urzędu.

---

## EPIC-03: Zgłaszanie Usterek i Prewencja Duplikatów

### US-07: Zgłaszanie nowego problemu ze wskazaniem lokalizacji na mapie lub przez GPS
* **Rola:** Mieszkaniec (`user`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-07`, `FR-08`, `FR-09` (odpowiada `US-03` z PRD)
* **Historyjka:**
  > **Jako** zalogowany mieszkaniec,  
  > **chcę** zgłosić nowy problem wskazując miejsce na mapie lub pobierając moją lokalizację GPS, wybierając kategorię oraz dodając opis i zdjęcia,  
  > **aby** odpowiednie służby miejskie mogły precyzyjnie zlokalizować i naprawić usterkę.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Użytkownik może kliknąć w dowolne miejsce na mapie, aby postawić pinezkę usterki, lub kliknąć „Użyj mojej lokalizacji” (HTML5 Geolocation API).
  - [ ] Użytkownik wybiera kategorię z listy (m.in. *uszkodzona latarnia*, *dziura w drodze*, *uszkodzony znak*, *śmieci*, *zniszczona ławka*).
  - [ ] Formularz wymaga podania opisu tekstowego oraz umożliwia załączenie zdjęć usterki (walidacja formatu JPG/PNG/WebP, maks. 5 MB/plik).
  - [ ] Nowe zgłoszenie otrzymuje domyślny status `zgłoszone` (`reported`), a autor zostaje automatycznie dodany do listy obserwujących to zgłoszenie.

---

### US-08: Wyświetlanie podobnych zgłoszeń w pobliżu przed dodaniem nowego zgłoszenia
* **Rola:** Mieszkaniec (`user`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 8 Story Points
* **Powiązane wymagania PRD:** `FR-10`, `NFR-03` (odpowiada `US-04` z PRD)
* **Historyjka:**
  > **Jako** mieszkaniec wypełniający formularz zgłoszenia,  
  > **chcę** przed ostatecznym wysłaniem formularza zobaczyć listę podobnych zgłoszeń znajdujących się w pobliżu wskazanego punktu,  
  > **aby** upewnić się, czy ktoś inny nie zgłosił już tej samej usterki.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Po wskazaniu współrzędnych na mapie (oraz opcjonalnie kategorii) aplikacja wysyła zapytanie `POST /api/v1/issues/check-similar`.
  - [ ] Backend wyszukuje aktywne zgłoszenia w promieniu geograficznym (domyślnie 100 metrów) z wykorzystaniem natywnych funkcji przestrzennych PostGIS (`ST_DWithin`, `ST_Distance`) oraz indeksu przestrzennego GiST.
  - [ ] Jeśli w pobliżu istnieją zgłoszenia, użytkownik widzi modal/sekcję ostrzegawczą z listą tych zgłoszeń, wyliczoną odległością w metrach (np. „32 m od wybranego punktu”) oraz ich aktualnym statusem.
  - [ ] Użytkownik może zdecydować, czy dołącza do istniejącego zgłoszenia („Ten problem również mnie dotyczy”), czy mimo to tworzy nowe zgłoszenie.

---

### US-09: Oznaczenie istniejącego zgłoszenia jako „Ten problem również mnie dotyczy”
* **Rola:** Mieszkaniec (`user`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-11` (odpowiada `US-05` z PRD)
* **Historyjka:**
  > **Jako** zalogowany mieszkaniec,  
  > **chcę** oznaczyć istniejące zgłoszenie jako „ten problem również mnie dotyczy” zamiast tworzyć nowe zgłoszenie,  
  > **aby** zwiększyć priorytet sprawy i pokazać urzędowi skalę uciążliwości problemu bez generowania duplikatów.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Przycisk „Ten problem również mnie dotyczy” jest dostępny zarówno na liście podobnych zgłoszeń w pobliżu (podczas zgłaszania), jak i bezpośrednio na karcie szczegółów każdego aktywnego zgłoszenia.
  - [ ] Kliknięcie zwiększa licznik poparcia zgłoszenia (`upvotes_count`) o 1 i zapisuje rekord w tabeli `issue_upvotes`.
  - [ ] Jeden użytkownik może oznaczyć dane zgłoszenie maksymalnie jeden raz (ograniczenie kluczem unikalnym `(issue_id, user_id)`).
  - [ ] Oznaczenie zgłoszenia jako „dotyczy mnie” automatycznie zapisuje użytkownika na listę obserwujących to zgłoszenie.

---

### US-10: Przeglądanie listy własnych zgłoszeń przez mieszkańca
* **Rola:** Mieszkaniec (`user`)
* **Priorytet (MoSCoW):** Should Have
* **Estymacja:** 2 Story Points
* **Powiązane wymagania PRD:** `FR-06`, `FR-12`
* **Historyjka:**
  > **Jako** zalogowany mieszkaniec,  
  > **chcę** mieć dostęp do zakładki „Moje zgłoszenia”,  
  > **aby** w jednym miejscu widzieć wszystkie usterki, które sam zgłosiłem lub poparłem.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Widok „Moje zgłoszenia” prezentuje tabelę/karty zgłoszeń utworzonych przez zalogowanego użytkownika wraz z ich aktualnym statusem i datą ostatniej aktualizacji.
  - [ ] Użytkownik może jednym kliknięciem przejść z listy do szczegółów wybranego zgłoszenia na mapie.

---

## EPIC-04: Obserwowanie Zgłoszeń i System Powiadomień

### US-11: Obserwowanie i przestawanie obserwowania wybranych zgłoszeń
* **Rola:** Mieszkaniec (`user`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-12` (rozwinięcie `US-05` z PRD)
* **Historyjka:**
  > **Jako** zalogowany mieszkaniec,  
  > **chcę** móc dodać dowolne zgłoszenie do obserwowanych (oraz zrezygnować z obserwowania),  
  > **aby** śledzić postępy naprawy usterek, które znajdują się np. na mojej drodze do pracy.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Na karcie zgłoszenia znajduje się przełącznik „Obserwuj zgłoszenie / Obserwujesz”.
  - [ ] Wywołanie `POST /api/v1/issues/{id}/follow` dodaje lub usuwa wpis w tabeli `issue_followers`.
  - [ ] Użytkownik może filtrować listę zgłoszeń tak, aby wyświetlić tylko zgłoszenia aktualnie przez niego obserwowane.

---

### US-12: Otrzymywanie powiadomień po zmianie statusu obserwowanego zgłoszenia
* **Rola:** Mieszkaniec (`user`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-13` (rozwinięcie `US-05` z PRD)
* **Historyjka:**
  > **Jako** mieszkaniec obserwujący zgłoszenie,  
  > **chcę** otrzymać powiadomienie natychmiast po tym, jak pracownik urzędu zmieni status tego zgłoszenia,  
  > **aby** wiedzieć, kiedy problem został przyjęty, przekazany do realizacji lub ostatecznie rozwiązany.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Każda zmiana statusu zgłoszenia przez pracownika urzędu wyzwala wysyłkę powiadomień (`Laravel Notifications`) do wszystkich użytkowników zapisanych w tabeli `issue_followers` dla danego zgłoszenia.
  - [ ] W górnym pasku nawigacji aplikacji React widoczna jest ikona dzwonka z licznikiem nieprzeczytanych powiadomień.
  - [ ] Rozwinięcie listy powiadomień pokazuje poprzedni i nowy status zgłoszenia, datę zmiany oraz link prowadzący bezpośrednio do karty zgłoszenia.
  - [ ] Użytkownik może oznaczyć pojedyncze lub wszystkie powiadomienia jako przeczytane (`PATCH /api/v1/notifications/{id}/read`).

---

## EPIC-05: Obsługa Zgłoszeń przez Pracownika Urzędu

### US-13: Zmiana statusu zgłoszenia w pełnym cyklu życia sprawy
* **Rola:** Pracownik Urzędu (`official`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-14` (odpowiada `US-06` z PRD)
* **Historyjka:**
  > **Jako** pracownik urzędu,  
  > **chcę** zmieniać status zgłoszenia pomiędzy stanami: `zgłoszone`, `przyjęte`, `przekazane do realizacji`, `w realizacji`, `rozwiązane` oraz `odrzucone`,  
  > **aby** odzwierciedlać w systemie rzeczywisty etap prac nad usterką.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Panel pracownika urzędu umożliwia filtrowanie i sortowanie wpływających zgłoszeń (np. według liczby głosów „dotyczy mnie”, kategorii, daty lub statusu).
  - [ ] Pracownik urzędu może ustawić jeden z 6 wymaganych statusów: `zgłoszone`, `przyjęte`, `przekazane do realizacji`, `w realizacji`, `rozwiązane`, `odrzucone`.
  - [ ] Każda zmiana statusu jest odnotowywana w tabeli `issue_status_histories` (kto zmienił, kiedy, z jakiego stanu na jaki) oraz w logach audytowych aplikacji.
  - [ ] Zmiana statusu na `rozwiązane` automatycznie zapisuje znacznik czasu `resolved_at`, wykorzystywany do obliczania średniego czasu realizacji.

---

### US-14: Dodawanie publicznego komentarza dotyczącego realizacji zgłoszenia
* **Rola:** Pracownik Urzędu (`official`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-15` (rozwinięcie `US-06` z PRD)
* **Historyjka:**
  > **Jako** pracownik urzędu,  
  > **chcę** dodać publiczny komentarz dotyczący realizacji zgłoszenia,  
  > **aby** przekazać mieszkańcom szczegółowe wyjaśnienia (np. której firmie zlecono naprawę, jaki jest planowany termin lub dlaczego zgłoszenie zostało odrzucone).
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Formularz komentarza urzędowego jest dostępny wyłącznie dla ról `official` oraz `admin` (`POST /api/v1/official/issues/{id}/comments`).
  - [ ] Dodany komentarz pojawia się publicznie na karcie zgłoszenia wraz z oznaczeniem autora („Pracownik Urzędu”) oraz datą i godziną publikacji.

---

### US-15: Dołączanie zdjęcia wykonanego po naprawie do rozwiązanego zgłoszenia
* **Rola:** Pracownik Urzędu (`official`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-16` (rozwinięcie `US-06` z PRD)
* **Historyjka:**
  > **Jako** pracownik urzędu zamykający zgłoszenie,  
  > **chcę** dołączyć zdjęcie wykonane po naprawie przy zmianie statusu na `rozwiązane`,  
  > **aby** udokumentować usunięcie usterki i przedstawić mieszkańcom wizualny dowód naprawy.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Podczas zmiany statusu na `rozwiązane` (lub w edycji rozwiązanego zgłoszenia) pracownik urzędu może przesłać plik graficzny oznaczony w systemie jako typ `after` (`issue_attachments.type = 'after'`).
  - [ ] Zdjęcie po naprawie jest wyświetlane na publicznej karcie zgłoszenia w sekcji porównawczej „Przed naprawą / Po naprawie”.

---

## EPIC-06: Statystyki Publiczne

### US-16: Przeglądanie publicznych statystyk realizacji zgłoszeń
* **Rola:** Gość / Mieszkaniec
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-17` (odpowiada `US-07` z PRD)
* **Historyjka:**
  > **Jako** mieszkaniec,  
  > **chcę** przeglądać publiczne statystyki obejmujące liczbę zgłoszeń według kategorii, średni czas realizacji oraz liczbę rozwiązanych zgłoszeń,  
  > **aby** móc ocenić efektywność służb miejskich i zobaczyć, jakie problemy najczęściej występują w mieście.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Widok statystyk jest dostępny publicznie bez konieczności logowania (`GET /api/v1/statistics`).
  - [ ] Prezentowane jest zestawienie oraz wykres liczby zgłoszeń w podziale na poszczególne kategorie.
  - [ ] Wyświetlana jest łączna liczba rozwiązanych zgłoszeń (status `rozwiązane`) na tle wszystkich zgłoszeń.
  - [ ] Wyświetlany jest średni czas realizacji zgłoszenia wyliczany na podstawie różnicy między datą utworzenia (`created_at`) a datą rozwiązania (`resolved_at`).

---

## EPIC-07: Panel Administratora i Analityka Przestrzenna

### US-17: Wyświetlanie mapy ciepła (obszarów z największą liczbą zgłoszeń)
* **Rola:** Administrator (`admin`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-18` (odpowiada części `US-08` z PRD)
* **Historyjka:**
  > **Jako** administrator systemu,  
  > **chcę** widzieć specjalną mapę obszarów z największą liczbą zgłoszeń (mapę ciepła / heatmap),  
  > **aby** identyfikować najbardziej problematyczne rejony miasta wymagające kompleksowych inwestycji.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Widok mapy ciepła w panelu administratora pobiera dane z zabezpieczonego endpointu `GET /api/v1/admin/heatmap`.
  - [ ] Warstwa `leaflet.heat` renderuje natężenie zgłoszeń z uwzględnieniem wagi (np. liczby zgłoszeń i głosów „dotyczy mnie” w danym punkcie).
  - [ ] Administrator może filtrować mapę ciepła po wybranej kategorii usterek oraz zakresie dat.

---

### US-18: Zarządzanie słownikiem kategorii zgłoszeń
* **Rola:** Administrator (`admin`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-20` (rozwinięcie `US-08` z PRD)
* **Historyjka:**
  > **Jako** administrator systemu,  
  > **chcę** dodawać, edytować i dezaktywować kategorie zgłoszeń,  
  > **aby** dostosowywać listę obsługiwanych typów usterek do potrzeb miasta.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Panel administratora umożliwia utworzenie nowej kategorii (nazwa, opis, ikona/kolor znacznika) oraz edycję istniejących kategorii.
  - [ ] Domyślny słownik zawiera kategorie: *uszkodzona latarnia*, *dziura w drodze*, *uszkodzony znak*, *śmieci*, *zniszczona ławka*.
  - [ ] Usunięcie/dezaktywacja kategorii nie usuwa historycznych zgłoszeń przypisanych do tej kategorii, lecz ukrywa ją przy tworzeniu nowych zgłoszeń.

---

### US-19: Zarządzanie użytkownikami oraz ich rolami
* **Rola:** Administrator (`admin`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 3 Story Points
* **Powiązane wymagania PRD:** `FR-19` (rozwinięcie `US-08` z PRD)
* **Historyjka:**
  > **Jako** administrator systemu,  
  > **chcę** przeglądać listę użytkowników, nadawać im role (Mieszkaniec, Pracownik Urzędu, Administrator) oraz blokować konta naruszające regulamin,  
  > **aby** kontrolować uprawnienia w platformie.
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Tabela użytkowników w panelu admina pozwala wyszukiwać konta po nazwie lub adresie e-mail.
  - [ ] Administrator może zmienić rolę użytkownika (`user`, `official`, `admin`) za pomocą pakietu `spatie/laravel-permission`.
  - [ ] Zablokowanie konta (`is_active = false`) natychmiast unieważnia aktywne tokeny Sanctum danego użytkownika i uniemożliwia mu dodawanie nowych zgłoszeń.

---

### US-20: Przeglądanie i filtrowanie logów z aplikacji
* **Rola:** Administrator (`admin`)
* **Priorytet (MoSCoW):** Must Have
* **Estymacja:** 5 Story Points
* **Powiązane wymagania PRD:** `FR-21` (rozwinięcie `US-08` z PRD)
* **Historyjka:**
  > **Jako** administrator systemu,  
  > **chcę** przeglądać logi z aplikacji bezpośrednio w panelu administracyjnym,  
  > **aby** monitorować błędy systemowe oraz audytować kluczowe operacje (logowania, zmiany statusów, zmiany ról).
* **Kryteria akceptacji (Acceptance Criteria):**
  - [ ] Widok logów pobiera stronicowane wpisy z endpointu `GET /api/v1/admin/logs` (tabela `system_logs`).
  - [ ] Każdy wpis wyświetla: datę i godzinę, poziom (`AUDIT`, `INFO`, `WARNING`, `ERROR`), typ akcji, wykonującego użytkownika, adres IP oraz szczegóły kontekstu (JSON).
  - [ ] Administrator może filtrować logi po poziomie ważności, typie akcji oraz wyszukiwać frazy tekstowe.
