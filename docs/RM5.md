# Comestero RM5 Evolution — V1.3.0

## Połączenie

- pin 7 CH1 -> X27
- pin 6 INHIBIT <- Y23
- pin 1 GND -> 0 V / COM
- pin 2 -> +12…24 VDC

## Akceptacja impulsu

```text
LDP X27
AND M300
AND M413
AND M415
OUT M320
```

Impuls jest przyjmowany tylko gdy:
- automat jest dostępny,
- D520 jest poprawne,
- mnożnik/dzielnik zegara są poprawne.

## INHIBIT

```text
/M300 OR /M413 OR /M415 -> Y23
```

## Nominalny czas

D520 = nominalne sekundy automatyki na jeden impuls CREDIT.

Każdy zaakceptowany impuls zwiększa D350 o D520, a po zakończeniu paczki ta sama liczba impulsów trafia do kolejki D330 i jest wysyłana na Y0.

## Diagnostyka

- D524 = impulsy zaakceptowane z RM5
- D533 = impulsy CREDIT faktycznie wysłane na Y0
- D525:D526 = nominalny czas zaakceptowanych impulsów
- D534:D535 = nominalny czas wysłanych impulsów

Różnica D524-D533 pozwala zobaczyć, czy w kolejce CREDIT pozostały jeszcze impulsy.
