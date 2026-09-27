# Comestero Gamma Pit / PitStart-F manual

Docelowy plik instrukcji:

```text
docs/manuals/pitstart/Gamma_Pit_PitStart_F_c27-M-PIT-EK_2009-10-13.pdf
```

## Identyfikacja dokumentu

```text
Gamma Pit
Sistemi di attivazione
Manuale operativo
PitStart-F - c27-M-PIT-EK
Data: 13/10/2009
```

Na stronie 2 PDF widnieje `Rev. 01`, natomiast nagłówki dalszej części instrukcji wskazują `Rev. 00`. Z tego powodu nazwa pliku w repo celowo nie zawiera numeru rewizji.

SHA-256 przesłanego PDF:

```text
5aee72cd4493d97b29da9af7580e7e59f43920e795419c86142f48a109ed735e
```

## Najważniejsze rozdziały dla projektu

Sekcja `9.5 PitStart` obejmuje:
- 9.5.1 — podłączenie elektryczne,
- 9.5.2 — połączenie z automatyką/myjnią,
- 9.5.3 — wejście obecności automatu i tryb manualny,
- 9.5.4 — konfigurację przez MultiConfig,
- wartość maksymalną,
- wspólną cenę bazową,
- tabelę wartości RM5,
- język,
- poziomy wejść,
- programowanie ręczne,
- inicjalizację systemu.

Schemat elektryczny PitStart znajduje się w rozdziale `10.5`.

## Znaczenie dla tego repo

Manual jest dokumentem referencyjnym oryginalnego Comestero PitStart. Aktualny PLC nie kopiuje go 1:1 — implementuje zgodne funkcje Counter, Pilotaggio i programy P1…P6 oraz rozszerza je o HMI, obsługę RM5 i synchronizację z RUN.

Opis mapowania oryginalnego PitStart na aktualny PLC:
[../../PITSTART.md](../../PITSTART.md)
