# ADR-010: Maszyna stanów zgłoszenia, obserwowanie oraz system powiadomień o zmianie statusu

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Wymagania funkcjonalne precyzują pełny cykl życia zgłoszenia obsługiwanego przez pracownika urzędu oraz powiązane z nim interakcje:
1. Pracownik urzędu może zmieniać status zgłoszenia na jeden z 6 stanów:
   * `zgłoszone` (`reported`),
   * `przyjęte` (`accepted`),
   * `przekazane do realizacji` (`forwarded`),
   * `w realizacji` (`in_progress`),
   * `rozwiązane` (`resolved`),
   * `odrzucone` (`rejected`).
2. Pracownik urzędu może dodać **publiczny komentarz dotyczący realizacji**.
3. Użytkownicy mogą **obserwować zgłoszenia** i muszą **dostawać powiadomienie po zmianie statusu obserwowanego zgłoszenia**.
4. System musi wyliczać w statystykach publicznych **średni czas realizacji** oraz **liczbę rozwiązanych zgłoszeń**.

Należało zaprojektować spójny mechanizm zarządzania zmianą statusów, rejestrowania historii zmian i rozsyłania powiadomień.

---

## 2. Rozważane opcje

### Opcja 1: Bezpośrednia podmiana pola `status` w kontrolerze CRUD bez historii i zdarzeń
* **Opis:** Zwykły `Issue::where('id', $id)->update(['status' => $request->status])`.
* **Zalety:** Minimalna ilość kodu.
* **Wady:** Brak historii zmian statusów; utrudnione wyliczanie średniego czasu realizacji; silne sprzężenie kontrolera z wysyłką powiadomień.

### Opcja 2: Natywny PHP 8.1+ Backed Enum (`IssueStatus`) + Architektura Zdarzeniowa (Laravel Events & Notifications) + znacznik czasu `resolved_at`
* **Opis:**
  * Zdefiniowanie typu wyliczeniowego `App\Enums\IssueStatus` reprezentującego 6 wymaganych statusów wraz z ich polskimi etykietami i dozwolonymi przejściami.
  * Każda zmiana statusu przez pracownika urzędu przechodzi przez `IssueStatusWorkflowService`, który w transakcji bazodanowej:
    1. Aktualizuje status w tabeli `issues` (oraz ustawia kolumnę `resolved_at = now()`, jeśli nowy status to `resolved`, co pozwala błyskawicznie liczyć średni czas realizacji w SQL).
    2. Zapisuje wpis w tabeli `issue_status_histories` (kto zmienił, z jakiego statusu, na jaki, kiedy oraz opcjonalny komentarz).
    3. Emituje zdarzenie domenowe `IssueStatusChanged`.
  * Listener `SendIssueStatusChangedNotifications` pobiera wszystkich użytkowników obserwujących dane zgłoszenie (relacja `followers` w tabeli `issue_followers` - obejmująca autora, osoby które kliknęły „dotyczy mnie” oraz osoby które kliknęły „Obserwuj”) i wysyła im powiadomienie przez **Laravel Notifications** (kanał `database` dostępny w dzwoneczku powiadomień w SPA oraz opcjonalnie kanał `mail`).

---

## 3. Decyzja

Wybrano **Opcję 2**: Wykorzystanie silnie typowanego **`IssueStatus` Enum**, dedykowanego serwisu przepływu pracy zapisującego historię zmian i znacznik `resolved_at`, oraz systemu **Laravel Events + Laravel Notifications** do powiadamiania obserwujących użytkowników.

---

## 4. Konsekwencje

### Pozytywne:
* **Pełna transparentność dla mieszkańców:** Na karcie zgłoszenia widoczna jest kompletna oś czasu (timeline) pokazująca kiedy zgłoszenie zostało przyjęte, przekazane do realizacji, wzięte w realizację i ostatecznie rozwiązane, wraz z publicznymi komentarzami urzędnika.
* **Niezawodne powiadomienia:** Każdy obserwujący użytkownik natychmiast po zmianie statusu widzi w interfejsie React nieprzeczytane powiadomienie (np. *„Zgłoszenie #42 (Uszkodzona latarnia) zmieniło status z 'w realizacji' na 'rozwiązane'”*).
* **Dokładne i szybkie statystyki publiczne:** Kolumna `resolved_at` w tabeli `issues` umożliwia obliczenie średniego czasu realizacji jednym zapytaniem agregującym `AVG(EXTRACT(EPOCH FROM (resolved_at - created_at)))` w PostgreSQL bez konieczności przeszukiwania logów.

### Negatywne i mitygacja:
* **Ponowne otwarcie zgłoszenia:** W przypadku zmiany statusu z `rozwiązane` z powrotem na `w realizacji`, serwis automatycznie czyści pole `resolved_at` (`null`), aby nie przekłamywać statystyk średniego czasu naprawy.
