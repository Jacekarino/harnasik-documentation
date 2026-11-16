# ADR-005: Uwierzytelnianie użytkowników - Laravel Sanctum oraz integracja OAuth 2.0 (Laravel Socialite)

* **Status:** Zaakceptowane
* **Data:** 2026-10-28
* **Autorzy:** Jakub Broszko, Maciej Wróblewski, Jacek Kowalczyk

---

## 1. Kontekst i problem

Wymagania projektowe określają dwa obowiązkowe sposoby rejestracji i logowania użytkownika w systemie:
1. **Klasycznie:** przy użyciu adresu e-mail oraz hasła.
2. **Przez zewnętrznego dostawcę OAuth:** np. konto Google lub GitHub.

Ponieważ architektura systemu (ADR-001) zakłada pełne odseparowanie frontendu (React SPA) od backendu (Laravel REST API), mechanizm uwierzytelniania musi działać bezstanowo w oparciu o nagłówki autoryzacyjne HTTP i bezpiecznie przekazywać stan uwierzytelnienia po zakończeniu przepływu OAuth 2.0.

---

## 2. Rozważane opcje

### Opcja 1: Sesje oparte wyłącznie o ciasteczka (Cookie-based Session Auth)
* **Opis:** Tradycyjne uwierzytelnianie sesyjne z ciasteczkami `HttpOnly`.
* **Zalety:** Odporność ciasteczek `HttpOnly` na odczyt z poziomu JS.
* **Wady:** Wymaga ścisłej zgodności domen/subdomen pomiędzy SPA a API oraz skomplikowanej obsługi tokenów CSRF przy każdym zapytaniu oraz podczas testowania endpointów w Swagger UI (OpenAPI).

### Opcja 2: Pełny serwer OAuth2 po stronie backendu (Laravel Passport / JWT)
* **Opis:** Wdrożenie własnego serwera autoryzacyjnego wystawiającego tokeny JWT z kluczami asymetrycznymi RSA.
* **Zalety:** Samowystarczalne tokeny JWT.
* **Wady:** Nadmierna złożoność konfiguracyjna; utrudnione natychmiastowe unieważnianie tokenów (np. w momencie zablokowania użytkownika przez administratora).

### Opcja 3: Laravel Sanctum (API Bearer Tokens) + Laravel Socialite (OAuth 2.0 Client)
* **Opis:**
  * **Laravel Sanctum** odpowiada za generowanie, weryfikację i unieważnianie tokenów dostępowych (`Personal Access Tokens`) przesyłanych w nagłówku `Authorization: Bearer <token>`.
  * **Laravel Socialite** obsługuje komunikację z zewnętrznym dostawcą OAuth 2.0 (np. Google / GitHub) w trybie *stateless*. Po poprawnym callbacku od dostawcy OAuth backend znajduje lub tworzy użytkownika w tabeli `users`, generuje dla niego token Sanctum i przekierowuje przeglądarkę z powrotem do aplikacji React SPA wraz z tokenem.
* **Zalety:** Lekka i natywna integracja z Laravel; bardzo proste testowanie zabezpieczonych endpointów w dokumentacji OpenAPI (Swagger UI posiada wbudowane pole na token `Bearer`); możliwość natychmiastowego unieważnienia sesji wybranego użytkownika przez administratora.
* **Wady:** Konieczność bezpiecznej obsługi callbacku przekierowującego między API a SPA.

---

## 3. Decyzja

Wybrano **Opcję 3**: Połączenie **Laravel Sanctum** (tokeny Bearer API dla logowania klasycznego i autoryzacji żądań REST) oraz **Laravel Socialite** (do obsługi zewnętrznych dostawców OAuth 2.0).

---

## 4. Konsekwencje

### Pozytywne:
* **Ujednolicona autoryzacja po zalogowaniu:** Niezależnie od tego, czy użytkownik zalogował się klasycznie (e-mail + hasło), czy przez OAuth 2.0, aplikacja React SPA otrzymuje ten sam typ tokenu `Bearer`, który dołącza w interceptorze Axios do każdego kolejnego żądania API.
* **Automatyczne łączenie kont (Account Linking):** Jeśli użytkownik zarejestrowany wcześniej klasycznie na dany adres e-mail użyje logowania OAuth z tym samym zweryfikowanym adresem e-mail, system powiąże jego konto z identyfikatorem `oauth_id` zamiast tworzyć duplikat.
* **Pełna kompatybilność z OpenAPI:** Schemat bezpieczeństwa `securityScheme: bearerAuth` w Swagger UI działa bezpośrednio z tokenami generowanymi przez Sanctum.

### Negatywne i mitygacja:
* **Przekazanie tokenu z callbacku OAuth do SPA:** Aby uniknąć długotrwałego eksponowania tokenu w historii przeglądarki, komponent odbiorczy w React SPA (`/auth/oauth/callback`) natychmiast po odczytaniu parametru zapisuje token i czyści pasek adresu za pomocą `window.history.replaceState` (lub wymienia jednorazowy kod na token przez `POST`).
