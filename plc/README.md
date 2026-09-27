# PLC V1.5.6

## Pliki

- `MAIN_GXDEV_ENTRY.txt` — aktualna lista instrukcji do GX Developer,
- `MAIN.txt` — wersja komentowana,
- `main_v1.5.5.csv` — ostatni zapisany CSV do importu GX; źródła V1.5.6 są w `MAIN_GXDEV_ENTRY.txt` i `MAIN.txt`,
- `DEVICE_MAP.csv` — mapa urządzeń,
- `DEVICE_COMMENTS.csv` / `DEVICE_COMMENTS.txt` — komentarze urządzeń,
- `IMPORT_CSV.md` — mapowanie kolumn importu.

## Aktualna mapa wyjść

```text
Y0   PITSTART_CREDITS
Y1   PILOTAGGIO
Y2   RM5_INHIBIT_PRIMARY
Y3   PROGRAM_1
Y4   PROGRAM_2
Y5   PROGRAM_3
Y6   PROGRAM_4
Y7   PROGRAM_5
Y10  PROGRAM_6
Y23  RM5_INHIBIT_COMPAT
Y27  WORK_LIGHTS
```

FX numeruje X/Y ósemkowo, dlatego po Y7 występuje Y10.

Y2 i Y23 mają identyczną logikę INHIBIT:

```text
X0=0 OR M435=1 -> ON
X0=1 AND M435=0 -> OFF
```

Na obecnym sprzęcie używany jest Y2; Y23 pozostaje dla przyszłego sterownika z większą liczbą wyjść.

## RM5

```text
RM5 CH1 -> X27
RM5 INHIBIT <- Y2
```

Wszystkie używane kanały/nominały RM5 muszą generować impulsy na CH1. PLC nie odczytuje CH2…CH6.

`D300` określa wartość pojedynczego impulsu CH1 w jednostkach po 0,10 EUR.

## Parametry

```text
D300 = RM5 CH1 value [x0,10 EUR]
D528 = clock correction x100
D549 = max credit [x0,10 EUR], 1..999
D550 = base price [x0,10 EUR]
D551..D556 = P1..P6 time [s]
D584 = Auto Start Program 1..6
```

## HMI

```text
D350 = remaining time [s]
D540 = minutes
D541 = seconds
D559 = TIME_BAR_MAX
D560 = remaining credit cents
D582 = credit cents mirror
M435 = max credit reached
D585 = HMI_SCREEN_INDEX (0 Main, 1 Admin, 2 inactive, 3 free)
D586 = HMI_CURRENT_SCREEN_INDEX (HMI -> PLC)
D587 = HMI_WAKE_REQUEST_WORD
```

## Import CSV

Kolumny pliku CSV importowanego do GX Developer:

- A = Step number
- B = Skip
- C = Instruction
- D = I/O(Device)
- E…I = Skip

Komentarze urządzeń importuj osobno, jeśli używana wersja GX Developer na to pozwala.


## Defaults V1.5.6

```text
M429 = 1 default  ; Sync countdown with PRACA
M431 = 1 default  ; Auto Start Program
```


## RM5 top-up

```text
M435 = MAX_CREDIT_REACHED
M436 = RM5_TOPUP_BUSY
```

Nowa paczka RM5 jest dopisywana do istniejącej kolejki D330. Liczba nowych impulsów Y0 jest ograniczana do wolnego miejsca wynikającego z D549.

Przy M436=1 tick zużycia kredytu jest wstrzymany do końca obsługi zaakceptowanego doładowania. Aktywny program i D558 nie są zmieniane przez samo doładowanie.


## Startup reset

```text
T204 = startup reset window
M438 = STARTUP_RESET_ACTIVE
```

Przez początkowe okno po wejściu PLC w RUN zerowane są rejestry i bity bieżącej sesji. Reset nie zależy wyłącznie od M8002.

## RM5 inhibit threshold

`D594` zawiera liczbę pełnych impulsów Y0, które jeszcze mieszczą się poniżej D549.

`M435` jest ustawiane, gdy:
- D594 jest mniejsze od D300,
- albo bieżąca paczka RM5 osiągnęła dostępne D594.

Wtedy Y2 i Y23 przechodzą w stan INHIBIT.


## STOP during RUN

```text
M448 = STOP_RUN_COUNTDOWN
M449 = COUNTDOWN_ACTIVE
```

Jeżeli STOP zostanie naciśnięty przy aktywnym M301/RUN i M412/WORK_ACTIVE, M448 podtrzymuje zużycie kredytu do zaniku RUN. Wyjścia programu pozostają wyłączone, ponieważ nadal zależą od M412.

## Pilotaggio default

```text
M439 = PILOTAGGIO_DEFAULT
```

M439 jest HMI RW i domyślnie ON. Po zakończeniu kolejki Y0, jeśli M410 jest OFF, bieżące M439 decyduje o przywróceniu Pilotaggio.

## Default tariff

```text
D300 = 10
D550 = 10
D551..D556 = 300
```

1 raw RM5 pulse = 10 Y0 pulses = 300 s.
