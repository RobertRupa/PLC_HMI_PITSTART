# HMI — WSStudio — V1.4.5

## Ekran Home

### Przyciski

| Funkcja | zapis HMI | wskaźnik aktywnego stanu |
|---|---:|---:|
| STOP | M400 momentary | M412 / WORK_ACTIVE |
| P1 | M401 momentary | M420 |
| P2 | M402 momentary | M421 |
| P3 | M403 momentary | M422 |
| P4 | M404 momentary | M423 |
| P5 | M405 momentary | M424 |
| P6 | M406 momentary | M425 |

STOP nie kasuje kredytu. Po STOP ostatni wybrany program pozostaje w D558 i można nadal wyświetlać pozostały czas dla tej taryfy.

### MM:SS

Użyj dwóch osobnych Value Display:

- minuty: `D540`
- sekundy: `D541`

Dla obu:
- DataType: **16-Bit Unsigned Int**
- Offset address: **OFF**
- Allow input: **OFF**

Pomiędzy obiektami dodaj statyczny znak `:`.

Dla D541 ustaw dwie cyfry z zerem wiodącym.

`M426=1` oznacza, że czas jest ważny. Gdy `M426=0`, zamiast 00:00 można pokazać tekst **SELECT PROGRAM**.

### Pozostały kredyt

`D582` = pozostały kredyt w centach.

Jeżeli WSStudio umożliwia skalowanie/decimal point:
- monitor D582,
- 2 miejsca po przecinku,
- opis EUR.

Przykład: D582=100 -> 1,00 EUR.

### Wybudzenie HMI

- `M419` = HMI_WAKE_REQUEST, utrzymywany około 3 s po zaakceptowanej monecie,
- `D565` = HMI_WAKE_EVENT_COUNTER.

Preferowane: wybudzenie/przejście na Home na zboczu M419.
Alternatywa: reagowanie na zmianę D565.

## Ekran Admin

### Parametry taryfy

| Adres | Nazwa | Typ HMI | Zakres | Domyślnie |
|---|---|---|---:|---:|
| D300 | RM5 CH1 value [x0.10 EUR] | 16-bit unsigned RW | 1…100 | 10 |
| D549 | Max credit [x0.10 EUR] | 16-bit unsigned RW | 1…50 | 50 |
| D550 | Base price [x0.10 EUR] | 16-bit unsigned RW | 1…D549 | 5 |
| D551 | P1 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D552 | P2 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D553 | P3 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D554 | P4 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D555 | P5 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D556 | P6 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D528 | Clock correction x100 | 16-bit unsigned RW | 50…200 | 100 |
| D584 | Auto start program | 16-bit unsigned RW | 1…6 | 1 |

Interpretacja:
- D300=10 oznacza 1,00 EUR za zaakceptowany impuls RM5 CH1,
- D550=5 oznacza wspólną cenę bazową 0,50 EUR,
- D551=70 oznacza 70 s programu P1 za 0,50 EUR,
- D528=100 oznacza 1,00x.

Jeżeli chcesz pokazywać D528 jako 1.00 zamiast 100, użyj na HMI dwóch miejsc po przecinku / skali 0,01. PLC nadal przechowuje 100.

### Przełączniki

| Adres | Funkcja |
|---|---|
| M410 | Pilotaggio enable |
| M414 | Work lights enable |
| M429 | Sync countdown with PRACA |
| M431 | Auto start program after CREDIT output |

M429:
- OFF — odliczanie bazuje na WORK_ACTIVE; najlepsze do uruchomienia/testu,
- ON — kredyt jest zużywany tylko gdy X1/M301 PRACA=1; docelowo najlepsza synchronizacja z automatyką, jeśli X1 jest poprawnym feedbackiem.

### Diagnostyka

| Adres | Znaczenie |
|---|---|
| M300 | automat dostępny |
| M301 | PRACA z automatyki |
| M413 | konfiguracja wartości RM5 poprawna |
| M415 | konfiguracja taryfy poprawna |
| M419 | wake request |
| M426 | czas może być wyświetlany |
| M427 | kredyt dostępny |
| D521 | impulsy RM5 ostatniej paczki |
| D522 | impulsy Counter wygenerowane dla ostatniej paczki |
| D524 | impulsy RM5 sesji |
| D525:D526 | oczekiwane impulsy Counter sesji |
| D533 | impulsy Counter faktycznie wysłane Y0 |
| D557 | czas taryfy ostatniego programu |
| D558 | numer ostatniego programu |
| D560 | kredyt wewnętrzny x100 |
| D580 | pełne pozostałe jednostki 0,10 EUR |
| D582 | pozostały kredyt w centach |
| D350 | pozostały czas [s] |

## Wersja PLC

```text
D500..D503 = V1.4.5
D516 = 1
D517 = 4
D518 = 5
```


## Pasek pozostałego czasu

Do paska postępu użyj:
- wartość: `D350` = pozostały czas,
- maksimum dynamiczne: `D559` = TIME_BAR_MAX,
- minimum: 0.

`D559`:
- jest zerowane na początku nowej sesji,
- jest zerowane przy zmianie programu i od razu odbudowywane z czasu nowej taryfy,
- zwiększa się, gdy doładowanie podniesie dostępny czas,
- nie maleje podczas normalnego odliczania.

Dzięki temu pasek pokazuje procent pozostałego czasu aktualnej sesji/programu.


### Auto start program

`M431 = AUTO_START_PROGRAM_ENABLE` — przełącznik HMI.

`D584 = AUTO_START_PROGRAM_NO` — numer programu uruchamianego automatycznie:
- 1 = Program 1,
- 2 = Program 2,
- 3 = Program 3,
- 4 = Program 4,
- 5 = Program 5,
- 6 = Program 6.

Najlepiej ustawić D584 jako pole/selector 16-bit unsigned z zakresem 1…6.

Działanie:
1. klient wrzuca monetę,
2. PLC wysyła pełną kolejkę `Y0/PITSTART_CREDITS`,
3. PLC czeka aż `D330=0`, `M321=0`, `M330=0`, `M331=0`,
4. jeżeli `M431=1`, kredyt jest dostępny, automat ma X0 i żaden program nie jest już aktywny, PLC generuje `M432=AUTO_START_TRIGGER`,
5. D584 wybiera P1…P6,
6. program jest uruchamiany przez tę samą logikę wyboru co przyciski ręczne/HMI.

Jeśli użytkownik ręcznie wybierze program przed końcem wysyłania CREDIT, Auto Start **nie przełącza go później** na inny program.

STOP podczas oczekiwania anuluje automatyczny start dla bieżącej sesji, aby program nie uruchomił się sam po zwolnieniu STOP.

Status:
- `M432` — jednocyklowy impuls Auto Start,
- `M433` — Auto Start wykonany/anulowany dla bieżącej sesji,
- `M434` — D584 ma poprawną wartość 1…6.

Odliczanie czasu nie jest już opóźniane przez M431. Po uruchomieniu programu działa normalnie według `M412` i opcjonalnie `M429/PRACA`.


### Przełącznik ADMIN X14

PLC kopiuje stan fizycznego wejścia:

```text
X14 -> M303
```

`M303 = ADMIN_SCREEN_REQUEST` jest przeznaczony dla HMI:

- M303=1 — przełącz / utrzymuj ekran **Admin**,
- M303=0 — wróć / utrzymuj ekran **Home**.

M303 jest tylko odczytem stanu fizycznego przełącznika X14; HMI nie powinno zapisywać do M303.

Jeżeli WSStudio ma funkcję przełączania ekranu sterowaną bitem PLC, użyj M303 bezpośrednio. Dla przełącznika bistabilnego nie jest potrzebny dodatkowy impuls zbocza.
