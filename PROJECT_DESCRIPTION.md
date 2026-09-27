# Opis projektu — V1.5.3

Projekt zastępuje funkcje Comestero PitStart w sterowniku myjni i współpracuje z HMI oraz akceptorem monet RM5 Evolution.

## Model kredytu

Źródłem prawdy jest pozostały kredyt pieniężny.

- Y0 / PITSTART_CREDITS = impulsy po 0,10 EUR,
- D300 = wartość jednego impulsu wejściowego RM5 CH1 w jednostkach 0,10 EUR,
- D550 = wspólna cena bazowa,
- D551…D556 = czasy P1…P6 dla ceny bazowej,
- D560 = pozostały kredyt w centach,
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
X0=0 lub M435=1 -> Y2=1 oraz Y23=1
X0=1 i M435=0 -> Y2=0 oraz Y23=0
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

Auto Start czeka na całkowite zakończenie kolejki Y0/CREDIT i wymaga D558=0. Jeżeli program jest już wybrany, kolejne doładowanie zwiększa kredyt i czas bez zmiany programu.

Paczki RM5 są dopisywane do D330. M436 pozostaje aktywne do zakończenia obsługi pakietu i kolejki Y0. Podczas M436=1 zużycie kredytu jest wstrzymane.

## Synchronizacja czasu

M429 wybiera synchronizację z PRACA. Przy M429=1 X1/M301 jest wymagane tylko gdy M410/Pilotaggio jest włączone. Przy M410=0 odliczanie trwa niezależnie od X1.

## HMI

- D559 = TIME_BAR_MAX,
- D582 = pozostały kredyt w centach,
- D585 = HMI_SCREEN_INDEX: 0 Main, 1 Admin, 2 Nieczynne, 3 Wolne,
- D586 = aktualny indeks ekranu zapisywany przez HMI,
- D587 = HMI_WAKE_REQUEST_WORD, mirror M419,
- D565 = licznik zaakceptowanych zdarzeń RM5.

X14 steruje M303. D585 wybiera ekran według priorytetu:
- 1 Admin, gdy X14=1,
- 2 Stanowisko nieczynne, gdy X14=0 i X0=0,
- 3 Stanowisko wolne, gdy X14=0, X0=1 i M427=0,
- 0 Main/Work w pozostałym przypadku.

Po zaakceptowanym impulsie RM5 M419 jest aktywne około 3 s, a D565 zwiększa licznik zdarzeń.


## Domyślne przełączniki

M429 i M431 startują domyślnie w stanie ON:
- M429 = Sync countdown with PRACA,
- M431 = Auto Start Program.


## Limit kredytu

D549: 1…999 jednostek po 0,10 EUR. M435 blokuje RM5, gdy kolejny pełny impuls RM5 nie mieści się już w wolnym limicie albo bieżąca paczka zarezerwowała całe dostępne miejsce.


## Restart

Po wejściu PLC w RUN T204 tworzy okno resetu startowego. Gdy M438=1 zerowane są stan sesji, kredyt, czas, kolejka Y0, liczniki diagnostyczne i rejestry robocze D588…D594.


## Pilotaggio default

`M439` przechowuje domyślny stan Pilotaggio i jest dostępne z HMI.

Przy starcie programu PLC sprawdza M410. Jeżeli M410=0, a M439=1, ustawia M410=1. M439 domyślnie startuje w stanie ON.

## Domyślna taryfa

```text
D300=10
D550=10
D551..D556=300
```

Jeden impuls RM5 daje wtedy 5 minut dla P1…P6.
