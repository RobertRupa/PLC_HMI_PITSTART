# PLC_HMI_PITSTART

Projekt integracji sterownika PLC/HMI SEEKU (PLC zgodny z rodziną Mitsubishi FX), akceptora monet **Comestero RM5 Evolution** oraz sterownika myjni w sposób zgodny funkcjonalnie z **Comestero PitStart**.

## Oprogramowanie

### PLC
Do edycji programu PLC używany jest:
- **MELSOFT GX Developer**

Połączenie programujące używane w projekcie:
- RS232 / USB→RS232,
- PLC side: `FXCPU`,
- 38.4 kbps.

### HMI
Do edycji panelu HMI:
- **WSStudio** firmy Winsun/SEEKU.

PLC i HMI są programowane osobno.

## Aktualna architektura

### Wejścia

| Adres | Funkcja |
|---|---|
| `X0` | `AUTOMATE_PRESENT / INVERTER_OK` — główny sygnał zezwolenia |
| `X1` | `PRACA` — sygnał informacyjny; nie blokuje RM5 |
| `X2` | rezerwa / przyszły status |
| `X3` | rezerwa / przyszły status |
| `X27` | impulsy z jednego kanału RM5 |

### Wyjścia

| Adres | Funkcja |
|---|---|
| `Y0` | CREDIT / COMPTEUR — impulsy do sterownika myjni |
| `Y1` | PILOTAGGIO POMPA |
| `Y2` | Program 1 |
| `Y3` | Program 2 |
| `Y4` | Program 3 |
| `Y5` | Program 4 |
| `Y6` | Program 5 |
| `Y7` | Program 6 / rezerwa, jeśli HMI ma tylko P1…P5 |
| `Y23` | RM5 INHIBIT |
| `Y27` | oświetlenie podczas pracy |

## RM5 — aktualna logika

Aktualnie używany jest jeden kanał RM5 podłączony do `X27`.

1. Impuls RM5 jest wykrywany na `X27`.
2. `M320` jest bitem roboczym impulsu.
3. Każdy impuls zwiększa `D320`.
4. `M321` oznacza aktywne zbieranie paczki.
5. Każdy kolejny impuls restartuje `T200`.
6. Po około 1 s bez impulsu `T200` kończy paczkę.
7. `D320 × D300` jest zapisywane do `D350`.
8. Liczba impulsów `D320` jest kopiowana do `D330`.
9. `D320` jest zerowane.
10. `D330` steruje generatorem CREDIT na `Y0`.

Aktualna sekwencja końca paczki:

```text
T200 -> MUL D320 D300 D350
T200 -> MOV D320 D330
T200 -> MOV K0 D320
T200 -> RST M321
```

## Generator CREDIT — Y0

```text
D330 > 0 AND /M330 AND /M331 -> SET M330

M330 -> Y0
M330 -> T201 K10

T201 -> DEC D330
T201 -> RST M330
T201 -> SET M331

M331 -> T202 K10
T202 -> RST M331
```

Przy aktualnym ustawieniu:
- `T201 K10` ≈ 0,1 s ON,
- `T202 K10` ≈ 0,1 s OFF,
- początek kolejnych impulsów wypada co około 0,2 s.

**Uwaga:** oryginalny PitStart opisuje wyjście Counter jako impuls za każde 0,10 € przyjętej wartości. Obecna implementacja wysyła na Y0 liczbę impulsów zapisaną w `D330` = bieżące `D320`. Jeżeli później ma być emulowana dokładnie semantyka 0,10 €/impuls, należy odpowiednio przeliczyć wartość ładowaną do `D330`.

## RM5 INHIBIT — Y23

```text
/M300 -> Y23
```

Założenie:
- `X0/M300 = 1` — automat/myjnia dostępna, RM5 aktywny,
- `X0/M300 = 0` — Y23 aktywuje INHIBIT i blokuje RM5.

`X1/PRACA` nie uczestniczy w blokowaniu RM5.

## Odliczanie wartości D350

Jeżeli `D350` ma maleć o 1 dokładnie raz na sekundę:

```text
LDP M8013
AND> D350 K0
DEC D350
```

Dzięki warunkowi `D350 > 0` licznik nie schodzi poniżej zera.

## Pilotaggio pompa — Y1

Oryginalny PitStart uruchamia wyjście „pilotaggio pompa” przy żądaniu pierwszego programu, utrzymuje je dopóki jest dostępny kredyt i wyłącza po STOP.

W projekcie:
- `M410 = PILOTAGGIO_ENABLE` — opcja z HMI,
- `M412 = WORK_ACTIVE` — aktywna praca.

```text
M410 AND M412 -> Y1
```

## Oświetlenie — Y27

```text
M412 -> Y27
```

Oświetlenie działa zawsze podczas `WORK_ACTIVE`, niezależnie od tego, czy Pilotaggio jest włączone.

## Programy i HMI

Planowany ekran HMI:
- STOP,
- Program 1,
- Program 2,
- Program 3,
- Program 4,
- Program 5,
- opcjonalnie Program 6,
- opcja Pilotaggio,
- mnożnik `D300`,
- pozostała wartość `D350`,
- diagnostycznie `D320` i `D330`.

Zalecane bity HMI:
- `M400` STOP,
- `M401…M406` Program 1…6,
- `M410` Pilotaggio enable,
- `M412` Work active.

## Co wynika z dokumentacji PitStart

Dla PitStart potwierdzone są:
- wyjścia suchego styku NO: Counter, Pump/Pilotaggio, P1…P6,
- wejście `Presence automatic device` na CN8 1–2,
- wejście `Manual/Free` na CN8 3–4.

Szczegóły i rozpiska CN4/CN5/CN8: `docs/PITSTART.md`.

## RM5 — podstawowe piny

| Pin | Funkcja |
|---:|---|
| 1 | GND |
| 2 | +12…24 VDC |
| 6 | INHIBIT |
| 7 | CH1 |
| 8 | CH2 |
| 9 | CH3 |
| 10 | CH4 |
| 3 | CH5 |
| 4 | CH6 |

W tym projekcie używany jest tylko jeden kanał:
- RM5 pin 7 CH1 → `X27`,
- RM5 pin 6 INHIBIT ← `Y23`.

## Dokumentacja

- `IO_TABLE.txt` — aktualna mapa projektu.
- `PROJECT_DESCRIPTION.md` — opis funkcjonalny.
- `docs/PITSTART.md` — potwierdzona funkcjonalność i złącza PitStart.
- `docs/RM5.md` — RM5.
- `docs/WIRING.md` — połączenia elektryczne.
- `docs/LADDER_LOGIC.md` — logika drabinki.
- `docs/wiring.svg` — uproszczony schemat.

Źródła:
- Comestero Gamma Pit / PitStart Operating Manual, Rev. 00, 13/10/2009.
- RM5: https://www.casino-software.de/download/manual_rm5.pdf
