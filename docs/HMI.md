# HMI — WSStudio — V1.5.0

## Ekran Home

### Przyciski

| Funkcja | zapis HMI | wskaźnik aktywnego stanu |
|---|---:|---:|
| STOP | M400 momentary | M412 / WORK_ACTIVE |
| P1 | M401 momentary | M420 |
| P2 | M402 momentary | M421 |
| P3 | M403 momentary | M422 |
| P4 | M404 momentary | M423 |
| P5 | M405 momentary | M424 |
| P6 | M406 momentary | M425 |

STOP nie kasuje kredytu. Po STOP ostatni wybrany program pozostaje w D558 i można nadal wyświetlać pozostały czas dla tej taryfy.

### MM:SS

Użyj dwóch osobnych Value Display:

- minuty: `D540`
- sekundy: `D541`

Dla obu:
- DataType: **16-Bit Unsigned Int**
- Offset address: **OFF**
- Allow input: **OFF**

Pomiędzy obiektami dodaj statyczny znak `:`.

Dla D541 ustaw dwie cyfry z zerem wiodącym.

`M426=1` oznacza, że czas jest ważny. Gdy `M426=0`, zamiast 00:00 można pokazać tekst **SELECT PROGRAM**.

### Pozostały kredyt

`D582` = pozostały kredyt w centach.

Jeżeli WSStudio umożliwia skalowanie/decimal point:
- monitor D582,
- 2 miejsca po przecinku,
- opis EUR.

Przykład: D582=100 -> 1,00 EUR.

### Zdarzenie RM5 / wake

- `M419` = HMI_WAKE_REQUEST, około 3 s,
- `D587` = HMI_WAKE_REQUEST_WORD, 0/1 mirror M419,
- `D565` = HMI_WAKE_EVENT_COUNTER.

Przełączanie ekranów jest realizowane przez `D585` w funkcji **PLC Control / Control Screen Switch**. `D586` jest zapisywany przez HMI i zawiera indeks aktualnie wyświetlanego ekranu.

## Ekran Admin

### Parametry taryfy

| Adres | Nazwa | Typ HMI | Zakres | Domyślnie |
|---|---|---|---:|---:|
| D300 | RM5 CH1 value [x0.10 EUR] | 16-bit unsigned RW | 1…100 | 10 |
| D549 | Max credit [x0.10 EUR] | 16-bit unsigned RW | 1…999 | 50 |
| D550 | Base price [x0.10 EUR] | 16-bit unsigned RW | 1…D549 | 5 |
| D551 | P1 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D552 | P2 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D553 | P3 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D554 | P4 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D555 | P5 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D556 | P6 time [s] | 16-bit unsigned RW | 1…600 | 70 |
| D528 | Clock correction x100 | 16-bit unsigned RW | 50…200 | 100 |
| D584 | Auto start program | 16-bit unsigned RW | 1…6 | 1 |

Interpretacja:
- D300=10 oznacza 1,00 EUR za zaakceptowany impuls RM5 CH1,
- D550=5 oznacza wspólną cenę bazową 0,50 EUR,
- D551=70 oznacza 70 s programu P1 za 0,50 EUR,
- D528=100 oznacza 1,00x.

Jeżeli chcesz pokazywać D528 jako 1.00 zamiast 100, użyj na HMI dwóch miejsc po przecinku / skali 0,01. PLC nadal przechowuje 100.

### Przełączniki

| Adres | Funkcja |
|---|---|
| M410 | Pilotaggio enable |
| M414 | Work lights enable |
| M429 | Sync countdown with PRACA — default ON |
| M431 | Auto start program after CREDIT output — default ON |

M429:
- domyślnie ON po starcie PLC,
- OFF — odliczanie bazuje tylko na WORK_ACTIVE,
- ON — synchronizacja z X1/M301 jest aktywna tylko wtedy, gdy M410/Pilotaggio jest włączone,
- gdy M410=0, odliczanie nie zatrzymuje się przy chwilowym zaniku X1/PRACA.

### Diagnostyka

| Adres | Znaczenie |
|---|---|
| M300 | automat dostępny |
| M301 | PRACA z automatyki |
| M413 | konfiguracja wartości RM5 poprawna |
| M415 | konfiguracja taryfy poprawna |
| M419 | wake request |
| M426 | czas może być wyświetlany |
| M427 | kredyt dostępny |
| D521 | impulsy RM5 ostatniej paczki |
| D522 | impulsy Counter wygenerowane dla ostatniej paczki |
| D524 | impulsy RM5 sesji |
| D525:D526 | oczekiwane impulsy Counter sesji |
| D533 | impulsy Counter faktycznie wysłane Y0 |
| D557 | czas taryfy ostatniego programu |
| D558 | numer ostatniego programu |
| D560 | kredyt wewnętrzny x100 |
| D580 | pełne pozostałe jednostki 0,10 EUR |
| D582 | pozostały kredyt w centach |
| D350 | pozostały czas [s] |

## Wersja PLC

```text
D500..D503 = V1.5.0
D516 = 1
D517 = 4
D518 = 9
```


## Pasek pozostałego czasu

Do paska postępu użyj:
- wartość: `D350` = pozostały czas,
- maksimum dynamiczne: `D559` = TIME_BAR_MAX,
- minimum: 0.

`D559`:
- jest zerowane na początku nowej sesji,
- jest zerowane przy zmianie programu i od razu odbudowywane z czasu nowej taryfy,
- zwiększa się, gdy doładowanie podniesie dostępny czas,
- nie maleje podczas normalnego odliczania.

Dzięki temu pasek pokazuje procent pozostałego czasu aktualnej sesji/programu.


### Auto start program

`M431 = AUTO_START_PROGRAM_ENABLE` — przełącznik HMI.

`D584 = AUTO_START_PROGRAM_NO` — numer programu uruchamianego automatycznie:
- 1 = Program 1,
- 2 = Program 2,
- 3 = Program 3,
- 4 = Program 4,
- 5 = Program 5,
- 6 = Program 6.

Najlepiej ustawić D584 jako pole/selector 16-bit unsigned z zakresem 1…6.

Działanie:
1. klient wrzuca monetę,
2. PLC wysyła pełną kolejkę `Y0/PITSTART_CREDITS`,
3. PLC czeka aż `D330=0`, `M321=0`, `M330=0`, `M331=0`,
4. jeżeli `M431=1`, kredyt jest dostępny, automat ma X0 i żaden program nie jest już aktywny, PLC generuje `M432=AUTO_START_TRIGGER`,
5. D584 wybiera P1…P6,
6. program jest uruchamiany przez tę samą logikę wyboru co przyciski ręczne/HMI.

Jeśli użytkownik ręcznie wybierze program przed końcem wysyłania CREDIT, Auto Start **nie przełącza go później** na inny program.

STOP podczas oczekiwania anuluje automatyczny start dla bieżącej sesji, aby program nie uruchomił się sam po zwolnieniu STOP.

Status:
- `M432` — jednocyklowy impuls Auto Start,
- `M433` — Auto Start wykonany/anulowany dla bieżącej sesji,
- `M434` — D584 ma poprawną wartość 1…6.

Odliczanie czasu nie jest już opóźniane przez M431. Po uruchomieniu programu działa normalnie według `M412` i opcjonalnie `M429/PRACA`.


### Sterowanie ekranem przez D585

`D585 = HMI_SCREEN_INDEX` jest rejestrem tylko do odczytu dla HMI.

Mapowanie:

```text
D585 = 0 -> ekran główny / praca
D585 = 1 -> ekran Admin
D585 = 2 -> ekran "Stanowisko nieczynne"
D585 = 3 -> ekran "Stanowisko wolne"
```

Warunki:

- `D585=1` gdy `X14=1` / `M303=1`,
- `D585=2` gdy Admin nie jest włączony i `X0=0`,
- `D585=3` gdy Admin nie jest włączony, `X0=1` i `M427=0`,
- `D585=0` w pozostałym przypadku, czyli typowo gdy automat jest dostępny i jest kredyt.

Priorytet:

```text
ADMIN > STANOWISKO NIECZYNNE > STANOWISKO WOLNE > MAIN
```

Dzięki temu przy `X0=0` ekran "Stanowisko nieczynne" ma pierwszeństwo nad "Stanowisko wolne", nawet jeśli `M427=0`.

W WSStudio użyj `D585` jako rejestru numeru/indeksu ekranu.


### Domyślne ustawienia

Po uruchomieniu PLC:

```text
M429 = 1   Sync countdown with PRACA
M431 = 1   Auto Start Program enabled
```

Oba bity nadal mogą być zmieniane z HMI podczas pracy.


## Konfiguracja systemowa WSStudio

Projekt HMI używa czterech ekranów:

| Index | Nazwa w projekcie | Funkcja |
|---:|---|---|
| 0 | Home | ekran pracy / wybór programu |
| 1 | Admin | parametry i ustawienia |
| 2 | Error | „Stanowisko nieczynne” |
| 3 | Ready | „Stanowisko wolne” |

### PLC Control

`System Settings -> plcCtrl`:

- **Control Screen Switch**: ON
- adres: `[Mitsubishi_Fx1n]D585`

PLC wpisuje do D585 numer ekranu według warunków procesu.

### HMI State

`System Settings -> HMI Status`:

- **Screen Index**: ON
- adres: `[Mitsubishi_Fx1n]D586`

HMI zapisuje do D586 numer aktualnie wyświetlanego ekranu. D586 jest więc kanałem **HMI -> PLC** i PLC nie może go nadpisywać.

### Wake / impuls RM5

- `M419` — wake request przez około 3 s,
- `D587` — word mirror M419 (0/1),
- `D565` — licznik zaakceptowanych zdarzeń RM5.

Aktualna konfiguracja WSStudio nie ma osobnego, potwierdzonego pola „wake by PLC word”, dlatego D587 pozostaje dostępny do diagnostyki lub przyszłej konfiguracji.


## Podgląd ekranów

### 000: Home

![Home](images/hmi/home.png)

Ekran pracy zawiera wybór programu, pozostały czas, pasek czasu i STOP.

### 001: Admin

![Admin](images/hmi/admin.png)

Ekran administracyjny zawiera:
- Touch control,
- Work lights,
- Pilotaggio pompa,
- Sync time with RUN,
- Auto start program,
- D300, D549, D550, D528,
- D551…D556,
- D584,
- Touch Calibration.

### 002: Error — Stanowisko nieczynne

![Stanowisko nieczynne](images/hmi/station_inactive.png)

Wyświetlany przy `D585=2`, czyli gdy Admin jest wyłączony i `X0=0`.

### 003: Ready — Stanowisko wolne

![Stanowisko wolne](images/hmi/station_free.png)

Wyświetlany przy `D585=3`, czyli gdy Admin jest wyłączony, `X0=1` i `M427=0`.

## Konfiguracja System Settings

Widok konfiguracji:

![System Parameters](images/hmi/system_parameters.png)

### PLC Control / Control Screen Switch

![PLC Control](images/hmi/plc_control_screen_switch.png)

Ustaw:
- **Control Screen Switch**: ON,
- adres: `[Mitsubishi_Fx1n]D585`.

D585 jest indeksem ekranu zadawanym przez PLC:

| D585 | Ekran |
|---:|---|
| 0 | 000: Home |
| 1 | 001: Admin |
| 2 | 002: Error / Stanowisko nieczynne |
| 3 | 003: Ready / Stanowisko wolne |

### HMI Status / Screen Index

![HMI State](images/hmi/hmi_state_screen_index.png)

Ustaw:
- **Screen Index**: ON,
- adres: `[Mitsubishi_Fx1n]D586`.

Kierunek tego rejestru jest przeciwny do D585:
- `D585`: PLC -> HMI, żądany ekran,
- `D586`: HMI -> PLC, aktualnie wyświetlany ekran.

PLC nie zapisuje D586.


### Limit kredytu i RM5

`D549` ma zakres 1…999 w jednostkach 0,10 EUR. Przykład:

```text
D549 = 50  -> 5,00 EUR
D549 = 999 -> 99,90 EUR
```

Po osiągnięciu limitu:
- `M435=1`,
- `Y2=1` i `Y23=1`,
- RM5 jest zablokowany przez INHIBIT,
- kolejne impulsy X27 nie są przyjmowane przez PLC.

W ekranie Admin ustaw dla pola D549:
- DataType: 16-Bit Unsigned Int,
- Min: 1,
- Max: 999.

### Doładowanie podczas aktywnego programu

Dołożenie monet nie zmienia aktywnego programu. Kredyt jest dopisywany do bieżącej sesji, a pozostały czas zwiększa się dla aktualnie wybranej taryfy.

Auto Start Program może wystąpić tylko wtedy, gdy `D558=0`, czyli w sesji nie został jeszcze wybrany program. Po wybraniu programu kolejne impulsy RM5 nie mogą przełączyć go na D584.

### Restart PLC

Na pierwszym skanie PLC zerowane są dane bieżącej sesji i liczniki runtime, w tym:
- D320, D330,
- D350, D540, D541,
- D521…D526,
- D533…D535,
- D557…D563,
- D565…D583,
- aktywne programy M420…M425.

Po restarcie nie powinien pozostać kredyt ani czas poprzedniej sesji.
