# Logika PLC — V1.3.1

## Najważniejsze poprawki

1. Usunięto D529 z algorytmu korekcji czasu.
2. D528 jest stałoprzecinkowym współczynnikiem ×100.
3. Niepoprawne D528 nie blokuje RM5.
4. Dodano M417 = NEW_SESSION_PULSE.
5. Liczniki sesji są zerowane automatycznie na pierwszym impulsie nowej sesji.
6. M414 pozostaje WORK_LIGHTS_ENABLE.

## D528 — korekcja zegara

```text
D528=100 -> 1.00x
D528=101 -> 1.01x
D528=99  -> 0.99x
```

M415=1, gdy D528 jest w zakresie 1…500.

Przy każdym 1-sekundowym ticku podczas WORK_ACTIVE:

```text
D530 = D530 + D528
DIV D530 K100 D531
D531 = liczba sekund do odjęcia
D532 = reszta
D530 = D532
```

To zachowuje część ułamkową korekcji.

Jeżeli M415=0, istniejący czas jest odliczany awaryjnie 1:1.

## RM5

RM5 jest blokowany tylko przez:
- brak M300,
- niepoprawny D520.

Korekcja zegara D528 nie blokuje przyjmowania monet.

## Automatyczny reset sesji

```text
M320 AND D350<=0 -> M417
```

M417 zeruje przed pierwszym impulsem:
- D320,
- D521:D526,
- D530:D535.

Następnie ten sam impuls jest normalnie liczony do D320 i D524.

STOP nie wywołuje M417 i nie zeruje sesji.

## MM:SS

```text
DIV D350 K60 D540
```

- D540 = minuty,
- D541 = sekundy / reszta.

W HMI używaj dwóch 16-bitowych pól bez Offset address.

## Oświetlenie

```text
M414 AND M412 -> Y27
```

## Pełny program

- `plc/MAIN_GXDEV_ENTRY.txt`
- `plc/MAIN.txt`
- `plc/main_v1.3.1.csv`
