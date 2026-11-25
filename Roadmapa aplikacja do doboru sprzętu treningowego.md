# Roadmapa: aplikacja do doboru sprzętu treningowego

Nov 3, 2026 · @Olaf

Roadmapa prowadzi od obecnego kodu do gotowego programu w dwóch fazach: najpierw kod (etapy 0–11), potem zamrożenie kodu, dane i wyniki do pracy (etapy 12–14).


## Faza I. Kod, etapy 0–5

### Etap 1. Porządki

1. `src/config.py` z `AREAS` (7 obszarów) i progiem niskiego budżetu. Importuj je w selectorze, coverage i loaderze.
2. `.gitignore`, `src/__init__.py`, usunięcie plików `.pyc`.

*Dlaczego:* lista obszarów jest teraz w trzech miejscach i łatwo o rozjazd. Skompilowane pliki nie należą do repozytorium, a pakiet ma się importować z testów.

**Bez zmian:** `compute_coverage_flag_count` liczy 6 partii, bez cardio. `coverage` w SAW mierzy wszechstronność mięśniową, a cardio dostaje punkty przez `goal_fit`. Problem gum przed bieżnią dla kondycji rozwiązujesz wagami i danymi, nie kodem.

### Etap 2. Wczytywanie danych

1. Walidacja `exercise_count` (liczba całkowita ≥ 1) i unikalnych `id`.
2. Czytelny błąd dla pustego CSV.

*Dlaczego:* przy 15–20 pozycjach literówki są bardziej prawdopodobne, a pusty plik daje teraz nieczytelny błąd z `min()`.

### Etap 3. Wybór zestawu (najważniejsza zmiana)

1. Selekcja dwufazowa. Faza 1: po rankingu bierzesz sprzęt mieszczący się w budżecie i dodający nową partię. Faza 2: drugi przebieg po rankingu, dodajesz niewybrane pozycje mieszczące się w reszcie budżetu.
2. Każda pozycja ma powód i numer fazy. Zaktualizuj `coverage_summary` i raport do nowego formatu zwracanych danych.
3. Docstring: to heurystyka zachłanna i nie gwarantuje optimum.

*Dlaczego:* teraz zestaw przy 8000 zł kosztuje 3300 zł i przestaje rosnąć, więc scenariusz B i analiza wrażliwości na budżet nie działają. Wytyczne (pkt 31) chcą uzasadnienia wyboru, a numer fazy przyda się w eksperymentach. Praca nie może twierdzić, że heurystyka daje optimum (pkt 29).

### Etap 4. Przypadki brzegowe (4.6 spisu)

1. Walidacja budżetu (liczba > 0).
2. Komunikat, gdy budżet nie wystarcza na żaden sprzęt, z ceną najtańszej pozycji.
3. Ostrzeżenie przy niepełnym pokryciu, z listą brakujących obszarów.

*Dlaczego:* to scenariusze F i G. Program nie może kończyć się błędem ani wypisywać "0%" bez wyjaśnienia.

### Etap 5. Metoda bazowa (6.6 spisu)

1. `baseline_select`: idzie po rankingu SAW i bierze wszystko, co się mieści, bez reguły pokrycia.

*Dlaczego:* bez niej nie ma z czym porównać algorytmu.

## Faza I. Kod, etapy 6–11

### Etap 6. Raport (5.6 spisu)

1. Elementy z cenami i wynikami SAW, powód i faza, koszt, reszta budżetu, kompletność, brakujące obszary, ostrzeżenia. Ceny bez ".0".

*Dlaczego:* to lista z pkt 31 wytycznych i gotowy przykład do 5.6.

### Etap 7. Interfejs (3.6 i 5.5 spisu)

1. `argparse` (`--cel`, `--budzet`) lub `input()` zamiast wartości wpisanych w `main.py`.

*Dlaczego:* wymóg to "użytkownik wybiera cel i budżet". Konsola wystarcza, jeśli zgoda.

### Etap 8. Eksperymenty (6.4–6.6 spisu)

1. `experiments.py`: pętla po 3 celach i budżetach (np. 200, 500, 1000, 2000, 5000). Zapis do CSV: zestaw, koszt, kompletność, osobno elementy z fazy 1 i fazy 2.
2. Porównanie z metodą bazową dla tych samych par.
3. Zmiana wag: dla każdego z 4 kryteriów ±20% względem wartości bazowej, reszta skalowana proporcjonalnie tak, żeby suma = 1. To 8 wariantów na cel. Dla każdego policz, ile pozycji zestawu się zmieniło. Funkcja `perturb_weights`.
4. *(Opcjonalnie)* pętla po progu uniwersalności (6, 7, 8), jeśli zostanie reguła R2.

Wzór na przeskalowanie wag, gdzie k to zmieniane kryterium, a j to pozostałe:

```latex
w'_k = w_k (1 + \delta), \qquad w'_j = w_j \frac{1 - w'_k}{1 - w_k}
```

*Dlaczego:* to wymaganie o wpływie celu, budżetu i wag. Bez określonego mechanizmu zmiany wag wyniki byłyby nieporównywalne.

### Etap 9. System ekspercki (do ustalenia)

1. **9A, niewymagany:** reguła "nowa partia" zostaje w selektorze. W 4.5 opisujesz ją jako wiedzę dziedzinową zakodowaną w algorytmie, **nie jako system ekspercki**.
2. **9B, wymagany:** dodaj `expert_rules.py` z 2–3 regułami. Każda zwraca decyzję i uzasadnienie, raport je wypisuje:
   - R0: reguła "nowa partia", nazwana i opisana jako reguła.
   - R2: JEŻELI budżet < próg, TO w fazie 1 tylko sprzęt o uniwersalności ≥ 7 (próg w `config.py`, uzasadnienie jednym zdaniem).
   - R3: JEŻELI po obu fazach brakuje obszaru, TO podaj najtańszy sprzęt, który go pokrywa, i ile zł brakuje.
3. W wariancie 9B dodaj podrozdziały do spisu treści (4 i 5).

*Dlaczego:* reguły wpływają na wybór zestawu, a nie na wynik SAW, więc nie premiujesz dwa razy tych samych cech.

### Etap 10. Testy (6.2 i 6.3 spisu)

1. Jednostkowe: `normalize` (w tym wartości stałe), `rank`, `select` (fazy 1 i 2), `coverage_summary`, `load_equipment` (zła cena, flaga 2, brak kolumny, puste CSV, duplikat `id`).
2. SAW policzony ręcznie na 3 pozycjach.
3. Scenariusze A, C, E, F, G na małej bazie zapisanej w teście. B i D zostają w `experiments.py`.
4. Koszt zestawu ≤ budżet dla każdego celu i budżetów 0–10000 co 100.
5. Wagi po `perturb_weights` sumują się do 1 i są nieujemne.

*Dlaczego:* własna mała baza oddziela ocenę logiki od jakości CSV. Scenariusz C jest dziś niespełniony, więc test go wykryje.

### Etap 11. Domknięcie kodu

1. `README.md`: uruchomienie programu, testów i eksperymentów.
2. Komentarze przy decyzjach (dlaczego min-max, dlaczego dwie fazy).

*Dlaczego:* recenzent ma móc odtworzyć wyniki bez Twojej pomocy, a komentarze skrócą pisanie rozdziałów 4 i 5.

## Faza II. Zamrożenie kodu i dane, etapy 12–14

### Etap 12. Pierwsze uruchomienie eksperymentów (na obecnym CSV)

1. Uruchom `experiments.py` i testy. Zapisz osobno, co jest błędem kodu, a co problemem danych lub wag.
2. Błędy kodu popraw, po poprawce uruchom testy i eksperymenty od nowa.

*Dlaczego:* bez tego rozróżnienia po drugim uruchomieniu nie wiesz, czy głupie wyniki to wina kodu, czy danych. Notatki przydadzą się w 6.7.

**Kod zamrożony, gdy:**

- etapy 1–8, 10, 11 są zrobione (9 według decyzji promotora),
- wszystkie testy przechodzą,
- eksperymenty działają od zera jednym poleceniem,
- README jest,
- pierwsze uruchomienie nie wykazało już błędów w kodzie.

### Etap 13. Dane i wagi

1. CSV do 15–20 pozycji, z jawnymi zasadami ocen 1–10 i flag partii (3.4, 4.1). Sprawdź m.in. gumy (6 jedynek w partiach) i bieżnię (`core=1`).
2. Wagi dla trzech celów z uzasadnieniem w pracy (4.2). Rozwiąż problem gum przed bieżnią dla kondycji.
3. Uzasadnienie źródła i daty cen (3.3).

*Dlaczego:* to jakość danych decyduje o sensowności wyników. Kod jest już gotowy.

### Etap 14. Drugie uruchomienie (wyniki do pracy)

1. Uruchom testy i `experiments.py` na docelowym CSV i wagach. Kodu nie zmieniasz, żeby wyniki lepiej wyglądały.
2. Jeśli wynik wygląda źle, najpierw sprawdź dane i wagi. Kod zmieniasz tylko przy udokumentowanym błędzie.
3. Zapisz tabele i wykresy do rozdziału 6.

**Dane gotowe, gdy:**

- CSV ma 15–20 pozycji z opisanymi zasadami,
- wagi mają uzasadnienie w pracy,
- analiza wrażliwości jest wykonana,
- drugie uruchomienie daje sensowne wyniki.

## Co dalej

Po etapach 0–14 zostaje praca pisemna: rozdziały 1–3 i 7, literatura, tabele i rysunki. Rozdziały 4–6 opierają się na kodzie i wynikach z etapów 12–14.
