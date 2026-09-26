# PLC — V1.3.1

## Pliki

- `MAIN_GXDEV_ENTRY.txt` — czysta lista instrukcji
- `MAIN.txt` — wersja komentowana
- `main_v1.3.1.csv` — CSV w układzie zgodnym z dostarczonym eksportem
- `DEVICE_MAP.csv` — mapa urządzeń
- `GX_DEVELOPER_NO_CONVERTER.md` — procedura ręczna bez GX Converter

## Parametry HMI

- M414 WORK_LIGHTS_ENABLE
- D520 nominal seconds / CREDIT pulse
- D528 CLOCK_SCALE_X100
- D540 minutes left
- D541 seconds left

D529 nie jest używany od V1.3.1.

## Korekcja

```text
rate = D528 / 100
```

Przykłady:
- 100 = 1.00x
- 101 = 1.01x
- 99 = 0.99x

D528 jest integerem w PLC; na HMI należy go wyświetlać z 2 miejscami po przecinku.

## Sesja

M417 powstaje przy pierwszym zaakceptowanym impulsie RM5, gdy D350<=0. Wtedy PLC zeruje D320, D521:D526 i D530:D535 przed zliczeniem pierwszego impulsu.

STOP nie resetuje sesji.

## Import

Sam GX Developer bez GX Converter nie ma oficjalnego importu programu z CSV. CSV zachowuje 9-kolumnowy układ dostarczonego eksportu. Alternatywnie użyj `MAIN_GXDEV_ENTRY.txt` w trybie Instruction List.
