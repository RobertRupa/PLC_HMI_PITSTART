# Logika drabinki — V1.2.0

Docelowy PLC: **FX3UC**.

Źródłem wykonywalnej listy instrukcji jest:
- `plc/MAIN_GXDEV_ENTRY.txt`

Wersja z komentarzami:
- `plc/MAIN.txt`

## 1. Start M8002

Pierwszy skan:
- zeruje robocze liczniki i kolejkę CREDIT,
- kasuje HMI STOP/P1…P6 oraz aktywne programy,
- ustawia `M410=1`,
- ustawia wersję `V1.2.0`,
- zachowuje poprawne `D520`; gdy D520<=0 lub D520>600 ustawia 10.

Najważniejsze:

```text
LD M8002
MOV K0 D330

LD M8002
AND<= D520 K0
MOV K10 D520

LD M8002
AND> D520 K600
MOV K10 D520

LD M8002
SET M410
```

## 2. Statusy

```text
X0  -> M300
X1  -> M301
X13 -> M302
```

M302 jest w V1.2.0 statusem Manual / Free.

## 3. Walidacja czasu i RM5 INHIBIT

```text
D520 > 0 AND D520 <= 600 -> M413

/M300 OR /M413 -> Y23
```

## 4. Akceptacja impulsu RM5

```text
LDP X27
AND M300
AND M413
OUT M320
```

Dzięki temu impuls nie jest liczony, gdy automat jest niedostępny lub parametr czasu jest błędny.

## 5. Dodawanie czasu

```text
M320 -> INC D320
M320 AND D524<30000 -> INC D524

M413 -> SUB K32000 D520 D527

M320 AND D350<=D527 -> ADD D350 D520 D350
M320 AND D350>D527  -> MOV K32000 D350
```

## 6. Koniec paczki

```text
T200 -> MOV D320 D521
T200 -> MUL D320 D520 D522
T200 -> MOV D320 D330
T200 -> MOV K0 D320
T200 -> RST M321
```

## 7. Licznik sesji HMI

```text
M8000 -> MUL D524 D520 D525

M411 -> MOV K0 D524
M411 -> MOV K0 D525
M411 -> MOV K0 D526
M411 -> RST M411
```

## 8. CREDIT / Y0

```text
D330>0 AND /M330 AND /M331 -> SET M330
M330 -> Y0
M330 -> T201 K10
T201 -> DEC D330
T201 -> RST M330
T201 -> SET M331
M331 -> T202 K10
T202 -> RST M331
```

## 9. Komendy mechaniczne + HMI

```text
X4  OR M400 -> M440
X5  OR M401 -> M441
X6  OR M402 -> M442
X7  OR M403 -> M443
X10 OR M404 -> M444
X11 OR M405 -> M445
X12 OR M406 -> M446
```

## 10. Jednocyklowe impulsy wyboru

M451…M456 powstają tylko gdy:
- M300=1,
- STOP nieaktywny,
- D350>0.

Przykład:

```text
LDP M441
AND M300
ANI M440
AND> D350 K0
OUT M451
```

## 11. STOP i utrata zezwolenia

Każdy M420…M425 jest kasowany gdy:
- M440=1 lub
- M300=0.

W Instruction List:

```text
LD M440
ORI M300
RST M420
```

Analogicznie dla M421…M425.

## 12. Programy wzajemnie wykluczające

M451 resetuje M421…M425 i ustawia M420.
M452 resetuje pozostałe i ustawia M421.
Analogicznie do M456/M425.

## 13. Koniec czasu

```text
D350<=0 -> RST M420...M425
```

## 14. Wyjścia P1…P6

```text
M420 AND M300 AND D350>0 -> Y2
M421 AND M300 AND D350>0 -> Y3
M422 AND M300 AND D350>0 -> Y4
M423 AND M300 AND D350>0 -> Y5
M424 AND M300 AND D350>0 -> Y6
M425 AND M300 AND D350>0 -> Y7
```

## 15. WORK_ACTIVE

```text
(M420 OR M421 OR M422 OR M423 OR M424 OR M425)
AND M300
AND D350>0
AND /M440
-> M412
```

## 16. Odliczanie

```text
LDP M8013
AND M412
AND> D350 K0
DEC D350
```

Czas jest więc odliczany tylko podczas aktywnego programu. STOP pauzuje czas.

## 17. Pilotaggio i oświetlenie

```text
M410 AND M412 -> Y1
M412          -> Y27
```

M410 startuje jako 1.

## 18. Wersja PLC

```text
D500 = "V1"
D501 = ".2"
D502 = ".0"
D503 = 0

D516 = 1
D517 = 2
D518 = 0
```

## 19. Manual / Free

```text
X13 -> M302
```

W V1.2.0 jest to status tylko do odczytu. Nie omija wymogu D350>0.
