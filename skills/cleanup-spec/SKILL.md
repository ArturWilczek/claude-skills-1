---
name: cleanup-spec
description: Porządkuje pliki ./SPEC.md i ./CLAUDE.md — usuwa duplikacje między nimi i względem ./API.md, przenosi treść do właściwego pliku, poprawia układ sekcji. SPEC.md = warstwa biznesowa/funkcjonalna; CLAUDE.md = warstwa techniczna/architektoniczna; pliki zewnętrzne (np. API.md) przywoływane przez link.
---

Uporządkuj `./SPEC.md` i `./CLAUDE.md` — usuń duplikacje, przenieś treści do właściwych dokumentów, zlinkuj zewnętrzne źródła prawdy zamiast je powielać, popraw układ sekcji.

---

**Cel:**

Oba pliki rozrastają się i zaczynają rozdmuchiwać kontekst agenta. Należy je zredukować i uporządkować tak, aby:

- `./SPEC.md` — **specyfikacja biznesowa i funkcjonalna**: model danych, ekrany, procesy, reguły biznesowe, opisy funkcjonalności użytkownika. Z naciskiem na to, **co** aplikacja robi, a nie **jak** jest zbudowana.
- `./CLAUDE.md` — **pamięć techniczna projektu**: wzorce projektowe, zalecenia architektoniczne, konwencje kodu, komendy CLI, struktura katalogów, zmienne środowiskowe. Z naciskiem na to, **jak** agent ma pracować z kodem.
- Pliki zewnętrzne (`./API.md` i inne) — pełne źródło prawdy w swoim obszarze. SPEC.md i CLAUDE.md **wskazują na nie linkiem** i opisują w jednym zdaniu co zawierają, zamiast powielać treść.

---

**Kroki:**

1. **Odczytaj bieżący stan dokumentów**
   - `./SPEC.md` — w całości
   - `./CLAUDE.md` — w całości
   - `./API.md` — w całości (jeśli istnieje); jest pełną dokumentacją REST API
   - Zinwentaryzuj pozostałe pliki MD w katalogu głównym projektu (Glob `*.md`), bo mogą istnieć inne zewnętrzne źródła prawdy (`README.md`, `CHANGELOG.md` itp.)

2. **Zbuduj mapę treści**

   Dla każdego pliku spisz listę sekcji (nagłówki ## i ###) i krótko zaznacz, czego dotyczy każda sekcja. Cel: zobaczyć duplikacje i niewłaściwe ulokowanie treści.

3. **Wykryj duplikacje i niewłaściwe ulokowanie**

   Zidentyfikuj w szczególności:
   - **SPEC.md ↔ CLAUDE.md** — te same informacje opisane dwa razy (np. lista endpoint-ów, opis wzorca thin-handler, opis store'a `appStatus`, opis modułu `nuxt-security`). Każda taka treść powinna być w **jednym** pliku.
   - **SPEC.md / CLAUDE.md ↔ API.md** — tabele endpoint-ów lub szczegółowe kontrakty endpoint-ów obecne w SPEC.md lub CLAUDE.md, gdy są już w pełni opisane w `./API.md`. W SPEC.md i CLAUDE.md zostaw tylko link i jednozdaniowy opis.
   - **Treści w niewłaściwym pliku** — np. szczegóły techniczne (helpery, struktura katalogów, wzorce) w SPEC.md albo opisy procesów biznesowych w CLAUDE.md. Przenieś je tam, gdzie pasują.
   - **Sekcje w nielogicznym miejscu** — np. „Usuwanie pomiarów" (funkcja użytkownika) wciśnięte między „Skrypty administracyjne" a „Zmienne środowiskowe". Przegrupuj.

4. **Zaplanuj docelową strukturę obu plików**

   Przed jakąkolwiek edycją zaproponuj użytkownikowi krótki plan w punktach: jakie sekcje SPEC.md zostają / są przeniesione / scalone / usunięte, to samo dla CLAUDE.md, gdzie zostaną wstawione linki do `./API.md` lub innych zewnętrznych plików. **Poczekaj na zgodę** — to operacja masowo modyfikująca dwa kluczowe dokumenty, użytkownik musi zaakceptować kierunek.

   Sugerowany docelowy kształt:

   **`./SPEC.md` — kolejność sekcji (od ogółu do szczegółu, biznes przed techniką):**
   1. Cel aplikacji
   2. Model danych (tabele i ich pola — opis biznesowy, nie techniczny)
   3. Funkcjonalności użytkownika (opis każdego ekranu/procesu z punktu widzenia użytkownika; tu m.in. usuwanie pomiarów, formularze, statystyki, profil, logowanie, skanowanie zdjęć ciśnieniomierza)
   4. Procesy biznesowe (agent przypomnień, analiza AI — co realizują, kiedy, dla kogo; bez szczegółów implementacyjnych)
   5. Bezpieczeństwo (z punktu widzenia wymagań: co musi być chronione, jakie nagłówki, jakie limity — bez szczegółów konfiguracji modułu)
   6. Stack technologiczny (krótka tabela — szczegóły konwencji w CLAUDE.md)
   7. Endpoint-y REST API → **link** do `./API.md` z jednozdaniowym opisem
   8. Skrypty administracyjne (z punktu widzenia administratora — co robią, jak uruchomić)
   9. Zmienne środowiskowe (lista — wymagane przez aplikację, niezależnie od pliku konsumującego)
   10. Poza zakresem (out-of-scope)

   **`./CLAUDE.md` — kolejność sekcji (techniczna pamięć projektu):**
   1. Specyfikacja → **link** do `./SPEC.md` (jednozdaniowy opis: pełna specyfikacja biznesowa i funkcjonalna)
   2. Dokumentacja API → **link** do `./API.md` (jednozdaniowy opis: kontrakty wszystkich endpoint-ów REST)
   3. Komendy (npm scripts, CLI tooling)
   4. Stack technologiczny — perspektywa techniczna (z konkretnymi pakietami, wersjami)
   5. Struktura katalogów (`app/`, `server/`, `scripts/`, `tests/`) — bez powtarzania opisu z SPEC.md
   6. Wzorce architektoniczne (thin handler, walidacja przez helpery, `appStatus` + `AppToaster`, `apiCall`, store'y Pinia, globalny loader, kontrakt komunikatów błędów)
   7. Konwencje kodu (`if` bez nawiasów dla jednego polecenia, język UI, kody błędów HTTP)
   8. Testy integracyjne (stack, lokalizacja, baza testowa, mocki, reguła aktualizacji testów)
   9. Tryb debug
   10. Zmienne środowiskowe → **link** do sekcji w SPEC.md albo krótka tabela bez powtórzeń

5. **Wprowadź zmiany**

   Po akceptacji planu zaktualizuj oba pliki. Zasady:
   - **Treść opisaną w API.md** — w SPEC.md i CLAUDE.md zostaw wyłącznie nazwę obszaru + link i jednozdaniowy opis. Przykład:

     ```markdown
     ## Endpoint-y REST API

     Pełna dokumentacja kontraktów wszystkich endpoint-ów REST (URL, metoda, walidacja wejścia, kody odpowiedzi, opis biznesowy) znajduje się w [./API.md](./API.md).
     ```

   - **Duplikat między SPEC.md i CLAUDE.md** — zostaw w pliku, do którego treść należy zgodnie z podziałem (biznes → SPEC, technika → CLAUDE), w drugim usuń. Jeśli drugi plik mocno na niej polega, zostaw jednolinijkowy odsyłacz: „Szczegóły: [./SPEC.md#nazwa-sekcji](...)".
   - **Tabele Markdown** — zachowaj, ale tylko tam, gdzie wnoszą wartość. Jeśli tabela powtarza dane z API.md / Prisma schema, zastąp ją linkiem.
   - **Spójność nazewnictwa sekcji** — używaj polskich nagłówków zgodnie ze stylem oryginałów; zachowaj poziomy (## / ###).
   - **Nie usuwaj informacji, które nie są nigdzie indziej** — jeśli coś nie ma odpowiednika w żadnym innym pliku, zostaw to we właściwym dokumencie.
   - **Minimalna ingerencja w semantykę** — porządkujesz strukturę i usuwasz duplikacje; nie zmieniasz wymagań ani konwencji projektu.

6. **Zweryfikuj wynik**

   Po zapisie:
   - Sprawdź długość obu plików (`wc -l SPEC.md CLAUDE.md`) i raportuj redukcję.
   - Zweryfikuj, że każdy link wewnętrzny (`./API.md`, kotwica `#...`) wskazuje na realnie istniejący plik/sekcję.
   - Upewnij się, że żadna kluczowa informacja nie zniknęła całkowicie — przejrzyj listę sekcji oryginałów i potwierdź, że każda jest reprezentowana albo w nowej wersji, albo świadomie usunięta jako duplikat (z odsyłaczem do źródła).

7. **Podsumuj zmiany**

   Poinformuj użytkownika:
   - Ile linii / sekcji zostało usuniętych z SPEC.md i CLAUDE.md
   - Które sekcje przeniesiono między plikami
   - Które bloki zostały zastąpione linkiem do `./API.md` lub innego zewnętrznego pliku
   - Czy wykryto rozbieżności merytoryczne między plikami (np. SPEC.md mówi co innego niż CLAUDE.md o tej samej rzeczy) — w takim wypadku zgłoś użytkownikowi do rozstrzygnięcia, nie wybieraj sam

---

**Ważne zasady:**

- **Nie modyfikuj `./API.md` ani plików kodu** — ten skill porządkuje wyłącznie `./SPEC.md` i `./CLAUDE.md`. Jeśli zauważysz, że `./API.md` jest niespójny ze stanem kodu lub ze SPEC.md, zgłoś to do uruchomienia `/update-api-doc`, ale sam nie poprawiaj.
- **Plan przed edycją** — zawsze pokaż użytkownikowi proponowaną nową strukturę obu plików i poczekaj na zgodę przed zapisem. To dwa kluczowe dokumenty projektu.
- **Brak ubytków merytorycznych** — porządkujesz formę i podział, nie zmieniasz wymagań biznesowych ani konwencji technicznych. Każda decyzja typu „to chyba już nieaktualne, usunę" wymaga jawnego potwierdzenia użytkownika.
- **Język** — wszystko po polsku, zachowując konwencje istniejących plików (tabele Markdown, styl nagłówków, terminologia).
- **Linki względne** — używaj `./API.md`, `./SPEC.md`, `./CLAUDE.md` (z prefiksem `./`), aby działały zarówno w GitHub, jak i w lokalnych przeglądarkach Markdown.
- **Nie twórz nowych plików** — żadnych zmian poza istniejącymi `./SPEC.md` i `./CLAUDE.md`.
