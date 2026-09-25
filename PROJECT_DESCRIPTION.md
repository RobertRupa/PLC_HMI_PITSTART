# Opis projektu PLC_HMI_PITSTART

## Wersja

Aktualna logika PLC: **V1.2.0**, docelowo dla **FX3UC** i GX Developer.

## Cel

Sterownik SEEKU PLC/HMI realizuje:
- odbiór impulsów RM5,
- blokowanie RM5 przy braku zezwolenia lub błędnym parametrze czasu,
- generowanie CREDIT / COMPTEUR,
- konwersję impulsów RM5 na czas,
- sterowanie P1…P6,
- obsługę HMI i przycisków mechanicznych równolegle,
- Pilotaggio i oświetlenie,
- bezpieczny restart bez wznowienia starej kolejki CREDIT.

## Wejścia

- X0 = AUTOMATE_PRESENT / INVERTER_OK
- X1 = PRACA — status
- X4 = STOP
- X5 = P1
- X6 = P2
- X7 = P3
- X10 = P4
- X11 = P5
- X12 = P6
- X13 = MANUAL / FREE — w V1.2.0 tylko status
- X27 = RM5 CH1

## Wyjścia

- Y0 = CREDIT / COMPTEUR
- Y1 = PILOTAGGIO POMPA
- Y2…Y7 = P1…P6
- Y23 = RM5 INHIBIT
- Y27 = oświetlenie

## HMI + przyciski

HMI:
- M400 = STOP
- M401…M406 = P1…P6
- M410 = PILOTAGGIO_ENABLE
- M411 = RESET_RM5_SESSION_COUNTERS

Komendy wspólne:
- M440 = X4 OR M400
- M441 = X5 OR M401
- M442 = X6 OR M402
- M443 = X7 OR M403
- M444 = X10 OR M404
- M445 = X11 OR M405
- M446 = X12 OR M406

Jednocyklowe impulsy wyboru:
- M451…M456 = P1…P6 SELECT PULSE

Aktywne programy:
- M420…M425 = P1…P6 ACTIVE

## Warunki programu

Program może zostać wybrany tylko gdy:
- M300=1,
- STOP nieaktywny,
- D350>0.

Wybór nowego programu kasuje pozostałe M420…M425.

STOP oraz utrata M300 kasują aktywny program. Po D350<=0 wybór programu także jest kasowany.

Wyjścia P1…P6 są dodatkowo warunkowane M300 i D350>0.

## RM5

- X27 = CH1
- Y23 = INHIBIT
- M320 = jednocyklowy impuls roboczy
- M321 = aktywna paczka
- T200 = timeout końca paczki

Impuls RM5 jest przyjmowany tylko przy M300=1 i M413=1.

M413=1 gdy:

```text
D520 > 0 AND D520 <= 600
```

Y23 jest aktywne gdy:

```text
/M300 OR /M413
```

## Konwersja impulsów na czas

- D520 = sekundy / impuls, HMI RW, 1…600
- D350 = pozostały czas
- D524 = licznik impulsów sesji
- D525:D526 = równoważny czas sesji
- D521 = ostatnia paczka impulsów
- D522:D523 = czas ostatniej paczki

Każdy zaakceptowany impuls dodaje D520 do D350. D350 jest saturacyjnie ograniczone do 32000 s.

D520 jest ustawiane na 10 tylko wtedy, gdy podczas pierwszego skanu ma wartość spoza 1…600. Zachowanie poprawnej wartości po zaniku zasilania wymaga odpowiedniej retencji/latch pamięci lub ponownego zapisu z HMI.

## CREDIT

D330 jest kolejką impulsów Y0.

T201 K10 = około 0,1 s ON.
T202 K10 = około 0,1 s OFF.

D330 jest bezwarunkowo zerowane w pierwszym skanie, aby nie wznowić starej kolejki po restarcie.

## Czas i STOP

D350 jest odliczane tylko podczas M412=WORK_ACTIVE:

```text
LDP M8013
AND M412
AND> D350 K0
DEC D350
```

STOP zatrzymuje program, ale zachowuje pozostały czas.

## Pilotaggio i oświetlenie

```text
M410 AND M412 -> Y1
M412          -> Y27
```

M410 jest ustawiane na ON w pierwszym skanie, aby funkcja działała bez podłączonego HMI.

## Manual / Free

```text
X13 -> M302
```

W V1.2.0 M302 jest statusem tylko do odczytu. Pełne działanie FREE bez kredytu nie jest jeszcze aktywne.

## Wersja dla HMI

```text
D500..D503 = "V1.2.0"
D516 = 1
D517 = 2
D518 = 0
```
