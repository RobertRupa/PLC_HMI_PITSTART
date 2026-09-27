# GX Developer CSV import — V1.5.7

Aktualny plik do importu:

```text
main_v1.5.7.csv
```

Plik został wygenerowany z `MAIN_GXDEV_ENTRY.txt` i zweryfikowany instrukcja po instrukcji.

## Mapowanie kolumn

| CSV | GX Developer |
|---|---|
| A | Step number |
| B | Do not Import / Skip |
| C | Instruction |
| D | I/O(Device) |
| E | Do not Import / Skip |
| F | Do not Import / Skip |
| G | Do not Import / Skip |
| H | Do not Import / Skip |
| I | Do not Import / Skip |

Komentarze urządzeń są przechowywane osobno:
- `DEVICE_COMMENTS.csv`,
- `DEVICE_COMMENTS.txt`.

Starsze pliki `main_v*.csv` pozostają historią kolejnych wersji. Do nowego projektu używaj V1.5.7.
