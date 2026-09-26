# Opis projektu PLC_HMI_PITSTART

Aktualna wersja: **V1.3.1**.

Najważniejsze funkcje:
- FX3UC / GX Developer,
- RM5 CH1 na X27,
- STOP X4,
- P1…P6 na X5…X12,
- Y0 CREDIT / COMPTEUR,
- Y1 Pilotaggio,
- Y2…Y7 P1…P6,
- Y23 RM5 INHIBIT,
- Y27 oświetlenie pracy,
- HMI i mechaniczne przyciski działają równolegle.

V1.3.1:
- M414 = WORK_LIGHTS_ENABLE,
- D528 = korekcja zegara ×100; 100=1.00×, 101=1.01×, 99=0.99×,
- D529 nie jest używany,
- M415 = CLOCK_SCALE_OK,
- M416 = wewnętrzny tick 1 s,
- M417 = impuls rozpoczęcia nowej sesji,
- nowa sesja zaczyna się przy pierwszym zaakceptowanym impulsie RM5 przy D350<=0,
- liczniki sesji są wtedy automatycznie zerowane przed policzeniem pierwszego impulsu,
- D533 = licznik faktycznie wysłanych impulsów CREDIT,
- D534:D535 = nominalny czas odpowiadający wysłanym impulsom,
- D540/D541 = minuty/sekundy dla HMI.

Touch control pozostaje wyłącznie po stronie HMI.

D520 ustala nominalny czas na jeden impuls CREDIT. D350 jest odliczany tylko przy M412=WORK_ACTIVE. STOP zatrzymuje program i pauzuje czas, ale nie kasuje D350 ani liczników bieżącej sesji.
