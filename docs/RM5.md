# Comestero RM5 Evolution

## Zastosowanie w projekcie

Aktualnie:
- RM5 CH1 -> PLC X27 jako główne wejście impulsów,
- PLC Y23 -> RM5 INHIBIT,
- opcjonalnie kanały RM5 mogą być obserwowane przez AD0...AD5 po dopasowaniu elektrycznym.

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

## Charakter wyjść

RM5 Evolution ma wyjścia kanałów NPN open-collector. Stan aktywny jest stanem niskim.
Wejście globalnego INHIBIT jest aktywne stanem HIGH.

## Główne wejście cyfrowe

CH1:
- pin 7 -> X27,
- pin 1 GND -> COM/0V wejść PLC.

## Opcjonalne wejścia analogowe AD0...AD5

Można wykorzystać AD jako wolne wejścia detekcji impulsów, ale NIE są one zamiennikiem
wejść cyfrowych bez interfejsu:

- AD0...AD2: w typowej konfiguracji WSB są 0-10V. Można wykorzystać pull-up/interfejs,
  który daje stan wysoki w spoczynku i 0V po zwarciu open-collector RM5.
- AD3...AD5: w typowej konfiguracji WSB są 0-20mA. Wymagają konwersji sygnału lub
  przełączenia kanału na tryb napięciowy, jeśli dana wersja sprzętu to obsługuje.

Odczyty aktualnego programu:
- AD0 -> D10
- AD1 -> D0
- AD2 -> D1
- AD3 -> D2
- AD4 -> D3
- AD5 -> D4

Detekcja impulsu może być zrobiona przez porównanie wartości RD3A z progiem,
a następnie użycie zbocza bitu pomocniczego M340...M345.

## INHIBIT

```text
/M300 ---------------- (Y23)
```

Brak M300 blokuje akceptor.

## Źródła

- https://www.casino-software.de/download/manual_rm5.pdf
- https://eu.suzohapp.com/pdf/manual/OM_RM5_Evolution__EN.pdf
