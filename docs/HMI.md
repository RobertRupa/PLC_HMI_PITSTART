# HMI — WSStudio — V1.3.0

## Ekran główny

Sterowanie:
- M400 — STOP, momentary
- M401…M406 — Program 1…6, momentary

Wskaźniki:
- M420…M425 — aktywny Program 1…6
- M412 — WORK_ACTIVE
- M301 — fizyczny status PRACA
- M300 — automat dostępny
- Y23 — RM5 INHIBIT

Czas:
- D350 — pozostały czas [s]
- D540 — pozostałe minuty
- D541 — pozostałe sekundy

Do wyświetlenia MM:SS użyj dwóch pól D540 i D541 z separatorem ':'.

## Ekran Admin

| Adres | Funkcja | Dostęp |
|---|---|---|
| D520 | Nominalny czas automatyki / 1 impuls CREDIT [s] | RW |
| D528 | Mnożnik zegara | RW |
| D529 | Dzielnik zegara | RW |
| M413 | D520 poprawne | RO |
| M415 | Mnożnik/dzielnik poprawny | RO |
| M410 | Pilotaggio enable | RW |
| M414 | Work lights enable | RW |
| M411 | Reset liczników sesji | momentary |
| D524 | zaakceptowane impulsy RM5 | RO |
| D533 | wysłane impulsy CREDIT | RO |
| D525:D526 | nominalny czas zaakceptowanych impulsów | RO |
| D534:D535 | nominalny czas wysłanych impulsów | RO |

Touch control pozostaje funkcją wyłącznie HMI i nie używa bitu PLC.

## Kalibracja czasu

Domyślnie:
- D528=100
- D529=100

Efektywne tempo odliczania:

```text
tempo = D528 / D529
```

Przykłady:
- 100/100 = 1.000x
- 105/100 = zegar HMI odlicza 5% szybciej
- 95/100 = zegar HMI odlicza 5% wolniej

Jeżeli automat kończy kredyt szybciej niż licznik HMI, zwiększ D528 względem D529.
Jeżeli automat kończy kredyt wolniej, zmniejsz D528 względem D529.

Zakres obu parametrów: 1…1000.

Nie zaleca się zmiany D528/D529 podczas aktywnego programu.

## Work lights

M414 jest zezwoleniem. Fizyczne Y27 działa tylko podczas pracy:

```text
M414 AND M412 -> Y27
```

## Wersja PLC

D500..D503 = V1.3.0
D516=1
D517=3
D518=0
