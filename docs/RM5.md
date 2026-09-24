# Comestero RM5 Evolution

## Zastosowanie

Projekt korzysta z jednego kanału RM5.

- RM5 CH1 -> PLC `X27`
- PLC `Y23` -> RM5 `INHIBIT`
- RM5 GND -> wspólne 0 V / COM wejść

## Standardowe złącze 10-pin

| Pin | Funkcja |
|---:|---|
| 1 | GND |
| 2 | +12…24 VDC |
| 3 | CH5 |
| 4 | CH6 |
| 5 | N.U. / zależne od konfiguracji |
| 6 | INHIBIT |
| 7 | CH1 |
| 8 | CH2 |
| 9 | CH3 |
| 10 | CH4 |

## CH1

```text
RM5 pin 7 CH1  -> X27
RM5 pin 1 GND  -> COM/0V PLC
```

Wyjście kanału RM5 jest typu open collector; podczas impulsu linia jest aktywowana do GND.

## INHIBIT

```text
/M300 -> Y23
```

Założenie projektu:
- M300=1 -> RM5 dozwolony,
- M300=0 -> Y23 podaje stan blokady na pin 6 INHIBIT.

## Algorytm paczki impulsów

```text
X27 -> M320
M320 -> INC D320
M320 -> SET M321
M320 -> RST T200

M321 -> T200 K100

T200 -> MUL D320 D300 D350
T200 -> MOV D320 D330
T200 -> MOV K0 D320
T200 -> RST M321
```

## Źródło

RM5 manual:
https://www.casino-software.de/download/manual_rm5.pdf
