# Bezpieczny start i obsługa przycisków

## Cel

Ta część projektu rozwiązuje dwa problemy:
1. sterowanie ma działać zarówno z HMI, jak i z fizycznych przycisków,
2. po restarcie PLC nie może kontynuować starej kolejki CREDIT.

## Mapowanie wejść

- X20 = STOP
- X21 = P1
- X22 = P2
- X23 = P3
- X24 = P4
- X25 = P5
- X26 = rezerwa
- X27 = RM5 CH1

HMI:
- M400 = STOP
- M401…M405 = P1…P5

## Łączenie HMI i mechaniki

```text
X20 OR M400 -> M440
X21 OR M401 -> M441
X22 OR M402 -> M442
X23 OR M403 -> M443
X24 OR M404 -> M444
X25 OR M405 -> M445
```

STOP jest obsługiwany poziomem i ma najwyższy priorytet.

Programy są wybierane zboczem M441…M445.

## Bity aktywnego programu

- M420 = P1 ACTIVE
- M421 = P2 ACTIVE
- M422 = P3 ACTIVE
- M423 = P4 ACTIVE
- M424 = P5 ACTIVE

Wybór programu resetuje wszystkie pozostałe.

## Warunki startu programu

Program może wystartować tylko wtedy, gdy:
- M300=1 / automat dostępny,
- M440=0 / STOP nieaktywny,
- D350>0 / jest kredyt.

## STOP

STOP:
- kasuje M420…M424,
- wyłącza Y2…Y6 przez brak aktywnego programu,
- wyłącza M412/WORK_ACTIVE,
- wyłącza Y27,
- wyłącza Pilotaggio Y1,
- NIE kasuje kredytu D350.

## Koniec kredytu

Po D350<=0:
- M420…M424 są resetowane.

To zapobiega automatycznemu wznowieniu starego programu po następnym doładowaniu.

## Bezpieczna inicjalizacja M8002

```text
M8002 -> MOV K10 D300
M8002 -> MOV K0 D320
M8002 -> MOV K0 D330
M8002 -> MOV K0 D350
M8002 -> MOV K0 D351

M8002 -> RST M321
M8002 -> RST M330
M8002 -> RST M331
M8002 -> RST M410
M8002 -> RST M420
M8002 -> RST M421
M8002 -> RST M422
M8002 -> RST M423
M8002 -> RST M424
```

Najważniejsze jest `MOV K0 D330`. To usuwa zapamiętaną kolejkę impulsów CREDIT przed uruchomieniem generatora.

## Wartości startowe

| Adres | Wartość |
|---|---:|
| D300 | 10 |
| D320 | 0 |
| D330 | 0 |
| D350 | 0 |
| D351 | 0 |
| M410 | 0 |
| M420…M424 | 0 |

D300=10 jest domyślną wartością kanału 1 RM5 / mnożnikiem.

## WORK_ACTIVE

```text
(M420 OR M421 OR M422 OR M423 OR M424)
AND D350>0
AND /M440
---------------- (M412)
```

M412 jest zwykłym OUT.

## Wyjścia

```text
M420 AND D350>0 -> Y2
M421 AND D350>0 -> Y3
M422 AND D350>0 -> Y4
M423 AND D350>0 -> Y5
M424 AND D350>0 -> Y6

M410 AND M412 -> Y1
M412          -> Y27
```
