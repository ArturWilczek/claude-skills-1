---
name: update-spec
description: Aktualizuje SPEC.md (i jeśli potrzeba CLAUDE.md) o nową funkcjonalność, zmianę koncepcyjną lub nowy proces opisany w argumencie.
---

Zaktualizuj specyfikację projektu. Opis zmiany: **$ARGS**

---

**Cel:**
Wbuduj zmianę opisaną w `$ARGS` w dokumenty specyfikacji projektu — `./SPEC.md` i opcjonalnie `./CLAUDE.md` — tak, żeby oba pliki były wewnętrznie spójne, zgodne z `./API.md` i gotowe do pracy przez `/update-tasks-list` lub `/do-task`.

---

**Podział treści (OBOWIĄZKOWY — taki sam jak w `/cleanup-spec`):**

- `./SPEC.md` — **warstwa biznesowa i funkcjonalna**: cel aplikacji, model danych (biznesowo), opis ekranów i procesów użytkownika, procesy biznesowe (agent przypomnień, analiza AI — co i kiedy, bez detali implementacyjnych), wymagania bezpieczeństwa, zmienne środowiskowe (lista), out-of-scope. Odpowiada na pytanie **co** aplikacja robi.
- `./CLAUDE.md` — **warstwa techniczna i architektoniczna**: komendy, stack techniczny, struktura katalogów, wzorce projektowe (`appStatus`, `apiCall`, thin handlers, walidacja przez helpery, store'y Pinia, globalny loader, kontrakt błędów), konwencje kodu, testy integracyjne, tryb debug. Odpowiada na pytanie **jak** agent ma pracować z kodem.
- `./API.md` — **pełne kontrakty REST** (URL, metoda, body / URL / query params, kody odpowiedzi, opis biznesowy). Jedyne źródło prawdy o API. SPEC.md i CLAUDE.md odnoszą się do niego **linkiem** (`[./API.md](./API.md)`) z jednozdaniowym opisem; **nigdy** nie powielają listy ani szczegółów endpoint-ów.

Każda informacja powinna mieszkać w **dokładnie jednym** pliku. W razie wątpliwości użyj tabeli z `/cleanup-spec` jako tie-breaker; jeśli temat należy do obu, krótko opisz biznesowo w SPEC.md i technicznie w CLAUDE.md (np. „bezpieczeństwo wymagane" vs „konfiguracja `nuxt-security`"), bez powtarzania tych samych treści.

---

**Kroki:**

1. **Odczytaj bieżący stan dokumentów**

   Odczytaj kolejno:
   - `./SPEC.md` — pełna specyfikacja biznesowa i funkcjonalna
   - `./CLAUDE.md` — pamięć techniczna projektu
   - `./API.md` — pełne kontrakty REST (jeśli istnieje)

2. **Przeanalizuj żądaną zmianę**

   Na podstawie `$ARGS` ustal:
   - Jakiego obszaru aplikacji dotyczy zmiana (model danych, API, UI, autoryzacja, integracje zewnętrzne, konfiguracja)?
   - Czy zmiana rozbudowuje istniejącą funkcjonalność, zastępuje ją, czy wprowadza zupełnie nową?
   - **Które dokumenty wymagają aktualizacji** — SPEC.md (biznes/UX/proces), CLAUDE.md (wzorzec/konwencja/komenda/zmienna), API.md (kontrakt endpoint-u)?
   - Czy zmiana wpływa na wzorce architektoniczne opisane w CLAUDE.md (`appStatus`, `apiCall`, thin handler, helpery walidacyjne, `nuxt-security` itd.)?

   Jeśli opis w `$ARGS` jest zbyt ogólny lub niejasny — **zapytaj użytkownika o doprecyzowanie** zanim cokolwiek zmienisz.

3. **Zaktualizuj `./SPEC.md` (warstwa biznesowo-funkcjonalna)**

   Zasady:

   - **Modyfikuj istniejące sekcje** — nie duplikuj treści; zaktualizuj to, co się zmieniło.
   - **Dodawaj nowe sekcje** — jeśli zmiana wprowadza zupełnie nowy obszar (np. nową tabelę, nowy ekran, nowy proces biznesowy).
   - **Zachowaj zalecaną kolejność sekcji** (taką samą jak ustala `/cleanup-spec`):
     1. Cel aplikacji
     2. Model danych (opis biznesowy pól, nie szczegóły techniczne)
     3. Funkcjonalności użytkownika (ekrany, formularze, statystyki, profil, logowanie, skanowanie zdjęć)
     4. Procesy biznesowe (agent przypomnień, analiza AI — **co** robią, kiedy, dla kogo; bez detali implementacji)
     5. Bezpieczeństwo (wymagania: co chronić, jakie nagłówki, jakie limity — bez konfiguracji modułu)
     6. Stack technologiczny (krótka tabela; szczegóły w CLAUDE.md)
     7. Endpoint-y REST API → **wyłącznie link** do `./API.md` z jednozdaniowym opisem
     8. Skrypty administracyjne (perspektywa administratora)
     9. Zmienne środowiskowe (lista wymaganych przez aplikację)
     10. Poza zakresem
   - **NIE wpisuj do SPEC.md treści technicznych** — nazw helperów, struktur katalogów, sygnatur funkcji, nazw store'ów Pinia, konfiguracji `nuxt-security`, szczegółów `apiCall` itd. Te należą do CLAUDE.md.
   - **NIE wpisuj do SPEC.md tabel endpoint-ów** ani szczegółowych kontraktów — odeślij linkiem do `./API.md`.
   - **Bądź precyzyjny biznesowo** — nazwy pól, dozwolone zakresy, reguły walidacji, języki interfejsu, role użytkowników — konkretnie, nie ogólnikowo.
   - **Out-of-scope** — jeśli zmiana usuwa coś z out-of-scope, przenieś do właściwej sekcji; jeśli coś wyraźnie wykracza poza zakres nowej funkcjonalności — dopisz do „Poza zakresem".

4. **Oceń potrzebę aktualizacji `./CLAUDE.md` (warstwa techniczna)**

   Aktualizuj CLAUDE.md **tylko gdy zmiana:**
   - Wprowadza nowy wzorzec architektoniczny lub modyfikuje istniejący (np. nowa warstwa pośrednia, zmiana w `apiCall`, nowa konwencja toastów, nowa zasada o store'ach).
   - Dodaje nową zależność zewnętrzną wymagającą opisu konwencji użycia.
   - Zmienia listę / znaczenie zmiennych środowiskowych istotnych z perspektywy pracy z kodem (np. `NUXT_DEBUG_LEVEL`, `NUXT_SECURITY_ENABLED`).
   - Dodaje nowe komendy CLI lub modyfikuje istniejące (`npm run …`, `npx tsx scripts/…`).
   - Zmienia konwencje kodu obowiązujące w projekcie (styl `if`, język UI, kody błędów HTTP).
   - Dodaje nowy plik tasków Nitro, nową lokalizację dla DTO / serwisów, nowe miejsce testów.

   Jeśli wpisujesz nową sekcję do CLAUDE.md, zachowaj kolejność dokumentu zgodną z `/cleanup-spec`:
   1. Linki do `./SPEC.md` i `./API.md`
   2. Komendy
   3. Stack technologiczny (perspektywa pakietów/wersji)
   4. Struktura katalogów
   5. Wzorce architektoniczne
   6. Konwencje kodu
   7. Testy integracyjne
   8. Tryb debug
   9. Zmienne środowiskowe (link do SPEC.md albo krótka tabela bez powtórzeń)

   Jeśli CLAUDE.md nie wymaga aktualizacji — pomiń ten krok.

5. **NIE modyfikuj `./API.md`**

   Ten skill **nie aktualizuje** kontraktów endpoint-ów. Jeśli zmiana z `$ARGS` wprowadza, modyfikuje lub usuwa endpoint:
   - W SPEC.md wzmiankuj funkcjonalność biznesowo (np. „użytkownik może oznaczyć pomiar jako fałszywy") i zostaw link do `./API.md`.
   - W CLAUDE.md (jeśli dotyczy) opisz wzorzec techniczny (np. nowy serwis, nowe DTO).
   - **Zaproponuj użytkownikowi uruchomienie `/update-api-doc`** w podsumowaniu (krok 7), żeby zsynchronizować `./API.md` z nowym kontraktem.

6. **Sprawdź wewnętrzną spójność**

   Po edycji upewnij się, że:
   - Żadna informacja nie została wpisana do dwóch dokumentów naraz (SPEC vs CLAUDE).
   - W SPEC.md i CLAUDE.md nie ma świeżo dopisanych tabel ani list endpoint-ów — tylko link do `./API.md`.
   - Linki względne (`./API.md`, `./SPEC.md`, `./CLAUDE.md`, kotwice `#…`) wskazują na realne pliki/sekcje.
   - Nazewnictwo i terminologia zgodne z istniejącym stylem (polski, tabele Markdown, poziomy nagłówków).

7. **Podsumuj zmiany**

   Poinformuj użytkownika:
   - Które sekcje `./SPEC.md` zostały dodane / zmodyfikowane / usunięte.
   - Czy `./CLAUDE.md` został zaktualizowany i dlaczego (lub dlaczego nie).
   - Czy zmiana wymaga aktualizacji `./API.md` — **jeśli tak, zaproponuj uruchomienie `/update-api-doc`**.
   - Czy w dokumentach pojawiło się ryzyko duplikacji lub niewłaściwego ulokowania treści — **jeśli tak, zaproponuj uruchomienie `/cleanup-spec`**.
   - Czy zmiana implikuje konieczność aktualizacji listy zadań — jeśli tak, zaproponuj uruchomienie `/update-tasks-list`.

---

**Ważne zasady:**

- Wszystkie teksty w dokumentach pisz po polsku, zgodnie z istniejącym stylem (tabele Markdown, polskie nagłówki, terminologia).
- Trzymaj się ścisłego podziału: **biznes → SPEC.md, technika → CLAUDE.md, kontrakty API → API.md (linkiem)**. Nigdy nie powielaj tej samej treści w dwóch plikach.
- Linki względne z prefiksem `./` (`./API.md`, `./SPEC.md`, `./CLAUDE.md`), żeby działały w GitHub i lokalnie.
- Nie zmieniaj niczego poza tym, czego dotyczy `$ARGS` — minimalne, celowe zmiany. Drobne porządki struktury zostaw skillowi `/cleanup-spec`.
- Nie twórz nowych plików (ADR, changelog, dodatkowe MD) — tylko aktualizuj `./SPEC.md` i `./CLAUDE.md`.
- Nie modyfikuj `./API.md` — to robi `/update-api-doc`.
- Jeśli zmiana pociąga za sobą decyzje projektowe z kilkoma opcjami — przedstaw opcje użytkownikowi i poczekaj na wybór zanim zapiszesz.
