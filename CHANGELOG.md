# Changelog

## V1.4.2

- dodano `D559 = TIME_BAR_MAX` dla paska czasu HMI,
- D559 jest zerowane przy nowej sesji i zmianie programu,
- D559 rośnie, gdy doładowanie zwiększa dostępny czas,
- D559 nie maleje podczas odliczania D350,
- wersja PLC/HMI V1.4.2.


## V1.4.1

- Y23/RM5_INHIBIT zależy wyłącznie od X0,
- X0=0 -> Y23=1,
- X0=1 -> Y23=0,
- M413/M415 nadal filtrują programowe przyjęcie impulsu X27, ale nie sterują INHIBIT,
- wersja PLC/HMI V1.4.1.


## V1.4.0

- przebudowano model na kredyt pieniężny zgodny z PitStart,
- Y0/Counter = jednostka 0,10 EUR,
- D300 = wartość RM5 CH1 w jednostkach 0,10 EUR,
- D549 = maksymalny kredyt,
- D550 = wspólna cena bazowa,
- D551…D556 = czas P1…P6 dla ceny bazowej,
- D560 = pozostały kredyt z rozdzielczością 1/100 jednostki Counter,
- D350/D540/D541 są obliczane z kredytu i taryfy wybranego programu,
- zmiana programu zachowuje kredyt i zmienia wynikowy czas,
- dodano M429 do opcjonalnej synchronizacji zużycia kredytu z X1/M301 PRACA,
- D528 pozostaje globalną korekcją zegara x100,
- dodano M419 HMI_WAKE_REQUEST na około 3 s po monecie,
- dodano D565 HMI_WAKE_EVENT_COUNTER,
- dodano D582 = pozostały kredyt w centach,
- wersja PLC/HMI V1.4.0.

## V1.3.2

- D300 działał jako Coin multiplier,
- czas był naliczany podczas wysyłki impulsów Y0.

## V1.3.0

- dodano M414 WORK_LIGHTS_ENABLE,
- dodano pierwszą wersję korekcji zegara i MM:SS.

## V1.2.0

- przygotowano czysty program FX3UC,
- dodano P1…P6, STOP i bezpieczny restart.
