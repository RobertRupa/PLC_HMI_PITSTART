# Bezpieczny start i obsługa przycisków — V1.2.0

## Mapowanie

- X4 = STOP
- X5 = P1
- X6 = P2
- X7 = P3
- X10 = P4
- X11 = P5
- X12 = P6
- X13 = MANUAL / FREE status
- X27 = RM5 CH1

HMI:
- M400 = STOP
- M401…M406 = P1…P6
- M410 = Pilotaggio enable
- M411 = reset licznika sesji

## Łączenie HMI i mechaniki

```text
X4  OR M400 -> M440
X5  OR M401 -> M441
X6  OR M402 -> M442
X7  OR M403 -> M443
X10 OR M404 -> M444
X11 OR M405 -> M445
X12 OR M406 -> M446
```

## Start programu

Program może wystartować tylko przy:
- M300=1,
- M440=0,
- D350>0.

M451…M456 są impulsami jednocyklowymi wyboru. Aktywny program jest zapisany w M420…M425.

## STOP

STOP:
- resetuje M420…M425,
- wyłącza Y2…Y7,
- wyłącza M412,
- wyłącza Y1 i Y27,
- nie kasuje D350.

Ponieważ D350 jest odliczane tylko przy M412=1, STOP pauzuje pozostały czas.

## Utrata X0

M300=0:
- blokuje RM5 przez Y23,
- resetuje wybrany program,
- blokuje Y2…Y7.

## Pierwszy skan M8002

Bezwarunkowo:
- D300=10
- D320=0
- D330=0
- D350=0
- D351=0
- D521…D527=0
- M320/M321/M330/M331=0
- M400…M406=0
- M411/M412=0
- M420…M425=0
- M410=1

D520:
- jeśli 1…600 — pozostaje bez zmian,
- jeśli <=0 lub >600 — zostaje ustawione na 10.

Wersja:
- D500…D503 = V1.2.0
- D516=1
- D517=2
- D518=0

## Retencja D520

Kod nie nadpisuje poprawnego D520 w pierwszym skanie, ale trwałość tej wartości przez zanik zasilania zależy od konfiguracji latched/retentive device w FX3UC. Jeżeli D520 nie jest w obszarze retencyjnym, po zimnym starcie może wrócić do 0 i PLC ustawi domyślne 10.

## Koniec czasu

Po D350<=0 M420…M425 są kasowane. Kolejne doładowanie nie uruchomi automatycznie poprzedniego programu.
