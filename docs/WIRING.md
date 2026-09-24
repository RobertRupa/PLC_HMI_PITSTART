# Schemat połączeniowy

## 1. RM5 -> PLC

```text
RM5 pin 2  +12...24 V  -> zasilanie RM5
RM5 pin 1  GND         -> 0 V / COM wejść PLC
RM5 pin 7  CH1         -> X27

PLC Y23                -> RM5 pin 6 INHIBIT
```

`Y23` należy podłączyć tak, aby jego aktywacja podawała stan HIGH na wejście INHIBIT RM5.

## 2. Wyjścia PLC -> sterownik myjni

```text
Y0  -> CREDIT / COMPTEUR
Y1  -> PILOTAGGIO POMPA
Y2  -> PROGRAM 1
Y3  -> PROGRAM 2
Y4  -> PROGRAM 3
Y5  -> PROGRAM 4
Y6  -> PROGRAM 5
Y7  -> PROGRAM 6
Y27 -> OŚWIETLENIE
```

Oryginalny PitStart ma wyjścia jako **suche styki NO**. Dla wersji PLC z wyjściami przekaźnikowymi należy wykorzystywać Y/COM jako styki bezpotencjałowe zgodnie z wejściami sterownika myjni.

## 3. Oryginalny PitStart — CN4/CN5, Fig. 33

Numeracja złącza jest liczona od strony oznaczonej `1` na rysunku.

### CN4

| Piny | Funkcja |
|---|---|
| 1 + 3 | Program 1 |
| 4 + 6 | Program 2 |
| 7 + 9 | Program 3 |
| 10 + 12 | Program 4 |

Piny 2, 5, 8 i 11 nie są użyte w pokazanym schemacie.
Jedna strona P1…P4 jest połączona wspólną magistralą `COMMUN`.

### CN5

| Piny | Funkcja |
|---|---|
| 1 + 3 | Program 5 |
| 4 + 6 | Program 6 |
| 7 + 9 | Pilotaggio pompa |
| 10 + 12 | Compteur / Credit |

Piny 2, 5, 8 i 11 nie są użyte w pokazanym schemacie.

## 4. Oryginalny PitStart — CN8, Fig. 34

| Piny | Funkcja |
|---|---|
| CN8-1 = +24 V, CN8-2 = GND | Presence automatic device |
| CN8-3 = +24 V, CN8-4 = GND | Manual / Free |

Presence automatic device:
- obecność sygnału oznacza dostępny automat,
- brak sygnału może przełączyć PitStart w OUT OF SERVICE,
- wymóg tego wejścia można wyłączyć w MultiConfig.

Manual / Free:
- obecność napięcia wymusza ciągłą pracę bez monety.

## 5. Mapowanie do naszego PLC

Aktualnie:
- funkcję Presence automatic device realizuje `X0 = INVERTER_OK/AUTOMATE_PRESENT`,
- `X1 = PRACA` jest dodatkowym sygnałem naszej myjni,
- Manual/Free nie ma jeszcze przypisanego wejścia w finalnym mapowaniu.

## 6. Pilotaggio i oświetlenie

```text
M410 AND M412 -> Y1
M412          -> Y27
```

- `M410` — opcja Pilotaggio,
- `M412` — praca aktywna.

## 7. Uwaga elektryczna

Nie podawać napięcia na wejścia sterownika myjni bez potwierdzenia ich typu. Jeżeli oczekują zwarcia styku, używać wyjść przekaźnikowych jako suchych styków.
