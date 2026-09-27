# Projekt HMI — pitstart.hs

## Plik źródłowy

Aktualny edytowalny projekt:

```text
projects/hmi/pitstart.hs
```

Plik jest obecny w repo i ma rozmiar 1 177 910 B.

```text
Git blob: 2fa650d6e956c4f8e480eb724d410f79400b2e97
SHA-256:  18f415d87c903069cb58ad295e9d1d824015716edd6013c4ed2f6fd358e0438b
```

## Dane odczytane z projektu

Nagłówek i zasoby pliku zawierają:

```text
Editor / format: KinSealStudio V1.0.2
Panel profile:   SUP070
Profile string:  wsb-070-16M
Display:         7.0 inch
Resolution:      800 x 480
Supply:          DC24V (+/-15%)
Ports:           COM1 / COM2
Download:        USB device
Language:        English
PLC driver:      Mitsubishi_Fx1n
```

W pliku występuje również ścieżka zasobów `WSZKHMI5.1En`. Manual producenta dla serii WSB używa nazwy HMI Studio / Weisheng 5.1. `KinSealStudio V1.0.2` jest identyfikatorem zapisanym w samym pliku projektu.

Ciąg opisujący ekran/kolory w nagłówku projektu jest metadanym profilu programu. Nie należy go traktować jako pewnej identyfikacji wariantu sprzętowego bez odczytu etykiety fizycznego panelu.

## Wersja HMI i PLC

Na ekranie projektu znajduje się stały tekst:

```text
HMI: V1.0.0
```

Jest to wersja interfejsu HMI. Wersja programu PLC jest osobna i aktualnie wynosi **V1.5.7**.

## Ekrany

```text
000 Home
001 Admin
002 Error / Stanowisko nieczynne
003 Ready / Stanowisko wolne
```

Sterowanie ekranami:
- `D585` — PLC -> HMI, żądany indeks ekranu,
- `D586` — HMI -> PLC, aktualny indeks ekranu.

## Główne adresy PLC używane przez HMI

```text
M400        STOP
M401..M406 P1..P6
M410        PILOTAGGIO_ENABLE
M414        WORK_LIGHTS_ENABLE
M429        SYNC_COUNTDOWN_WITH_PRACA
M431        AUTO_START_PROGRAM_ENABLE
M439        PILOTAGGIO_DEFAULT

D300        Coin multiplier
D528        Time correction
D549        Max credits
D550        Base price
D551..D556 P1..P6 time
D584        Auto Start Program
D585        PLC -> HMI screen index
D586        HMI -> PLC screen feedback
```

Dodatkowo w pliku projektu występują lokalne obiekty HMI, m.in. `LB555` przy funkcji Touch control oraz `LW4057` przy Touch Calibration. Nie są to urządzenia PLC.

Pełna dokumentacja: [../../docs/HMI.md](../../docs/HMI.md).
