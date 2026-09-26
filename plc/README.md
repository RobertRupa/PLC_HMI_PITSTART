# PLC V1.4.0

## Import

- `MAIN_GXDEV_ENTRY.txt` — aktualna lista instrukcji,
- `main_v1.4.0.csv` — CSV w układzie eksportu GX,
- `MAIN.txt` — lista z komentarzami,
- `DEVICE_MAP.csv` — mapa adresów.

## Model

```text
RM5 -> D300 jednostek po 0,10 EUR -> Y0 Counter
Y0 faktycznie wysłany -> +0,10 EUR do wewnętrznego kredytu
kredyt + wybrany program -> obliczony czas
```

## Najważniejsze ustawienia testowe

```text
D300 = 10
D549 = 50
D550 = 5
D551..D556 = 70
D528 = 100
M429 = 0
```

Po potwierdzeniu sygnału X1/PRACA można ustawić M429=1.
