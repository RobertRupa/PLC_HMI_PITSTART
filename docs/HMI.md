# HMI — WSStudio — V1.2.0

## Wersja PLC

HMI czyta string od D500:

| Adres | Zawartość |
|---|---|
| D500 | `V1` |
| D501 | `.2` |
| D502 | `.0` |
| D503 | terminator |
| D516 | major = 1 |
| D517 | minor = 2 |
| D518 | patch = 0 |

Kod PLC:

```text
MOV H3156 D500
MOV H322E D501
MOV H302E D502
MOV H0000 D503
```

W WSStudio użyj ASCII/String Display od D500, np. 16 znaków, RO.

Jeżeli pary znaków są odwrócone, należy odwrócić kolejność bajtów stałych HEX.

## Sterowanie

| Adres | Funkcja | Dostęp |
|---|---|---|
| M400 | STOP | momentary |
| M401…M406 | P1…P6 | momentary |
| M410 | Pilotaggio enable | RW |
| M411 | Reset licznika sesji | momentary |
| M413 | Parametr D520 OK | RO |
| M302 | Manual / Free status | RO |

M410 startuje w PLC jako 1. HMI może je przełączyć na 0.

## Czas / impuls

| Adres | Nazwa | Typ | Dostęp |
|---|---|---|---|
| D520 | Czas / impuls [s] | 16-bit | RW |
| D350 | Pozostały czas [s] | 16-bit | RO |
| D521 | Ostatnia paczka [imp] | 16-bit | RO |
| D522:D523 | Ostatnia paczka [s] | 32-bit | RO |
| D524 | Licznik impulsów sesji | 16-bit | RO |
| D525:D526 | Czas równoważny sesji [s] | 32-bit | RO |

D520:
- min 1,
- max 600,
- domyślnie 10 jeśli PLC wykryje wartość niepoprawną.

Jeżeli D520 jest poza zakresem, M413=0 i RM5 jest blokowany.

### Retencja

PLC zachowuje poprawne D520 podczas pierwszego skanu, ale retencja po zaniku zasilania zależy od parametrów pamięci FX3UC. Jeżeli D520 nie jest latched, HMI może zapisywać parametr przy połączeniu albo należy ustawić odpowiedni obszar retencyjny PLC.

## Licznik sesji

D524 zwiększa się przy każdym zaakceptowanym impulsie RM5, maksymalnie do 30000.

```text
D525:D526 = D524 * D520
```

M411 zeruje D524/D525/D526 i jest automatycznie resetowane przez PLC.

## Diagnostyka

- M300 — automat dostępny
- M301 — PRACA
- M302 — MANUAL/FREE
- M320 — zaakceptowany impuls RM5
- M321 — aktywna paczka RM5
- M412 — WORK_ACTIVE
- M413 — parametr czasu OK
- D320 — bieżąca paczka
- D330 — kolejka CREDIT
- D527 — próg saturacji

## Manual / Free

W V1.2.0 X13/M302 jest tylko statusem. Nie uruchamia programu bez D350.
