# Logika PLC — V1.5.3

## RM5 INHIBIT

Y2 jest głównym fizycznym wyjściem RM5 INHIBIT. Y23 pozostaje kompatybilnym mirrorem dla przyszłego PLC:

```text
LDI X0
OUT Y2

LDI X0
OUT Y23
```

- X0=0 -> Y2=1 i Y23=1 -> RM5 zablokowany,
- X0=1 -> Y2=0 i Y23=0 -> RM5 odblokowany.

M413/M415 nadal warunkują programowe przyjęcie impulsu X27, ale nie sterują INHIBIT.

## Założenie PitStart

`Y0 / Counter` jest traktowane jako wyjście jednostek 0,10 EUR.

Po paczce RM5:

```text
COUNTER_PULSES = RM5_PULSES × D300
```

gdzie D300 jest wartością RM5 CH1 w jednostkach 0,10 EUR.

## Cena bazowa i czasy programów

```text
D550 = cena bazowa w jednostkach 0,10 EUR
D551 = P1 sekundy za D550
D552 = P2 sekundy za D550
...
D556 = P6 sekundy za D550
```

## Pozostały kredyt

PLC utrzymuje kredyt w `D560` w centach:

```text
10  = 0,10 EUR
100 = 1,00 EUR
```

Kredyt jest dodawany dopiero po rzeczywiście wysłanym impulsie Y0/T201.

## Zużycie kredytu

Dla wybranego programu:

```text
base_cents = D550 × 10
rate = base_cents × D528 / 100
```

Co sekundę podczas aktywnej pracy:

```text
D561 += rate
consume = D561 / D557
remainder = D561 % D557
D560 -= consume
D561 = remainder
```

Przy zmianie programu D561…D563 są zerowane. Błąd spowodowany zmianą taryfy jest mniejszy niż 1 wewnętrzna podjednostka kredytu.

## Synchronizacja z PRACA

`M430 = M301 OR /M429 OR /M410`.

Domyślnie po pierwszym skanie `M429=1`.

- M429=0: M430 jest zawsze aktywne, więc czas jest zużywany zgodnie z WORK_ACTIVE.
- M429=1: M430 = M301 OR /M429 OR /M410. Przy M429=1 sygnał PRACA zatrzymuje odliczanie tylko wtedy, gdy M410/Pilotaggio jest włączone.

Tick:

```text
LDP M8013
AND M412
AND M430
AND M426
OUT M416
```

## Obliczenie czasu

Dla ostatnio wybranego programu:

```text
D350 = D560 × D557 / (D550 × 10)
```

Obliczenie używa MUL + DDIV.

```text
DIV D350 K60 D540
```

- D540 = minuty,
- D541 = sekundy.

## HMI wake

```text
M320 -> SET M419
M419 -> T203 K300
T203 -> RST M419
M320 -> INC D565
```

M419 pozostaje aktywne około 3 s.

## Maksymalny kredyt

D549 jest parametrem maksymalnego kredytu w jednostkach 0,10 EUR, domyślnie 50 = 5,00 EUR.

Wewnętrzny limit:

```text
MAX_CREDIT_CENTS = D549 × 10
```

Zakres D549 i maksymalny czas programu zostały ograniczone tak, aby D350 pozostało bezpiecznie w dodatnim zakresie 16-bit dla najgorszej kombinacji ustawień.

## STOP

STOP:
- kasuje aktywny program,
- wyłącza Y1/Y3…Y7/Y10/Y27,
- nie kasuje D560,
- nie kasuje D558,
- nie kasuje pozostałego czasu.

Po ponownym wyborze programu pozostały kredyt jest przeliczany według wybranej taryfy.


## TIME_BAR_MAX

`D559` przechowuje maksimum paska czasu.

Po obliczeniu D350:

```text
LD> D350 D559
MOV D350 D559
```

Na początku nowej sesji oraz przy zmianie programu D559 jest zerowane, więc nowe maksimum jest wyznaczane z aktualnego D350. Podczas odliczania D559 nie maleje.


## Auto Start Program

Adresy:

```text
M431 = AUTO_START_PROGRAM_ENABLE
D584 = AUTO_START_PROGRAM_NO (1..6)
M432 = AUTO_START_TRIGGER
M433 = AUTO_START_DONE
M434 = AUTO_START_PROGRAM_OK

M460..M465 = AUTO_SELECT_P1..P6
M470..M475 = EFFECTIVE_SELECT_P1..P6
```

Warunek automatycznego startu:

```text
M431
AND M434
AND M300
AND /M440
AND D560>0
AND D330<=0
AND /M321
AND /M330
AND /M331
AND /M412
AND /M433
-> M432
```

M432 wybiera jeden z M460…M465 według D584. Następnie:

```text
M470 = M451 OR M460
M471 = M452 OR M461
...
M475 = M456 OR M465
```

Dalsza logika programu korzysta z M470…M475, dlatego start automatyczny i ręczny przechodzą przez ten sam latch programu, ustawienie D557/D558 oraz D559.

M433 jest ustawiane po Auto Start i blokuje kolejne automatyczne uruchomienia w tej samej sesji. Nowa sesja zeruje M433. STOP podczas oczekiwania ustawia M433 i anuluje automatyczne uruchomienie.

Odliczanie czasu wróciło do standardowej bramki:

```text
LDP M8013
AND M412
AND M430
AND M426
OUT M416
```


## HMI screen index

```text
X14 -> M303
D585 = HMI_SCREEN_INDEX
```

Logika:

```text
default                         -> D585=0
/M303 AND /M300                 -> D585=2
/M303 AND M300 AND /M427        -> D585=3
M303                            -> D585=1
```

Znaczenie:
- 0 = Main/Work,
- 1 = Admin,
- 2 = Stanowisko nieczynne,
- 3 = Stanowisko wolne.

Zapis Admin wykonywany jest jako ostatni, więc ma najwyższy priorytet.

## Default settings

Na pierwszym skanie:

```text
SET M429   ; Sync countdown with PRACA
SET M431   ; Auto Start Program
```

Oba ustawienia domyślnie startują w stanie ON.


## HMI current screen feedback

`D585` steruje ekranem HMI (PLC -> HMI).

`D586` jest odwrotnym kanałem statusowym (HMI -> PLC): WSStudio zapisuje tam indeks aktualnie wyświetlanego ekranu przez **HMI Status -> Screen Index**.

PLC nie wykonuje instrukcji MOV do D586.

Wake request pozostaje w `M419`, a jego word mirror został przeniesiony do `D587`.


## Max credit / RM5 inhibit

```text
D566 = D549 * 10
D588 = D330 * 10
D590 = D560 + D588
D592 = max(D566 - D590, 0)
D594 = D592 / 10
D542 = D320 * D300
M437 = M321 AND (D542 >= D594)
M435 = (D594 < D300) OR M437

Y2  = /X0 OR M435
Y23 = /X0 OR M435
```

D330 jest kolejką impulsów Y0. D594 określa, ile pełnych impulsów Y0 można jeszcze dopisać. M435 przechodzi w stan wysoki, zanim PLC zaakceptuje impuls RM5, którego pełnej wartości nie da się już zaliczyć.

D549 ma zakres 1…999, czyli do 99,90 EUR.

## Doładowanie bez zmiany programu

Auto Start wymaga:

```text
D558 = 0
```

Po wybraniu programu D558 ma wartość 1…6. Kolejne paczki RM5 nie generują Auto Select.

Przy zamknięciu paczki T200 nowa liczba impulsów jest ograniczana do wolnego miejsca wynikającego z D549 i dodawana do bieżącej kolejki:

```text
free_cents  = max_credit_cents - D560
free_pulses = free_cents / 10
free_slots  = max(free_pulses - D330, 0)
packet_y0   = min(packet_y0, free_slots)

D330 = D330 + packet_y0
```

`M436 = RM5_TOPUP_BUSY`:

```text
M436 = (D330>0) OR M321 OR M330 OR M331
```

Tick zużycia kredytu ma dodatkowy warunek `/M436`. Jeżeli moneta została zaakceptowana podczas aktywnej sesji, odliczanie czeka do zakończenia paczki i kolejki Y0.

## Countdown / PRACA / Pilotaggio

```text
M430 = M301 OR /M429 OR /M410
```

Dla domyślnego `M429=1`:
- M410=1 -> odliczanie wymaga X1/M301,
- M410=0 -> odliczanie trwa niezależnie od X1/M301.


## Restart PLC

Po wejściu PLC w RUN działa T204. Dopóki T204 nie upłynie, M438=1 i PLC zeruje kredyt, czas, liczniki sesji, kolejkę Y0, D588…D594 oraz stany M435…M437. Parametry taryfy i konfiguracja HMI nie są zerowane.


## Pilotaggio default

```text
M439 = PILOTAGGIO_DEFAULT
```

Po utworzeniu M470…M475:

```text
M470 OR M471 OR M472 OR M473 OR M474 OR M475
AND /M410
AND M439
SET M410
```

Domyślnie M439=1.

## Domyślna taryfa

```text
D300=10
D550=10
D551..D556=300
```

Jeden impuls RM5 generuje 10 impulsów Y0. Przy cenie bazowej 1,00 EUR i czasie 300 s daje to 5 minut.
