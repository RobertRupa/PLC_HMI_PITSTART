# PLC — FX3UC / GX Developer — V1.2.0

## Pliki

- `MAIN_GXDEV_ENTRY.txt` — czysta lista instrukcji do wprowadzenia w GX Developer.
- `MAIN.txt` — ta sama logika z komentarzami.
- `DEVICE_MAP.csv` — mapa X/Y/M/D/T.
- `GX_DEVELOPER_NO_CONVERTER.md` — procedura bez GX Converter.

## GX Converter

Nie jest wymagany.

W GX Developer:
1. otwórz kopię projektu,
2. Program -> MAIN,
3. Alt+F1 / View -> Instruction List,
4. usuń testowy MAIN,
5. wprowadź `MAIN_GXDEV_ENTRY.txt`,
6. F4 / Convert,
7. wróć do Ladder,
8. wykonaj Check program,
9. dopiero wtedy zapisz do PLC.

## Najważniejsze cechy V1.2.0

- FX3UC,
- X4 STOP,
- X5…X12 P1…P6,
- X13 MANUAL/FREE status,
- X27 RM5 CH1,
- Y2…Y7 P1…P6,
- M451…M456 jednocyklowe impulsy wyboru,
- utrata X0 kasuje aktywny program,
- D350 odlicza się tylko przy M412=1,
- STOP pauzuje czas,
- M410 startuje jako ON,
- D520 jest parametrem HMI 1…600 i nie jest nadpisywane, jeśli przy starcie ma poprawną wartość,
- M411 resetuje licznik sesji i sam się zeruje,
- wersja dla HMI: V1.2.0.

## Uwaga o D520

Zachowanie poprawnej wartości D520 po zaniku zasilania wymaga skonfigurowanej retencji/latch w FX3UC albo ponownego zapisu parametru przez HMI.

## Manual / Free

W V1.2.0 X13/M302 jest statusem tylko do odczytu. Funkcja FREE bez kredytu nie jest aktywna.
