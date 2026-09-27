# PLC V1.4.6

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


## Admin screen request

```text
X14 -> M303 = ADMIN_SCREEN_REQUEST
```

M303 jest statusem tylko do odczytu dla HMI. Stan 1 oznacza żądanie ekranu Admin, stan 0 oznacza Home.


## HMI word triggers

Dla HMI bez obsługi bitów M jako triggerów ekranów:

```text
D585 = ADMIN_SCREEN_REQUEST_WORD
       0/1 mirror X14 -> M303

D586 = HMI_WAKE_REQUEST_WORD
       0/1 mirror M419
```

D585=1 oznacza ekran Admin, D585=0 ekran Home.
D586=1 jest aktywne około 3 s po zaakceptowanym impulsie RM5.
