# Comestero PitStart — odniesienie dla V1.4.0

Źródło projektu: **Gamma Pit – Sistemi di attivazione – Manuale operativo**, PitStart-F, Rev. 00, 13/10/2009.

## Wyjścia maszyny

PitStart ma:
- wyjście Counter / Compteur,
- Pilotaggio pompa,
- P1…P6.

W modelu V1.4.0:
- Y0 = Counter / PITSTART_CREDITS,
- Y1 = Pilotaggio,
- Y2…Y7 = P1…P6.

Każdy impuls Y0 reprezentuje jedną jednostkę 0,10 EUR.

## Cena bazowa i czasy programów

PitStart ma jedną wspólną cenę bazową dla sześciu programów, natomiast czas przypisany do tej ceny może być osobny dla każdego programu.

W projekcie:
- D550 = cena bazowa w jednostkach 0,10 EUR,
- D551…D556 = czas P1…P6 w sekundach dla ceny bazowej.

Przykład:
- D550=5 -> 0,50 EUR,
- D551=70 -> P1 = 70 s za 0,50 EUR.

## RM5

D300 jest wartością kanału RM5 CH1 wyrażoną w jednostkach 0,10 EUR.

Przykład:
- D300=10 -> zaakceptowany impuls RM5 CH1 reprezentuje 1,00 EUR,
- PLC generuje 10 impulsów Counter/Y0.

## Kredyt i zmiana programu

PLC utrzymuje pozostały kredyt pieniężny w D560.

Po zmianie programu kredyt nie jest kasowany. Zmienia się tylko przeliczony dostępny czas zgodnie z czasem wybranego programu.

## Pilotaggio

M410 AND M412 -> Y1.

M412 powstaje po wyborze programu przy dostępnym kredycie. STOP wyłącza program i Pilotaggio, ale pozostawia kredyt.

## Manual / Free

X13 -> M302 pozostaje w V1.4.0 statusem. Pełne działanie FREE bez kredytu nie jest jeszcze aktywowane.

## PRACA

X1/M301 nie jest standardowym wejściem PitStart z manuala. Jest dodatkowym feedbackiem naszej automatyki.

M429 pozwala zdecydować, czy zużycie kredytu ma być synchronizowane z tym feedbackiem:
- M429=0 -> WORK_ACTIVE,
- M429=1 -> WORK_ACTIVE + PRACA.

## HMI wake

M419 = około 3 s po zaakceptowanej monecie.
D565 = licznik zdarzeń monet.
