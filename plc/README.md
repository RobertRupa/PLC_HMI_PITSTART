# PLC V1.4.4

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

## CSV column mapping

See `IMPORT_CSV.md`. Column E of `main_v1.4.0.csv` is mapped as **Note** during import.


## TIME_BAR_MAX

`D559` = maksymalny czas paska HMI dla aktualnej sesji/programu.

- reset przy nowej sesji,
- reset przy zmianie programu,
- aktualizacja w górę gdy D350 wzrośnie po doładowaniu,
- nie maleje podczas normalnego odliczania.


## Auto Start Program

- `M431` — HMI RW, Auto Start Program enable.
- `M432` — RO, Auto Timer active.
- `M433` — RO, final countdown enable.

When M431=ON, countdown waits until the whole Y0/CREDIT queue is finished. If the queue finished before a program was selected, countdown starts after the program is selected.


## Auto Start Program

- `M431` — enable,
- `D584` — program 1…6,
- start następuje po zakończeniu pełnej kolejki Y0/CREDIT,
- ręczny wybór programu ma pierwszeństwo; Auto Start nie nadpisuje już pracującego programu.
