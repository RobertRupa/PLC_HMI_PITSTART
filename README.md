# PLC_HMI_PITSTART

Aktualna wersja PLC: **V1.5.7**.

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
D550 = 10  -> cena bazowa = 1,00 EUR
D551 = 300 -> P1 daje 300 s za 1,00 EUR
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

`M429 = Sync time with RUN` wybiera źródło sygnału odliczania:

- `M429=0` — czas jest zużywany przy `M412=WORK_ACTIVE`,
- `M429=1` — czas jest zużywany przy `M301=PRACA/RUN`.

Przy `M429=1` STOP może wyłączyć aktywny program i wyjścia, ale **nie zatrzymuje czasu, dopóki M301/RUN pozostaje w stanie 1**. Odliczanie zatrzymuje się dopiero po zaniku RUN.

```text
M429=0 -> COUNTDOWN_ACTIVE = M412
M429=1 -> COUNTDOWN_ACTIVE = M301
```

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
- `plc/main_v1.5.7.csv` — aktualny CSV do importu w GX Developer,
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
- dokumentacja producenta WSB7020R i HMI_Setup V5.1 jest podlinkowana bezpośrednio w `docs/HMI.md`,
- `docs/images/hmi/` — podglądy ekranów i ustawień systemowych,
- `docs/RM5.md` — konfiguracja RM5, pinout CN5 i złącza programującego TTL, wymóg COM1 dla Clone5 Professional oraz dokumentacja ustawień i danych referencyjnych konkretnego egzemplarza.
- `docs/SOFTWARE.md` — linki do Clone5 Professional, Clone5, Unio oraz dokumentacji Unio.


## Limit kredytu RM5

`D549` ma zakres 1…999 w jednostkach 0,10 EUR. PLC oblicza liczbę pełnych impulsów Y0, które jeszcze mieszczą się poniżej limitu. `M435` przechodzi w stan wysoki, gdy kolejny pełny impuls RM5 nie może już zostać zaliczony albo bieżąca paczka RM5 wypełniła całe wolne miejsce. Y2/Y23 wtedy aktywują INHIBIT.

```text
D549=50  -> 5,00 EUR
D549=999 -> 99,90 EUR
```

## Doładowanie aktywnej sesji

Dodanie impulsów RM5 nie zmienia aktywnego programu. Kolejna paczka jest dodawana do istniejącej kolejki Y0 zamiast ją zastępować.

`M436 = RM5_TOPUP_BUSY` jest aktywne podczas odbioru paczki RM5 i wysyłania oczekujących impulsów Y0. W tym czasie zużycie kredytu jest wstrzymane. Zapobiega to wyzerowaniu sesji w czasie, gdy zaakceptowane doładowanie nie zostało jeszcze dopisane do D560.

Auto Start wymaga `D558=0`, więc późniejsze doładowanie nie wybiera ponownie programu.

## Restart

Po wejściu PLC w RUN timer T204 utrzymuje przez krótki czas `M438=STARTUP_RESET_ACTIVE`. W tym czasie zerowane są kredyt, czas, kolejki impulsów, liczniki sesji, D588…D594 oraz stany M435…M437. Parametry konfiguracyjne pozostają bez zmian.


## STOP podczas aktywnego RUN

Przy `M429=1` STOP wyłącza wyjścia programu, ale nie zatrzymuje odliczania, jeśli `M301/RUN=1`. `M449` jest wtedy sterowane bezpośrednio przez RUN. Przy `M429=0` odliczanie zależy od `M412/WORK_ACTIVE`.

## Pilotaggio default

`M439 = PILOTAGGIO_DEFAULT` określa stan, do którego ma wrócić sterowanie Pilotaggio po zakończeniu wysyłania całej kolejki impulsów `Y0`, ale tylko wtedy, gdy bieżące sterowanie `M410` jest wyłączone.

Po ostatnim impulsie `Y0` i zakończeniu `T202`:
- jeśli `M410=1` -> PLC niczego nie zmienia,
- jeśli `M410=0` i `M439=1` -> PLC wykonuje `SET M410`,
- jeśli `M410=0` i `M439=0` -> `M410` pozostaje wyłączone.

PLC czeka także, aż nie będzie zbierana nowa paczka RM5 (`M321=0`). Dzięki temu wartość domyślna jest stosowana dopiero po rzeczywistym zakończeniu obsługi kredytu.

`M439` nie jest już kopiowane do `M410` przy wyborze programu. `M447 = NEW_PROGRAM_START` pozostaje tylko sygnałem diagnostycznym.

## Domyślna taryfa

```text
D300 = 10
D550 = 10
D551..D556 = 300
```

Przy tych wartościach jeden impuls z RM5 daje 300 s, czyli 5 minut, dla każdego programu.
