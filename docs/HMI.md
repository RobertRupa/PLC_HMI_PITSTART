# HMI — adresy i wersja PLC

## Oprogramowanie

Panel HMI jest edytowany w **WSStudio**.

## Wersja PLC

Aktualna wersja projektu:

```text
V1.0.0
```

PLC udostępnia ją jako tekst ASCII:

| Adres | Zawartość |
|---|---|
| D500 | `V1` |
| D501 | `.0` |
| D502 | `.0` |
| D503 | terminator `0x0000` |
| D504…D507 | rezerwa dla dłuższej wersji |
| D516 | major = 1 |
| D517 | minor = 0 |
| D518 | patch = 0 |

Kod inicjalizacji:

```text
M8002 -> MOV H3156 D500
M8002 -> MOV H302E D501
M8002 -> MOV H302E D502
M8002 -> MOV H0000 D503

M8002 -> MOV K1 D516
M8002 -> MOV K0 D517
M8002 -> MOV K0 D518
```

## WSStudio

Dla wyświetlenia wersji:
- obiekt: ASCII/String Display,
- adres startowy: `D500`,
- długość: 16 znaków,
- tylko odczyt.

Jeżeli wyświetlony zostanie tekst z odwróconymi parami znaków, np. `1V0.0.`, oznacza to przeciwną kolejność bajtów w używanym sterowniku/HMI. Wtedy należy odwrócić bajty stałych HEX, np. użyć `H5631` zamiast `H3156`.

## Pozostałe adresy HMI

- `M400` — STOP
- `M401…M406` — Program 1…6
- `M410` — Pilotaggio enable
- `D300` — wartość/mnożnik RM5 CH1
- `D350` — aktualna wartość/kredyt
- `D320` — diagnostyka bieżącej paczki RM5
- `D330` — diagnostyka kolejki CREDIT
