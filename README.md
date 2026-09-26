# PLC_HMI_PITSTART

Aktualna wersja PLC: **V1.4.0**.

Logika V1.4.0 została przebudowana na model kredytowy zgodny z zachowaniem PitStart:
- `Y0 / PITSTART_CREDITS` reprezentuje jednostki **0,10 EUR**,
- `D300` określa wartość RM5 CH1 w jednostkach 0,10 EUR,
- wspólna cena bazowa jest w `D550`,
- czasy P1…P6 dla ceny bazowej są w `D551…D556`,
- PLC prowadzi pozostały kredyt pieniężny i z niego oblicza czas dla wybranego programu.

## I/O

### Wejścia
- X0 — AUTOMATE_PRESENT / INVERTER_OK
- X1 — PRACA
- X4 — STOP
- X5…X12 — P1…P6
- X13 — MANUAL / FREE status
- X27 — RM5 CH1

### Wyjścia
- Y0 — PITSTART_CREDITS / Counter
- Y1 — PILOTAGGIO
- Y2…Y7 — P1…P6
- Y23 — RM5 INHIBIT
- Y27 — WORK LIGHTS

## Model kredytu

```text
1 impuls Y0 = 0,10 EUR

D300 = wartość jednego impulsu RM5 CH1 w jednostkach 0,10 EUR
D550 = wspólna cena bazowa w jednostkach 0,10 EUR
D551..D556 = czas P1..P6 w sekundach dla D550
```

Przykład:

```text
D300 = 10  -> RM5 CH1 = 1,00 EUR
D550 = 5   -> cena bazowa = 0,50 EUR
D551 = 70  -> P1 daje 70 s za 0,50 EUR
```

Jeden impuls RM5 wygeneruje wtedy 10 impulsów Counter/Y0.

## Pozostały kredyt i czas

`D560` jest wewnętrznym pozostałym kredytem o rozdzielczości 1/100 impulsu 0,10 EUR:

```text
D560 = 100   -> 0,10 EUR
D560 = 1000  -> 1,00 EUR
```

Czas dla ostatnio wybranego programu:

```text
time_left = D560 * PROGRAM_TIME / (D550 * 100)
```

Rejestry HMI:
- D350 — pozostały czas [s],
- D540 — minuty,
- D541 — sekundy,
- D582 — pozostały kredyt w centach,
- M426 — TIME_DISPLAY_VALID,
- M427 — CREDIT_AVAILABLE.

## Synchronizacja z automatyką

Domyślnie `M429=0`: czas/kredyt jest zużywany podczas `WORK_ACTIVE`.

Po potwierdzeniu, że `X1/M301 = PRACA` jest wiarygodnym sygnałem rzeczywistej pracy automatyki, ustaw na HMI:

```text
M429 = 1
```

Wtedy kredyt jest zużywany tylko gdy jednocześnie:
- program jest aktywny,
- M301/PRACA = 1.

`D528` pozostaje globalną korekcją zegara x100:
- 100 = 1,00x,
- 101 = 1,01x,
- 99 = 0,99x.

## HMI wake

Po zaakceptowanym impulsie RM5:
- `M419=1` przez około 3 s — żądanie wybudzenia ekranu,
- `D565` zwiększa się o 1 — licznik zdarzeń monet.

Jeżeli WSStudio potrafi wybudzać/przełączać ekran po bicie PLC, użyj M419. Jeżeli wymaga detekcji zmiany wartości, użyj D565.

## HMI sterowanie

- M400 — STOP, momentary
- M401…M406 — P1…P6, momentary
- M410 — PILOTAGGIO_ENABLE
- M411 — reset liczników diagnostycznych sesji
- M414 — WORK_LIGHTS_ENABLE
- M429 — synchronizacja odliczania z PRACA

Touch control pozostaje lokalną funkcją HMI i nie ma bitu PLC.

## Pliki

- `plc/MAIN_GXDEV_ENTRY.txt` — aktualny program Instruction List,
- `plc/MAIN.txt` — wersja komentowana,
- `plc/main_v1.4.0.csv` — CSV w układzie eksportu GX,
- `plc/DEVICE_MAP.csv` — mapa urządzeń,
- `docs/HMI.md` — konfiguracja HMI,
- `docs/LADDER_LOGIC.md` — opis logiki,
- `CHANGELOG.md` — historia zmian.
