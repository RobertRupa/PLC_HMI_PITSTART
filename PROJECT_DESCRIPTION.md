# Opis projektu PLC_HMI_PITSTART — V1.3.2

## Przepływ kredytu

1. RM5 CH1 generuje impuls na X27.
2. PLC zbiera paczkę impulsów RM5 w D320.
3. Po końcu paczki:

```text
D542 = D320 × D300
D330 = D542
```

D300 jest **Coin multiplier** — liczbą impulsów PITSTART_CREDITS generowanych na jeden impuls RM5.

4. Generator Y0 wysyła D330 impulsów CREDIT.
5. Dopiero po faktycznie wysłanym impulsie CREDIT (T201) PLC dodaje D520 sekund do D350.

Domyślne ustawienia:

```text
D300 = 10 CREDIT / RM5
D520 = 10 s / CREDIT
```

czyli 1 impuls RM5 daje 10 impulsów CREDIT i 100 s czasu.

## Czas

- D350 — czas pozostały; maleje podczas pracy,
- D540 — minuty,
- D541 — sekundy,
- D528 — korekcja zegara x100; 100=1.00x,
- D534:D535 — skumulowany czas już wysłany do automatyki; rośnie.

## Sesja

Pierwszy zaakceptowany impuls RM5 przy pustym czasie i pustej kolejce generuje M417 i zeruje liczniki diagnostyczne sesji przed policzeniem pierwszego impulsu.

STOP nie zeruje D350 ani sesji; zatrzymuje aktywny program i odliczanie.

## HMI

- D300 — Coin multiplier, RW, 1…100,
- D520 — seconds/CREDIT, RW, 1…600,
- D528 — clock factor x100, RW, 50…200,
- M414 — WORK_LIGHTS_ENABLE,
- M410 — PILOTAGGIO_ENABLE,
- M411 — reset liczników sesji,
- M413 — D520 valid,
- M415 — D528 valid,
- M418 — D300 valid.

Touch control pozostaje wyłącznie po stronie HMI.
