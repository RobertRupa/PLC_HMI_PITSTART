# Comestero RM5 Evolution — V1.4.1

## Połączenie

- pin 7 CH1 -> X27
- pin 6 INHIBIT <- Y23
- pin 1 GND -> 0 V / COM
- pin 2 -> +12…24 VDC

## Wartość kanału

D300 = wartość RM5 CH1 w jednostkach 0,10 EUR.

Przykład:
- D300=10 -> 1,00 EUR za zaakceptowany impuls CH1,
- po zakończeniu paczki PLC generuje 10 impulsów Y0/Counter na każdy taki impuls.

## Akceptacja

Impuls X27 jest przyjmowany programowo tylko przy:
- M300=1,
- M413=1 (D300 poprawne),
- M415=1 (taryfa poprawna).

Natomiast fizyczne/logiczne wyjście INHIBIT Y23 zależy wyłącznie od X0.

## INHIBIT

```text
X0 = 0 -> Y23 = 1  (RM5 INHIBIT aktywny)
X0 = 1 -> Y23 = 0  (RM5 odblokowany)
```

W Instruction List:

```text
LDI X0
OUT Y23
```

Y23 nie jest już uzależnione od M413/M415.

## Paczka RM5

- D320 = bieżące impulsy,
- D521 = impulsy ostatniej paczki,
- D522 = liczba impulsów Counter wygenerowana z ostatniej paczki,
- D330 = kolejka Counter do wysłania.

Po końcu paczki:

```text
D542 = D320 * D300
D522 = D542
D330 = D542
```

## Faktycznie wysłane Counter

D533 zwiększa się po każdym zakończonym impulsie Y0.

D560, czyli pozostały kredyt pieniężny, jest zwiększany dopiero w chwili faktycznej wysyłki impulsu Counter. Dzięki temu HMI nie wyprzedza automatyki.

## Wake HMI

Po zaakceptowanym X27:
- M419 jest aktywne około 3 s,
- D565 zwiększa się o 1.
