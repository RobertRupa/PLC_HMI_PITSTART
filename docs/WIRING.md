# Schemat połączeniowy

## RM5 -> PLC X27

```text
RM5 pin 2 +12...24 V  -> zasilanie +
RM5 pin 1 GND         -> 0 V / COM wejść PLC
RM5 pin 7 CH1         -> X27
```

RM5 podczas impulsu zwiera wyjście kanału do GND.

## RM5 -> PLC AD0...AD5 (opcjonalnie)

Nie łączyć kanału open-collector RM5 bezpośrednio z wejściem analogowym bez
dopasowania.

Dla kanału 0-10V koncepcja jest następująca:

```text
zewnętrzne napięcie <=10V
          |
       pull-up / interfejs
          |
          +---------------- ADx
          |
RM5 CHx --+   (open collector do GND)
RM5 GND ------------------ AGND/0V
```

W spoczynku ADx ma wartość wysoką, a podczas impulsu RM5 jest ściągane do 0V.
Próg jest wykrywany programowo.

Dla AD3...AD5, jeśli kanały są skonfigurowane jako 0-20mA, potrzebny jest
interfejs prądowy lub zmiana konfiguracji kanału.

## RM5 INHIBIT

```text
+24 V -> COM grupy Y23
Y23   -> RM5 pin 6 INHIBIT
0 V   -> RM5 pin 1 GND
```

- Y23=1 -> INHIBIT HIGH -> RM5 zablokowany.
- Y23=0 -> RM5 aktywny.

## Wyjścia do sterownika myjni / emulacja PitStart

```text
Y0  -> CREDIT / COMPTEUR
Y1  -> PILOTAGGIO POMPA
Y2  -> PROGRAM 1
Y3  -> PROGRAM 2
Y4  -> PROGRAM 3
Y5  -> PROGRAM 4
Y6  -> PROGRAM 5
Y7  -> PROGRAM 6
Y27 -> OŚWIETLENIE W CZASIE PRACY
```

Oryginalny PitStart realizuje wyjścia jako suche styki NO. Jeżeli wyjścia SEEKU są
przekaźnikowe, należy używać ich jako styków bezpotencjałowych zgodnie z wymaganiami
wejść sterownika myjni.

UWAGA: Y4 jest używane w starej logice analogowej D6/D7 i trzeba ten konflikt usunąć.

## Pilotaggio i oświetlenie

Proponowane:
- M410 = PILOTAGGIO_ENABLE z HMI
- M412 = WORK_ACTIVE

```text
M410 AND M412 -> Y1
M412          -> Y27
```

WORK_ACTIVE:
- SET po uruchomieniu P1...P6,
- SET podczas aktywnego trybu ręcznego/free,
- RST przez STOP,
- RST po zakończeniu kredytu/pracy.

Dzięki temu oświetlenie działa podczas pracy nawet wtedy, gdy funkcja Pilotaggio jest wyłączona.

## Wejścia z myjni

Z dokumentacji PitStart:
- AUTOMATE PRESENT / obecność urządzenia automatycznego,
- MANUAL/FREE / uruchomienie ręczne bez monety.

Projekt może dodatkowo wykorzystywać X0...X3 jako statusy specyficzne dla istniejącego
sterownika myjni, ale ich znaczenie trzeba potwierdzić na obiekcie.
