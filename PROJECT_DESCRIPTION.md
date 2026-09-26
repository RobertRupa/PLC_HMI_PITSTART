# Opis projektu PLC_HMI_PITSTART

Aktualna wersja: **V1.3.0**.

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

V1.3.0 dodaje:
- M414 WORK_LIGHTS_ENABLE,
- D528/D529 korekcję zegara mnożnik/dzielnik,
- M415 walidację korekcji,
- M416 wewnętrzny tick 1 s,
- D533 licznik faktycznie wysłanych impulsów CREDIT,
- D534:D535 nominalny czas odpowiadający wysłanym impulsom,
- D540/D541 minuty/sekundy dla HMI.

Touch control pozostaje wyłącznie po stronie HMI.

Nominalny czas przypadający na jeden impuls CREDIT jest ustawiany w D520. Pozostały czas D350 jest odliczany tylko przy M412=WORK_ACTIVE. STOP zatrzymuje program i pauzuje czas, ale nie kasuje D350.
