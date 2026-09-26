# PLC — V1.3.0

## Pliki

- `MAIN_GXDEV_ENTRY.txt` — czysta lista instrukcji
- `MAIN.txt` — wersja komentowana
- `main_v1.3.0.csv` — CSV w układzie zgodnym z dostarczonym eksportem
- `DEVICE_MAP.csv` — mapa urządzeń
- `GX_DEVELOPER_NO_CONVERTER.md` — procedura ręczna bez GX Converter

## Nowe parametry HMI

- M414 WORK_LIGHTS_ENABLE
- D520 nominal seconds / CREDIT pulse
- D528 clock multiplier
- D529 clock divider
- D540 minutes left
- D541 seconds left

## Korekcja

```text
rate = D528 / D529
```

Domyślnie 100/100.

## Import

Sam GX Developer bez GX Converter nie ma oficjalnego importu programu z CSV. Plik CSV jest zachowany w tym samym 9-kolumnowym formacie co aktualny eksport użytkownika i może być używany w środowisku obsługującym odczyt list-format CSV. Alternatywnie użyj `MAIN_GXDEV_ENTRY.txt` w trybie Instruction List.
