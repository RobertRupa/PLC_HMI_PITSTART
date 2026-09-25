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

W naszym projekcie:
- `X13 = MANUAL / FREE`,
- `M302 = status MANUAL_FREE` do odczytu przez logikę/HMI.

## 4. Sygnał PRACA

`X1 = PRACA` pochodzi z naszego sterownika myjni.

Nie jest on opisany w manualu jako standardowe wejście PitStart. Traktujemy go jako dodatkowy feedback/status i **nie używamy go do blokowania RM5**.

## 5. Mapowanie naszego PLC

| Funkcja PitStart | PLC |
|---|---|
| Manual / Free | X13 / M302 |
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


## 8. Sterowanie lokalne bez HMI

Aktualny projekt dodaje mechaniczne przyciski:
- X4 STOP,
- X5 P1,
- X6 P2,
- X7 P3,
- X10 P4,
- X11 P5,
- X12 P6.

RM5 CH1 jest przypisany do X27.

Są one logicznie łączone z przyciskami HMI, dzięki czemu panel HMI nie jest wymagany do podstawowej obsługi stanowiska.


## 9. Różnice V1.2.0 względem oryginalnego PitStart

### Manual / Free

Oryginalny PitStart podczas aktywnego Manual / Free może pracować ciągle bez monety. W aktualnej wersji PLC **V1.2.0**:

```text
X13 -> M302
```

jest tylko statusem dla PLC/HMI. Nie omija warunku `D350>0` i nie uruchamia programu bez kredytu.

### Counter / Credit

Manual PitStart opisuje Counter jako impulsy odpowiadające wielokrotnościom 0,10 EUR. Aktualna implementacja PLC wysyła na `Y0` **jeden impuls za każdy zaakceptowany impuls CH1 na X27**:

```text
D330 = liczba impulsów ostatniej paczki RM5
```

Dlatego zgodność wartości pieniężnej Y0 zależy od konfiguracji RM5 CH1. `D300=10` nie zmienia liczby impulsów Y0; aktualna konwersja czasu korzysta z `D520`.
