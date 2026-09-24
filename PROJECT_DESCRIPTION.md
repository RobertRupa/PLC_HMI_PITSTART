# Opis projektu PLC_HMI_PITSTART

## Cel

Projekt łączy:
1. sterownik myjni,
2. PLC/HMI SEEKU kompatybilny z Mitsubishi FX,
3. akceptor monet Comestero RM5 Evolution,
4. wejście impulsowe PITStart.

PLC odbiera stany maszyny i impulsy RM5, kontroluje blokadę akceptora, przelicza kredyt oraz generuje impulsy dla PITStart.

## Przepływ sygnału

```text
Sterownik myjni
   | X0..X3
   v
PLC SEEKU <---- X27 ---- RM5 CH1
   |
   +---- Y23 ---------> RM5 INHIBIT
   |
   +---- Y0 ----------> CREDIT / COMPTEUR
   +---- Y1 ----------> PILOTAGGIO POMPA
   +---- Y2..Y7 ------> PROGRAM 1..6
   +---- Y27 ---------> OŚWIETLENIE
   |
   +---- D/M <--------> HMI WSStudio
```

## Zbieranie impulsów RM5

`X27` jest wejściem impulsowym jednego kanału RM5.

Każde wykrycie `M320`:
- zwiększa `D320`,
- ustawia `M321`,
- zeruje `T200`.

`T200 K100` jest timeoutem paczki. Po ok. 1 s bez kolejnego impulsu paczka zostaje zakończona.

## Przeliczenie

```text
MUL D320 D300 D350
MOV D320 D330
MOV K0 D320
RST M321
```

- `D320` — liczba impulsów RM5,
- `D300` — mnożnik / parametr z HMI,
- `D350` — wynik,
- `D330` — kolejka impulsów do PITStart.

## Generator PITStart

- `M330` — faza ON,
- `M331` — faza OFF,
- `T201 K10` — ok. 100 ms ON,
- `T202 K10` — ok. 100 ms OFF.

Dopóki `D330 > 0`, generator wysyła kolejne impulsy CREDIT na `Y0`.

## Blokada RM5

`Y23` steruje wejściem `INHIBIT` RM5.

Aktualna logika:

```text
/M300 -> Y23
```

Brak `INVERTER_OK` powoduje załączenie `Y23`, podanie HIGH na INHIBIT i zablokowanie RM5.

## HMI

WSStudio powinno udostępniać:
- M300...M303 — statusy,
- D300 — mnożnik,
- D350 — wynik / wartość,
- opcjonalnie D320 i D330 — diagnostyka.

Planowane przyciski:
- STOP,
- Program 1...5.

## Oryginalna logika analogowa

Nie usuwać bloku `RD3A/WR3A`.

Odczyty:
- AI0 -> D10
- AI1 -> D0
- AI2 -> D1
- AI3 -> D2
- AI4 -> D3
- AI5 -> D4

Wyjścia:
- D6 -> AO0
- D7 -> AO1

`Y4` jest już używane przez logikę D6/D7.


## Oświetlenie Y27

`Y27` jest przewidziane jako wyjście oświetlenia aktywne podczas pracy. Zalecany bit pośredni:

```text
M412 = WORK_ACTIVE
M412 -> Y27
```

`WORK_ACTIVE` jest ustawiane przy aktywnym programie lub trybie manual/free i kasowane przez STOP albo koniec pracy.

## Pilotaggio

```text
M410 = PILOTAGGIO_ENABLE
M410 AND M412 -> Y1
```

Pozwala to włączyć/wyłączyć Pilotaggio z HMI bez wyłączania oświetlenia.

## Opcjonalne wejścia RM5 przez AD0...AD5

Możliwe są wejścia pomocnicze z kanałów RM5 przez AD, ale wyłącznie po dopasowaniu elektrycznym sygnału open-collector RM5 do wejścia analogowego. AD0...AD2 są typowo wejściami napięciowymi 0-10V; AD3...AD5 są typowo wejściami prądowymi 0-20mA.
