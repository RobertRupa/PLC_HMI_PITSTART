# Wprowadzenie V1.5.7 do GX Developer bez GX Converter

Docelowa rodzina PLC projektu: **Mitsubishi FX3UC / kompatybilny FX**.

Najprostszą metodą jest import [main_v1.5.7.csv](main_v1.5.7.csv). Jeżeli import CSV nie jest dostępny lub sprawia problemy, można wprowadzić listę instrukcji z `MAIN_GXDEV_ENTRY.txt`.

## Wprowadzanie Instruction List

1. Zrób kopię projektu: **Project -> Save As**.
2. Otwórz `Program -> MAIN`.
3. Przełącz na **Instruction List**.
4. Usuń poprzednią zawartość MAIN lub utwórz pusty MAIN.
5. Wprowadź instrukcje z `MAIN_GXDEV_ENTRY.txt`.
6. Wykonaj **F4 / Convert**.
7. Przełącz do Ladder i sprawdź wynik konwersji.
8. Uruchom **Check program / Diagnostics**.
9. Dopiero po poprawnym sprawdzeniu zapisz program do PLC.

GX Converter nie jest wymagany do ręcznego wprowadzania Instruction List.

## Weryfikacja wersji

V1.5.7 zapisuje:

```text
D516 = 1
D517 = 5
D518 = 7
```

Aktualny program zawiera także ciąg wersji w D500…D503 używany przez HMI.

## Pliki pomocnicze

- [MAIN_GXDEV_ENTRY.txt](MAIN_GXDEV_ENTRY.txt)
- [MAIN.txt](MAIN.txt)
- [main_v1.5.7.csv](main_v1.5.7.csv)
- [IMPORT_CSV.md](IMPORT_CSV.md)
- [DEVICE_MAP.csv](DEVICE_MAP.csv)

## Manual / Free

`X13/M302` pozostaje wyłącznie statusem. Tryb FREE bez kredytu nie jest zaimplementowany.
