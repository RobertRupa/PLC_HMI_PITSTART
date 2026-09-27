# Logika PLC — V1.4.4

## RM5 INHIBIT

Y23 jest sterowane wyłącznie wejściem X0:

```text
LDI X0
OUT Y23
```

- X0=0 -> Y23=1 -> RM5 zablokowany,
- X0=1 -> Y23=0 -> RM5 odblokowany.

M413/M415 nadal warunkują programowe przyjęcie impulsu X27, ale nie sterują Y23.

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

PLC utrzymuje kredyt w `D560` z rozdzielczością 1/100 impulsu Counter:

```text
100  = 0,10 EUR
1000 = 1,00 EUR
```

Kredyt jest dodawany dopiero po rzeczywiście wysłanym impulsie Y0/T201.

## Zużycie kredytu

Dla wybranego programu:

```text
base_scaled = D550 × 100
rate = base_scaled × D528 / 100
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

`M430 = M301 OR /M429`.

- M429=0: M430 jest zawsze aktywne, więc czas jest zużywany zgodnie z WORK_ACTIVE.
- M429=1: M430=M301, więc czas jest zużywany tylko podczas rzeczywistego PRACA z automatyki.

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
D350 = D560 × D557 / (D550 × 100)
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
MAX_SCALED = D549 × 100
```

Zakres D549 i maksymalny czas programu zostały ograniczone tak, aby D350 pozostało bezpiecznie w dodatnim zakresie 16-bit dla najgorszej kombinacji ustawień.

## STOP

STOP:
- kasuje aktywny program,
- wyłącza Y1/Y2…Y7/Y27,
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
