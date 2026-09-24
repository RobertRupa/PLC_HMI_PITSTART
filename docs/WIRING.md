# Schemat połączeniowy

## RM5 -> PLC

```text
                 ZASILACZ 24 VDC
               +24V          0V
                 |            |
                 |            +-----------------------------+
                 |                                          |
                 v                                          v
          +-------------+                            +---------------+
          | RM5         |                            | PLC SEEKU     |
pin 2 ----| +12..24 V   |                            |               |
pin 1 ----| GND         |----------------------------| INPUT COM/0V  |
pin 7 ----| CH1         |----------------------------| X27           |
          | open coll.  |                            |               |
          +-------------+                            +---------------+
```

RM5 podczas impulsu zwiera CH1 do GND.

## Sterowanie INHIBIT

```text
          +24 V
            |
            +---------------------+
                                  |
                          PLC OUTPUT COM
                                  |
                              [ relay ]
                                  |
                                Y23
                                  |
                                  +---------- RM5 pin 6 INHIBIT

RM5 pin 1 GND ------------------------------ 0 V
```

- `Y23 = 1` -> HIGH na INHIBIT -> RM5 zablokowany.
- `Y23 = 0` -> RM5 aktywny, o ile INHIBIT pozostaje LOW.

Aktualna logika: `/M300 -> Y23`.

## PLC -> PITStart

Jeżeli Y1 jest przekaźnikiem, traktować je jako styk bezpotencjałowy:

```text
PITStart PULSE/COIN IN  -------- Y1
PITStart PULSE COMMON   -------- COM grupy Y1
```

Nie podawać 24 V na wejście PITStart bez potwierdzenia parametrów wejścia.

Aktualny timing:
- ON: `T201 K10` ≈ 100 ms,
- OFF: `T202 K10` ≈ 100 ms,
- okres: ≈ 200 ms.

## Sygnały sterownika myjni

```text
Sterownik myjni                     PLC
----------------------------------------
INVERTER_OK / status 1  ---------- X0
PRACA / status 2        ---------- X1
status 3                ---------- X2
status 4                ---------- X3
wspólny sygnałów NPN    ---------- COM/0V
```
