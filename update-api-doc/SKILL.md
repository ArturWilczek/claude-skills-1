---
name: update-api-doc
description: Tworzy (jeśli nie istnieje) lub aktualizuje plik ./API.md — kompletną dokumentację wszystkich endpoint-ów REST API aplikacji (URL, metoda HTTP, dane wejściowe z podziałem na body / URL params / query params, kontrola autentykacji, biznesowy opis operacji).
---

Przeanalizuj backend i wygeneruj/aktualizuj `./API.md` — dokumentację wszystkich endpoint-ów REST API.

---

**Cel:**
Utrzymywać `./API.md` jako pojedyncze, aktualne źródło prawdy o wszystkich endpoint-ach REST API aplikacji. Plik ma być kompletny, spójny z bieżącą zawartością `server/api/`, `server/services/` i `server/dtos/`, oraz czytelny dla osoby integrującej się z API (frontend, testy E2E, klient zewnętrzny).

---

**Kroki:**

1. **Odczytaj kontekst projektu**
   - `./SPEC.md` — ogólna specyfikacja (sekcje o endpoint-ach i autoryzacji)
   - `./CLAUDE.md` — konwencje (m.in. helpery walidacji, wzorzec thin handler + service)
   - `./API.md` — jeśli istnieje, jako punkt odniesienia (zachowaj strukturę / styl, jeśli jest dobrej jakości; w razie potrzeby przebuduj)

2. **Zinwentaryzuj wszystkie endpoint-y**

   Za pomocą Glob znajdź wszystkie pliki w `server/api/**/*.ts`. Dla każdego ustal:
   - **URL** — z nazwy pliku i ścieżki katalogu (zgodnie z konwencją Nuxt: `server/api/foo/[id].delete.ts` → `DELETE /api/foo/:id`)
   - **Metoda HTTP** — z sufiksu pliku (`.get.ts`, `.post.ts`, `.put.ts`, `.patch.ts`, `.delete.ts`)
   - **Czy wymaga sesji** — sprawdź czy handler wywołuje `requireUserSession(event)` (zgodnie z konwencją z `CLAUDE.md`: każdy endpoint poza `POST /api/login` to robi)
   - **Schematy walidacji wejścia** — sprawdź wywołania `checkBodyData(event, ...)`, `checkUrlParams(event, ...)`, `checkQueryParams(event, ...)` i prześledź użyte schematy Zod do plików w `server/dtos/`
   - **Logika biznesowa** — prześledź wywołania serwisów z `server/services/` aby ustalić, co operacja faktycznie robi (jakie dane czyta/zapisuje, jakie reguły stosuje, jakie kody błędów może rzucić)

3. **Dla każdego pola wejściowego ustal:**
   - Nazwę
   - Typ (z Zod / TypeScript)
   - Czy jest wymagane / opcjonalne
   - Zakres / format (np. `int 60..250`, `email`, `ISO datetime`, dozwolone wartości enum)
   - Krótki opis biznesowy (jeśli nazwa nie jest oczywista)

   Schematy Zod są źródłem prawdy — odczytaj je z `server/dtos/<obszar>.ts`, nie zgaduj.

4. **Wygeneruj treść `./API.md`**

   **Wymagana struktura dokumentu:**

   ```markdown
   # API CPW — dokumentacja endpoint-ów REST

   <Wstęp: krótko o adresie bazowym (`/api`), konwencji autoryzacji
   (sesja przez `nuxt-auth-utils` cookie, każdy endpoint poza `POST /api/login`
   wymaga sesji), formacie odpowiedzi (JSON), kodach błędów
   (`400` walidacja, `401` brak sesji, `403` brak dostępu, `413` payload za duży,
   `422` błąd analizy AI, `429` rate limit, `5xx` błąd serwera) oraz CSRF dla
   metod mutujących.>

   ## Spis endpoint-ów

   <Tabela: Metoda | Ścieżka | Sesja | Krótki opis — wszystkie endpoint-y, w kolejności obszarów>

   ## <Obszar 1: np. Autentykacja>

   ### `POST /api/login`

   <Opis biznesowy 1–3 zdania.>

   - **Sesja:** nie / tak (`requireUserSession`)
   - **Body** (`application/json`):

     | Pole       | Typ    | Wymagane | Walidacja    | Opis              |
     | ---------- | ------ | -------- | ------------ | ----------------- |
     | `email`    | string | tak      | format email | Login użytkownika |
     | `password` | string | tak      | min. 1 znak  | Hasło             |

   - **URL params:** brak
   - **Query params:** brak
   - **Odpowiedź sukcesu (`200`):** <opis kształtu lub: pusta>
   - **Możliwe błędy:**
     - `400` — niepoprawne dane wejściowe (Zod)
     - `401` — niepoprawny email lub hasło
     - `429` — przekroczony rate limit (zaostrzony dla logowania)

   ### <kolejny endpoint>

   ...
   ```

   **Zasady redakcyjne:**
   - Język polski; nazwy techniczne (HTTP, JSON, body) zostawiaj po angielsku tam, gdzie to naturalne
   - Pogrupuj endpoint-y wg obszarów funkcjonalnych: Autentykacja, Ciśnienie, Puls, Waga, Statystyki, Profil, Analiza zdjęcia (AI)
   - Każda sekcja endpoint-u podaje: opis biznesowy, sesję, body, URL params, query params, kształt odpowiedzi sukcesu, możliwe błędy. Sekcję pomijaj („brak"), a nie usuwaj — czytelnik powinien jednoznacznie widzieć, że dany element nie występuje
   - Jeśli endpoint ma specjalne ograniczenia (zaostrzony rate limit, większy limit rozmiaru body, nietypowy `content-type`) — odnotuj to wprost
   - Dla pól z Zod podaj walidację dosłownie (`int 60..250`, `min 8 znaków`, `enum: '7d' | '14d' | '1m' | '3m' | '1y'`)
   - Dla odpowiedzi nie zgaduj kształtu — odczytaj go z kodu serwisu (typ zwracany)
   - Stosuj tabele Markdown dla pól; zachowaj jednolity nagłówek tabeli we wszystkich endpoint-ach

5. **Zapisz / zaktualizuj `./API.md`**
   - Jeśli plik nie istnieje — utwórz go.
   - Jeśli istnieje — zaktualizuj go, zachowując strukturę z punktu 4. Nie zostawiaj nieaktualnych endpoint-ów (jeśli plik handler-a nie istnieje, odpowiednia sekcja musi zniknąć).

6. **Podsumuj zmiany**

   Poinformuj użytkownika:
   - Czy plik `./API.md` został utworzony, czy zaktualizowany
   - Ile endpoint-ów zostało udokumentowanych (łącznie)
   - Które endpoint-y zostały dodane / usunięte / zmienione względem poprzedniej wersji (jeśli plik istniał)
   - Czy wykryto rozbieżności między kodem a `SPEC.md` (np. endpoint istnieje w kodzie, ale nie ma go w spec lub odwrotnie) — zgłoś je użytkownikowi do decyzji, nie poprawiaj `SPEC.md` samodzielnie

---

**Ważne zasady:**

- **Źródłem prawdy jest kod**, nie `SPEC.md`. Jeśli `SPEC.md` mówi co innego niż kod — dokumentuj kod, a rozbieżność zgłoś użytkownikowi.
- Nie wymyślaj endpoint-ów, pól, ani kodów błędów. Każdy element musi mieć potwierdzenie w `server/api/`, `server/services/` lub `server/dtos/`.
- Nie dokumentuj wewnętrznych funkcji serwisów — tylko publiczne endpoint-y REST.
- Nie modyfikuj `SPEC.md`, `CLAUDE.md` ani plików kodu — ten skill aktualizuje wyłącznie `./API.md`.
- Zachowaj porządek: spójny styl tabel, jednolite nagłówki, ten sam zestaw podsekcji w każdym endpoint-cie.
- Jeśli endpoint zwraca surowe encje Prisma — opisz pola encji raz w sekcji wstępnej („Modele danych zwracane przez API") i odwołuj się do nich, zamiast powtarzać schemat w każdym endpoint-cie.
