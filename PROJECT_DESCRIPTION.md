# Opis projektu — V1.4.6

Projekt emuluje funkcje PitStart dla sterownika myjni.

Najważniejsza zmiana V1.4.6: źródłem prawdy nie jest już sztuczny licznik sekund przypisywany do monety. Źródłem prawdy jest pozostały **kredyt pieniężny**, a czas jest wyliczany z taryfy programu.

## Taryfa

- D300 — wartość RM5 CH1, jednostka 0,10 EUR,
- D549 — maksymalny kredyt, jednostka 0,10 EUR,
- D550 — wspólna cena bazowa, jednostka 0,10 EUR,
- D551…D556 — czasy P1…P6 dla ceny bazowej,
- D528 — globalna korekcja zegara x100.

## Counter

Każdy impuls Y0 reprezentuje 0,10 EUR.

Dla D300=10 jeden impuls RM5 daje 10 impulsów Y0 = 1,00 EUR.

## Dostępny czas

Po wyborze programu PLC oblicza czas z aktualnego kredytu i czasu tego programu. Zmiana programu zmienia przeliczenie czasu, ale zachowuje kredyt pieniężny.

## Synchronizacja

M429 pozwala wybrać:
- odliczanie według WORK_ACTIVE,
- albo według rzeczywistego X1/M301 PRACA.

## HMI wake

M419 jest 3-sekundowym żądaniem wybudzenia po monecie. D565 jest licznikiem zdarzeń monet.


## Auto Start Program

`M431` enables delayed start of the countdown. With this option enabled, the PLC waits until the full `Y0/PITSTART_CREDITS` queue has been transmitted. The actual active state is `M432`, and `M433` is the final countdown gate.

STOP, loss of X0 and exhausted credit reset the auto timer.


## Auto Start Program

- `M431` — enable,
- `D584` — program 1…6,
- start następuje po zakończeniu pełnej kolejki Y0/CREDIT,
- ręczny wybór programu ma pierwszeństwo; Auto Start nie nadpisuje już pracującego programu.


## Admin screen request

```text
X14 -> M303 = ADMIN_SCREEN_REQUEST
```

M303 jest statusem tylko do odczytu dla HMI. Stan 1 oznacza żądanie ekranu Admin, stan 0 oznacza Home.


## HMI word triggers

Dla HMI bez obsługi bitów M jako triggerów ekranów:

```text
D585 = ADMIN_SCREEN_REQUEST_WORD
       0/1 mirror X14 -> M303

D586 = HMI_WAKE_REQUEST_WORD
       0/1 mirror M419
```

D585=1 oznacza ekran Admin, D585=0 ekran Home.
D586=1 jest aktywne około 3 s po zaakceptowanym impulsie RM5.
