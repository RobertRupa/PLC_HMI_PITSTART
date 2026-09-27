# HMI manuals

Oba manuale są już obecne w repo.

## WSB HMI&PLC All-in-one User Manual V1.79

[WSB_HMI_PLC_All_in_one_User_Manual_V1.79.pdf](WSB_HMI_PLC_All_in_one_User_Manual_V1.79.pdf)

To podstawowa instrukcja dla używanej rodziny WSB. Manual wymienia model **WSB7020R**.

Dla projektu szczególnie istotne są:
- HMI Studio 5.1,
- download programu HMI przez USB,
- komunikacja HMI <-> PLC,
- dla WSB: `Mitsubishi FX1N`, **38400 bps**, tryb **232**,
- informacja, że po downloadzie kabel HMI należy odłączyć, ponieważ przy podłączonym kablu HMI i PLC nie komunikują się,
- programowanie części PLC przez GX Developer / GX Works2,
- zasilanie 24 VDC, porty i schematy elektryczne.

## WSC HMI&PLC All-in-one User Manual V1.13

[WSC_HMI_PLC_All_in_one_User_Manual_V1.13.pdf](WSC_HMI_PLC_All_in_one_User_Manual_V1.13.pdf)

Manual dotyczy rodziny **WSC/WSCH**, nie WSB7020R. Jest przechowywany jako dokumentacja porównawcza.

Istotna różnica: instrukcja WSC podaje `Mitsubishi FX3U`, **19200 bps**, tryb **232**. Tych parametrów nie należy przenosić do WSB7020R. Dla tego projektu obowiązuje konfiguracja zgodna z manualem WSB i plikiem `pitstart.hs`.

## Powiązane pliki

- [../../HMI.md](../../HMI.md)
- [../../../projects/hmi/README.md](../../../projects/hmi/README.md)
- [../../SOFTWARE.md](../../SOFTWARE.md)
