# PLC_HMI_PITSTART

Projekt integracji sterownika PLC/HMI SEEKU (PLC zgodny z rodziną Mitsubishi FX), akceptora monet **Comestero RM5 Evolution** oraz sterownika myjni w sposób zgodny funkcjonalnie z **Comestero PitStart**.

## Oprogramowanie

### PLC
Do edycji programu PLC używany jest:
- **MELSOFT GX Developer**

Połączenie programujące używane w projekcie:
- RS232 / USB→RS232,
- PLC side: `FXCPU`,
- 38.4 kbps.

### HMI
Do edycji panelu HMI:
- **WSStudio** firmy Winsun/SEEKU.

PLC i HMI są programowane osobno. Sterowanie podstawowe działa również bez HMI dzięki mechanicznym przyciskom STOP + Program 1…6.

## Aktualna architektura

### Wejścia

Lokalne wejścia panelu wykorzystują zakres `X0…X14` zgodnie z mapą poniżej. Kanał RM5 CH1 pozostaje przypisany w projekcie do `X27`.

| Adres | Funkcja |
|---|---|
| `X0` | `AUTOMATE_PRESENT / INVERTER_OK` — główny sygnał zezwolenia |
| `X1` | `PRACA` — sygnał informacyjny; nie blokuje RM5 |
| `X4` | mechaniczny STOP |
| `X5` | mechaniczny Program 1 |
| `X6` | mechaniczny Program 2 |
| `X7` | mechaniczny Program 3 |
| `X10` | mechaniczny Program 4 |
| `X11` | mechaniczny Program 5 |
| `X12` | mechaniczny Program 6 |
| `X13` | rezerwa |
| `X14` | rezerwa |
| `X27` | RM5 CH1 — wejście impulsów |

### Wyjścia

| Adres | Funkcja |
|---|---|
| `Y0` | CREDIT / COMPTEUR — impulsy do sterownika myjni |
| `Y1` | PILOTAGGIO POMPA |
| `Y2` | Program 1 |
| `Y3` | Program 2 |
| `Y4` | Program 3 |
| `Y5` | Program 4 |
| `Y6` | Program 5 |
| `Y7` | Program 6 |
| `Y23` | RM5 INHIBIT |
| `Y27` | oświetlenie podczas pracy |

## Sterowanie mechaniczne + HMI

HMI i przyciski mechaniczne są łączone logicznym OR. Żaden z nich nie jest wymagany do działania drugiego.

| Funkcja | Przycisk fizyczny | HMI | Komenda wspólna |
|---|---|---|---|
| STOP | X4 | M400 | M440 |
| P1 | X5 | M401 | M441 |
| P2 | X6 | M402 | M442 |
| P3 | X7 | M403 | M443 |
| P4 | X10 | M404 | M444 |
| P5 | X11 | M405 | M445 |
| P6 | X12 | M406 | M446 |

Przykład:

```text
X4 OR M400 -> M440
X5 OR M401 -> M441
...
X11 OR M405 -> M445
X12 OR M406 -> M446
```

STOP ma priorytet. Programy są wybierane zboczem komendy i są wzajemnie wykluczające.

Aktywne programy:
- `M420` = P1,
- `M421` = P2,
- `M422` = P3,
- `M423` = P4,
- `M424` = P5,
- `M425` = P6.

Wyjścia:

```text
M420 AND D350>0 -> Y2
M421 AND D350>0 -> Y3
M422 AND D350>0 -> Y4
M423 AND D350>0 -> Y5
M424 AND D350>0 -> Y6
M425 AND D350>0 -> Y7
```

STOP zeruje wybór programu, ale nie kasuje kredytu `D350`. Po wyczerpaniu kredytu wybór programu jest kasowany, aby po kolejnym doładowaniu myjnia nie uruchomiła poprzedniego programu automatycznie.

## Bezpieczny start PLC

Problem zaobserwowany wcześniej: PLC zapamiętywał `D330` i po restarcie kontynuował wysyłanie impulsów CREDIT.

Dlatego na **pierwszym skanie PLC (`M8002`)** wymuszamy bezpieczne wartości:

```text
M8002 -> MOV K10 D300   ; wartość/mnożnik kanału 1 RM5 = 10
M8002 -> MOV K0  D320   ; bieżąca paczka RM5
M8002 -> MOV K0  D330   ; kolejka CREDIT - krytyczne, zapobiega wznowieniu impulsów
M8002 -> MOV K0  D350   ; kredyt/wartość
M8002 -> MOV K0  D351   ; starsze słowo MUL

M8002 -> RST M321
M8002 -> RST M330
M8002 -> RST M331
M8002 -> RST M410       ; Pilotaggio domyślnie OFF
M8002 -> RST M420
M8002 -> RST M421
M8002 -> RST M422
M8002 -> RST M423
M8002 -> RST M424
M8002 -> RST M425
```

Po restarcie:
- nie ma oczekujących impulsów,
- kredyt jest równy 0,
- żaden program nie jest wybrany,
- Pilotaggio jest wyłączone,
- RM5 pozostaje blokowany, dopóki nie ma poprawnego `X0/AUTOMATE_PRESENT`.

## RM5 — kanał 1

Używany jest jeden kanał RM5 podłączony do `X27`.

Domyślna wartość kanału 1 / mnożnik:
- `D300 = 10`.

Algorytm:
1. impuls RM5 → `M320`,
2. każdy impuls zwiększa `D320`,
3. `M321` oznacza aktywne zbieranie paczki,
4. każdy kolejny impuls restartuje `T200`,
5. po około 1 s bez impulsu `T200` kończy paczkę,
6. każdy impuls dodaje `D520` sekund do `D350`,
7. po końcu paczki `D320` → `D521` i `D330`,
8. `D320 × D520` → `D522:D523` jako czas ostatniej paczki,
9. `D320` jest zerowane,
10. `D330` steruje generatorem CREDIT na `Y0`.

```text
T200 -> MOV D320 D521
T200 -> MUL D320 D520 D522
T200 -> MOV D320 D330
T200 -> MOV K0 D320
T200 -> RST M321
```

## Konwersja impulsów RM5 na czas + licznik HMI

Dodany został osobny parametr czasu, niezależny od `D300=10`:

| Adres | Funkcja | Dostęp HMI |
|---|---|---|
| `D520` | sekundy dodawane za 1 impuls RM5 | RW |
| `D521` | liczba impulsów w ostatniej paczce | RO |
| `D522:D523` | czas ostatniej paczki w sekundach, wynik 32-bit | RO |
| `D524` | licznik impulsów RM5 od startu/resetu sesji | RO |
| `D525:D526` | równoważny czas licznika sesji = D524 × D520 | RO, 32-bit |
| `M411` | reset licznika sesji D524/D525:D526 | przycisk chwilowy |
| `M413` | parametr D520 poprawny | RO |

Domyślnie `D520=10 s/impuls`. Zachowuje to dotychczasowe efektywne przeliczenie, ale od teraz współczynnik czasu jest osobnym parametrem HMI.

Dopuszczalny zakres `D520`:

```text
1 ... 600 s/impuls
```

Poza tym zakresem `M413=0` i RM5 jest blokowany przez INHIBIT.

Każdy prawidłowo odebrany impuls:
- zwiększa `D320` i licznik sesji `D524`,
- dodaje `D520` sekund do pozostałego czasu `D350`,
- `D350` jest ograniczony do maks. 32000 s, aby uniknąć przepełnienia 16-bit.

Po zakończeniu paczki PLC zapisuje:
- `D521 = D320`,
- `D522:D523 = D320 × D520`,
- `D330 = D320` dla generatora CREDIT.

Dzięki temu kolejne monety **dodają czas do pozostałego czasu**, zamiast nadpisywać `D350`.

## Generator CREDIT — Y0

```text
D330 > 0 AND /M330 AND /M331 -> SET M330

M330 -> Y0
M330 -> T201 K10

T201 -> DEC D330
T201 -> RST M330
T201 -> SET M331

M331 -> T202 K10
T202 -> RST M331
```

Aktualnie:
- `T201 K10` ≈ 0,1 s ON,
- `T202 K10` ≈ 0,1 s OFF,
- okres start-start ≈ 0,2 s.

## RM5 INHIBIT — Y23

```text
/M300 -> Y23
```

- `X0/M300 = 1` — RM5 aktywny,
- `X0/M300 = 0` — RM5 zablokowany.

`X1/PRACA` nie uczestniczy w blokowaniu RM5.

## Odliczanie D350

```text
LDP M8013
AND> D350 K0
DEC D350
```

Dzięki warunkowi `D350 > 0` licznik nie schodzi poniżej zera.

## Pilotaggio pompa — Y1

- `M410 = PILOTAGGIO_ENABLE`,
- `M412 = WORK_ACTIVE`.

```text
M410 AND M412 -> Y1
```

Domyślnie po restarcie `M410=0` — bezpiecznie OFF.

## WORK_ACTIVE i oświetlenie

`M412` jest aktywne, gdy:
- wybrany jest P1…P6,
- `D350 > 0`,
- STOP nie jest aktywny.

```text
M412 -> Y27
```

Oświetlenie działa podczas pracy niezależnie od opcji Pilotaggio.


## Wersja PLC dla HMI

Wersja projektu PLC jest udostępniona HMI jako tekst ASCII od `D500`.

Aktualna wersja:

```text
V1.1.0
```

Mapa:

| Adres | Znaczenie |
|---|---|
| `D500…D507` | `PLC_VERSION_STRING` — pole tekstowe dla HMI |
| `D516` | VERSION_MAJOR = 1 |
| `D517` | VERSION_MINOR = 1 |
| `D518` | VERSION_PATCH = 0 |

Na pierwszym skanie PLC:

```text
M8002 -> MOV H3156 D500   ; "V1"
M8002 -> MOV H312E D501   ; ".1"
M8002 -> MOV H302E D502   ; ".0"
M8002 -> MOV H0000 D503   ; terminator

M8002 -> MOV K1 D516
M8002 -> MOV K1 D517
M8002 -> MOV K0 D518
```

W WSStudio użyj obiektu ASCII/String Display od adresu `D500`, długość np. 16 znaków. Jeżeli HMI pokaże pary znaków odwrócone, należy odwrócić bajty stałych HEX dla używanego sterownika/HMI.

## Dokumentacja

- `IO_TABLE.txt` — aktualna mapa projektu.
- `PROJECT_DESCRIPTION.md` — opis funkcjonalny.
- `docs/PITSTART.md` — funkcjonalność i złącza PitStart.
- `docs/RM5.md` — RM5.
- `docs/WIRING.md` — połączenia elektryczne.
- `docs/LADDER_LOGIC.md` — logika drabinki.
- `docs/STARTUP_AND_BUTTONS.md` — inicjalizacja i przyciski mechaniczne/HMI.
- `docs/wiring.svg` — uproszczony schemat.
- `docs/HMI.md` — adresy HMI, w tym string wersji PLC.
- `plc/MAIN.txt` — aktualna logika PLC w formie mnemonic/instruction list.
- `plc/DEVICE_MAP.csv` — mapa urządzeń do wersjonowania i analizy.
- `plc/README.md` — sposób pracy z eksportem tekstowym.
