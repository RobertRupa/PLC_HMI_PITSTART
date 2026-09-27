# Logika PLC — V1.5.7

Aktualne źródła:

- `plc/MAIN_GXDEV_ENTRY.txt` — lista instrukcji,
- `plc/MAIN.txt` — wersja komentowana,
- `plc/main_v1.5.7.csv` — plik importowy GX Developer,
- `plc/DEVICE_MAP.csv` — mapa urządzeń.

## Start i reset sesji

Po wejściu PLC w RUN:

```text
M8000 -> T204 K100
/T204 -> M438 STARTUP_RESET_ACTIVE
```

W oknie M438 zerowane są dane bieżącej sesji: kredyt, czas, kolejki RM5/Y0, liczniki i stany robocze. Parametry konfiguracyjne nie są bezwarunkowo kasowane.

Na pierwszym skanie domyślnie ustawiane są:

```text
M410 PILOTAGGIO_ENABLE
M414 WORK_LIGHTS_ENABLE
M429 SYNC_COUNTDOWN_WITH_PRACA
M431 AUTO_START_PROGRAM_ENABLE
M439 PILOTAGGIO_DEFAULT
```

## RM5 INHIBIT

Aktualna logika obu wyjść INHIBIT:

```text
LDI X0
OR M435
OUT Y2

LDI X0
OR M435
OUT Y23
```

Znaczenie:
- `X0=0` -> RM5 zablokowany,
- `M435=1` -> RM5 zablokowany przez limit kredytu,
- `X0=1 AND M435=0` -> RM5 odblokowany.

**Y2** jest głównym fizycznym wyjściem używanym z RM5. Y23 jest kompatybilnym mirrorem.

## Model impulsów RM5 i Counter

`X27` odbiera tylko CH1 z RM5.

```text
D300 = wartość 1 impulsu CH1 w jednostkach 0,10 EUR
Y0   = Counter / PITSTART_CREDITS
1 impuls Y0 = 0,10 EUR
```

Po zakończeniu paczki:

```text
packet_y0 = RM5_CH1_pulses * D300
```

Wartość jest ograniczana do wolnego miejsca wynikającego z D549 i dodawana do kolejki D330.

## Odbiór paczki RM5

Zbocze X27 tworzy impuls M320. PLC:
- zwiększa D320 — liczbę impulsów aktualnej paczki,
- zwiększa D524 — liczbę impulsów RM5 w sesji,
- ustawia M321,
- restartuje T200.

Po zakończeniu T200:

```text
D521 = D320
D542 = D320 * D300
D542 = min(D542, D594)
D522 = D542
D330 = D330 + D542
D320 = 0
M321 = 0
```

## Generator Y0

Jeżeli D330>0 i generator jest wolny, ustawiane jest M330.

```text
M330 -> Y0
M330 -> T201 K10
```

Po T201:
- D330 jest zmniejszane,
- D533 jest zwiększane,
- do D560 dopisywane jest 10 centów,
- D560 jest ograniczane do D566 = D549*10,
- M330 jest kasowane,
- M331 uruchamia przerwę T202.

M436 pozostaje aktywne, dopóki trwa paczka RM5 albo kolejka Y0:

```text
M436 = (D330>0) OR M321 OR M330 OR M331
```

Podczas M436 tick zużycia kredytu jest wstrzymany.

## Limit kredytu

```text
D566 = D549 * 10
D588 = D330 * 10
D590 = D560 + D588
D592 = max(D566 - D590, 0)
D594 = D592 / 10
```

D594 oznacza liczbę pełnych impulsów Y0, które można jeszcze dopisać.

Dla aktualnej paczki:

```text
D542 = D320 * D300
M437 = M321 AND (D542 >= D594)
M435 = (D594 < D300) OR M437
```

Dzięki temu RM5 jest blokowany zanim kolejny pełny impuls CH1 przekroczy limit.

## Wybór programu

Źródła ręczne:

```text
P1: X5  OR M401
P2: X6  OR M402
P3: X7  OR M403
P4: X10 OR M404
P5: X11 OR M405
P6: X12 OR M406
STOP: X4 OR M400
```

Impulsy wyboru trafiają do M451…M456. Auto Start używa M460…M465.

```text
M470 = M451 OR M460
...
M475 = M456 OR M465
```

M470…M475 są wspólnymi impulsami wyboru programu.

Po wyborze programu:
- ustawiany jest dokładnie jeden M420…M425,
- D557 otrzymuje D551…D556,
- D558 otrzymuje numer 1…6,
- D561…D563 są zerowane,
- D559 jest zerowane, aby wyznaczyć nowe maksimum paska czasu.

## WORK_ACTIVE

```text
M412 =
  (M420 OR M421 OR M422 OR M423 OR M424 OR M425)
  AND M300
  AND D560>0
  AND NOT M440
```

M412 steruje wyjściami programu, Pilotaggio i Work Lights. Nie jest już jedynym źródłem bramki odliczania.

## Sync time with RUN — M429

V1.5.7 używa selektora:

```text
LDI M429
AND M412
OUT M448

LD M429
AND M301
OR M448
OUT M449

LD M449
OUT M430
```

Czyli:

```text
M429=0 -> M449 = M412 / WORK_ACTIVE
M429=1 -> M449 = M301 / PRACA-RUN
M430 = M449
```

M448 nie jest latchem. Jest bieżącą gałęzią `NOT M429 AND M412`.

## Tick zużycia kredytu

Warunek poprawnego czasu:

```text
M426 = D558>0 AND D557>0 AND D560>0
```

Tick:

```text
LDP M8013
AND M449
AND M426
ANI M436
OUT M416
```

Przy `M429=1` aktywny RUN jest więc wystarczającym źródłem czasu po wcześniejszym wybraniu programu i przy dostępnym kredycie.

## Zużycie kredytu

```text
D568 = D550 * 10
D570:D571 = D568 * D528
D574:D575 = (D570:D571) / 100
```

D574 jest skorygowaną stawką wewnętrzną.

Co tick:

```text
D561 += D574
D562 = D561 / D557
D563 = D561 MOD D557
D561 = D563
D560 -= D562
```

D528 działa więc jako korekcja szybkości zużycia:
- 100 = 1,00x,
- 101 = 1,01x,
- 99 = 0,99x.

## Obliczanie pozostałego czasu

```text
D568 = D550 * 10
D570:D571 = D560 * D557
D574:D575 = (D570:D571) / D568
D350 = D574
```

Następnie:

```text
DIV D350 K60 D540
```

Wynik:
- D540 — minuty,
- D541 — reszta sekund.

D528 nie zmienia czasu początkowego; zmienia tempo zużywania D560, przez co D350 może maleć szybciej lub wolniej niż jedna jednostka na sekundę.

## STOP

STOP kasuje M420…M425. W efekcie natychmiast wyłączają się:
- Y1 Pilotaggio,
- Y3…Y7/Y10 programy,
- Y27 Work Lights.

STOP nie kasuje D560, D557 ani D558.

Zachowanie odliczania:

```text
M429=0 -> STOP zeruje M412, więc czas staje
M429=1 -> M449 śledzi M301/RUN; jeśli RUN=1, czas dalej schodzi
```

Nie ma już latcha „STOP_RUN_COUNTDOWN” z V1.5.6.

## Auto Start Program

```text
M431 = AUTO_START_PROGRAM_ENABLE
D584 = AUTO_START_PROGRAM_NO 1..6
M432 = AUTO_START_TRIGGER
M433 = AUTO_START_DONE
M434 = AUTO_START_PROGRAM_OK
```

Warunek M432 obejmuje:
- M431=1,
- D584 w zakresie 1…6,
- M300=1,
- STOP nieaktywny,
- D560>0,
- `D558=0`,
- D330<=0,
- brak M321/M330/M331,
- brak M412,
- brak M433.

M433 blokuje wielokrotny Auto Start w tej samej sesji. Nowa sesja zeruje M433. STOP podczas oczekiwania ustawia M433 i anuluje Auto Start dla tej sesji.

## Pilotaggio default

```text
M439 = PILOTAGGIO_DEFAULT
```

Po zakończeniu całej kolejki Y0 i T202:

```text
jeśli M410=0 i M439=1 -> SET M410
jeśli M410=1          -> bez zmiany
jeśli M439=0          -> M410 pozostaje OFF
```

Przywrócenie jest blokowane, jeżeli trwa już następna paczka RM5.

M447 = NEW_PROGRAM_START pozostaje wyłącznie markerem diagnostycznym.

## Wyjścia

```text
Y1  = M410 AND M412
Y3  = M420 AND M300 AND D560>0
Y4  = M421 AND M300 AND D560>0
Y5  = M422 AND M300 AND D560>0
Y6  = M423 AND M300 AND D560>0
Y7  = M424 AND M300 AND D560>0
Y10 = M425 AND M300 AND D560>0
Y27 = M414 AND M412
```

## HMI screen index

```text
D585=1  Admin                  gdy M303=1
D585=2  Stanowisko nieczynne   gdy M303=0 i M300=0
D585=3  Stanowisko wolne       gdy M303=0, M300=1 i M427=0
D585=0  Home                   w pozostałym przypadku
```

D586 jest zapisywany przez HMI i nie jest nadpisywany przez PLC.

## TIME_BAR_MAX

D559 jest zerowane przy wyborze programu. Później rośnie tylko wtedy, gdy nowe D350 jest większe od dotychczasowego D559. Nie maleje podczas odliczania.

## Znane ograniczenie D350

D350 jest 16-bitowym rejestrem czasu. Bieżące M415 sprawdza osobne zakresy D549, D550 i D551…D556, ale nie sprawdza ich kombinacji pod kątem przepełnienia wynikowego czasu.

Konserwatywnie utrzymuj:

```text
D549 * D557 / D550 <= 32767 s
```

Dla domyślnych wartości:

```text
50 * 300 / 10 = 1500 s
```

czyli 25 minut — bezpiecznie.

## Domyślna taryfa

```text
D300=10
D550=10
D551..D556=300
```

Jeden impuls RM5 CH1 daje wtedy 10 impulsów Y0 i 300 s dla każdego programu.
