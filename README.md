# PLC_HMI_PITSTART

Projekt integracji sterownika PLC/HMI SEEKU (PLC zgodny z rodziną Mitsubishi FX3UC), akceptora monet **Comestero RM5 Evolution** oraz sterownika myjni w sposób zgodny funkcjonalnie z **Comestero PitStart**.

Aktualna wersja logiki PLC: **V1.2.0**.

## Oprogramowanie

### PLC
- **MELSOFT GX Developer**
- połączenie używane w projekcie: RS232 / USB→RS232
- PLC side: `FXCPU`
- 38.4 kbps

GX Converter nie jest wymagany. Plik `plc/MAIN_GXDEV_ENTRY.txt` jest przygotowany do ręcznego wprowadzenia w widoku **Instruction List** w GX Developer.

### HMI
- **WSStudio** firmy Winsun/SEEKU

PLC i HMI są programowane osobno. Sterowanie podstawowe działa również bez HMI dzięki mechanicznym przyciskom STOP + Program 1…6.

## I/O

### Wejścia

| Adres | Funkcja |
|---|---|
| `X0` | AUTOMATE_PRESENT / INVERTER_OK |
| `X1` | PRACA — status |
| `X4` | STOP |
| `X5` | Program 1 |
| `X6` | Program 2 |
| `X7` | Program 3 |
| `X10` | Program 4 |
| `X11` | Program 5 |
| `X12` | Program 6 |
| `X13` | MANUAL / FREE — w V1.2.0 status tylko do odczytu |
| `X14` | rezerwa |
| `X27` | RM5 CH1 |

### Wyjścia

| Adres | Funkcja |
|---|---|
| `Y0` | CREDIT / COMPTEUR |
| `Y1` | PILOTAGGIO POMPA |
| `Y2…Y7` | Program 1…6 |
| `Y23` | RM5 INHIBIT |
| `Y27` | oświetlenie podczas pracy |

## Przyciski mechaniczne + HMI

| Funkcja | Mechaniczny | HMI | Komenda wspólna |
|---|---|---|---|
| STOP | X4 | M400 | M440 |
| P1 | X5 | M401 | M441 |
| P2 | X6 | M402 | M442 |
| P3 | X7 | M403 | M443 |
| P4 | X10 | M404 | M444 |
| P5 | X11 | M405 | M445 |
| P6 | X12 | M406 | M446 |

Programy są wybierane zboczem komendy. W V1.2.0 używane są jednocyklowe bity `M451…M456`, a aktywny program jest zapamiętywany w `M420…M425`.

STOP:
- kasuje aktywny program,
- wyłącza P1…P6, Pilotaggio i oświetlenie,
- **nie kasuje `D350`**.

Utrata `X0/AUTOMATE_PRESENT` również kasuje aktywny program.

## RM5 i czas

RM5 CH1:

```text
RM5 pin 7 CH1 -> X27
PLC Y23       -> RM5 pin 6 INHIBIT
```

Impuls `X27` jest akceptowany tylko gdy:
- `M300=1` / automat jest dostępny,
- `M413=1` / parametr czasu jest poprawny.

### Parametr czasu

`D520` = liczba sekund dodawanych za jeden impuls RM5.

- HMI: RW
- zakres: 1…600 s/impuls
- wartość domyślna: 10 s/impuls

Jeżeli po starcie `D520` ma poprawną wartość, PLC jej nie nadpisuje. Jeżeli jest poza zakresem, ustawia 10. Zachowanie wartości po zaniku zasilania zależy od konfiguracji retencji/latch pamięci FX3UC lub ponownego zapisu przez HMI.

Każdy zaakceptowany impuls:
- zwiększa `D320`,
- zwiększa `D524` do maks. 30000,
- dodaje `D520` sekund do `D350`,
- ogranicza `D350` do 32000 s.

Po zakończeniu paczki:
- `D521` = impulsy ostatniej paczki,
- `D522:D523` = czas ostatniej paczki,
- `D330` = kolejka impulsów CREDIT dla Y0.

`D300=10` pozostaje polem wartości kanału 1 RM5; aktualna konwersja na czas korzysta z `D520`.

## CREDIT / COMPTEUR

`Y0` generuje impulsy z kolejki `D330`:

- około 0,1 s ON,
- około 0,1 s OFF,
- około 0,2 s start-start.

Po restarcie `D330` jest zerowane, dzięki czemu PLC nie kontynuuje starej kolejki CREDIT.

## Czas pracy

`D350` jest pozostałym czasem w sekundach.

Odliczanie następuje tylko podczas rzeczywistej pracy:

```text
LDP M8013
AND M412
AND> D350 K0
DEC D350
```

Dlatego STOP pauzuje pozostały czas.

## WORK_ACTIVE / Pilotaggio

`M412 = WORK_ACTIVE` gdy:
- aktywny jest jeden z P1…P6,
- `M300=1`,
- `D350>0`,
- STOP nie jest aktywny.

```text
M410 AND M412 -> Y1
M412          -> Y27
```

W V1.2.0 `M410/PILOTAGGIO_ENABLE` jest ustawiane na ON przy starcie, aby mechaniczny panel działał bez HMI. HMI może je później wyłączyć.

## HMI — liczniki i wersja

| Adres | Funkcja |
|---|---|
| `D350` | pozostały czas [s] |
| `D520` | czas / impuls [s], RW |
| `D521` | impulsy ostatniej paczki |
| `D522:D523` | czas ostatniej paczki [s] |
| `D524` | licznik impulsów sesji |
| `D525:D526` | równoważny czas sesji [s] |
| `M411` | reset licznika sesji |
| `M413` | parametr czasu poprawny |
| `M302` | MANUAL / FREE status |

Wersja `V1.2.0` jest dostępna od `D500`:

```text
D500 = "V1"
D501 = ".2"
D502 = ".0"
D503 = 0

D516 = 1
D517 = 2
D518 = 0
```

## Bez GX Converter

Do wprowadzenia programu użyj:
- `plc/MAIN_GXDEV_ENTRY.txt` — czysta lista instrukcji,
- `plc/GX_DEVELOPER_NO_CONVERTER.md` — procedura krok po kroku.

W GX Developer: `MAIN -> Alt+F1 / Instruction List -> wpisz program -> F4 Convert -> kontrola Ladder -> Check program`.

## Dokumentacja

- `IO_TABLE.txt` — aktualna mapa urządzeń.
- `PROJECT_DESCRIPTION.md` — opis funkcjonalny.
- `docs/LADDER_LOGIC.md` — logika V1.2.0.
- `docs/HMI.md` — adresy HMI.
- `docs/RM5.md` — RM5.
- `docs/PITSTART.md` — odniesienie do oryginalnego PitStart.
- `docs/WIRING.md` — połączenia.
- `docs/STARTUP_AND_BUTTONS.md` — start i przyciski.
- `plc/MAIN.txt` — udokumentowana lista instrukcji.
- `plc/MAIN_GXDEV_ENTRY.txt` — lista do wpisania w GX Developer.
- `plc/DEVICE_MAP.csv` — mapa urządzeń.
