# Start i obsługa — V1.3.2

## Start PLC

Na pierwszym skanie:
- kolejka CREDIT D330 = 0,
- pozostały czas D350 = 0,
- liczniki sesji = 0,
- programy P1…P6 = OFF,
- M410 Pilotaggio enable = ON,
- M414 Work lights enable = ON.

Parametry HMI są korygowane do wartości domyślnych tylko gdy są poza zakresem:
- D300: 1…100, domyślnie 10,
- D520: 1…600, domyślnie 10,
- D528: 50…200, domyślnie 100.

## Nowa sesja

M417 powstaje przy pierwszym zaakceptowanym impulsie RM5, gdy:
- nie ma aktywnej paczki,
- D350<=0,
- D330<=0,
- generator CREDIT jest bezczynny.

M417 zeruje liczniki sesji przed policzeniem pierwszego impulsu.

## STOP

STOP resetuje aktywny program, ale:
- nie kasuje D350,
- nie kasuje D330,
- nie resetuje liczników sesji.

Po STOP czas jest pauzowany, ponieważ D350 odlicza tylko gdy M412=WORK_ACTIVE.

## Programy

HMI:
- M400 STOP,
- M401…M406 P1…P6.

Fizyczne:
- X4 STOP,
- X5…X12 P1…P6.

Stany aktywnych programów:
- M420…M425.
