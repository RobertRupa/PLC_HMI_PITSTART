# Comestero RM5 Evolution

## Zastosowanie w projekcie

Aktualnie:
- RM5 `CH1` -> PLC `X27`,
- PLC `Y23` -> RM5 `INHIBIT`.

## Standardowe złącze 10-pin

| Pin | Funkcja |
|---:|---|
| 1 | GND |
| 2 | +12…24 VDC |
| 3 | CH5 |
| 4 | CH6 |
| 5 | N.U. / zależne od trybu |
| 6 | INHIBIT |
| 7 | CH1 |
| 8 | CH2 |
| 9 | CH3 |
| 10 | CH4 |

## Wyjścia kanałów

Kanały RM5 są typowo wyjściami open collector. Aktywny impuls zwiera linię kanału do GND.

CH1:
- pin 7 -> X27,
- pin 1 GND -> 0 V / COM wejść PLC.

## INHIBIT

Pin 6 jest globalnym wejściem inhibit:
- HIGH blokuje akceptor.

W projekcie:
- COM grupy Y23 jest zasilony napięciem zgodnym z INHIBIT,
- Y23 podaje to napięcie na pin 6.

Logika:

```text
/M300 ---------------- (Y23)
```

## Multipulse

RM5 może występować w wielu konfiguracjach. W trybie RM5 X 0M Multipulse Validator wartość monety może być przekazywana jako wielokrotne impulsy.

Obecna drabinka:
- zlicza impulsy,
- kończy paczkę po ok. 1 s bezczynności,
- przelicza wynik przez mnożnik.

## Źródła

- https://www.casino-software.de/download/manual_rm5.pdf
- https://eu.suzohapp.com/pdf/manual/OM_RM5_Evolution__EN.pdf
