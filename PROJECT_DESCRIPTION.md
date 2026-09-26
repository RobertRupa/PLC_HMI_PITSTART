# Opis projektu — V1.4.0

Projekt emuluje funkcje PitStart dla sterownika myjni.

Najważniejsza zmiana V1.4.0: źródłem prawdy nie jest już sztuczny licznik sekund przypisywany do monety. Źródłem prawdy jest pozostały **kredyt pieniężny**, a czas jest wyliczany z taryfy programu.

## Taryfa

- D300 — wartość RM5 CH1, jednostka 0,10 EUR,
- D549 — maksymalny kredyt, jednostka 0,10 EUR,
- D550 — wspólna cena bazowa, jednostka 0,10 EUR,
- D551…D556 — czasy P1…P6 dla ceny bazowej,
- D528 — globalna korekcja zegara x100.

## Counter

Każdy impuls Y0 reprezentuje 0,10 EUR.

Dla D300=10 jeden impuls RM5 daje 10 impulsów Y0 = 1,00 EUR.

## Dostępny czas

Po wyborze programu PLC oblicza czas z aktualnego kredytu i czasu tego programu. Zmiana programu zmienia przeliczenie czasu, ale zachowuje kredyt pieniężny.

## Synchronizacja

M429 pozwala wybrać:
- odliczanie według WORK_ACTIVE,
- albo według rzeczywistego X1/M301 PRACA.

## HMI wake

M419 jest 3-sekundowym żądaniem wybudzenia po monecie. D565 jest licznikiem zdarzeń monet.
