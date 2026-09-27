# PLC_HMI_PITSTART

Aktualna wersja PLC: **V1.5.1**.

## Model kredytu

- `Y0 / PITSTART_CREDITS` reprezentuje jednostki **0,10 EUR**,
- `D300` określa wartość RM5 CH1 w jednostkach 0,10 EUR,
- wspólna cena bazowa jest w `D550`,
- czasy P1…P6 dla ceny bazowej są w `D551…D556`,
- PLC prowadzi pozostały kredyt pieniężny i z niego oblicza czas dla wybranego programu.

## I/O

### Wejścia
- X0 — AUTOMATE_PRESENT / INVERTER_OK
- X1 — PRACA
- X4 — STOP
- X5…X12 — P1…P6
- X13 — MANUAL / FREE status
- X14 — ADMIN switch -> M303
- X27 — RM5 CH1

### Wyjścia

> **Adresacja FX jest ósemkowa:** po `Y7` występuje `Y10`. Nie ma adresu `Y8`. Dlatego sześć kolejnych wyjść programów od Y3 to Y3, Y4, Y5, Y6, Y7, Y10.
- Y0 — PITSTART_CREDITS / Counter
- Y1 — PILOTAGGIO
- Y2 — RM5 INHIBIT, główne fizyczne wyjście
- Y3 — Program 1
- Y4 — Program 2
- Y5 — Program 3
- Y6 — Program 4
- Y7 — Program 5
- Y10 — Program 6
- Y23 — RM5 INHIBIT, kompatybilny mirror Y2 dla przyszłego PLC z większą liczbą wyjść
- Y27 — WORK LIGHTS

## Model kredytu

```text
1 impuls Y0 = 0,10 EUR

D300 = wartość jednego impulsu RM5 CH1 w jednostkach 0,10 EUR
D550 = wspólna cena bazowa w jednostkach 0,10 EUR
D551..D556 = czas P1..P6 w sekundach dla D550
```

Przykład:

```text
D300 = 10  -> RM5 CH1 = 1,00 EUR
D550 = 5   -> cena bazowa = 0,50 EUR
D551 = 70  -> P1 daje 70 s za 0,50 EUR
```

Jeden impuls RM5 wygeneruje wtedy 10 impulsów Counter/Y0.

## Pozostały kredyt i czas

`D560` przechowuje pozostały kredyt w centach:

```text
D560 = 10   -> 0,10 EUR
D560 = 100  -> 1,00 EUR
```

Czas dla ostatnio wybranego programu:

```text
time_left = D560 * PROGRAM_TIME / (D550 * 10)
```

Rejestry HMI:
- D350 — pozostały czas [s],
- D540 — minuty,
- D541 — sekundy,
- D559 — TIME_BAR_MAX dla paska postępu,
- D582 — pozostały kredyt w centach,
- M426 — TIME_DISPLAY_VALID,
- M427 — CREDIT_AVAILABLE.

## Synchronizacja z automatyką

Domyślnie `M429=1`. Synchronizacja z `X1/M301 = PRACA` zatrzymuje odliczanie tylko wtedy, gdy `M410/PILOTAGGIO_ENABLE=1`. Przy `M410=0` czas jest odliczany dalej mimo zaniku PRACA.

Po potwierdzeniu, że `X1/M301 = PRACA` jest wiarygodnym sygnałem rzeczywistej pracy automatyki, ustaw na HMI:

```text
M429 = 1
```

Wtedy kredyt jest zużywany tylko gdy jednocześnie:
- program jest aktywny,
- M301/PRACA = 1.

`D528` pozostaje globalną korekcją zegara x100:
- 100 = 1,00x,
- 101 = 1,01x,
- 99 = 0,99x.

## HMI — zdarzenie RM5

Po zaakceptowanym impulsie RM5:
- `M419=1` przez około 3 s,
- `D565` zwiększa się o 1.

Przełączanie ekranów jest realizowane przez `D585` w funkcji PLC Control / Control Screen Switch. `D586` jest przeznaczony na zapis aktualnego indeksu ekranu przez HMI.

## HMI ekran Admin

- M303 — ADMIN_SCREEN_REQUEST, bezpośrednie odwzorowanie X14.
- D585 — HMI_SCREEN_INDEX:
  - 0 = Main/Work,
  - 1 = Admin,
  - 2 = Stanowisko nieczynne,
  - 3 = Stanowisko wolne.
- D586 — HMI_CURRENT_SCREEN_INDEX, zapisywany przez HMI.

## HMI sterowanie

- M400 — STOP, momentary
- M401…M406 — P1…P6, momentary
- M410 — PILOTAGGIO_ENABLE
- M411 — reset liczników diagnostycznych sesji
- M414 — WORK_LIGHTS_ENABLE
- M429 — synchronizacja odliczania z PRACA, default ON
- M431 — AUTO_START_PROGRAM_ENABLE; default ON; automatycznie uruchamia wybrany program po zakończeniu wysyłania Y0
- D584 — numer programu Auto Start 1…6

Touch control pozostaje lokalną funkcją HMI i nie ma bitu PLC.

## Pliki

- `plc/MAIN_GXDEV_ENTRY.txt` — aktualny program Instruction List,
- `plc/MAIN.txt` — wersja komentowana,
- `plc/main_v1.5.1.csv` — CSV w układzie eksportu GX,
- `plc/DEVICE_MAP.csv` — mapa urządzeń,
- `docs/HMI.md` — konfiguracja HMI,
- `docs/LADDER_LOGIC.md` — opis logiki,
- `CHANGELOG.md` — historia zmian.


## Wymagana konfiguracja RM5 Evolution

PLC odczytuje tylko `X27 = RM5 CH1` (pin 7 RM5). Wszystkie używane nominały/kanały walidatora muszą być skonfigurowane tak, aby ich impulsy były wysyłane na wyjście **CH1**.

W praktyce:
- fizycznie do PLC podłączony jest tylko CH1,
- CH2…CH6 nie są odczytywane przez PLC,
- jeżeli różne nominały mają różną wartość, RM5 musi zakodować wartość liczbą impulsów CH1 zgodnie z przyjętą jednostką,
- PLC traktuje każdy impuls na X27 jednakowo i mnoży liczbę impulsów przez `D300`.

Jeżeli RM5 wyśle po jednym impulsie niezależnie od nominału, PLC potraktuje wszystkie takie monety jako tę samą wartość.


## Priorytet ekranów HMI

`D585` jest aktualnym indeksem ekranu:

```text
1 Admin                 jeśli X14=1
2 Stanowisko nieczynne  jeśli X14=0 i X0=0
3 Stanowisko wolne      jeśli X14=0, X0=1 i M427=0
0 Main/Work             w pozostałym przypadku
```

Priorytet: Admin > Nieczynne > Wolne > Main.


## Pliki projektów

- `projects/plc/PitStart.zip` — projekt PLC,
- `projects/hmi/pitstart_hmi.zip` — projekt HMI w archiwum transportowym.

Dokumentacja ekranów i konfiguracji:
- `docs/HMI.md` — mapowanie ekranów, D585/D586, ustawienia Home/Admin i WSStudio,
- `docs/images/hmi/` — podglądy ekranów i ustawień systemowych,
- `docs/RM5.md` — konfiguracja RM5.


## Limit kredytu RM5

`D549` ma zakres 1…999 w jednostkach 0,10 EUR. `M435` uwzględnia kredyt już zapisany w `D560` oraz impulsy oczekujące w kolejce `D330`. Po wykorzystaniu całego dostępnego limitu Y2/Y23 aktywują INHIBIT RM5.

```text
D549=50  -> 5,00 EUR
D549=999 -> 99,90 EUR
```

## Doładowanie aktywnej sesji

Dodanie impulsów RM5 nie zmienia aktywnego programu. Kolejna paczka jest dodawana do istniejącej kolejki Y0 zamiast ją zastępować.

`M436 = RM5_TOPUP_BUSY` jest aktywne podczas odbioru paczki RM5 i wysyłania oczekujących impulsów Y0. W tym czasie zużycie kredytu jest wstrzymane. Zapobiega to wyzerowaniu sesji w czasie, gdy zaakceptowane doładowanie nie zostało jeszcze dopisane do D560.

Auto Start wymaga `D558=0`, więc późniejsze doładowanie nie wybiera ponownie programu.

## Restart

Pierwszy skan PLC zeruje pozostały kredyt, czas, kolejki impulsów, liczniki sesji, rejestry obliczeniowe D588…D592 oraz stan M436. Parametry konfiguracyjne pozostają bez zmian.
