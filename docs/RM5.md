# Comestero RM5 Evolution — V1.2.0

## Połączenie

- RM5 pin 7 CH1 -> PLC X27
- RM5 pin 6 INHIBIT <- PLC Y23
- RM5 pin 1 GND -> wspólne 0 V / COM wejść
- RM5 pin 2 -> +12…24 VDC

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

Wyjście CH1 jest open collector / aktywne do GND.

## Akceptacja impulsu

```text
LDP X27
AND M300
AND M413
OUT M320
```

M300 = automat dostępny.
M413 = D520 w zakresie 1…600.

## INHIBIT

```text
/M300 OR /M413 -> Y23
```

RM5 jest blokowany, gdy automat nie jest dostępny lub parametr czasu jest niepoprawny.

## Liczenie

Każdy M320:
- zwiększa D320,
- zwiększa D524 do 30000,
- dodaje D520 sekund do D350,
- ogranicza D350 do 32000 s,
- ustawia M321 i restartuje T200.

## Koniec paczki

Po około 1 s bez kolejnego impulsu:

```text
D521 = D320
D522:D523 = D320 * D520
D330 = D320
D320 = 0
M321 = 0
```

D330 steruje generatorem CREDIT Y0.

## Parametry

- D300 = 10 — wartość kanału 1 RM5 / pole konfiguracyjne
- D520 = sekundy za impuls, 1…600
- D350 = pozostały czas
- D524 = licznik impulsów sesji
- D525:D526 = równoważny czas sesji

Aktualna logika czasu używa D520; D300 nie jest mnożnikiem czasu.

## Restart

D330 jest zerowane na pierwszym skanie, aby nie wysyłać ponownie starych impulsów CREDIT.

D520 jest ustawiane na 10 tylko wtedy, gdy wartość startowa jest poza 1…600.
