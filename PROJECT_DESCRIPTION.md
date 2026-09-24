# Opis projektu PLC_HMI_PITSTART

## Cel

SEEKU PLC/HMI zastępuje / rozszerza logikę Comestero PitStart:
- odbiera impulsy z RM5,
- blokuje RM5, gdy myjnia nie jest dostępna,
- przelicza odebraną paczkę impulsów,
- generuje CREDIT/COMPTEUR,
- steruje Pilotaggio pompa,
- wybiera programy mycia,
- steruje oświetleniem w czasie pracy,
- udostępnia obsługę przez HMI.

Finalny projekt nie korzysta z demo-bloku analogowego ani AD0…AD5.

## Schemat funkcjonalny

```text
RM5 CH1 --------------------> X27
RM5 INHIBIT <--------------- Y23

myjnia AUTOMATE_PRESENT ---> X0
myjnia PRACA --------------> X1

Y0  -----------------------> CREDIT / COMPTEUR
Y1  -----------------------> PILOTAGGIO POMPA
Y2  -----------------------> PROGRAM 1
Y3  -----------------------> PROGRAM 2
Y4  -----------------------> PROGRAM 3
Y5  -----------------------> PROGRAM 4
Y6  -----------------------> PROGRAM 5
Y7  -----------------------> PROGRAM 6
Y27 -----------------------> OŚWIETLENIE
```

## RM5

Jeden kanał RM5 trafia na `X27`.

Każdy impuls:
- zwiększa `D320`,
- ustawia `M321`,
- restartuje `T200`.

Po ok. 1 s ciszy:

```text
MUL D320 D300 D350
MOV D320 D330
MOV K0 D320
RST M321
```

`D350` jest wynikiem mnożenia i wartością do wyświetlania/odliczania.
`D330` jest kolejką generatora CREDIT.

## CREDIT / COMPTEUR

Generator wyjścia `Y0` używa `M330/M331` i `T201/T202`.

Aktualnie:
- ON ≈ 0,1 s,
- OFF ≈ 0,1 s.

Oryginalny PitStart opisuje Counter jako impulsy odpowiadające wielokrotnościom 0,10 € przy każdym przyjęciu wartości. Obecny kod wysyła na Y0 tyle impulsów, ile zostało skopiowane do `D330`.

## Blokada RM5

`X0` jest głównym sygnałem dostępności:

```text
X0 -> M300
/M300 -> Y23
```

Brak `X0` blokuje RM5.

`X1 = PRACA` jest statusem i nie bierze udziału w blokowaniu akceptora.

## Pilotaggio pompa

Dokumentacja PitStart określa zachowanie Pilotaggio:
- startuje przy żądaniu pierwszego programu,
- pozostaje aktywne, dopóki istnieje kredyt,
- STOP je wyłącza.

Projekt:
- `M410 = PILOTAGGIO_ENABLE`,
- `M412 = WORK_ACTIVE`,
- `M410 AND M412 -> Y1`.

## WORK_ACTIVE

`M412` jest wspólnym stanem pracy.

Powinien być ustawiany:
- po uruchomieniu Program 1…6,
- opcjonalnie podczas trybu MANUAL/FREE,
- ewentualnie na podstawie potwierdzenia PRACA z X1.

Powinien być kasowany:
- przez STOP,
- po zakończeniu pracy/kredytu.

## Oświetlenie

```text
M412 -> Y27
```

Oświetlenie jest niezależne od opcji Pilotaggio.

## Programy

Mapowanie:
- `Y2` = P1,
- `Y3` = P2,
- `Y4` = P3,
- `Y5` = P4,
- `Y6` = P5,
- `Y7` = P6.

HMI ma docelowo STOP + P1…P5; P6 pozostaje dostępny opcjonalnie.

## Wejścia udokumentowane dla oryginalnego PitStart

Potwierdzone przez manual:
- CN8 1–2 — Presence automatic device,
- CN8 3–4 — Manual/Free.

Manual PitStart nie opisuje osobnego wejścia RUN/PRACA. `X1=PRACA` jest więc sygnałem specyficznym dla naszego sterownika myjni, a nie standardową funkcją PitStart.

Szczegółowa rozpiska wyjść CN4/CN5 i wejść CN8 znajduje się w `docs/PITSTART.md`.
