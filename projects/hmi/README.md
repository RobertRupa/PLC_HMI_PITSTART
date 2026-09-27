# Projekt HMI

## Główny plik projektu

Aktualny edytowalny projekt HMI:

```text
pitstart.hs
```

Plik należy otwierać w **KinSealStudio** / oprogramowaniu zgodnym z formatem `.hs`.

> Plik `pitstart.hs` jest dodawany do repo ręcznie. Dokumentacja w repo zakłada ścieżkę `projects/hmi/pitstart.hs`.

## Parametry odczytane z projektu

Z nagłówka aktualnego pliku `.hs`:

```text
Editor / format: KinSealStudio V1.0.2
Panel profile:   SUP070
Hardware:        wsb-070-16M
Display:         7.0 inch
Resolution:      800 x 480
Display type:    1667M Colors 7 TFT LCD
Supply:          DC24V (+/-15%)
Ports:           COM1 / COM2
Download:        USB device
Language:        English
PLC driver:      Mitsubishi_Fx1n
```

W projekcie występują również obiekty powiązane z urządzeniami Mitsubishi FX, m.in. adresy X/Y/M/D używane przez aktualną logikę PLC.

## Powiązanie z PLC

Aktualna mapa HMI i PLC jest opisana w:

- [../../docs/HMI.md](../../docs/HMI.md)
- [../../docs/LADDER_LOGIC.md](../../docs/LADDER_LOGIC.md)
- [../../plc/DEVICE_MAP.csv](../../plc/DEVICE_MAP.csv)

Najważniejsze adresy HMI używane przez bieżący projekt:

```text
M400        STOP
M401..M406 P1..P6
M410        PILOTAGGIO_ENABLE
M414        WORK_LIGHTS_ENABLE
M429        SYNC_WITH_PRACA / Sync time with RUN
M431        AUTO_START_PROGRAM_ENABLE
D300        Coin multiplier
D528        Time correction
D549        Max credits
D550        Base price
D551..D556 P1..P6 time
D584        Auto Start Program number
D585        PLC -> HMI screen index
D586        HMI -> PLC current screen index
D587        HMI wake request
```

## Ekrany

Projekt korzysta z czterech ekranów operatorskich:

```text
000 Home
001 Admin
002 Error / Stanowisko nieczynne
003 Ready / Stanowisko wolne
```

Opis działania ekranów i zrzuty konfiguracji znajdują się w [../../docs/HMI.md](../../docs/HMI.md).

## Archiwum

`pitstart_hmi.zip` pozostaje w repo jako wcześniejsze archiwum transportowe. Przy dalszej edycji jako źródło należy traktować `pitstart.hs`.
