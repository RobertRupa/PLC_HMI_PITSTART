# PLC_HMI_PITSTART

Aktualna wersja PLC: **V1.3.0**.

## Najważniejsze adresy

### Wejścia
- X0 AUTOMATE_PRESENT / INVERTER_OK
- X1 PRACA
- X4 STOP
- X5…X12 Program 1…6
- X13 MANUAL/FREE status
- X27 RM5 CH1

### Wyjścia
- Y0 CREDIT / COMPTEUR
- Y1 PILOTAGGIO
- Y2…Y7 Program 1…6
- Y23 RM5 INHIBIT
- Y27 WORK LIGHTS

### HMI
- M400 STOP
- M401…M406 P1…P6
- M410 PILOTAGGIO_ENABLE
- M411 RESET_SESSION_COUNTERS
- M414 WORK_LIGHTS_ENABLE
- M413 BASE_TIME_OK
- M415 CLOCK_RATIO_OK

Touch control jest wyłącznie funkcją HMI.

## Czas

- D350 = pozostały czas [s]
- D520 = nominalne sekundy / impuls CREDIT
- D528 = mnożnik zegara
- D529 = dzielnik zegara
- D540 = minuty
- D541 = sekundy

Domyślnie D528/D529 = 100/100.

## Diagnostyka impulsów

- D524 = impulsy RM5 przyjęte
- D533 = impulsy CREDIT faktycznie wysłane
- D525:D526 = nominalny czas przyjętych impulsów
- D534:D535 = nominalny czas wysłanych impulsów

## Pliki PLC

- `plc/MAIN_GXDEV_ENTRY.txt` — instruction list
- `plc/MAIN.txt` — wersja komentowana
- `plc/main_v1.3.0.csv` — CSV w takim samym 9-kolumnowym układzie jak aktualny eksport użytkownika
- `plc/DEVICE_MAP.csv` — mapa urządzeń
