# Logika PLC — V1.3.2

## Zmiana sposobu naliczania

W V1.3.2 czas nie jest już dodawany przy wejściu RM5.

`D300` jest faktycznym Coin multiplier:

```text
CREDIT_COUNT = RM5_PULSES × D300
```

Po końcu paczki:

```text
MUL D320 D300 D542
MOV D542 D330
```

`D330` jest kolejką impulsów `Y0/PITSTART_CREDITS`.

## Dodawanie czasu

Każdy faktycznie wysłany CREDIT kończy fazę ON na `T201`. Dopiero wtedy:

```text
T201 -> INC D533
T201 -> D350 = D350 + D520
```

`D520` to sekundy / CREDIT. Domyślnie 10 s.

Domyślnie `D300=10`, więc:

```text
1 RM5 -> 10 CREDIT -> 10 × 10 s = 100 s
```

## Odliczanie

`D350` jest pozostałym czasem. Odliczanie działa tylko przy `M412=WORK_ACTIVE`.

`D528` jest korekcją x100:
- 100=1.00x,
- 101=1.01x,
- 99=0.99x.

Jeżeli D528 jest poza zakresem 50…200, odliczanie przechodzi awaryjnie na 1 s / tick.

## Sesja

Nowa sesja jest wykrywana tylko na pierwszym zaakceptowanym impulsie RM5, gdy:
- D350<=0,
- D330<=0,
- M321=0,
- M330=0,
- M331=0.

M417 zeruje liczniki sesji przed policzeniem pierwszego impulsu.

## Diagnostyka

- D524 = impulsy RM5 w sesji,
- D533 = CREDIT faktycznie wysłane,
- D534:D535 = skumulowany czas wysłanych CREDIT — rośnie,
- D350 = czas pozostały — maleje,
- D540/D541 = MM/SS czasu pozostałego.

## Oświetlenie

```text
M414 AND M412 -> Y27
```

## Pełny program

- `plc/MAIN_GXDEV_ENTRY.txt`
- `plc/MAIN.txt`
- `plc/main_v1.3.2.csv`
