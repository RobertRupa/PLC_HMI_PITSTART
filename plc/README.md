# PLC V1.3.2

## Pliki

- `MAIN_GXDEV_ENTRY.txt` — aktualna instruction list.
- `MAIN.txt` — wersja komentowana.
- `main_v1.3.2.csv` — CSV zgodny z układem aktualnego eksportu.
- `DEVICE_MAP.csv` — mapa nowych adresów.

## Kluczowe parametry

```text
D300 = CREDIT pulses / RM5 pulse      default 10
D520 = seconds / sent CREDIT pulse    default 10
D528 = clock scale x100               default 100 = 1.00x
```

Domyślnie:

```text
1 RM5 -> 10 CREDIT -> 100 s
```

Czas D350 jest naliczany dopiero w momencie faktycznej wysyłki CREDIT na Y0.

## Countdown

- D350 = sekundy pozostałe,
- D540 = minuty,
- D541 = sekundy.

D534:D535 jest licznikiem skumulowanego czasu już wysłanego i nie powinien być używany jako countdown.

## MM:SS w HMI

Dwa pola 16-bit unsigned:
- D540
- D541

Offset Address OFF. Pomiędzy nimi statyczny `:`. Sekundy z zerem wiodącym.
