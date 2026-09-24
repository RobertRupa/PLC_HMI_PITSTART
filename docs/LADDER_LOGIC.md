# Aktualna logika drabinki

## Analog

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

## Statusy

```text
X0 -> M300
X1 -> M301
X2 -> M302
X3 -> M303

X0 AND /X1 AND X2 AND X3 -> M304
```

## RM5

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

## Generator PITStart

```text
(D330 > K0) AND /M330 AND /M331 -> SET M330

M330 -> Y1
M330 -> T201 K10

T201 -> DEC D330
T201 -> RST M330
T201 -> SET M331

M331 -> T202 K10
T202 -> RST M331
```

## Odliczanie

Jeżeli funkcją ma być dokładnie -1 co sekundę w standardowym FX, zastosować zbocze `M8013` oraz `D350 > K0`.

## D300 / D200

Na aktualnym zrzucie:

```text
X1 -> INC D200
X1 -> INC D300
```

Jeżeli X1 pozostaje ON dłużej niż jeden skan, zwykłe INC wykona się wielokrotnie. Jeśli D300 ma być ustawiane z HMI, ten fragment należy usunąć albo zastąpić logiką zboczową.
