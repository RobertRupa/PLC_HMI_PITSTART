# Opis projektu PLC_HMI_PITSTART

## Cel

SEEKU PLC/HMI realizuje logikę zgodną funkcjonalnie z Comestero PitStart:
- odbiera impulsy RM5,
- blokuje RM5, gdy myjnia nie jest dostępna,
- generuje CREDIT/COMPTEUR,
- steruje Pilotaggio,
- steruje programami P1…P5,
- obsługuje jednocześnie HMI i mechaniczne przyciski,
- steruje oświetleniem,
- uruchamia się zawsze w bezpiecznym stanie.

## Wejścia użytkownika

Mechaniczne:
- X4 = STOP,
- X5 = P1,
- X6 = P2,
- X7 = P3,
- X10 = P4,
- X11 = P5,
- X12 = P6.

HMI:
- M400 = STOP,
- M401…M406 = P1…P6.

PLC scala oba źródła:

```text
X4 OR M400 -> M440 STOP_CMD
X5 OR M401 -> M441 P1_CMD
X6 OR M402 -> M442 P2_CMD
X7 OR M403 -> M443 P3_CMD
X10 OR M404 -> M444 P4_CMD
X11 OR M405 -> M445 P5_CMD
X12 OR M406 -> M446 P6_CMD
```

Dzięki temu:
- HMI może być odłączone, a przyciski fizyczne nadal działają,
- brak panelu mechanicznego nie blokuje sterowania HMI,
- STOP ma priorytet.

## Aktywny program

Bity:
- M420 = P1,
- M421 = P2,
- M422 = P3,
- M423 = P4,
- M424 = P5,
- M425 = P6.

Programy są wzajemnie wykluczające. Komenda nowego programu resetuje poprzedni i ustawia tylko nowy.

Wyjścia:
- M420 AND D350>0 -> Y2,
- M421 AND D350>0 -> Y3,
- M422 AND D350>0 -> Y4,
- M423 AND D350>0 -> Y5,
- M424 AND D350>0 -> Y6,
- M425 AND D350>0 -> Y7.

STOP kasuje wybór programu, ale nie kasuje D350.

Po zejściu D350 do 0 wszystkie M420…M425 są resetowane. To zapobiega automatycznemu wznowieniu poprzedniego programu po wrzuceniu kolejnej monety.

## Bezpieczny restart

Na pierwszym skanie PLC (`M8002`) ustawiane są wartości:

```text
D300 = 10
D320 = 0
D330 = 0
D350 = 0
D351 = 0
D520 = 10
D521...D527 = 0 / wartości robocze

M321 = 0
M330 = 0
M331 = 0
M410 = 0
M420...M425 = 0
```

Kluczowa zmiana to `D330=0`: PLC nie kontynuuje po restarcie wcześniej rozpoczętej serii impulsów CREDIT.

`D300=10` jest domyślną wartością kanału 1 RM5 / mnożnikiem.

## RM5

- CH1 -> X27,
- INHIBIT <- Y23,
- brak X0/AUTOMATE_PRESENT blokuje RM5,
- X1/PRACA nie blokuje RM5.

Paczka impulsów kończy się po około 1 s bezczynności.

## Impulsy RM5 -> czas

`D300=10` pozostaje wartością kanału 1 RM5. Konwersja na czas jest odseparowana:

- `D520` = sekundy za 1 impuls RM5, parametr HMI,
- domyślnie `D520=10`,
- zakres bezpieczny: 1…600 s/impuls,
- `D350` = pozostały czas w sekundach,
- `D524` = licznik impulsów sesji,
- `D525:D526` = równoważny czas sesji w sekundach.

Każdy impuls zwiększa licznik oraz dodaje `D520` do `D350`. Czas jest saturacyjnie ograniczony do 32000 s, aby nie przepełnić dodatniego zakresu używanego przez 16-bitową logikę czasu.

Po końcu paczki:
- `D521` przechowuje liczbę impulsów ostatniej paczki,
- `D522:D523` przechowuje przeliczony czas tej paczki,
- `D330` przechowuje liczbę impulsów do wysłania na Y0.

`M411` z HMI resetuje licznik sesji. `M413` sygnalizuje poprawność parametru `D520`.

## CREDIT

Y0 generuje CREDIT z kolejki D330.

## Pilotaggio

- M410 = PILOTAGGIO_ENABLE,
- M412 = WORK_ACTIVE,
- M410 AND M412 -> Y1.

Domyślnie po restarcie Pilotaggio jest wyłączone.

## WORK_ACTIVE

M412 jest aktywne, gdy:
- wybrany jest jeden z P1…P6,
- D350 > 0,
- STOP nie jest aktywny.

M412 steruje:
- Y27 = oświetlenie,
- Y1 = Pilotaggio, jeśli M410=1.

## Oświetlenie

```text
M412 -> Y27
```

## Wyjścia

- Y0 = CREDIT,
- Y1 = Pilotaggio,
- Y2…Y6 = P1…P5,
- Y7 = Program 6,
- Y23 = RM5 inhibit,
- Y27 = oświetlenie.
