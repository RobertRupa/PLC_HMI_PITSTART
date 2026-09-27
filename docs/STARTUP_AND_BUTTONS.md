# Start, przyciski i STOP — V1.5.7

## Start PLC

Po wejściu PLC w RUN działa okno resetu `T204/M438`. W tym czasie zerowany jest stan bieżącej sesji, m.in.:
- kolejka Y0 `D330`,
- kredyt `D560`,
- czas `D350`,
- liczniki sesji,
- M420…M425,
- stany RM5 M435…M437,
- wewnętrzne stany odliczania M448/M449.

Na pierwszym skanie ustawiane są domyślnie:

```text
M410 = ON   Pilotaggio enable
M414 = ON   Work lights enable
M429 = ON   Sync time with RUN
M431 = ON   Auto Start Program
M439 = ON   Pilotaggio default
```

Parametry są korygowane tylko wtedy, gdy są poza zakresem:

```text
D300      1..100   default 10
D528     50..200   default 100
D549      1..999   default 50
D550      1..50*   default 10
D551..556 1..600   default 300
D584      1..6     default 1
```

`* D550` jest dodatkowo sprawdzane przez M415 względem D549.

## Nowa sesja RM5

`M417` powstaje przy pierwszym zaakceptowanym impulsie RM5, gdy:
- `D560<=0`,
- `D330<=0`,
- nie trwa M321/M330/M331.

M417 zeruje dane poprzedniej sesji przed zliczaniem nowej.

## Przyciski

HMI:

```text
M400 STOP
M401 P1
M402 P2
M403 P3
M404 P4
M405 P5
M406 P6
```

Fizyczne wejścia:

```text
X4  STOP
X5  P1
X6  P2
X7  P3
X10 P4
X11 P5
X12 P6
```

M420…M425 przechowują aktywny program P1…P6.

## STOP

STOP:
- kasuje M420…M425,
- wyłącza wyjścia programów,
- wyłącza Y1/Pilotaggio i Y27/Work Lights przez utratę M412,
- nie kasuje D560,
- nie kasuje D557/D558,
- nie kasuje obliczonego kredytu sesji.

Zachowanie czasu zależy od `M429`:

```text
M429=0 -> czas liczy się tylko przy M412/WORK_ACTIVE
M429=1 -> czas liczy się przy M301/PRACA-RUN
```

Dlatego przy włączonym **Sync time with RUN** naciśnięcie STOP nie zatrzyma czasu, jeżeli RUN nadal ma stan 1.

## Utrata AUTOMATE_PRESENT

Gdy `X0/M300=0`, aktywny program jest kasowany, wyjścia P1…P6 są blokowane, a RM5 jest blokowany przez INHIBIT.
