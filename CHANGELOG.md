# Changelog

## V1.2.0

- przygotowano czysty program docelowy dla FX3UC zamiast wcześniejszych testów,
- dodano `MAIN_GXDEV_ENTRY.txt` do wprowadzania bez GX Converter,
- dodano M451…M456 jako jednocyklowe impulsy wyboru P1…P6,
- utrata X0/AUTOMATE_PRESENT kasuje aktywny program,
- wyjścia P1…P6 są blokowane przez M300,
- D350 odlicza się tylko podczas WORK_ACTIVE; STOP pauzuje czas,
- M410/PILOTAGGIO_ENABLE startuje jako ON,
- D520 jest zachowywane, jeśli podczas startu ma wartość 1…600; w przeciwnym razie ustawiane jest 10,
- M411 resetuje licznik sesji i sam się zeruje,
- RM5 jest liczony tylko przy M300=1 i M413=1,
- X13/M302 pozostaje statusem Manual / Free,
- wersja HMI/PLC ustawiona na V1.2.0.
