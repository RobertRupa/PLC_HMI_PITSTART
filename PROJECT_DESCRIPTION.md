# Opis projektu — V1.4.7

Projekt zastępuje funkcje Comestero PitStart w sterowniku myjni i współpracuje z HMI oraz akceptorem monet RM5 Evolution.

## Model kredytu

Źródłem prawdy jest pozostały kredyt pieniężny.

- Y0 / PITSTART_CREDITS = impulsy po 0,10 EUR,
- D300 = wartość jednego impulsu wejściowego RM5 CH1 w jednostkach 0,10 EUR,
- D550 = wspólna cena bazowa,
- D551…D556 = czasy P1…P6 dla ceny bazowej,
- D560 = pozostały kredyt wewnętrzny,
- D350 / D540 / D541 = pozostały czas.

## Wejścia

```text
X0   AUTOMATE_PRESENT / INVERTER_OK
X1   PRACA
X4   STOP
X5   P1
X6   P2
X7   P3
X10  P4
X11  P5
X12  P6
X13  MANUAL / FREE status
X14  ADMIN switch
X27  RM5 CH1
```

## Wyjścia

```text
Y0   PITSTART_CREDITS / Counter
Y1   PILOTAGGIO
Y2   RM5 INHIBIT PRIMARY
Y3   Program 1
Y4   Program 2
Y5   Program 3
Y6   Program 4
Y7   Program 5
Y10  Program 6
Y23  RM5 INHIBIT compatibility mirror
Y27  WORK LIGHTS
```

FX używa ósemkowej numeracji X/Y: po Y7 jest Y10. Adres Y8 nie istnieje.

Y2 jest aktualnym fizycznym wyjściem INHIBIT. Y23 wykonuje tę samą funkcję logiczną i pozostaje w projekcie na wypadek użycia sterownika z większą liczbą fizycznych wyjść.

## RM5 Evolution

RM5 jest podłączony przez CH1 do X27. Wszystkie używane kanały/nominały muszą być skonfigurowane w RM5 tak, aby generować impulsy na wyjściu CH1.

PLC nie czyta CH2…CH6.

Jeżeli różne monety mają różne wartości, wartość powinna być zakodowana liczbą impulsów CH1. Każdy impuls X27 jest liczony jednakowo i mnożony przez D300.

INHIBIT:

```text
X0=0 -> Y2=1 oraz Y23=1
X0=1 -> Y2=0 oraz Y23=0
```

## Programy

Program może być wybrany z:
- fizycznych przycisków X5…X12,
- HMI M401…M406,
- Auto Start Program.

Aktywne stany:
- M420…M425 = P1…P6.

Zmiana programu zachowuje kredyt i przelicza pozostały czas według taryfy wybranego programu.

## Auto Start Program

- M431 = enable,
- D584 = numer programu 1…6,
- M432 = jednocyklowy trigger,
- M433 = wykonano/anulowano Auto Start,
- M434 = poprawny numer programu.

Auto Start czeka na całkowite zakończenie kolejki Y0/CREDIT. Jeżeli wcześniej wybrano program ręcznie, Auto Start go nie nadpisuje.

## Synchronizacja czasu

M429 wybiera sposób zużycia kredytu:
- M429=0 — podczas WORK_ACTIVE,
- M429=1 — dodatkowo wymagane X1/M301 PRACA.

## HMI

- D559 = TIME_BAR_MAX,
- D582 = pozostały kredyt w centach,
- D585 = ADMIN_SCREEN_REQUEST_WORD, mirror X14/M303,
- D586 = HMI_WAKE_REQUEST_WORD, mirror M419,
- D565 = licznik zaakceptowanych zdarzeń RM5.

X14 steruje M303 i D585 dla ekranu Admin.

Po zaakceptowanym impulsie RM5 M419/D586 są aktywne około 3 s.
