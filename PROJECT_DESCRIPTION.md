# Opis projektu — V1.5.7

Projekt odtwarza funkcje Comestero PitStart w sterowniku myjni i integruje PLC, HMI oraz akceptor monet RM5 Evolution.

## Aktualne pliki

```text
PLC project: projects/plc/PitStart.zip
PLC import:  plc/main_v1.5.7.csv
PLC IL:      plc/MAIN_GXDEV_ENTRY.txt
HMI project: projects/hmi/pitstart.hs
```

## Model kredytu

Źródłem prawdy jest pozostały kredyt pieniężny.

- `Y0 / PITSTART_CREDITS` = impulsy po 0,10 EUR,
- `D300` = wartość jednego impulsu RM5 CH1 w jednostkach 0,10 EUR,
- `D550` = wspólna cena bazowa w jednostkach 0,10 EUR,
- `D551…D556` = czas P1…P6 dla ceny bazowej,
- `D560` = pozostały kredyt w centach,
- `D350 / D540 / D541` = pozostały czas.

## Wejścia

```text
X0   AUTOMATE_PRESENT / INVERTER_OK
X1   PRACA / RUN
X4   STOP
X5   P1
X6   P2
X7   P3
X10  P4
X11  P5
X12  P6
X13  MANUAL / FREE status
X14  ADMIN switch
X27  RM5 CH1
```

## Wyjścia

```text
Y0   PITSTART_CREDITS / Counter
Y1   PILOTAGGIO
Y2   RM5 INHIBIT PRIMARY
Y3   Program 1
Y4   Program 2
Y5   Program 3
Y6   Program 4
Y7   Program 5
Y10  Program 6
Y23  RM5 INHIBIT compatibility mirror
Y27  WORK LIGHTS
```

FX używa ósemkowej numeracji X/Y: po Y7 występuje Y10.

## RM5 Evolution

Aktualne fizyczne połączenie:

```text
RM5 CH1 pin 7 -> X27
RM5 INHIBIT pin 6 <- Y2
```

Y23 ma identyczną logikę jak Y2, ale pozostaje tylko kompatybilnym mirrorem.

```text
X0=0 OR M435=1 -> Y2=1, Y23=1
X0=1 AND M435=0 -> Y2=0, Y23=0
```

Wszystkie używane wartości monet muszą być przekazane przez CH1. Szczegóły: [docs/RM5.md](docs/RM5.md).

## Wybór programu

Program może zostać wybrany przez:
- fizyczne wejścia X5…X12,
- HMI M401…M406,
- Auto Start Program.

Aktywne stany programu: `M420…M425`.

Zmiana programu nie kasuje kredytu. `D557` i wynikowy czas są aktualizowane zgodnie z taryfą wybranego programu.

## Auto Start Program

```text
M431 = AUTO_START_PROGRAM_ENABLE
D584 = program 1..6
M432 = AUTO_START_TRIGGER
M433 = AUTO_START_DONE
M434 = AUTO_START_PROGRAM_OK
```

Auto Start wymaga m.in. ważnego numeru programu, dostępnego kredytu, pustej kolejki Y0, braku aktywnego programu oraz `D558=0`. Późniejsze doładowanie nie zmienia już wybranego programu.

## Sync time with RUN

V1.5.7 używa jednoznacznego selektora źródła odliczania:

```text
M429=0 -> COUNTDOWN_ACTIVE = M412 / WORK_ACTIVE
M429=1 -> COUNTDOWN_ACTIVE = M301 / PRACA-RUN
```

`M449` jest końcową bramką odliczania, a `M430` jej mirrorem diagnostycznym.

### STOP przy aktywnym RUN

STOP kasuje M420…M425 i natychmiast wyłącza wyjścia programu. Jeżeli jednak `M429=1` i `M301/RUN=1`, kredyt/czas nadal jest zużywany. Po zaniku RUN odliczanie staje.

Przy `M429=0` STOP zatrzymuje odliczanie razem z `M412=WORK_ACTIVE`.

## HMI

Aktualny projekt: [projects/hmi/pitstart.hs](projects/hmi/pitstart.hs).

Projekt zawiera:
- KinSealStudio V1.0.2,
- profil SUP070 / wsb-070-16M,
- 800×480,
- sterownik `Mitsubishi_Fx1n`,
- ekrany 000 Home, 001 Admin, 002 Error, 003 Ready.

Sterowanie ekranami:

```text
D585 = PLC -> HMI screen index
D586 = HMI -> PLC current screen index
```

Szczegóły: [docs/HMI.md](docs/HMI.md).

## Restart PLC

`T204/M438` tworzy okno resetu stanu sesji po wejściu PLC w RUN. Zerowane są dane bieżącej sesji, ale parametry taryfy i przełączniki konfiguracyjne pozostają zachowane lub są korygowane tylko wtedy, gdy są poza dopuszczalnym zakresem.

Domyślnie ustawiane są m.in. `M410`, `M414`, `M429`, `M431` i `M439`.

## Pilotaggio default

`M439 = PILOTAGGIO_DEFAULT` jest wartością domyślną stosowaną po zakończeniu całej kolejki Y0, jeżeli bieżące `M410` jest wyłączone.

## Domyślna taryfa

```text
D300=10
D550=10
D551..D556=300
```

Przy tych ustawieniach jeden impuls RM5 CH1 daje 10 impulsów Y0 i 300 s dla każdego programu.

## Znane ograniczenie czasu

`D350` jest pojedynczym rejestrem 16-bit. Bieżąca walidacja `M415` nie sprawdza kombinacji maksymalnego kredytu, ceny bazowej i czasu programu pod kątem przepełnienia D350.

Konserwatywny warunek konfiguracji:

```text
D549 * D557 / D550 <= 32767 s
```

Domyślne ustawienia są daleko poniżej tego limitu.
