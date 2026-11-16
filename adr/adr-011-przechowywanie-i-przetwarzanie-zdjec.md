# ADR-011: Przechowywanie i obsługa załączników fotograficznych (zdjęcia usterek oraz zdjęcia po naprawie)

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

System wymaga obsługi dokumentacji fotograficznej w dwóch momentach cyklu życia zgłoszenia:
1. **Podczas zgłaszania problemu przez mieszkańca:** użytkownik dodaje opis oraz zdjęcia przedstawiające usterkę (typ `before`).
2. **Podczas zamykania zgłoszenia przez pracownika urzędu:** rozwiązane zgłoszenie może zawierać zdjęcie wykonane po naprawie (typ `after`), stanowiące publiczny dowód usunięcia awarii.

Należało określić sposób przesyłania, walidacji, składowania i serwowania plików graficznych w architekturze REST API + SPA.

---

## 2. Rozważane opcje

### Opcja 1: Przechowywanie obrazów jako BLOB / Base64 bezpośrednio w bazie SQLite
* **Opis:** Kodowanie zdjęć do formatu Base64 i zapis w kolumnie `TEXT` lub `BLOB` w pliku `database.sqlite`.
* **Zalety:** Wszystko w jednym pliku bazodanowym.
* **Wady:** Gwałtowny rozrost pliku bazy SQLite do setek megabajtów, drastyczny spadek wydajności zapytań i wysoka konsumpcja pamięci RAM przy serializacji odpowiedzi JSON.

### Opcja 2: Zewnętrzna chmura obiektowa (AWS S3 / Cloudflare R2 / Cloudinary)
* **Opis:** Przesyłanie zdjęć do zewnętrznego dostawcy chmurowego.
* **Zalety:** Odciążenie dysku serwera.
* **Wady:** Wymaga konfiguracji płatnych kont i kluczy dostępowych, co utrudnia lokalne uruchomienie projektu na uczelni.

### Opcja 3: Laravel Filesystem Storage (`public` disk) + tabela metadanych `issue_attachments` z rozróżnieniem roli zdjęcia (`before` / `after`)
* **Opis:**
  * Przesyłanie plików przez żądania `multipart/form-data`.
  * Zapis plików na dysku lokalnym w strukturze katalogów `storage/app/public/issues/{issue_id}/{type}/{uuid}.{ext}` udostępnianej przez dowiązanie symboliczne `php artisan storage:link` pod publicznym adresem URL.
  * Przechowywanie metadanych w tabeli `issue_attachments` z kolumną `type` przyjmującą wartości:
    * `before` - zdjęcia pierwotnej usterki dodane przez zgłaszającego mieszkańca,
    * `after` - zdjęcia powykonawcze dodane przez pracownika urzędu przy statusie `rozwiązane`.
* **Zalety:** Szybka praca bazy SQLite (przechowuje wyłącznie lekkie ścieżki tekstowe); natywne wsparcie dla pamięci podręcznej przeglądarki (HTTP caching dla statycznych obrazów); brak zależności od zewnętrznych płatnych usług chmurowych, przy jednoczesnym zachowaniu pełnej abstrakcji `Storage::disk()`, która w razie wdrożenia produkcyjnego pozwala przełączyć sterownik na `s3` jedną zmienną w `.env`.
* **Wady:** Wymaga wykonania komendy `php artisan storage:link` podczas pierwszej konfiguracji projektu.

---

## 3. Decyzja

Wybrano **Opcję 3**: Abstrakcja **Laravel Filesystem (`public` disk)** wraz z tabelą **`issue_attachments`** rozróżniającą zdjęcia usterki (`before`) oraz zdjęcia po naprawie (`after`).

---

## 4. Konsekwencje

### Pozytywne:
* **Czytelna prezentacja „Przed i Po naprawie”:** Dzięki rozdzieleniu typów załączników na `before` i `after`, komponent szczegółów zgłoszenia w React SPA może wyświetlić atrakcyjne wizualnie porównanie stanu przed interwencją urzędu i po wykonaniu naprawy.
* **Bezpieczeństwo:** Wszystkie przesyłane pliki przechodzą rygorystyczną walidację po stronie backendu (`image`, `mimes:jpg,jpeg,png,webp`, `max:5120` KB) oraz otrzymują losowe nazwy UUID, co eliminuje ryzyko nadpisania plików lub wykonania złośliwego skryptu (Path Traversal / RCE).

### Negatywne i mitygacja:
* **Usuwanie osieroconych plików:** W przypadku usunięcia zgłoszenia lub zastąpienia zdjęcia po naprawie nowym plikiem, odpowiedni Observer modelu `IssueAttachment` automatycznie usuwa fizyczny plik z dysku (`Storage::disk('public')->delete(...)`).
