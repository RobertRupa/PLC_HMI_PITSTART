# Logika PLC — V1.3.0

## Główne zmiany

1. `M414 = WORK_LIGHTS_ENABLE`.
2. Touch control pozostaje wyłącznie w HMI.
3. `D520` oznacza nominalne sekundy automatyki przypadające na jeden impuls CREDIT.
4. Dodano korekcję szybkości zegara przez `D528/D529`.
5. Dodano licznik faktycznie wysłanych impulsów CREDIT `D533`.
6. Dodano rejestry czasu `D540/D541` gotowe dla HMI.

## Korekcja zegara

```text
D528 = multiplier
D529 = divider

effective decrement rate = D528 / D529
```

M415=1 gdy oba parametry są w zakresie 1…1000.

Przy każdym zboczu 1-sekundowego M8013 podczas WORK_ACTIVE:

```text
D530 = D530 + D528
DIV D530 D529 D531
D531 = quotient
D532 = remainder
D530 = D532
D350 = D350 - D531
```

Jeżeli wynik przekracza pozostały czas, D350 jest ustawiane na 0.

Jeżeli M415=0, istniejący czas jest odliczany awaryjnie 1:1, a nowe impulsy RM5 są blokowane przez Y23.

## Czas MM:SS

```text
DIV D350 K60 D540
```

- D540 = minuty
- D541 = sekundy (remainder)

## Licznik wysłanych impulsów

Każde zakończenie fazy Y0 ON / T201 zwiększa D533:

```text
T201 AND D533<30000 -> INC D533
```

```text
D534:D535 = D533 * D520
```

Dzięki temu HMI może porównać:
- D524 — impulsy przyjęte z RM5
- D533 — impulsy faktycznie wysłane do automatyki

## Oświetlenie

```text
M414 AND M412 -> Y27
```

M414 jest ustawiane na 1 przy starcie.

## Pełny program

Źródła:
- `plc/MAIN_GXDEV_ENTRY.txt`
- `plc/MAIN.txt`
- `plc/main_v1.3.0.csv`
