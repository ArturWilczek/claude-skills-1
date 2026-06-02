---
name: update-task
description: Aktualizuje plik zadania NNNN.md o nowe wymagania i realizuje je w kodzie. Argumenty: nr zadania + opis zmian. Przykład: /update-task 4 Przycisk submit ma być nieaktywny gdy formularz niepoprawny
---

Zaktualizuj zadanie i zaimplementuj zmiany. Argumenty: **$ARGS**

---

Pierwszy token `$ARGS` to numer zadania, reszta to opis nowych wymagań.

**Przykład:** `4 Przycisk submit ma być nieaktywny gdy formularz niepoprawny`
→ numer zadania: `4`, wymagania: `Przycisk submit ma być nieaktywny gdy formularz niepoprawny`

---

**Kroki:**

1. **Odczytaj kontekst**

   Odczytaj kolejno:
   - `./_TASKS/NNNN.md` — aktualny plan zadania (gdzie NNNN to czterocyfrowy numer z pierwszego tokenu `$ARGS`)
   - `./CLAUDE.md` — konwencje architektoniczne

   Jeśli plik `./_TASKS/NNNN.md` nie istnieje — poinformuj użytkownika i zakończ.

2. **Zaktualizuj plik zadania**

   Zmodyfikuj `./_TASKS/NNNN.md` tak, aby odzwierciedlał nowe wymagania z `$ARGS`:
   - Jeśli wymaganie rozbudowuje istniejącą sekcję — zaktualizuj tę sekcję
   - Jeśli wymaganie wprowadza nową funkcję — dodaj ją do odpowiedniej sekcji (Frontend / Backend — API / Backend — serwisy)
   - Jeśli wymaganie jest poprawką zachowania — dodaj je do sekcji „Uwagi implementacyjne" jako zasadę
   - Nie usuwaj istniejących treści — tylko rozszerzaj i koryguj
   - Zachowaj strukturę i język polski

3. **Oceń zakres implementacji**

   Zdecyduj, czy nowe wymagania wymagają zmian w kodzie:
   - **Tak** — jeśli dotyczą działania funkcji, UI, API lub logiki biznesowej
   - **Nie** — jeśli to wyłącznie zmiana dokumentacji lub doprecyzowanie istniejącego opisu

   Jeśli zadanie ma status `Planowane` (nie zrealizowane), zaktualizuj tylko plik zadania — implementacji jeszcze nie ma, więc nie ma co poprawiać. Poinformuj użytkownika, że zmiany zostaną uwzględnione przy realizacji zadania przez `/do-task`.

4. **Zaimplementuj zmiany w kodzie** *(tylko gdy zadanie jest `Zrealizowane` i zmiany dotyczą kodu)*

   Wprowadź w kodzie wyłącznie to, czego dotyczą nowe wymagania z `$ARGS`. Przestrzegaj konwencji z `CLAUDE.md`:
   - Handlery API: wyłącznie `requireUserSession` + walidacja Zod + wywołanie serwisu
   - `if` bez nawiasów klamrowych gdy treść to jedno polecenie
   - UI i komunikaty w języku polskim
   - Formularze przez `UForm` z `:schema` (Zod) i `:state`; `UFormField name="..."` do wyświetlania błędów przy polach
   - Pinia setup stores (`defineStore` z Composition API)
   - Okna dialogowe przez `useOverlay` / `useConfirmDialog`
   - Powiadomienia przez `useAppMessages()`

5. **Zweryfikuj implementację** *(jeśli były zmiany w kodzie)*

   Uruchom:
   ```bash
   npm run typecheck
   ```
   Napraw wszystkie błędy TypeScript przed przejściem dalej.

6. **Podsumuj wynik**

   Poinformuj użytkownika:
   - Co zostało zmienione w pliku zadania
   - Co zostało zaimplementowane w kodzie (jeśli dotyczy)
   - Czy typecheck przeszedł bez błędów (jeśli dotyczy)
