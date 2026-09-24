# HMI — adresy i wersja PLC

## Oprogramowanie

Panel HMI jest edytowany w **WSStudio**.

## Wersja PLC

Aktualna wersja projektu:

```text
V1.1.0
```

PLC udostępnia ją jako tekst ASCII:

| Adres | Zawartość |
|---|---|
| D500 | `V1` |
| D501 | `.1` |
| D502 | `.0` |
| D503 | terminator `0x0000` |
| D504…D507 | rezerwa dla dłuższej wersji |
| D516 | major = 1 |
| D517 | minor = 1 |
| D518 | patch = 0 |

Kod inicjalizacji:

```text
M8002 -> MOV H3156 D500
M8002 -> MOV H312E D501
M8002 -> MOV H302E D502
M8002 -> MOV H0000 D503

M8002 -> MOV K1 D516
M8002 -> MOV K1 D517
M8002 -> MOV K0 D518
```

## WSStudio

Dla wyświetlenia wersji:
- obiekt: ASCII/String Display,
- adres startowy: `D500`,
- długość: 16 znaków,
- tylko odczyt.

Jeżeli wyświetlony zostanie tekst z odwróconymi parami znaków, np. `1V0.0.`, oznacza to przeciwną kolejność bajtów w używanym sterowniku/HMI. Wtedy należy odwrócić bajty stałych HEX, np. użyć `H5631` zamiast `H3156`.

## Licznik impulsów i przeliczenie na czas

Do ekranu serwisowego HMI dodaj:

| Adres | Nazwa na HMI | Typ | Dostęp |
|---|---|---|---|
| `D520` | Czas / impuls [s] | 16-bit unsigned | RW |
| `D521` | Ostatnia paczka [imp] | 16-bit unsigned | RO |
| `D522:D523` | Ostatnia paczka [s] | 32-bit unsigned | RO |
| `D524` | Licznik impulsów sesji | 16-bit unsigned | RO |
| `D525:D526` | Czas równoważny sesji [s] | 32-bit unsigned | RO |
| `D350` | Pozostały czas [s] | 16-bit | RO |
| `M411` | Reset licznika sesji | bit / momentary | WO |
| `M413` | Parametr czasu OK | bit | RO |

### Parametr D520

- domyślnie: **10 s/impuls**,
- minimum: **1**,
- maksimum: **600**,
- krok: 1 s.

Jeżeli `D520` wyjdzie poza zakres 1…600, `M413=0` i PLC blokuje RM5 przez `Y23 INHIBIT`.

### Licznik sesji

`D524` zwiększa się o 1 przy każdym zaakceptowanym zboczu `X27`.

`D525:D526` jest przeliczane jako:

```text
liczba_impulsów × sekundy_na_impuls
```

czyli:

```text
D525:D526 = D524 × D520
```

Przycisk HMI `M411` zeruje `D524`, `D525` i `D526`. Ustaw go jako przycisk chwilowy, nie przełącznik bistabilny.

## Pozostałe adresy HMI

- `M400` — STOP
- `M401…M406` — Program 1…6
- `M410` — Pilotaggio enable
- `M411` — reset licznika impulsów sesji
- `M413` — status poprawności parametru czasu
- `D300` — wartość/mnożnik RM5 CH1
- `D350` — pozostały czas w sekundach
- `D520` — sekundy za impuls RM5
- `D521` — ostatnia paczka impulsów
- `D522:D523` — czas ostatniej paczki
- `D524` — licznik impulsów sesji
- `D525:D526` — równoważny czas sesji
- `D320` — diagnostyka bieżącej paczki RM5
- `D330` — diagnostyka kolejki CREDIT
