---
name: commit-staged
description: Tworzy commit (z opisem po polsku) dla plików znajdujących się w statusie staged, a następnie — po jawnym potwierdzeniu użytkownika — wykonuje git push.
---

Wykonaj commit plików w statusie _staged_, a następnie zapytaj użytkownika, czy wypchnąć zmiany do zdalnego repozytorium.

---

**Cel:**
Na podstawie zawartości plików aktualnie znajdujących się w obszarze _staged_ przygotuj poprawny merytorycznie i stylistycznie komunikat commita w języku polskim, utwórz commit, a następnie zawsze zapytaj użytkownika, czy wykonać `git push` — push wykonuj WYŁĄCZNIE po jego potwierdzeniu.

---

**Kroki:**

1. **Sprawdź stan repozytorium**

   Wykonaj równolegle (jeden message, kilka wywołań Bash):
   - `git status` — aby ustalić, które pliki są w obszarze _staged_, a które tylko zmodyfikowane / nieśledzone
   - `git diff --staged` — pełna treść zmian, które wejdą do commita
   - `git log -n 10 --pretty=format:"%h %s"` — ostatnie commity, aby dopasować styl komunikatów obowiązujący w projekcie

2. **Zwaliduj, że jest co commitować**
   - Jeśli `git diff --staged` jest pusty (brak plików w _staged_) — **przerwij** i poinformuj użytkownika, że nie ma żadnych zmian do commita. Nie wykonuj `git add`, nie próbuj nic zgadywać.
   - Jeśli wśród plików _staged_ widoczne są pliki, które typowo nie powinny trafić do repozytorium (`.env`, klucze, prywatne tokeny, duże binaria) — zatrzymaj się i ostrzeż użytkownika przed wykonaniem commita.

3. **Przygotuj treść commita (po polsku)**
   - Komunikat ma być **w języku polskim**, z poprawnymi znakami diakrytycznymi (ą, ć, ę, ł, ń, ó, ś, ź, ż).
   - Pierwsza linia (subject) — zwięzła (≤ 72 znaki), w trybie oznajmującym, opisująca **co** zmiana wnosi (np. „Dodaje wyliczanie BMI w sekcji Waga", „Poprawia obsługę błędów 429 w apiCall", „Aktualizuje SPEC.md o pole height w profilu"). Bez kropki na końcu.
   - Jeśli zmiany są nietrywialne lub obejmują kilka obszarów — dodaj pustą linię i krótki opis (kilka punktów lub 1–3 zdania) wyjaśniający **dlaczego** i **co** się zmieniło logicznie. Jeśli zmiana jest jednoliniowa i oczywista — sam subject wystarczy.
   - Dopasuj styl (wielkość liter, użycie czasownika, format) do ostatnich commitów z `git log` — projekt CPW używa polskich, krótkich opisów typu „Zadanie 21 - Statystyki — wskaźnik BMI w sekcji Waga".
   - **Nie wymyślaj** zmian, których nie ma w diffie. Komunikat opisuje wyłącznie to, co faktycznie wchodzi do commita.

4. **Utwórz commit**

   Wykonaj `git commit` z przygotowanym komunikatem przekazanym przez HEREDOC, aby zachować formatowanie i znaki diakrytyczne:

   ```bash
   git commit -m "$(cat <<'EOF'
   <treść komunikatu>
   EOF
   )"
   ```

   - **Nie używaj** `git add .` ani `git add -A` — commit obejmuje wyłącznie to, co już jest w _staged_.
   - **Nie używaj** `--amend` — twórz nowy commit.
   - **Nie używaj** `--no-verify` ani innych flag pomijających hooki — jeśli pre-commit hook się wywróci, napraw przyczynę i utwórz **nowy** commit (nie amenduj).
   - **Nie dodawaj** stopki typu „Co-Authored-By: Claude" — komunikat ma zawierać wyłącznie treść opisaną w punkcie 3.

5. **Zapytaj o `git push` (ZAWSZE)**

   Po pomyślnym utworzeniu commita **NIE wykonuj** `git push` automatycznie. **Zawsze** zadaj użytkownikowi pytanie potwierdzające — użyj do tego narzędzia `AskUserQuestion` z dwoma opcjami:
   - **Tak, wykonaj `git push`** — domyślna, rekomendowana
   - **Nie, zostaw commit lokalnie**

   Pytanie zadawaj nawet wtedy, gdy w danej sesji push został już wcześniej zaakceptowany — odpowiedź nie przenosi się między wywołaniami skilla.

6. **Wypchnij commit (`git push`) — tylko po potwierdzeniu**

   - Jeśli użytkownik wybrał **Tak** — wykonaj `git push`.
   - Jeśli bieżąca gałąź nie ma jeszcze _upstream_ (komunikat „has no upstream branch"), wykonaj `git push -u origin <bieżąca-gałąź>`.
   - **Nigdy** nie używaj `--force` / `--force-with-lease`, chyba że użytkownik wprost o to poprosi. Jeśli `git push` zostanie odrzucony (np. z powodu zmian na zdalnym), **zatrzymaj się** i poinformuj użytkownika — nie próbuj samodzielnie rebase'ować, mergować ani forsować.
   - Jeśli użytkownik wybrał **Nie** — pomiń push i zakończ na podsumowaniu (commit pozostaje lokalnie).

7. **Podsumuj**

   Krótka informacja zwrotna do użytkownika:
   - Hash i pierwsza linia utworzonego commita
   - Liczba zmienionych plików
   - Status `git push`: sukces / komunikat błędu / „pominięto na życzenie użytkownika"

---

**Ważne zasady:**

- Komunikat commita zawsze po polsku, z pełnymi znakami diakrytycznymi.
- Commit obejmuje wyłącznie pliki będące już w obszarze _staged_ — nie dodawaj nic samodzielnie.
- Nie pomijaj hooków, nie amenduj, nie forsuj push — w razie problemu zatrzymaj się i zapytaj użytkownika.
- Nie dodawaj stopek atrybucji (Co-Authored-By itp.) do komunikatu commita.
