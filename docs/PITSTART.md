# Comestero PitStart — informacje użyte w projekcie

Źródło: **Gamma Pit – Sistemi di attivazione – Manuale operativo**, PitStart-F, Rev. 00, 13/10/2009.

## 1. Wyjścia PitStart

Manual podaje, że wszystkie wyjścia PitStart są realizowane jako **suche styki normalnie otwarte (NO)**.

Dostępne funkcje:
- **Compteur / Counter** — impulsy odpowiadające wielokrotnościom 0,10 € przy każdym przyjęciu wartości,
- **Pilotaggio pompa** — aktywowane przy żądaniu pierwszego programu; pozostaje aktywne, dopóki jest kredyt; STOP je wyłącza,
- **P1…P6** — wyjścia programów mycia.

## 2. Fig. 33 — złącza CN4 i CN5

Numeracja na rysunku ma pin `1` po prawej stronie każdego złącza.

### CN4

| Piny | Funkcja |
|---|---|
| 1 + 3 | P1 |
| 4 + 6 | P2 |
| 7 + 9 | P3 |
| 10 + 12 | P4 |

Piny 2, 5, 8 i 11 nie są użyte w pokazanym schemacie.

Dla P1…P4 jedna strona styku jest połączona wspólną magistralą `COMMUN`.

### CN5

| Piny | Funkcja |
|---|---|
| 1 + 3 | P5 |
| 4 + 6 | P6 |
| 7 + 9 | Pilotaggio pompa |
| 10 + 12 | Compteur / Counter |

Piny 2, 5, 8 i 11 nie są użyte w pokazanym schemacie.

## 3. Fig. 34 — złącze CN8

### Presence automatic device

- CN8-1 = +24 VDC
- CN8-2 = GND

Brak tego sygnału może przełączyć PitStart w stan poza obsługą. Wymóg obecności automatu może być wyłączony w MultiConfig.

W naszym projekcie najbliższym odpowiednikiem jest:
- `X0 = AUTOMATE_PRESENT / INVERTER_OK`.

### Manual / Free

- CN8-3 = +24 VDC
- CN8-4 = GND

Dopóki wejście jest aktywne, PitStart pracuje w trybie ciągłym bez monety.

To wejście nie ma jeszcze przypisanego finalnego adresu X w naszym projekcie.

## 4. Sygnał PRACA

`X1 = PRACA` pochodzi z naszego sterownika myjni.

Nie jest on opisany w manualu jako standardowe wejście PitStart. Traktujemy go jako dodatkowy feedback/status i **nie używamy go do blokowania RM5**.

## 5. Mapowanie naszego PLC

| Funkcja PitStart | PLC |
|---|---|
| Counter / Credit | Y0 |
| Pilotaggio pompa | Y1 |
| P1 | Y2 |
| P2 | Y3 |
| P3 | Y4 |
| P4 | Y5 |
| P5 | Y6 |
| P6 | Y7 |
| RM5 inhibit | Y23 |
| Oświetlenie podczas pracy | Y27 |

## 6. Pilotaggio w projekcie

- `M410 = PILOTAGGIO_ENABLE`
- `M412 = WORK_ACTIVE`

```text
M410 AND M412 -> Y1
```

## 7. Oświetlenie

```text
M412 -> Y27
```

## 8. Czego nie przenosimy z demo

Finalny projekt nie wykorzystuje:
- AD0…AD5,
- RD3A,
- WR3A,
- D0…D10 z demo-bloku analogowego,
- testowych zależności Y4 -> D6/D7.

Dzięki temu Y4 może być użyte bez konfliktu jako Program 3.
