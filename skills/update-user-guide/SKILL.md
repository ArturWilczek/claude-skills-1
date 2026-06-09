---
name: update-user-guide
description: "Tworzy (jeśli nie istnieje) lub aktualizuje plik ./GUIDE.md — instrukcję obsługi aplikacji dla użytkownika końcowego. Opisuje wszystkie funkcjonalności frontendu, dołącza zrzuty ekranów wykonane przez Playwright na bazie testowych danych (testowy użytkownik + seed pomiarów), wymaga uruchomionego serwera deweloperskiego. Argument opcjonalny: dodatkowe uwagi do bieżącego przebiegu aktualizacji instrukcji (np. „skup się na sekcji statystyk", „pomiń analizę AI", „użyj jasnego motywu")."
---

Wygeneruj/odśwież `./GUIDE.md` — kompletną, czytelną dla nietechnicznego użytkownika instrukcję obsługi aplikacji CPW, ze zrzutami ekranów wszystkich istotnych widoków i funkcji.

---

**Argument opcjonalny — dodatkowe uwagi do bieżącego przebiegu:**

Argument wywołania skilla (cały tekst po nazwie polecenia, jeśli podany) traktuj jako **dodatkowe uwagi użytkownika do tego konkretnego uruchomienia**. Mogą obejmować np.:

- wskazanie, na której sekcji się skupić („zaktualizuj tylko sekcję statystyk"),
- pominięcie wybranych zrzutów / funkcjonalności („nie generuj zrzutu analizy AI"),
- zmianę motywu („zrób zrzuty na ciemnym motywie"),
- specjalne stany do udokumentowania („pokaż też stan pusty na liście pomiarów"),
- doraźne preferencje redakcyjne („krótsze opisy", „bardziej formalny ton").

Uwagi mają **wyższy priorytet niż ustawienia domyślne tego skilla**, ale **niższy** niż twarde reguły bezpieczeństwa (zakaz dotykania produkcyjnej bazy, zakaz realnych wywołań OpenRouter/SMTP, zakaz wycieku prawdziwych danych użytkowników). Jeśli uwagi byłyby z nimi sprzeczne — przerwij i poproś o korektę.

Jeżeli argument nie został podany — pracuj z domyślnymi ustawieniami opisanymi poniżej.

---

**Cel:**
Utrzymywać `./GUIDE.md` jako pojedyncze źródło prawdy o tym, **jak korzystać** z aplikacji od strony użytkownika końcowego — wszystkie strony, formularze, dialogi, przepływy biznesowe (rejestracja pomiaru, analiza zdjęcia AI, statystyki, profil, przypomnienia e-mail). Każda funkcjonalność opisana w specyfikacji frontendowej musi być pokryta tekstem i co najmniej jednym zrzutem ekranu pokazującym faktyczny stan UI.

---

**Założenia środowiska:**

- Skill działa **lokalnie**, na maszynie deweloperskiej — nie modyfikuje bazy dev/prod, nie wysyła e-maili, nie wykonuje wywołań OpenRouter.
- Pracuje wyłącznie na **dedykowanej bazie poglądowej** (osobny plik SQLite, ścieżka z `GUIDE_DATABASE_URL`, fallback: `file:./data/cpw-guide.db`) — nigdy nie używaj `DATABASE_URL` ani `TEST_DATABASE_URL`. Zrzutom ekranu nie wolno wyciekać żadnych prawdziwych danych użytkowników.
- Wymaga dostępnych w `node_modules` pakietów `playwright` i `@playwright/test`. Jeśli ich brakuje — zainstaluj jako `devDependencies` przez `npm install -D playwright @playwright/test`, a następnie `npx playwright install chromium`. Skill korzysta tylko z Chromium.
- Język UI aplikacji: polski. Wszystkie napisy w `GUIDE.md` i nazwy plików zrzutów również po polsku (transliterowane do ASCII w nazwach plików, np. `cisnienie-formularz.png`).

---

**Kroki:**

1. **Odczytaj kontekst projektu**
   - `./SPEC.md` — sekcje „Funkcjonalności użytkownika", „Layout wspólny", opisy ekranów i procesów (agent przypomnień, analiza AI zdjęcia, statystyki). To źródło prawdy o tym, **co użytkownik powinien móc zrobić**.
   - `./CLAUDE.md` — żeby ustalić listę stron (`app/pages/**`), strukturę komponentów (`AppNavbar`, `AppLoader`, `AppToaster`, dialogi potwierdzeń), konwencje.
   - `./GUIDE.md` — jeśli istnieje, jako punkt odniesienia (zachowaj sensowną strukturę, jeśli jest dobra; w razie poważnej desynchronizacji z bieżącym UI — przebuduj).

2. **Zinwentaryzuj funkcjonalności do udokumentowania**

   Na podstawie `app/pages/**/*.vue` + opisu w `SPEC.md` wypisz listę widoków/przepływów do pokrycia. Standardowo:
   - Ekran logowania (`/login`)
   - Strona główna / dashboard (`/`)
   - Formularz pomiaru ciśnienia + opcjonalnego pulsu (`/blood-pressure`) wraz z analizą zdjęcia AI
   - Formularz pomiaru pulsu (`/pulse`)
   - Formularz pomiaru wagi (`/weight`)
   - Statystyki / wykresy (`/statistics`) z różnymi zakresami czasu
   - Profil użytkownika (`/profile`) — dane osobowe, włączenie przypomnień e-mail, zmiana hasła
   - Wspólne elementy UI: górny pasek nawigacji (zalogowany/niezalogowany), globalny loader, system powiadomień (toasty), dialog potwierdzenia usunięcia
   - Wylogowanie

   Każdy element listy = osobna sekcja w `GUIDE.md` z **co najmniej jednym zrzutem** ekranu.

3. **Przygotuj środowisko poglądowe (DB + użytkownik testowy + seed)**

   Wykonaj poniższe operacje w jednym skrypcie pomocniczym `scripts/seed-guide.ts` (utwórz, jeśli nie istnieje; w innym wypadku tylko nadpisz / dopisz to, co potrzebne; **nie usuwaj** istniejących skryptów):
   1. Załaduj `dotenv`. Wymuś `process.env.DATABASE_URL = process.env.GUIDE_DATABASE_URL ?? 'file:./data/cpw-guide.db'` **zanim** zaimportujesz Prisma Clienta.
   2. Wykonaj migracje: `execSync('npx prisma migrate deploy', { stdio: 'inherit', env: { ...process.env } })`.
   3. Wytrzyj wszystkie tabele (`reminderLog`, `bloodPressureReading`, `pulseReading`, `weightReading`, `user`) — kolejność z uwzględnieniem FK.
   4. Utwórz testowego użytkownika ze stałymi, łatwo rozpoznawalnymi danymi:
      - `name`: „Anna Kowalska"
      - `email`: `anna.kowalska@example.com`
      - hasło jawne (zwracane przez skrypt, używane przez kod Playwright do logowania): **`Demo!2025#Cpw`** — zahasłowane bcryptem (`bcrypt.hash(password, 12)`)
      - `birthDate`: 1985-06-15
      - `height`: 168
      - `remindersEnabled`: `true`
   5. Wygeneruj zestaw realistycznych pomiarów z ostatnich 12 miesięcy, **deterministyczny** (stały seed PRNG, np. własna funkcja `mulberry32` z literałem), tak by każdy run produkował identyczne wykresy:
      - **Ciśnienie**: 2 pomiary dziennie (poranny ~07:30, wieczorny ~21:00). Skurczowe 110–145, rozkurczowe 70–95, lekkie sezonowe trendy (kilka wyraźnych „skoków" — żeby wykres był ciekawy, nie płaski).
      - **Puls**: zapisywany razem z każdym pomiarem ciśnienia (55–85) + 1–2 dodatkowe samodzielne pomiary tygodniowo z `/pulse` (np. po treningu, 95–125).
      - **Waga**: 1 pomiar tygodniowo (poranek, 64–69 kg, lekki trend spadkowy).
        Razem ok. 700–800 pomiarów ciśnienia/pulsu i ~50 pomiarów wagi — wystarczy na ciekawe statystyki dla wszystkich zakresów (`7d`, `14d`, `1m`, `3m`, `1y`).
   6. Wypisz na stdout: ścieżkę bazy, email i hasło użytkownika testowego.

   Uruchom skrypt: `npx tsx scripts/seed-guide.ts`. Sprawdź exit code.

4. **Uruchom serwer deweloperski na bazie poglądowej**
   - W tle (`run_in_background: true`) uruchom: `cross-env DATABASE_URL=file:./data/cpw-guide.db NUXT_SECURITY_ENABLED=false npm run dev`. Jeśli `cross-env` nie jest dostępny — ustaw zmienne w PowerShellu (`$env:DATABASE_URL='...'; $env:NUXT_SECURITY_ENABLED='false'; npm run dev`) lub w bashu (`DATABASE_URL=... NUXT_SECURITY_ENABLED=false npm run dev`). Wybierz wariant zgodny z `Platform` ze środowiska.
   - Poczekaj aż serwer odpowie na `http://localhost:3000/login` (pollowanie HEAD/GET co 1 s, max 60 s). Nie używaj statycznych `sleep`.
   - **Wymóg**: po zakończeniu skill **MUSI** zatrzymać serwer dev (zabicie procesu w tle), niezależnie czy zrzuty się udały.

5. **Wykonaj zrzuty ekranu przez Playwright**

   Utwórz / zaktualizuj `scripts/capture-guide-screenshots.ts` (nowy plik, dedykowany dla tego skilla — **nie myl go z testami integracyjnymi w `tests/`**). Plik:
   1. Korzysta z `@playwright/test`'s `chromium.launch({ headless: true })`.
   2. Otwiera kontekst w trybie **smartphone, pion** — to jest podstawowe urządzenie docelowe aplikacji (użytkownicy korzystają głównie z telefonów trzymanych pionowo). Domyślnie:
      - `viewport: { width: 390, height: 844 }` — typowa logiczna rozdzielczość współczesnego smartfona (klasa iPhone 13/14/15, Pixel 7/8 itp.),
      - `deviceScaleFactor: 3` — ostre, „retinowe" zrzuty,
      - `isMobile: true`, `hasTouch: true` — żeby aplikacja zachowywała się jak na mobile (media queries, gesty),
      - `userAgent` typowy dla iOS Safari w wersji mobilnej (np. taki, jakiego używa preset `devices['iPhone 13']` z Playwright — można po prostu rozpakować `...devices['iPhone 13']` zamiast podawać pola ręcznie),
      - `locale: 'pl-PL'`, `timezoneId: 'Europe/Warsaw'`, `colorScheme: 'light'`.

      Wszystkie zrzuty mają być **pełnoekranowe w orientacji pionowej** (`page.screenshot({ fullPage: true })` dla widoków przewijalnych, zwykły `screenshot()` dla pojedynczych kadrów typu modal/dialog). **Nie używaj** desktopowych viewportów — instrukcja ma odzwierciedlać to, co użytkownik zobaczy na telefonie.
   3. **Zawsze loguje się** danymi z punktu 3 (email + jawne hasło) — wypełnia formularz i klika „Zaloguj", czeka aż URL przejdzie na `/`. Ten krok jest też okazją na zrzut ekranu logowania (przed kliknięciem) i komunikatu błędu (oddzielne wejście z błędnym hasłem).
   4. Dla każdej strony / przepływu nawiguje przez UI (link w `AppNavbar`, klik w przycisk akcji, otwarcie dialogu) — **nie przez bezpośrednie `goto` URL**, jeśli przejście testuje funkcjonalność (np. otwarcie dialogu usunięcia).
   5. Przed każdym zrzutem czeka na warunek końca animacji/ładowania: `waitForLoadState('networkidle')` + `waitForFunction(() => !document.querySelector('[data-app-loader]') )` (dopisz w razie potrzeby atrybut `data-app-loader` w komponencie `AppLoader.vue` — **NIE rób tego**, gdyby wymagało to zmian w kodzie produkcyjnym; zamiast tego użyj selektora opartego o istniejący CSS / aria-label albo prosty `waitForTimeout(300)` tylko jako fallback).
   6. Zapisuje pliki PNG do `docs/guide/screenshots/`. Stałe, semantyczne nazwy (bez sygnatur czasowych), żeby kolejne uruchomienia **nadpisywały** poprzednie zrzuty (idempotencja, czysty diff w git).
   7. Nazwy plików: ascii-kebab-case po polsku, np. `logowanie.png`, `logowanie-blad.png`, `dashboard.png`, `nawigacja-pasek.png`, `cisnienie-formularz.png`, `cisnienie-formularz-z-pulsem.png`, `cisnienie-analiza-zdjecia-modal.png`, `puls-formularz.png`, `waga-formularz.png`, `statystyki-7d.png`, `statystyki-3m.png`, `statystyki-1y.png`, `profil.png`, `profil-zmiana-hasla.png`, `dialog-usuniecia.png`, `toast-sukces.png`.
   8. Dla zrzutu „analiza zdjęcia AI" **nie wywołuj** rzeczywistego OpenRouter — zrób screenshot otwartego modala/komponentu wczytania zdjęcia (np. po wybraniu pliku z `tests/fixtures/sample-bp.jpg` — jeśli fixture nie istnieje, użyj pierwszego z jakiegoś istniejącego PNG w repo; jeżeli brak — pomiń ten konkretny zrzut i odnotuj to w podsumowaniu, **nie pobieraj zdjęć z Internetu**).
   9. Dla dialogu potwierdzenia usunięcia — najedź na listę pomiarów, kliknij ikonę kosza, zrób zrzut otwartego dialogu, **anuluj** (nie potwierdzaj — nie modyfikuj danych poglądowych).
   10. Dla toastów — zrób krótkie wejście np. „dodanie poprawnego pomiaru" i złap zrzut chwilę po submit, zanim toast zniknie (`page.waitForSelector('[role=status]')`).

   Uruchom: `npx tsx scripts/capture-guide-screenshots.ts`. Jeśli któryś zrzut zawiedzie — zaloguj który i kontynuuj resztę; w podsumowaniu wymień wszystkie nieudane.

6. **Wygeneruj treść `./GUIDE.md`**

   **Wymagana struktura dokumentu** (cały tekst po polsku, prosty język, perspektywa użytkownika końcowego, **bez** żargonu programistycznego, ścieżek do kodu, nazw komponentów Vue itp.):

   ```markdown
   # CPW — Instrukcja obsługi

   <Krótkie wprowadzenie (1–2 akapity): czym jest CPW, dla kogo, co umożliwia.
   Wzmianka, że konta zakłada administrator — użytkownik dostaje login i hasło
   poza aplikacją.>

   ## Spis treści

   <Linki do wszystkich sekcji poniżej.>

   ## Logowanie

   <Opis ekranu, opis pól, opis komunikatu błędu przy złym haśle.>

   <p align="center"><img src="docs/guide/screenshots/logowanie.png" alt="Ekran logowania" width="320"></p>

   <p align="center"><img src="docs/guide/screenshots/logowanie-blad.png" alt="Błąd logowania" width="320"></p>

   ## Pulpit (strona główna)

   <Co użytkownik widzi po zalogowaniu, jakie skróty/karty są dostępne.>

   <p align="center"><img src="docs/guide/screenshots/dashboard.png" alt="Pulpit" width="320"></p>

   ## Nawigacja

   <Opis paska nawigacji: linki, ikona profilu, wylogowanie.>

   <p align="center"><img src="docs/guide/screenshots/nawigacja-pasek.png" alt="Pasek nawigacji" width="320"></p>

   ## Pomiar ciśnienia (i pulsu)

   <Jak otworzyć formularz, jakie pola są wymagane, zakresy, że puls jest
   opcjonalny i trafia do tej samej historii co samodzielne pomiary pulsu,
   jak zapisać, co się dzieje po zapisie (toast, pojawienie się na liście).>

   <p align="center"><img src="docs/guide/screenshots/cisnienie-formularz.png" alt="Formularz ciśnienia" width="320"></p>

   <p align="center"><img src="docs/guide/screenshots/cisnienie-formularz-z-pulsem.png" alt="Formularz ciśnienia z pulsem" width="320"></p>

   ### Analiza zdjęcia ciśnieniomierza (AI)

   <Jak wgrać zdjęcie, ograniczenia rozmiaru, że AI rozpozna wartości
   i wstawi je do formularza, że można je poprawić ręcznie przed zapisem,
   jakie błędy mogą wystąpić (nieczytelne zdjęcie → kod 422).>

   <p align="center"><img src="docs/guide/screenshots/cisnienie-analiza-zdjecia-modal.png" alt="Wczytywanie zdjęcia" width="320"></p>

   ## Pomiar pulsu

   <Samodzielny pomiar pulsu (np. po wysiłku) — kiedy używać tego formularza
   zamiast formularza ciśnienia.>

   <p align="center"><img src="docs/guide/screenshots/puls-formularz.png" alt="Formularz pulsu" width="320"></p>

   ## Pomiar wagi

   <Formularz wagi, zakres dozwolonych wartości.>

   <p align="center"><img src="docs/guide/screenshots/waga-formularz.png" alt="Formularz wagi" width="320"></p>

   ## Statystyki

   <Opis zakresów czasu (`7d`, `14d`, `1m`, `3m`, `1y`), wykresów ciśnienia,
   pulsu, wagi, średnich i trendów.>

   <p align="center"><img src="docs/guide/screenshots/statystyki-7d.png" alt="Statystyki — ostatnie 7 dni" width="320"></p>

   <p align="center"><img src="docs/guide/screenshots/statystyki-3m.png" alt="Statystyki — ostatnie 3 miesiące" width="320"></p>

   <p align="center"><img src="docs/guide/screenshots/statystyki-1y.png" alt="Statystyki — ostatni rok" width="320"></p>

   ## Profil

   <Edycja danych osobowych: imię, data urodzenia, wzrost; włączenie
   przypomnień e-mail i co to oznacza (krótki opis godzin wysyłki — godziny
   z SPEC.md, bez wchodzenia w implementację); zmiana hasła.>

   <p align="center"><img src="docs/guide/screenshots/profil.png" alt="Profil" width="320"></p>

   <p align="center"><img src="docs/guide/screenshots/profil-zmiana-hasla.png" alt="Zmiana hasła" width="320"></p>

   ## Usuwanie pomiarów

   <Jak usunąć pojedynczy pomiar z listy, że pojawia się dialog potwierdzenia,
   że operacja jest nieodwracalna.>

   <p align="center"><img src="docs/guide/screenshots/dialog-usuniecia.png" alt="Dialog potwierdzenia" width="320"></p>

   ## Komunikaty systemowe

   <Toasty: sukces / ostrzeżenie / błąd; globalny loader podczas ładowania.>

   <p align="center"><img src="docs/guide/screenshots/toast-sukces.png" alt="Toast sukcesu" width="320"></p>

   ## Wylogowanie

   <Gdzie znaleźć przycisk wylogowania, co się dzieje po kliknięciu (powrót na ekran logowania, wyczyszczenie podręcznych danych).>

   ## Najczęstsze pytania (FAQ)

   <2–5 krótkich Q&A: „Nie pamiętam hasła", „Nie dostaję e-maili z przypomnieniami",
   „Skąd biorą się wartości po wgraniu zdjęcia ciśnieniomierza", „Czy mogę pobrać
   moje dane".>
   ```

   **Zasady redakcyjne:**
   - Tekst zwarty, ale kompletny. Każdy ekran/przepływ ma sekcję; każda sekcja ma akapit opisu **i** zrzut(y) ekranu.
   - Zrzuty osadzaj **WYŁĄCZNIE** w formacie HTML `<p align="center"><img src="docs/guide/screenshots/<plik>.png" alt="<opis>" width="320"></p>` (relatywna ścieżka). **NIE używaj** składni markdown `![alt](path)` — zrzuty są w orientacji pionowej (smartfon) i bez kontroli szerokości skalują się do pełnej szerokości dokumentu. Owijanie w `<p align="center">` daje wycentrowanie w poziomie (GitHub świadomie obsługuje atrybut `align` mimo formalnej deprekacji, bo `style=` jest stripowane). Domyślna szerokość `width="320"` jest dobrana pod zrzuty z viewportu 390 px (smartfon w pionie); zmieniaj tylko jeśli użytkownik wskaże inną wartość.
   - Język: prosty, konwersacyjny, polski. **Bez** nazw plików `.vue`, ścieżek API, nazw store'ów, nazw bibliotek.
   - Wartości graniczne (np. zakresy ciśnienia, wagi, pulsu) cytuj z `SPEC.md` jako naturalne zdanie („Skurczowe od 60 do 250 mmHg"), bez tabel typów Zod.
   - Jeśli widok ma stany puste — opisz je, choć zrzutem ekranu pokaż wypełniony (dzięki seedowi z punktu 3).

7. **Zapisz / zaktualizuj `./GUIDE.md`**
   - Jeśli plik nie istnieje — utwórz.
   - Jeśli istnieje — zaktualizuj zachowując strukturę z punktu 6. Sekcje, których funkcjonalność zniknęła z UI (np. usunięty ekran), **usuń** — `GUIDE.md` ma odzwierciedlać bieżący stan aplikacji, nie historię.

8. **Sprzątanie**
   - Zatrzymaj serwer deweloperski (jeśli uruchomiony w tle).
   - Plik bazy poglądowej `./data/cpw-guide.db` **pozostawiaj** na dysku (kolejne odpalenia będą szybsze; seed i tak nadpisze zawartość). Dopisz `data/cpw-guide.db*` do `.gitignore`, jeśli jeszcze go tam nie ma.
   - Pliki PNG zrzutów `docs/guide/screenshots/*.png` **commituj** razem z `GUIDE.md` — są częścią dokumentacji.

9. **Podsumuj zmiany**

   Poinformuj użytkownika:
   - Czy `./GUIDE.md` został utworzony, czy zaktualizowany.
   - Ile sekcji i ile zrzutów ekranu zawiera dokument.
   - Które zrzuty się nie udały (jeśli jakiekolwiek) — z krótkim powodem i propozycją następnego kroku.
   - Czy wykryto rozbieżność między `SPEC.md` a faktycznym UI (np. spec mówi o przycisku „Eksport CSV", a w UI go nie ma — albo odwrotnie). Zgłoś użytkownikowi do decyzji, **nie modyfikuj** `SPEC.md` samodzielnie.

---

**Ważne zasady:**

- **Źródłem prawdy o tym, co dokumentować, jest faktyczne UI** (zrzuty z działającego dev servera + treść stron w `app/pages/**`), a nie `SPEC.md`. Jeśli SPEC mówi co innego niż UI — dokumentuj UI, rozbieżność zgłoś użytkownikowi.
- **Nie modyfikuj** kodu produkcyjnego (`app/`, `server/`) ani głównej bazy danych. Wyłącznie: pliki w `scripts/` (`seed-guide.ts`, `capture-guide-screenshots.ts`), `docs/guide/screenshots/*.png`, `./GUIDE.md`, `.gitignore`. Inne pliki dotykaj tylko po jawnym potwierdzeniu użytkownika.
- **Bezpieczeństwo danych**: zrzuty mają zawierać wyłącznie dane testowe użytkownika „Anna Kowalska". Nigdy nie podpinaj się pod prawdziwego użytkownika / prawdziwą bazę. Jeśli `GUIDE_DATABASE_URL` przypadkiem wskazuje na bazę produkcyjną — **przerwij** i poproś o korektę.
- **AI / SMTP**: nie wywołuj zewnętrznych usług. Analiza zdjęcia AI dokumentowana zrzutem stanu „przed wysłaniem" / placeholder; przypomnienia e-mail dokumentowane słownie, bez realnej wysyłki.
- **Idempotencja**: kolejne uruchomienie skilla na niezmienionym kodzie ma dać identyczny `GUIDE.md` i identyczne pliki PNG (deterministyczny seed + headless Chromium + stały viewport).
- **Polski w UI i dokumentach** — bez wyjątków. Nazwy plików zrzutów zapisuj w ASCII (bez polskich znaków diakrytycznych), żeby uniknąć problemów z systemami plików / linkami.
- **Język techniczny zostaje w `SPEC.md` / `CLAUDE.md` / `API.md`** — `GUIDE.md` jest dokumentem dla użytkownika, nie programisty.
