# Aktualna / planowana logika drabinki

## Analog

Oryginalny odczyt pozostaje:

```text
M8011 -> RD3A K0 K0 D10
M8011 -> RD3A K0 K1 D0
M8011 -> RD3A K0 K2 D1
M8011 -> RD3A K0 K3 D2
M8011 -> RD3A K0 K4 D3
M8011 -> RD3A K0 K5 D4

M8011 -> WR3A K0 K0 D6
M8011 -> WR3A K0 K1 D7
```

## RM5 - główne wejście X27

```text
X27 -> M320

/M300 -> Y23

M320 -> INC D320
M320 -> SET M321
M320 -> RST T200

M321 -> T200 K100

T200 -> MUL D320 D300 D350
T200 -> MOV D320 D330
T200 -> MOV K0 D320
T200 -> RST M321
```

## CREDIT / COMPTEUR - Y0

Generator kredytu został przeniesiony z Y1 na Y0:

```text
(D330 > K0) AND /M330 AND /M331 -> SET M330

M330 -> Y0
M330 -> T201 K10

T201 -> DEC D330
T201 -> RST M330
T201 -> SET M331

M331 -> T202 K10
T202 -> RST M331
```

## Pilotaggio pompa - Y1

Proponowane bity:
- M410 = PILOTAGGIO_ENABLE
- M412 = WORK_ACTIVE

WORK_ACTIVE powinno zostać ustawione po wybraniu dowolnego programu P1...P6
lub podczas aktywnego trybu MANUAL/FREE, a skasowane przez STOP albo po zakończeniu pracy.

```text
M410 AND M412 -> Y1
```

Dzięki temu Pilotaggio można wyłączyć z HMI bez wpływu na programy i oświetlenie.

## Programy

```text
Y2 = Program 1
Y3 = Program 2
Y4 = Program 3
Y5 = Program 4
Y6 = Program 5
Y7 = Program 6
```

UWAGA: Y4 występuje w starym bloku analogowym D6/D7. Przed użyciem Y4 jako Program 3
należy usunąć albo zmienić stare rungi zależne od Y004.

## Oświetlenie - Y27

```text
M412 ---------------- (Y27)
```

Oświetlenie jest aktywne zawsze, gdy WORK_ACTIVE=1, niezależnie od opcji Pilotaggio.

## Opcjonalne kanały RM5 na AD0...AD5

RM5 ma wyjścia NPN open-collector. AD są wejściami analogowymi, dlatego wymagają
dopasowania poziomu.

Dla kanału napięciowego 0-10V można utworzyć bit logiczny na podstawie progu.
Przykład, gdy stan spoczynkowy jest wysoki, a impuls RM5 ściąga wejście do 0V:

```text
LD< D10 D360
OUT M340

LDP M340
; tutaj obsługa jednego impulsu z AD0
```

Analogicznie:
- AD0/D10 -> M340
- AD1/D0  -> M341
- AD2/D1  -> M342
- AD3/D2  -> M343
- AD4/D3  -> M344
- AD5/D4  -> M345

D360 jest przykładowym progiem. Dla zakresu 0-10V i wartości 0...4095
próg należy dobrać do rzeczywistego napięcia spoczynkowego i aktywnego.

AD3...AD5 w standardowej konfiguracji WSB mogą być wejściami prądowymi 0-20mA,
więc open-collector RM5 nie powinien być do nich podłączany bezpośrednio.

## Odliczanie

Jeżeli D350 ma maleć dokładnie o 1 na sekundę:

```text
LDP M8013
AND> D350 K0
DEC D350
```
