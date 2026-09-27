# Schemat połączeniowy — V1.5.7

## RM5 Evolution -> PLC

CN5 RM5:

```text
pin 1  GND          -> 0 V / COM
pin 2  +12...24 VDC -> zasilanie RM5
pin 6  INHIBIT      <- PLC Y2
pin 7  CH1          -> PLC X27
```

`Y23` ma w programie tę samą logikę INHIBIT co Y2, ale na obecnym sprzęcie jako fizyczne wyjście używany jest **Y2**.

Pełny pinout: [RM5.md](RM5.md).

## Wejścia sterujące

```text
AUTOMATE_PRESENT / INVERTER_OK -> X0
PRACA / RUN                    -> X1
STOP                           -> X4
PROGRAM 1                      -> X5
PROGRAM 2                      -> X6
PROGRAM 3                      -> X7
PROGRAM 4                      -> X10
PROGRAM 5                      -> X11
PROGRAM 6                      -> X12
MANUAL / FREE                  -> X13
ADMIN                          -> X14
RM5 CH1                        -> X27
```

FX numeruje wejścia i wyjścia ósemkowo.

## PLC -> sterownik myjni

```text
Y0  -> PITSTART_CREDITS / COUNTER
Y1  -> PILOTAGGIO POMPA
Y3  -> PROGRAM 1
Y4  -> PROGRAM 2
Y5  -> PROGRAM 3
Y6  -> PROGRAM 4
Y7  -> PROGRAM 5
Y10 -> PROGRAM 6
Y27 -> OŚWIETLENIE
```

Dodatkowo:

```text
Y2  -> RM5 INHIBIT
Y23 -> logiczny mirror RM5 INHIBIT
```

Jeden impuls Y0 reprezentuje 0,10 EUR.

## HMI <-> PLC

Aktualny projekt HMI używa sterownika `Mitsubishi_Fx1n`.

Manual WSB V1.79 dla WSB7020R podaje:

```text
Protocol: Mitsubishi FX1N
Baud:     38400
Mode:     RS232
```

Po downloadzie projektu HMI kabel programujący należy odłączyć. Manual WSB wskazuje, że przy podłączonym kablu downloadowym komunikacja HMI <-> PLC nie działa.

Lokalny manual: [manuals/hmi/WSB_HMI_PLC_All_in_one_User_Manual_V1.79.pdf](manuals/hmi/WSB_HMI_PLC_All_in_one_User_Manual_V1.79.pdf).

## Oryginalny PitStart

Oryginalny PitStart udostępnia wyjścia Counter, Pilotaggio i P1…P6. W aktualnym projekcie ich funkcje są odwzorowane na Y0, Y1 oraz Y3…Y10.

## Uwaga elektryczna

Przed podłączeniem należy potwierdzić typ wyjść SEEKU/PLC, wspólne COM, polaryzację i wymagania wejść automatyki. Dokumentacja logiczna nie zastępuje pomiaru elektrycznego na konkretnym sterowniku.
