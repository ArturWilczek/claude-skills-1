---
name: do-task
description: Realizuje zadanie o podanym numerze z TASKS.md i po zakończeniu aktualizuje jego status.
---

Zrealizuj zadanie numer: **$ARGS**

---

**Cel:**
Przeczytaj plan zadania, zaimplementuj wszystkie elementy z zakresu, a następnie zaktualizuj status zadania w `./TASKS.md`.

**Kroki:**

1. **Odczytaj kontekst projektu**

   Odczytaj kolejno:
   - `./TASKS.md` — lista zadań (sprawdź aktualny status zadania `$ARGS`)
   - `./_TASKS/$ARGS.md` — szczegółowy plan realizacji zadania
   - `./CLAUDE.md` — konwencje architektoniczne
   - `./SPEC.md` — specyfikacja projektu (tylko sekcje powiązane z zadaniem)

   Jeśli plik `./_TASKS/$ARGS.md` nie istnieje — poinformuj użytkownika i zakończ.
   Jeśli zadanie ma już status `Zrealizowane` — poinformuj użytkownika i zakończ.

2. **Sprawdź zależności**

   Na podstawie sekcji "Uwagi implementacyjne" w pliku zadania sprawdź, czy zadanie ma zależności od wcześniejszych zadań. Jeśli tak — zweryfikuj, że pliki wynikowe tych zadań faktycznie istnieją w repozytorium (np. sprawdź kluczowe pliki przez Glob/Grep). Jeśli wymagane pliki brakuje — poinformuj użytkownika, że trzeba najpierw zrealizować zadania poprzednie.

3. **Zaimplementuj zadanie**

   Zrealizuj pełny zakres opisany w pliku `./_TASKS/$ARGS.md`:
   - **Baza danych / Model danych** — utwórz lub zaktualizuj schemat Prisma, uruchom migrację
   - **Backend — serwisy** — utwórz pliki w `server/services/` z logiką biznesową i dostępem do Prisma
   - **Backend — API** — utwórz endpointy w `server/api/` (tylko `requireUserSession` + Zod + wywołanie serwisu)
   - **Frontend** — utwórz strony w `app/pages/`, komponenty w `app/components/`, stores w `app/stores/`

   Przestrzegaj konwencji z `CLAUDE.md`:
   - Handlery API: wyłącznie `requireUserSession` + walidacja Zod + wywołanie serwisu
   - `if` bez nawiasów klamrowych gdy treść to jedno polecenie
   - UI i komunikaty w języku polskim
   - Pinia setup stores (`defineStore` z Composition API)
   - Okna dialogowe przez `useOverlay` / `useConfirmDialog`
   - Powiadomienia przez `useAppMessages()`

4. **Zweryfikuj implementację**

   Po zaimplementowaniu uruchom:

   ```bash
   pnpm typecheck
   ```

   Napraw wszystkie błędy TypeScript przed przejściem dalej.

   Jeśli zadanie obejmuje schemat Prisma — upewnij się, że migracja została uruchomiona.

5. **Zaktualizuj status w TASKS.md**

   Zmień status zadania `$ARGS` w pliku `./TASKS.md` z `Planowane` na `Zrealizowane`.

   Format wiersza w tabeli:

   ```
   | [NNNN](./_TASKS/NNNN.md) | Zrealizowane | Tytuł zadania |
   ```

6. **Podsumuj wynik**

   Poinformuj użytkownika:
   - Co zostało zaimplementowane (lista kluczowych plików)
   - Czy typecheck przeszedł bez błędów
   - Czy są jakieś ręczne kroki do wykonania (np. uruchomienie serwera deweloperskiego, konfiguracja `.env`)
