---
name: update-tasks-list
description: Przegląda specyfikację w SPEC.md i CLAUDE.md, a następnie aktualizuje TASKS.md o nowe zadania wynikające ze specyfikacji oraz tworzy pliki planów realizacji w _TASKS/.
---

Aktualizuj listę zadań projektu na podstawie specyfikacji.

---

**Cel:**
Przejrzyj specyfikację projektu i zaktualizuj `./TASKS.md` o zadania, które jeszcze nie zostały uwzględnione. Każde zadanie musi mieć plik planu realizacji w `./_TASKS/`.

**Kroki:**

1. **Odczytaj specyfikację i obecny stan zadań**

   Odczytaj kolejno:
   - `./SPEC.md` — pełna specyfikacja projektu
   - `./CLAUDE.md` — wskazówki architektoniczne i konwencje
   - `./TASKS.md` — obecna lista zadań (jeśli istnieje)
   - Pliki `./_TASKS/*.md` — istniejące plany zadań (jeśli istnieją)

   Jeśli `TASKS.md` nie istnieje — to pierwsze uruchomienie; utwórz go i folder `./_TASKS/` z wszystkimi zadaniami.

2. **Przeanalizuj specyfikację pod kątem zadań**

   Na podstawie specyfikacji wyodrębnij wszystkie funkcjonalności wymagające implementacji. Grupuj logicznie — jedno zadanie może obejmować wiele warstw (baza danych + API + frontend), jeśli realizuje spójną funkcjonalność.

   **Obowiązkowa kolejność zadań** (od podstaw do UI):
   1. Infrastruktura i konfiguracja projektu (Prisma, zmienne środowiskowe, konfiguracja Nuxt)
   2. Model danych i migracje bazy danych
   3. Skrypty administracyjne (CLI)
   4. Warstwa API (server routes + services)
   5. Middleware i autoryzacja
   6. Frontend — strony i komponenty

   **Zasady granularności:**
   - NIE rozbijaj na bardzo małe zadania (np. osobno tabela, osobno model — to jedno zadanie)
   - TAK grupuj wszystko co realizuje jedną funkcjonalność: np. "Pomiary ciśnienia" = tabela + serwis + endpoint + strona
   - Wyjątek: infrastruktura i konfiguracja mogą być osobnym zadaniem inicjalizacyjnym

3. **Porównaj z istniejącymi zadaniami**

   Sprawdź które funkcjonalności ze specyfikacji:
   - Są już w `TASKS.md` (pomiń je)
   - Nie są jeszcze uwzględnione (dodaj jako nowe zadania)

   Przy porównaniu kieruj się tytułem i treścią planów w `./_TASKS/*.md`.

4. **Przypisz numery nowym zadaniom**
   - Format: czterocyfrowy numer, np. `0001`, `0002`, `0042`
   - Nowe zadania numeruj kontynuując od najwyższego istniejącego numeru
   - Zachowaj ciągłość numeracji

5. **Utwórz pliki planów dla nowych zadań**

   Dla każdego nowego zadania utwórz plik `./_TASKS/NNNN.md` według schematu:

   ```markdown
   # NNNN — Tytuł zadania

   ## Cel

   Krótki opis co realizuje to zadanie i dlaczego jest potrzebne.

   ## Zakres

   ### Baza danych / Model danych

   (jeśli dotyczy)

   - Lista tabel / modeli do utworzenia lub zmodyfikowania
   - Schemat Prisma

   ### Backend — serwisy

   (jeśli dotyczy)

   - Pliki do utworzenia: `server/services/...`
   - Logika biznesowa do zaimplementowania

   ### Backend — API

   (jeśli dotyczy)

   - Endpointy do utworzenia z metodami HTTP i ścieżkami
   - Walidacja Zod

   ### Frontend

   (jeśli dotyczy)

   - Strony: `app/pages/...`
   - Komponenty: `app/components/...`
   - Store Pinia (jeśli potrzebny)

   ## Uwagi implementacyjne

   Dodatkowe wskazówki, zależności od innych zadań, ważne decyzje architektoniczne.
   ```

   Pomijaj sekcje, które nie dotyczą danego zadania.

6. **Zaktualizuj TASKS.md**

   Format pliku `TASKS.md`:

   ```markdown
   # Zadania projektu CPW

   | Nr                       | Status       | Tytuł                                 |
   | ------------------------ | ------------ | ------------------------------------- |
   | [0001](./_TASKS/0001.md) | Zrealizowane | Inicjalizacja projektu i konfiguracja |
   | [0002](./_TASKS/0002.md) | Planowane    | Model danych i migracje               |
   ```

   - Dodaj tylko nowe zadania — nie modyfikuj istniejących wierszy
   - Nowe zadania mają status `Planowane`
   - Zachowaj kolejność numeryczną w tabeli

7. **Podsumuj wynik**

   Po zakończeniu poinformuj użytkownika:
   - Ile zadań istniało przed aktualizacją
   - Ile nowych zadań dodano
   - Lista nowo dodanych zadań z numerami i tytułami

**Ważne zasady:**

- Wszystkie teksty w plikach zadań pisz po polsku
- Tytuły zadań muszą być zwięzłe (maks. ~60 znaków)
- Status nowych zadań to zawsze `Planowane`
- Nie zmieniaj statusu ani treści istniejących zadań
- Jeśli specyfikacja nie wprowadza nic nowego — poinformuj o tym i zakończ bez zmian
