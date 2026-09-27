# PLC_HMI_PITSTART

Aktualna wersja PLC: **V1.5.7**.

## Stan projektu

- PLC: **V1.5.7**,
- HMI: `projects/hmi/pitstart.hs`; interfejs ma oznaczenie **HMI V1.0.0**,
- projekt PLC: `projects/plc/PitStart.zip`,
- import GX Developer: `plc/main_v1.5.7.csv`,
- `Y0 / PITSTART_CREDITS` reprezentuje jednostki **0,10 EUR**,
- `D300` określa wartość RM5 CH1 w jednostkach 0,10 EUR,
- wspólna cena bazowa jest w `D550`,
- czasy P1…P6 dla ceny bazowej są w `D551…D556`,
- PLC prowadzi pozostały kredyt pieniężny i z niego oblicza czas dla wybranego programu.

## HMI — podgląd ekranów

| Home | Admin |
|---|---|
| <img src="docs/images/hmi/hmi_v1_5_6_screen_000_home.png" width="390"> | <img src="docs/images/hmi/hmi_v1_5_6_screen_001_admin.png" width="390"> |
| Ekran pracy: Program 1…5, pozostały czas, pasek czasu i STOP. PLC obsługuje również P6, ale aktualny ekran Home nie ma osobnego przycisku P6. | Ekran konfiguracji: przełączniki funkcji oraz parametry kredytu, czasu i Auto Start. |

| Error / Stanowisko nieczynne | Ready / Stanowisko wolne |
|---|---|
| <img src="docs/images/hmi/hmi_v1_5_6_screen_002_error.png" width="390"> | <img src="docs/images/hmi/hmi_v1_5_6_screen_003_ready.png" width="390"> |
| Wyświetlany przy braku dostępności automatu: `D585=2`. | Wyświetlany gdy automat jest dostępny, ale nie ma kredytu: `D585=3`. |

### Konfiguracja przełączania ekranów

| PLC Control | HMI Status |
|---|---|
| <img src="docs/images/hmi/hmi_v1_5_6_system_plc_control.png" width="390"> | <img src="docs/images/hmi/hmi_v1_5_6_system_hmi_status.png" width="390"> |
| `Control Screen Switch = ON`, adres `[Mitsubishi_Fx1n]D585`. PLC wybiera ekran HMI. | `Screen Index = ON`, adres `[Mitsubishi_Fx1n]D586`. HMI zwraca do PLC indeks aktualnego ekranu. |

Indeksy ekranów:

```text
0 = Home
1 = Admin
2 = Error / Stanowisko nieczynne
3 = Ready / Stanowisko wolne
```

Priorytet PLC: **Admin > Nieczynne > Wolne > Home**.

## Konfiguracja dostępna z ekranu Admin

| Element HMI | Adres | Typ | Zakres / domyślne | Działanie |
|---|---|---|---|---|
| Touch control* | `LB555` | toggle lokalny HMI | ON/OFF | Lokalna funkcja panelu; nie zapisuje bitu PLC. |
| Work lights | `M414` | toggle | default ON | Zezwolenie na Y27. Fizyczne światła działają jako `M414 AND M412`. |
| Auto start program | `M431` | toggle | default ON | Po zaksięgowaniu kredytu może automatycznie wybrać program wskazany przez D584. |
| Auto Start Program | `D584` | liczba | 1…6, default 1 | `1=P1 ... 6=P6`. Pozwala również uruchomić P6 mimo braku bezpośredniego przycisku P6 na aktualnym Home. |
| Sync time with RUN | `M429` | toggle | default ON | OFF: odliczanie wg M412/WORK_ACTIVE. ON: odliczanie wg M301/PRACA-RUN; STOP nie zatrzyma czasu, jeśli RUN pozostaje 1. |
| Pilotaggio pompa | `M410` | toggle | default ON | Zezwolenie na Y1; wyjście działa jako `M410 AND M412`. |
| Coin multiplier | `D300` | liczba | 1…100, default 10 | Wartość jednego impulsu RM5 CH1 w jednostkach 0,10 EUR. `10 = 1,00 EUR`. |
| Max credits | `D549` | liczba | 1…999, default 50 | Maksymalny kredyt w jednostkach 0,10 EUR. `50=5,00 EUR`, `999=99,90 EUR`. |
| Base price | `D550` | liczba | HMI 1…50, default 10 | Wspólna cena bazowa P1…P6 w jednostkach 0,10 EUR. M415 dodatkowo wymaga `D550<=D549`. |
| Time correction | `D528` | liczba | 50…200, default 100 | Korekcja tempa zużycia kredytu: 100=1,00x, 50=0,50x, 200=2,00x. |
| Program 1 time | `D551` | liczba | 1…600 s, default 300 | Czas P1 przypisany do ceny bazowej D550. |
| Program 2 time | `D552` | liczba | 1…600 s, default 300 | Czas P2 przypisany do ceny bazowej D550. |
| Program 3 time | `D553` | liczba | 1…600 s, default 300 | Czas P3 przypisany do ceny bazowej D550. |
| Program 4 time | `D554` | liczba | 1…600 s, default 300 | Czas P4 przypisany do ceny bazowej D550. |
| Program 5 time | `D555` | liczba | 1…600 s, default 300 | Czas P5 przypisany do ceny bazowej D550. |
| Program 6 time | `D556` | liczba | 1…600 s, default 300 | Czas P6. P6 jest obsługiwany przez PLC i Auto Start, chociaż aktualny Home nie pokazuje przycisku Program 6. |
| Touch Calibration | `LW4057` | funkcja lokalna HMI | — | Otwiera kalibrację dotyku panelu. |

`* Touch control` i `Touch Calibration` używają lokalnych urządzeń HMI `LB/LW`, nie pamięci PLC.

Aktualny ekran Admin **nie pokazuje osobnego przełącznika `M439=PILOTAGGIO_DEFAULT`**. Bit M439 istnieje w PLC i domyślnie jest ON, ale nie należy traktować go jako obecnie dostępnej nastawy operatora na pokazanym ekranie HMI.

### Komunikacja HMI z PLC

Dla WSB7020R używana jest konfiguracja:

```text
Driver: Mitsubishi_Fx1n
Baud:   38400
Mode:   RS232
```

Po pobraniu projektu do HMI kabel download należy odłączyć; manual WSB podaje, że przy podłączonym kablu komunikacja HMI <-> PLC nie działa.

Szczegóły: [docs/HMI.md](docs/HMI.md).

Schemat elektryczny i zasady COM/RM5: [docs/WIRING.md](docs/WIRING.md).

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

## Sterowanie operatorskie HMI

Na aktualnym ekranie Home dostępne są:
- `M400` — STOP, momentary,
- `M401…M405` — Program 1…5, momentary.

PLC ma również `M406` dla P6, ale aktualny Home nie zawiera bezpośredniego przycisku P6.

Ekran Admin i wszystkie dostępne na nim nastawy są rozpisane w tabeli **Konfiguracja dostępna z ekranu Admin** powyżej. `M411` oraz `M439` istnieją w PLC, lecz nie są pokazane jako kontrolki na aktualnym ekranie Admin.

## Dokumentacja oryginalnego Comestero PitStart

Przesłana instrukcja to **Gamma Pit — Sistemi di attivazione — Manuale operativo**, identyfikowana jako `PitStart-F - c27-M-PIT-EK`, z datą 13/10/2009.

Docelowe miejsce w repo:

```text
docs/manuals/pitstart/Gamma_Pit_PitStart_F_c27-M-PIT-EK_2009-10-13.pdf
```

Nazwa celowo nie zawiera numeru rewizji: strona 2 PDF pokazuje `Rev. 01`, podczas gdy nagłówki dalszych stron pokazują `Rev. 00`.

Najważniejsze rozdziały dla tego projektu:
- `9.5 PitStart` — zasilanie, połączenie z automatyką, wejście obecności automatu i Manual/Free,
- `9.5.4` — konfiguracja MultiConfig: maksymalna wartość, wspólna cena bazowa, tabela RM5, język i poziomy wejść,
- `9.5.5` — programowanie ręczne cen/czasów,
- `9.5.6` — inicjalizacja,
- `10.5` — schemat elektryczny PitStart.

Oryginalny manual potwierdza m.in. suche styki NO wyjść, wyjście Counter generujące impulsy co 0,1 EUR, wyjście Pilotaggio pompa oraz programy P1…P6. Aktualny projekt odwzorowuje te funkcje na PLC, ale nie jest kopią firmware PitStart.

Opis mapowania: [docs/PITSTART.md](docs/PITSTART.md).  
Przygotowany katalog dokumentacji: [docs/manuals/pitstart/README.md](docs/manuals/pitstart/README.md).

## Pliki

- `plc/MAIN_GXDEV_ENTRY.txt` — aktualny program Instruction List,
- `plc/MAIN.txt` — wersja komentowana,
- `plc/main_v1.5.7.csv` — aktualny CSV do importu w GX Developer,
- `plc/DEVICE_MAP.csv` — mapa urządzeń,
- `docs/HMI.md` — konfiguracja HMI,
- `docs/WIRING.md` — techniczny schemat połączeń HMI/PLC, RM5 i automatyki myjni,
- `docs/wiring.svg` — graficzny schemat połączeń,
- `docs/manuals/hmi/WSB_HMI_PLC_All_in_one_User_Manual_V1.79.pdf` — główny manual WSB7020R,
- `docs/manuals/hmi/WSC_HMI_PLC_All_in_one_User_Manual_V1.13.pdf` — manual WSC/WSCH jako dokumentacja porównawcza,
- `docs/manuals/rm5/manual_rm5.pdf` — lokalny manual RM5 Evolution,
- `docs/manuals/pitstart/Gamma_Pit_PitStart_F_c27-M-PIT-EK_2009-10-13.pdf` — docelowa nazwa manuala oryginalnego PitStart,
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
- `projects/hmi/pitstart.hs` — główny edytowalny projekt HMI KinSealStudio,
- `projects/hmi/README.md` — parametry i opis pliku HMI,

Dokumentacja ekranów i konfiguracji:
- `docs/HMI.md` — mapowanie ekranów, D585/D586 oraz konfiguracja projektu KinSealStudio/HMI Studio,
- lokalne manuale WSB/WSC oraz HMI_Setup/HMI Studio 5.1 są opisane w `docs/HMI.md` i `docs/SOFTWARE.md`,
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

## Znane ograniczenie D350

`D350` jest pojedynczym rejestrem 16-bit. Aktualna walidacja M415 nie sprawdza kombinacji maksymalnego kredytu, ceny bazowej i czasu programu pod kątem przepełnienia.

Konserwatywnie utrzymuj:

```text
D549 * D557 / D550 <= 32767 s
```

Domyślne ustawienia dają maksymalnie 1500 s dla domyślnego limitu 5,00 EUR.
