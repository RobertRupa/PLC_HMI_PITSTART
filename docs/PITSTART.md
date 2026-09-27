# Comestero PitStart — odniesienie funkcjonalne dla V1.5.7

Źródło odniesienia projektu: **Gamma Pit – Sistemi di attivazione – Manuale operativo**, PitStart-F, Rev. 00, 13/10/2009.

Ten dokument opisuje, jak funkcje oryginalnego PitStart są odwzorowane w aktualnym PLC. Nie jest kopią instrukcji producenta.

## Wyjścia maszyny

Funkcje PitStart:
- Counter / Compteur,
- Pilotaggio pompa,
- P1…P6.

Aktualne mapowanie:

```text
Y0  Counter / PITSTART_CREDITS
Y1  Pilotaggio
Y3  P1
Y4  P2
Y5  P3
Y6  P4
Y7  P5
Y10 P6
```

Y2 jest używane przez RM5 INHIBIT, dlatego programy zaczynają się od Y3.

Każdy impuls Y0 reprezentuje 0,10 EUR.

## Cena bazowa i czasy

```text
D550 = wspólna cena bazowa [x0,10 EUR]
D551 = P1 seconds
D552 = P2 seconds
...
D556 = P6 seconds
```

Pozostały kredyt pieniężny jest przechowywany w D560. Zmiana programu nie kasuje kredytu, tylko przelicza pozostały czas według nowej taryfy.

## RM5 Evolution

`D300` określa wartość pojedynczego impulsu CH1 w jednostkach 0,10 EUR.

```text
D300=1  -> CH1 = 0,10 EUR
D300=10 -> CH1 = 1,00 EUR
```

PLC generuje odpowiednią liczbę impulsów Y0.

## Pilotaggio

```text
M410 AND M412 -> Y1
```

STOP wyłącza aktywny program i M412, więc Y1 wyłącza się natychmiast.

## Manual / Free

`X13 -> M302` jest obecnie statusem. Pełny tryb FREE bez kredytu nie jest zaimplementowany.

## PRACA / RUN

X1/M301 jest dodatkowym feedbackiem automatyki, którego nie należy mylić z funkcjami oryginalnego PitStart.

`M429 = Sync time with RUN`:

```text
M429=0 -> odliczanie wg M412/WORK_ACTIVE
M429=1 -> odliczanie wg M301/PRACA-RUN
```

Przy M429=1 czas może nadal schodzić po STOP, jeśli RUN pozostaje aktywny.

## HMI

- D585 — indeks ekranu zadawany przez PLC,
- D586 — indeks aktualnego ekranu zwracany przez HMI,
- M419/D587 — sygnał zdarzenia RM5,
- D565 — licznik zaakceptowanych impulsów RM5.

Aktualny projekt HMI: [../projects/hmi/pitstart.hs](../projects/hmi/pitstart.hs).
