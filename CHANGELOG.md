# Changelog

## V1.3.2

- D300 działa jako Coin multiplier: liczba impulsów PITSTART_CREDITS na impuls RM5,
- domyślnie D300=10,
- D520 jest czasem w sekundach przypadającym na jeden faktycznie wysłany CREDIT,
- czas D350 jest dodawany przy T201, czyli po rzeczywistym impulsie Y0,
- usunięto dodawanie czasu bezpośrednio przy impulsie RM5,
- D528 jest pojedynczym współczynnikiem zegara x100; D529 nie jest używany,
- poprawiono wykrywanie nowej sesji, aby wieloimpulsowa paczka RM5 nie resetowała licznika przy każdym impulsie,
- D534:D535 pozostaje licznikiem skumulowanego czasu wysłanego i nie służy jako countdown,
- wersja PLC/HMI V1.3.2.


## V1.3.1

- usunięto D529 z algorytmu zegara,
- D528 jest współczynnikiem stałoprzecinkowym ×100,
- 100=1.00×, 101=1.01×, 99=0.99×,
- zakres D528: 1…500,
- niepoprawne D528 nie blokuje RM5; odliczanie przechodzi na 1.00×,
- dodano M417 NEW_SESSION_PULSE,
- nowa sesja zaczyna się na pierwszym zaakceptowanym impulsie RM5 przy D350<=0,
- na początku sesji automatycznie zerowane są D320, D521:D526 i D530:D535,
- STOP nie resetuje sesji ani pozostałego czasu,
- poprawiono dokumentację MM:SS oraz 32-bitowych pól HMI,
- wersja PLC/HMI podniesiona do V1.3.1.

## V1.3.0

- dodano M414 WORK_LIGHTS_ENABLE,
- Touch control pozostawiono wyłącznie po stronie HMI,
- zmieniono opis D520 na nominalny czas automatyki / impuls CREDIT,
- dodano D528/D529 jako mnożnik/dzielnik szybkości zegara,
- dodano M415 CLOCK_RATIO_OK,
- dodano M416 wewnętrzny tick 1 s podczas WORK_ACTIVE,
- dodano zegar ułamkowy D530/D531/D532,
- dodano licznik faktycznie wysłanych impulsów CREDIT D533,
- dodano D534:D535 jako nominalny czas wysłanych CREDIT,
- dodano D540/D541 do wyświetlania MM:SS,
- Y27 jest teraz sterowane przez M414 AND M412,
- wersja PLC/HMI podniesiona do V1.3.0.

## V1.2.0

- przygotowano czysty program dla FX3UC zamiast kodu testowego,
- dodano bezpieczne impulsy wyboru M451…M456,
- utrata X0 kasuje aktywny program,
- STOP pauzuje D350,
- M410 startuje jako ON,
- dodano D520 i liczniki RM5/CREDIT.
