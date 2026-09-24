# Logika drabinki — wersja docelowa

## 1. Bezpieczna inicjalizacja — pierwszy skan

Standardowy bit pierwszego skanu FX: `M8002`.

```text
M8002 -> MOV K10 D300
M8002 -> MOV K0  D320
M8002 -> MOV K0  D330
M8002 -> MOV K0  D350
M8002 -> MOV K0  D351
M8002 -> MOV K10 D520
M8002 -> MOV K0  D521
M8002 -> MOV K0  D522
M8002 -> MOV K0  D523
M8002 -> MOV K0  D524
M8002 -> MOV K0  D525
M8002 -> MOV K0  D526
M8002 -> MOV K31990 D527

M8002 -> MOV H3156 D500   ; V1
M8002 -> MOV H312E D501   ; .1
M8002 -> MOV H302E D502   ; .0
M8002 -> MOV H0000 D503   ; terminator
M8002 -> MOV K1 D516
M8002 -> MOV K1 D517
M8002 -> MOV K0 D518

M8002 -> RST M321
M8002 -> RST M330
M8002 -> RST M331
M8002 -> RST M410
M8002 -> RST M411
M8002 -> RST M420
M8002 -> RST M421
M8002 -> RST M422
M8002 -> RST M423
M8002 -> RST M424
M8002 -> RST M425
```

Powód:
- `D330` nie może pozostać retencyjne, bo po restarcie PLC wznowiłoby wysyłanie starych impulsów CREDIT.
- `D350=0` oznacza brak kredytu po restarcie.
- `D300=10` ustawia wartość/mnożnik kanału 1 RM5.
- `M410=0` daje bezpieczny domyślny stan Pilotaggio OFF.
- żaden program nie jest wybrany.

## 2. Statusy i RM5 inhibit

```text
X0 ---------------- (M300)
X1 ---------------- (M301)

D520 > 0 AND D520 <= 600 ---- (M413)

/M300 OR /M413 -------------- (Y23)
```

X1/PRACA jest tylko statusem i nie blokuje RM5. Niepoprawny parametr czasu blokuje RM5.

## 3. Zbieranie impulsów RM5

```text
LDP X27 ------------ (M320)

M320 --------------- [INC D320]
M320 --------------- [INC D524]

M413 --------------- [SUB K32000 D520 D527]

M320 AND M413 AND D350 <= D527
--------------------- [ADD D350 D520 D350]

M320 AND M413 AND D350 > D527
--------------------- [MOV K32000 D350]

M320 --------------- [SET M321]
M320 --------------- [RST T200]

M321 --------------- [T200 K100]

M8000 -------------- [MUL D524 D520 D525]
```

Jeśli X27 jest dłuższym poziomem, a nie jednocyklowym impulsem, wejście M320 należy realizować zboczem zgodnym z rzeczywistą polaryzacją RM5.

## 4. Koniec paczki

```text
T200 --------------- [MOV D320 D521]
T200 --------------- [MUL D320 D520 D522]
T200 --------------- [MOV D320 D330]
T200 --------------- [MOV K0 D320]
T200 --------------- [RST M321]
```

## 5. CREDIT / COMPTEUR — Y0

```text
(D330 > K0) AND /M330 AND /M331 -> SET M330

M330 --------------- (Y0)
M330 --------------- [T201 K10]

T201 --------------- [DEC D330]
T201 --------------- [RST M330]
T201 --------------- [SET M331]

M331 --------------- [T202 K10]
T202 --------------- [RST M331]
```

## 6a. Reset licznika sesji z HMI

```text
M411 -> MOV K0 D524
M411 -> MOV K0 D525
M411 -> MOV K0 D526
```

`M411` powinien być przyciskiem chwilowym w HMI.

## 6. Odliczanie D350

```text
LDP M8013
AND> D350 K0
DEC D350
```

## 7. Przyciski fizyczne + HMI

Przyciski fizyczne:
- X4 STOP,
- X5 P1,
- X6 P2,
- X7 P3,
- X10 P4,
- X11 P5,
- X12 P6.

HMI:
- M400 STOP,
- M401 P1,
- M402 P2,
- M403 P3,
- M404 P4,
- M405 P5,
- M406 P6.

Scalanie:

```text
X4 OR M400 -> M440
X5 OR M401 -> M441
X6 OR M402 -> M442
X7 OR M403 -> M443
X10 OR M404 -> M444
X11 OR M405 -> M445
X12 OR M406 -> M446
```

## 8. STOP — najwyższy priorytet

STOP może być obsługiwany poziomem, nie tylko zboczem:

```text
M440 -> RST M420
M440 -> RST M421
M440 -> RST M422
M440 -> RST M423
M440 -> RST M424
M440 -> RST M425
```

Dzięki temu trzymany STOP blokuje wszystkie programy.

STOP nie zeruje `D350`.

## 9. Wybór P1…P6

Komendy programów powinny być obsługiwane zboczem narastającym połączonego bitu M441…M445. Zapobiega to automatycznemu ponownemu startowi programu, gdy przycisk jest nadal trzymany.

Warunki startu:
- STOP nieaktywny,
- `M300=1` / automat dostępny,
- `D350 > 0`.

Przykład P1:

```text
LDP M441
AND M300
ANI M440
AND> D350 K0

RST M421
RST M422
RST M423
RST M424
SET M420
```

P2:

```text
LDP M442
AND M300
ANI M440
AND> D350 K0

RST M420
RST M422
RST M423
RST M424
SET M421
```

Analogicznie P3…P6.

## 10. Automatyczne skasowanie wyboru po końcu kredytu

```text
LD<= D350 K0
RST M420
RST M421
RST M422
RST M423
RST M424
RST M425
```

To jest istotne bezpieczeństwo: po późniejszym doładowaniu kredytu poprzedni program nie wystartuje sam.

## 11. Wyjścia programów

```text
M420 AND D350>0 -> Y2
M421 AND D350>0 -> Y3
M422 AND D350>0 -> Y4
M423 AND D350>0 -> Y5
M424 AND D350>0 -> Y6
M425 AND D350>0 -> Y7
```

Y7 = Program 6, sterowany przez M425.

## 12. WORK_ACTIVE

```text
(M420 OR M421 OR M422 OR M423 OR M424 OR M425)
AND D350>0
AND /M440
---------------- (M412)
```

M412 jest zwykłą cewką OUT, nie SET.

## 13. Pilotaggio — Y1

```text
M410 AND M412 -> Y1
```

Po restarcie M410=0.

## 14. Oświetlenie — Y27

```text
M412 -> Y27
```

Oświetlenie działa podczas aktywnej pracy niezależnie od Pilotaggio.

## 15. Wersja PLC dla HMI

Aktualna wersja: `V1.1.0`.

```text
D500 = "V1"
D501 = ".1"
D502 = ".0"
D503 = 0

D516 = 1
D517 = 1
D518 = 0
```

HMI czyta tekst od `D500`. Rejestry `D516…D518` pozwalają dodatkowo porównywać wersję liczbowo.

## 16. Licznik RM5 i parametr czasu

- `D520` — sekundy na impuls, HMI RW, 1…600,
- `D521` — impulsy ostatniej paczki,
- `D522:D523` — czas ostatniej paczki,
- `D524` — licznik impulsów sesji,
- `D525:D526` — równoważny czas sesji,
- `D527` — roboczy próg saturacji czasu,
- `M411` — reset licznika sesji,
- `M413` — parametr czasu poprawny.

Każdy impuls dodaje `D520` sekund do `D350`; wartość jest ograniczona do 32000 s.
