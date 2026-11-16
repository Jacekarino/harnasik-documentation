# ADR-007: Dokumentowanie REST API w standardzie OpenAPI 3.0 - L5-Swagger

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Jednym z kluczowych wymagań projektu jest udokumentowanie odseparowanego backendu w formie **OpenAPI**. Dokumentacja ta musi precyzyjnie opisywać wszystkie punkty końcowe (endpointy), parametry ścieżki i zapytań, schematy żądań (w tym `multipart/form-data` dla zdjęć przed i po naprawie), kody odpowiedzi HTTP oraz wymagania autoryzacyjne (Bearer Token). Należało wybrać podejście gwarantujące, że dokumentacja nie zdezaktualizuje się w trakcie prac programistycznych.

---

## 2. Rozważane opcje

### Opcja 1: Ręczne pisanie i utrzymywanie pliku `openapi.yaml` (Design-First)
* **Opis:** Tworzenie specyfikacji w osobnym edytorze (np. Swagger Editor) bez powiązania z kodem PHP.
* **Zalety:** Pełna swoboda edycji pliku YAML.
* **Wady:** Bardzo wysokie ryzyko rozjazdu (dryfu dokumentacyjnego) między plikiem YAML a rzeczywistą implementacją kontrolerów i klas walidacji w Laravelu.

### Opcja 2: Generowanie z atrybutów PHP 8 (`darkaonline/l5-swagger` + `zircote/swagger-php`)
* **Opis:** Umieszczanie ustrukturyzowanych atrybutów PHP 8 (`#[OA\Get]`, `#[OA\Post]`, `#[OA\Schema]`, `#[OA\Response]`) bezpośrednio nad metodami kontrolerów oraz klasami `JsonResource` i `FormRequest`, skąd komenda `php artisan l5-swagger:generate` automatycznie buduje plik `api-docs.json` i wystawia interaktywny interfejs **Swagger UI**.
* **Zalety:**
  * Dokumentacja znajduje się bezpośrednio przy kodzie źródłowym kontrolera - każda zmiana endpointu w tym samym commicie aktualizuje specyfikację OpenAPI.
  * Wbudowany w aplikację panel Swagger UI (`/api/documentation`) umożliwia testowanie endpointów na żywo w przeglądarce.
* **Wady:** Większa liczba linii w plikach kontrolerów (możliwa do ograniczenia poprzez wydzielenie schematów do osobnych klas/interfejsów).

### Opcja 3: Automatyczna refleksja bez atrybutów (np. `dedoc/scramble`)
* **Opis:** Statyczna analiza kodu kontrolerów bez konieczności pisania adnotacji.
* **Zalety:** Brak narzutu w kodzie kontrolerów.
* **Wady:** Mniejsza precyzja przy niestandardowych strukturach odpowiedzi (np. zagregowane dane dla mapy ciepła lub wieloplikowe formularze `multipart/form-data`).

---

## 3. Decyzja

Wybrano **Opcję 2**: Pakiet **`darkaonline/l5-swagger`** wykorzystujący natywne **atrybuty PHP 8 (OpenAPI 3.0 Attributes)** do automatycznego generowania specyfikacji oraz udostępniania interaktywnego środowiska **Swagger UI**.

---

## 4. Konsekwencje

### Pozytywne:
* **Zawsze aktualny kontrakt API:** Specyfikacja `openapi.json` jest generowana bezpośrednio z kodu i udostępniana zarówno dla członków zespołu rozwijających frontend w React + TypeScript, jak i do weryfikacji zaliczeniowej projektu.
* **Interaktywne testowanie:** Możliwość autoryzacji tokenem Sanctum bezpośrednio w Swagger UI i przetestowania wszystkich ścieżek (zgłaszanie usterki, sprawdzanie duplikatów w pobliżu, zmiana statusu przez urzędnika, pobranie statystyk i logów).
* **Możliwość generowania typów TypeScript:** Plik `api-docs.json` może służyć jako źródło dla generatorów interfejsów TS po stronie frontendu.

### Negatywne i mitygacja:
* **Rozmiar plików kontrolerów:** Aby zachować wysoką czytelność metod kontrolerów, powtarzalne definicje modeli (`#[OA\Schema]`) wydzielono do osobnego katalogu `app/OpenApi/Schemas/`.
