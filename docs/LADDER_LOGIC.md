# Logika drabinki — wersja docelowa

## 1. Statusy

```text
X0 ---------------- (M300)    ; AUTOMATE_PRESENT / INVERTER_OK
X1 ---------------- (M301)    ; PRACA
```

`X1/PRACA` jest tylko statusem i nie blokuje RM5.

## 2. RM5 INHIBIT

```text
/M300 -------------- (Y23)
```

Brak X0 blokuje akceptor.

## 3. Zbieranie impulsów RM5

```text
X27 ---------------- (M320)

M320 --------------- [INC D320]
M320 --------------- [SET M321]
M320 --------------- [RST T200]

M321 --------------- [T200 K100]
```

Każdy nowy impuls restartuje T200. Koniec paczki następuje po około 1 s bez impulsu.

## 4. Koniec paczki

```text
T200 --------------- [MUL D320 D300 D350]
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

## 6. Odliczanie D350

```text
LDP M8013
AND> D350 K0
DEC D350
```

## 7. HMI — przyciski

Rezerwacja:
- `M400` = STOP,
- `M401` = Program 1,
- `M402` = Program 2,
- `M403` = Program 3,
- `M404` = Program 4,
- `M405` = Program 5,
- `M406` = Program 6 / opcjonalny,
- `M410` = Pilotaggio enable.

Programy powinny być wzajemnie wykluczające — aktywny jest tylko jeden z P1…P6.

Mapowanie wyjść:
- P1 -> Y2,
- P2 -> Y3,
- P3 -> Y4,
- P4 -> Y5,
- P5 -> Y6,
- P6 -> Y7.

## 8. WORK_ACTIVE

`M412` jest wspólnym stanem pracy.

Przykładowa logika funkcjonalna:

```text
wybranie P1...P6 przy aktywnej pracy -> SET M412
STOP                                -> RST M412
koniec kredytu/pracy                -> RST M412
```

Jeżeli sygnał `X1=PRACA` jest wiarygodnym potwierdzeniem pracy sterownika myjni, można go wykorzystać do podtrzymania lub potwierdzenia `M412`.

## 9. Pilotaggio — Y1

```text
M410 AND M412 -> Y1
```

Zgodnie z zachowaniem oryginalnego PitStart Pilotaggio powinno wystartować przy pierwszym żądaniu programu i pozostać aktywne do STOP albo końca kredytu/pracy.

## 10. Oświetlenie — Y27

```text
M412 -> Y27
```

Oświetlenie działa przez cały czas aktywnej pracy, niezależnie od opcji Pilotaggio.
