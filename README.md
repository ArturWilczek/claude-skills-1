# Claude Code Skills — AI-Driven Development Workflow

Zestaw SKILL-i [Claude Code](https://docs.anthropic.com/claude/docs/claude-code), które posłużyły do zbudowania i utrzymania realnej aplikacji w ramach **AI-Driven Development Workflow** — powtarzalnego procesu, w którym agenci AI pełnią rolę wyspecjalizowanych członków zespołu (planista, wykonawca, technical writer, sekretarz), a człowiek skupia się na architekturze, wymaganiach i weryfikacji.

Pełny opis procesu, kontekst powstania każdego SKILL-a oraz wnioski z pracy znajdziesz w artykule:
👉 **[AI-Driven Development Workflow](https://awfs.dev/blog/ai-driven-development-workflow)**

> SKILL-e powstały przy okazji aplikacji **CPW** (rejestrowanie pomiarów ciśnienia, pulsu i wagi), ale są na tyle ogólne, że nadają się do większości projektów prowadzonych z Claude Code. Część z nich zakłada konwencję plików `SPEC.md` / `CLAUDE.md` / `TASKS.md` / `API.md` / `GUIDE.md` opisaną w artykule — dostosuj je do własnego projektu, jeśli używasz innych nazw.

---

## Zawartość

| SKILL                                               | Rola                          | Opis                                                                                                                                                                                                    |
| --------------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`update-tasks-list`](./update-tasks-list/SKILL.md) | Planista                      | Czyta specyfikację (`SPEC.md`, `CLAUDE.md`) i dopisuje do `TASKS.md` zadania ułożone logicznie — od bazy danych, przez REST API, po frontend. Dla każdego zadania tworzy plik planu w `_TASKS/NNNN.md`. |
| [`do-task`](./do-task/SKILL.md)                     | Wykonawca                     | Realizuje zadanie o podanym numerze zgodnie z planem w `_TASKS/NNNN.md`, a po zakończeniu aktualizuje jego status w `TASKS.md`.                                                                         |
| [`update-task`](./update-task/SKILL.md)             | Korygujący                    | Aktualizuje plik zadania `NNNN.md` o nowe wymagania (nowa funkcja, rozbudowa, poprawka) i od razu wprowadza zmianę w kodzie.                                                                            |
| [`update-spec`](./update-spec/SKILL.md)             | Analityk biznesowy            | Wbudowuje nową funkcjonalność / zmianę koncepcyjną w `SPEC.md` (i w razie potrzeby `CLAUDE.md`), dbając o spójność dokumentów.                                                                          |
| [`cleanup-spec`](./cleanup-spec/SKILL.md)           | Porządkowy                    | Usuwa duplikacje między `SPEC.md`, `CLAUDE.md` i plikami zewnętrznymi (np. `API.md`), porządkuje strukturę i odchudza kontekst.                                                                         |
| [`update-api-doc`](./update-api-doc/SKILL.md)       | Technical writer (API)        | Tworzy lub aktualizuje `API.md` — kompletną dokumentację endpoint-ów REST API (URL, metoda, dane wejściowe, autoryzacja, opis biznesowy).                                                               |
| [`update-user-guide`](./update-user-guide/SKILL.md) | Technical writer (użytkownik) | Tworzy lub aktualizuje `GUIDE.md` — instrukcję dla użytkownika końcowego ze zrzutami ekranów wykonanymi przez Playwright na danych testowych.                                                           |
| [`commit-staged`](./commit-staged/SKILL.md)         | Sekretarz                     | Przygotowuje commit message (po polsku) dla plików w statusie _staged_, wykonuje commit, a na końcu **zawsze** pyta, czy zrobić `git push`.                                                             |

---

## Pętla rozwojowa

```
Główna:
  pomysł → /update-spec → /update-tasks-list → /do-task <nr> → (/update-task <nr> <opis>)* → /commit-staged

Pomocnicza:
  ↘ /update-api-doc, /update-user-guide   (regularnie)
  ↘ /cleanup-spec                          (okresowo)
```

---

## Jak dodać SKILL-e do swojego projektu

SKILL-e Claude Code to katalogi z plikiem `SKILL.md`, umieszczane w `.claude/skills/<nazwa-skilla>/`. Wywołuje się je z konsoli Claude Code poleceniem `/<nazwa-skilla>`.

### Wariant A — cały zestaw (kopiowanie)

W katalogu swojego projektu:

```bash
# 1. Sklonuj to repozytorium gdzieś obok
git clone https://github.com/ArturWilczek/claude-skills-1.git /tmp/claude-skills

# 2. Utwórz katalog na skille (jeśli nie istnieje) i skopiuj wybrane / wszystkie
mkdir -p .claude/skills
cp -r /tmp/claude-skills/update-tasks-list \
      /tmp/claude-skills/do-task \
      /tmp/claude-skills/update-task \
      /tmp/claude-skills/update-spec \
      /tmp/claude-skills/cleanup-spec \
      /tmp/claude-skills/update-api-doc \
      /tmp/claude-skills/update-user-guide \
      /tmp/claude-skills/commit-staged \
      .claude/skills/
```

Docelowa struktura:

```
twój-projekt/
└── .claude/
    └── skills/
        ├── update-tasks-list/SKILL.md
        ├── do-task/SKILL.md
        ├── update-task/SKILL.md
        ├── update-spec/SKILL.md
        ├── cleanup-spec/SKILL.md
        ├── update-api-doc/SKILL.md
        ├── update-user-guide/SKILL.md
        └── commit-staged/SKILL.md
```

### Wariant B — pojedynczy SKILL

Skopiuj tylko interesujący Cię katalog, np.:

```bash
mkdir -p .claude/skills
cp -r /tmp/claude-skills/commit-staged .claude/skills/
```

### Wariant C — globalnie (dla wszystkich projektów)

Aby SKILL-e były dostępne we wszystkich projektach, skopiuj je do katalogu domowego zamiast lokalnego katalogu projektu:

```bash
mkdir -p ~/.claude/skills
cp -r /tmp/claude-skills/* ~/.claude/skills/
```

### Aktywacja

1. Uruchom (lub zrestartuj) Claude Code w katalogu projektu.
2. Wpisz `/`, aby zobaczyć listę dostępnych poleceń — Twoje SKILL-e powinny się na niej pojawić.
3. Wywołaj wybrany SKILL, np.:

   ```
   /update-tasks-list
   /do-task 1
   /update-task 4 Przycisk submit ma być nieaktywny gdy formularz niepoprawny
   /commit-staged
   ```

---

## Konwencje plików projektu

Część SKILL-i zakłada następujący podział dokumentacji (możesz go dostosować, edytując pliki `SKILL.md`):

- **`SPEC.md`** — warstwa biznesowa i funkcjonalna: **co** robi aplikacja.
- **`CLAUDE.md`** — warstwa techniczna i pamięć projektu: **jak** agent ma pracować z kodem (generowany przez wbudowane `/init`).
- **`TASKS.md`** + **`_TASKS/NNNN.md`** — lista zadań i szczegółowe plany realizacji.
- **`API.md`** — pełna dokumentacja endpoint-ów REST API.
- **`GUIDE.md`** — instrukcja dla użytkownika końcowego.

---

## Licencja

Projekt jest dostępny na licencji **[MIT](./LICENSE)** — możesz go używać, kopiować, modyfikować i redystrybuować, także komercyjnie, zachowując notę o autorstwie. Jeśli SKILL-e okażą się pomocne — daj znać lub podlinkuj [artykuł](https://awfs.dev/blog/ai-driven-development-workflow). 🙌
