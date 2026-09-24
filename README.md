# PLC_HMI_PITSTART

Projekt integracji sterownika PLC/HMI SEEKU (PLC zgodny z Mitsubishi FX3U), akceptora monet **Comestero RM5 Evolution**, sterownika myjni oraz wejścia impulsowego **PITStart**.

## Status projektu

Aktualna drabinka:
- odbiera statusy z wejść PLC,
- blokuje / odblokowuje RM5 przez wejście `INHIBIT`,
- odbiera impulsy RM5 na `X27`,
- grupuje impulsy w paczkę zakończoną po ok. 1 s bezczynności,
- mnoży liczbę impulsów przez parametr `D300`,
- przechowuje wynik w `D350`,
- kopiuje liczbę impulsów do kolejki `D330`,
- wysyła kolejkę CREDIT na `Y0` jako impulsy 0,1 s ON / 0,1 s OFF,
- używa `Y1` jako opcjonalnego `Pilotaggio pompa`,
- używa `Y2...Y7` jako Program 1...6,
- używa `Y27` jako oświetlenia aktywnego podczas pracy,
- może odliczać wartość `D350`.

## Oprogramowanie

### PLC
Do edycji programu PLC:
- **MELSOFT GX Developer** — główne środowisko używane w projekcie.
- GX Works2 może być używany pomocniczo dla sterowników kompatybilnych z FX3U.

Typowe ustawienie programowania:
- interfejs: RS232 / USB→RS232,
- PLC side: `FXCPU`,
- prędkość: 38.4 kbps,
- kabel RS232: 2↔2, 3↔3, 5↔5.

### HMI
Do edycji HMI:
- **WSStudio** / pakiet HMI firmy Winsun/SEEKU.

HMI i PLC są programowane osobno.

## Najważniejsze adresy

| Adres | Znaczenie |
|---|---|
| `X0` | status 1 / `INVERTER_OK` |
| `X1` | status 2 / `PRACA` |
| `X2` | status 3 |
| `X3` | status 4 |
| `X27` | wejście impulsów z RM5 |
| `Y0` | CREDIT / COMPTEUR |
| `Y1` | PILOTAGGIO POMPA |
| `Y2...Y7` | Program 1...6 |
| `Y27` | oświetlenie w czasie pracy |
| `Y23` | sterowanie `INHIBIT` RM5 |
| `M300` | kopia/status X0 |
| `M301` | kopia/status X1 |
| `M302` | kopia/status X2 |
| `M303` | kopia/status X3 |
| `M304` | status złożony; aktualnie `X0 AND /X1 AND X2 AND X3` |
| `M320` | aktywny stan / impuls RM5 z X27 |
| `M321` | aktywna paczka impulsów RM5 |
| `M330` | faza ON generatora Y1 |
| `M331` | faza OFF generatora Y1 |
| `D300` | mnożnik / parametr roboczy |
| `D320` | licznik impulsów bieżącej paczki RM5 |
| `D330` | kolejka impulsów do wysłania na Y1 |
| `D350` | wynik `D320 × D300` / wartość robocza HMI |
| `T200` | timeout końca paczki RM5, `K100` ≈ 1 s |
| `T201` | czas ON Y1, `K10` ≈ 0,1 s |
| `T202` | czas OFF Y1, `K10` ≈ 0,1 s |

Pełna tabela: `IO_TABLE.txt`.

## Logika RM5

Aktualnie używany jest jeden kanał RM5.

1. RM5 generuje impulsy na `CH1`.
2. `CH1` jest podłączony do `X27`.
3. Każdy impuls zwiększa `D320`.
4. `M321` uruchamia `T200`.
5. Każdy kolejny impuls zeruje `T200`.
6. Po 1 s bez impulsu paczka zostaje zakończona.
7. Wykonywane jest:
   - `MUL D320 D300 D350`
   - `MOV D320 D330`
   - `MOV K0 D320`
   - `RST M321`
8. `D330` jest kolejką impulsów.
9. Generator CREDIT na Y0 pracuje dopóki `D330 > 0`:
   - ON: `T201 K10` ≈ 0,1 s,
   - `D330 = D330 - 1`,
   - OFF: `T202 K10` ≈ 0,1 s.
10. Okres start-start wynosi około 0,2 s.

## RM5 Evolution — standardowe złącze 10-pin

| Pin RM5 | Funkcja |
|---:|---|
| 1 | GND |
| 2 | +12…24 VDC |
| 3 | CH5 |
| 4 | CH6 |
| 5 | N.U. / zależne od konfiguracji |
| 6 | INHIBIT |
| 7 | CH1 |
| 8 | CH2 |
| 9 | CH3 |
| 10 | CH4 |

Aktualne połączenia:
- pin 7 `CH1` → PLC `X27`,
- pin 6 `INHIBIT` ← PLC `Y23`,
- pin 1 `GND` → 0 V / COM wejść PLC,
- pin 2 → +12…24 VDC.

## Istniejąca logika analogowa

Oryginalny program:
- `RD3A` odczytuje 6 kanałów do `D10, D0, D1, D2, D3, D4`,
- `WR3A` zapisuje `D6` i `D7` na dwa wyjścia analogowe,
- `M0/M1/Y4` wpływają na `D6`,
- `M2/M3/Y4` wpływają na `D7`.

`Y4` oraz `D0…D10` należy traktować jako zajęte.

## Uwaga o MUL

`MUL D320 D300 D350` daje wynik 32-bitowy w parze `D350:D351`. Przy małych wartościach używane jest dolne słowo `D350`, ale należy kontrolować przepełnienie.

## Odliczanie D350

W standardowym Mitsubishi FX:
- `M8011` — zegar 10 ms,
- `M8012` — zegar 100 ms,
- `M8013` — zegar 1 s.

Jeżeli `D350` ma być zmniejszane dokładnie raz na sekundę, należy użyć zbocza `M8013` i warunku `D350 > K0`.

## Dokumentacja

- RM5: https://www.casino-software.de/download/manual_rm5.pdf
- RM5 Evolution: https://eu.suzohapp.com/pdf/manual/OM_RM5_Evolution__EN.pdf

## Struktura

- `IO_TABLE.txt` — mapa I/O, M, D i T.
- `PROJECT_DESCRIPTION.md` — opis projektu.
- `docs/RM5.md` — RM5, pinout i inhibit.
- `docs/WIRING.md` — schemat połączeń.
- `docs/LADDER_LOGIC.md` — opis drabinki.
- `docs/wiring.svg` — schemat SVG.


## Opcjonalne kanały RM5 przez AD0...AD5

Kanały analogowe mogą zostać użyte jako dodatkowe wejścia RM5 tylko po dopasowaniu elektrycznym. RM5 daje sygnały NPN open-collector, natomiast WSB ma wejścia analogowe. W typowej konfiguracji WSB AD0...AD2 są 0-10V, a AD3...AD5 0-20mA. AD0...AD2 można wykorzystać przez pull-up/interfejs 0-10V i detekcję progu w PLC. AD3...AD5 wymagają konwersji na sygnał prądowy albo odpowiedniej rekonfiguracji kanału, jeśli wersja sprzętu ją obsługuje.

Aktualne odczyty RD3A:
- AD0 -> D10
- AD1 -> D0
- AD2 -> D1
- AD3 -> D2
- AD4 -> D3
- AD5 -> D4

Szczegóły: `docs/RM5.md` i `docs/LADDER_LOGIC.md`.
