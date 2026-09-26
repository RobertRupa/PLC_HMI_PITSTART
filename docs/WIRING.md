# Schemat połączeniowy — V1.4.0

## RM5 -> PLC

```text
RM5 pin 2  +12...24 V  -> zasilanie RM5
RM5 pin 1  GND         -> 0 V / COM wejść PLC
RM5 pin 7  CH1         -> X27

PLC Y23                -> RM5 pin 6 INHIBIT
```

## Przyciski

```text
STOP        -> X4
PROGRAM 1   -> X5
PROGRAM 2   -> X6
PROGRAM 3   -> X7
PROGRAM 4   -> X10
PROGRAM 5   -> X11
PROGRAM 6   -> X12
MANUAL/FREE -> X13
RM5 CH1     -> X27
```

## PLC -> sterownik myjni

```text
Y0  -> PITSTART_CREDITS / COUNTER
Y1  -> PILOTAGGIO POMPA
Y2  -> PROGRAM 1
Y3  -> PROGRAM 2
Y4  -> PROGRAM 3
Y5  -> PROGRAM 4
Y6  -> PROGRAM 5
Y7  -> PROGRAM 6
Y27 -> OŚWIETLENIE
```

Y0 w modelu V1.4.0 reprezentuje impuls 0,10 EUR.

## Oryginalny PitStart

CN4:
- P1…P4

CN5:
- P5
- P6
- Pilotaggio
- Counter / Compteur

CN8:
- 1–2 Presence automatic device -> w projekcie X0,
- 3–4 Manual / Free -> w projekcie X13.

## Pilotaggio i oświetlenie

```text
M410 AND M412 -> Y1
M414 AND M412 -> Y27
```

## Uwaga elektryczna

Oryginalny PitStart używa suchych styków NO. Przed podłączeniem SEEKU należy potwierdzić typ wyjść PLC, wspólne COM i wymagania wejść automatyki.
