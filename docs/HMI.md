# HMI — WSStudio — V1.3.2

## Ekran główny

Sterowanie:
- M400 — STOP, momentary
- M401…M406 — Program 1…6, momentary

Wskaźniki:
- M420…M425 — aktywny Program 1…6
- M412 — WORK_ACTIVE
- M301 — fizyczny status PRACA
- M300 — automat dostępny
- Y23 — RM5 INHIBIT

## MM:SS

PLC przygotowuje:
- D540 — minuty
- D541 — sekundy

Najprostszy i poprawny układ w WSStudio:
1. Value Display #1: 16-bit Unsigned, Monitor address = D540, Offset address = OFF.
2. Statyczny tekst: ":"
3. Value Display #2: 16-bit Unsigned, Monitor address = D541, Offset address = OFF.
4. Dla D541 ustaw 2 cyfry i leading zero / zero fill, jeśli ta opcja jest dostępna.

Przy obecnym limicie D350=32000 minuty mogą przekroczyć 99, więc technicznie bezpieczniej przeznaczyć 3 cyfry na D540.

**Nie używaj Offset address do składania MM:SS.**

Dla wartości 32-bit, np. D534:D535:
- wybierz DataType = 32-bit Unsigned Int,
- Monitor address = D534,
- Offset address = OFF.
D535 jest wtedy automatycznie starszym słowem.

## Ekran Admin

| Adres | Funkcja | Dostęp |
|---|---|---|
| D520 | Nominalny czas / 1 impuls CREDIT [s] | RW |
| D528 | Korekcja zegara ×100 | RW |
| M413 | D520 poprawne | RO |
| M415 | D528 poprawne | RO |
| M410 | Pilotaggio enable | RW |
| M414 | Work lights enable | RW |
| M411 | Reset liczników sesji | momentary |
| D524 | zaakceptowane impulsy RM5 | RO |
| D533 | wysłane impulsy CREDIT | RO |
| D525:D526 | nominalny czas zaakceptowanych impulsów | RO 32-bit |
| D534:D535 | nominalny czas wysłanych impulsów | RO 32-bit |

Touch control pozostaje wyłącznie funkcją HMI.

## D528 — korekcja zegara

PLC przechowuje wartość jako liczbę całkowitą ×100:

- D528=100 → 1.00×
- D528=101 → 1.01×
- D528=99 → 0.99×
- D528=1 → 0.01×

W WSStudio ustaw D528 jako 16-bit unsigned i wyświetlanie z **2 miejscami po przecinku**. Nie zapisuj liczby zmiennoprzecinkowej do PLC — rejestr nadal jest integerem.

Zakres PLC: 1…500 (0.01×…5.00×). Domyślnie 100 = 1.00×.

Jeśli D528 jest niepoprawne, M415=0, ale RM5 nie jest blokowany; odliczanie przechodzi awaryjnie na 1.00×.

## Sesja

Nowa sesja zaczyna się przy pierwszym zaakceptowanym impulsie RM5, gdy D350<=0.

Przed policzeniem tego pierwszego impulsu PLC zeruje:
- D320,
- D521:D526,
- D530:D535.

Dzięki temu liczniki nowej sesji startują od zera. STOP nie resetuje sesji — tylko pauzuje pracę i zachowuje D350.

## Work lights

M414 jest zezwoleniem:

```text
M414 AND M412 -> Y27
```

## Wersja PLC

D500..D503 = V1.3.2
D516=1
D517=3
D518=1


## Coin multiplier i naliczanie czasu

- `D300` = liczba impulsów `Y0/PITSTART_CREDITS` generowanych z jednego impulsu RM5.
- `D520` = sekundy dodawane za każdy impuls CREDIT faktycznie wysłany przez `Y0`.

Domyślnie:

```text
D300 = 10
D520 = 10 s
```

czyli:

```text
1 impuls RM5 -> 10 impulsów CREDIT -> 100 s
```

Czas jest dodawany dopiero po zakończeniu fazy ON danego impulsu CREDIT (`T201`), nie w momencie odebrania monety.

### Który rejestr wyświetlać

- `D350` = pozostały czas, maleje podczas pracy.
- `D540` = minuty pozostałego czasu.
- `D541` = sekundy pozostałego czasu.
- `D534:D535` = skumulowany czas już wysłany do automatyki; **rośnie**, nie maleje.

Na ekranie głównym nie używaj `D534` jako licznika czasu pozostałego.

### MM:SS

Użyj dwóch osobnych Value Display:
- minuty: `D540`, 16-bit unsigned,
- sekundy: `D541`, 16-bit unsigned,
- pomiędzy nimi statyczny znak `:`,
- Offset Address = OFF dla obu,
- dla sekund ustaw 2 cyfry / leading zero.
