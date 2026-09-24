# Schemat połączeniowy

## 1. RM5 -> PLC

```text
RM5 pin 2  +12...24 V  -> zasilanie RM5
RM5 pin 1  GND         -> 0 V / COM wejść PLC
RM5 pin 7  CH1         -> X14

PLC Y23                -> RM5 pin 6 INHIBIT
```

Y23 należy podłączyć tak, aby jego aktywacja podawała stan HIGH na wejście INHIBIT RM5.

## 2. Mechaniczne przyciski

Przyjęte mapowanie:

```text
STOP       -> X4
PROGRAM 1  -> X5
PROGRAM 2  -> X6
PROGRAM 3  -> X7
PROGRAM 4  -> X10
PROGRAM 5  -> X11
X12        -> rezerwa
X13        -> rezerwa
RM5 CH1    -> X14
```

Przyciski mają być chwilowe (momentary). Zalecane są styki NO dla P1…P5. Dla STOP można użyć rozwiązania dopasowanego do istniejącego panelu, ale logika PLC traktuje aktywny X4 jako STOP o najwyższym priorytecie.

Sposób połączenia przycisków z COM należy wykonać zgodnie z polaryzacją wejść konkretnego wariantu SEEKU/WSB. Nie podawać napięcia na X bez potwierdzenia typu wejścia.

## 3. Wyjścia PLC -> sterownik myjni

```text
Y0  -> CREDIT / COMPTEUR
Y1  -> PILOTAGGIO POMPA
Y2  -> PROGRAM 1
Y3  -> PROGRAM 2
Y4  -> PROGRAM 3
Y5  -> PROGRAM 4
Y6  -> PROGRAM 5
Y7  -> REZERWA
Y27 -> OŚWIETLENIE
```

Oryginalny PitStart ma wyjścia jako suche styki NO. Dla PLC z wyjściami przekaźnikowymi używać Y/COM jako styków bezpotencjałowych zgodnie z wejściami sterownika myjni.

## 4. Oryginalny PitStart — CN4/CN5, Fig. 33

### CN4

| Piny | Funkcja |
|---|---|
| 1 + 3 | Program 1 |
| 4 + 6 | Program 2 |
| 7 + 9 | Program 3 |
| 10 + 12 | Program 4 |

### CN5

| Piny | Funkcja |
|---|---|
| 1 + 3 | Program 5 |
| 4 + 6 | Program 6 |
| 7 + 9 | Pilotaggio pompa |
| 10 + 12 | Compteur / Credit |

Projekt wykorzystuje P1…P5. P6/Y7 pozostaje rezerwą.

## 5. Oryginalny PitStart — CN8, Fig. 34

| Piny | Funkcja |
|---|---|
| CN8-1 = +24 V, CN8-2 = GND | Presence automatic device |
| CN8-3 = +24 V, CN8-4 = GND | Manual / Free |

W naszym projekcie X0 pełni rolę AUTOMATE_PRESENT / INVERTER_OK.

## 6. Pilotaggio i oświetlenie

```text
M410 AND M412 -> Y1
M412          -> Y27
```

## 7. Uwaga elektryczna

Nie podawać napięcia na wejścia sterownika myjni bez potwierdzenia ich typu. Jeżeli oczekują zwarcia styku, używać wyjść przekaźnikowych jako suchych styków.
