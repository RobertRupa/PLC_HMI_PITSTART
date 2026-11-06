# WSC-7048R / PitStart — wariant okablowania (projekt do walidacji)
 
> **DRAFT / NIE PODŁĄCZAĆ BEZ POMIARÓW.** To wariant docelowy WSC-7048R, nie bieżąca mapa działającego PLC WSB. Wariant źródłowy logiki PLC i CSV do importu zapisano osobno; **nie są przetestowane ani wgrane**. Projekt HMI pozostał bez zmian. Zweryfikować etykietę modelu, podział i oznaczenia grup wyjść na obudowie, typ wejść, parametry styków i okablowanie maszyny. Wszystkie pomiary napięć wykonywać odpowiednimi przyrządami przez osobę uprawnioną.

## Inwentaryzacja przewodów po numerach (dane z wiązki)

| Złącze / pin PitStart | Przewód | Opis | Pewność / uwagi |
|---|---:|---|---|
| CN10.1 | 14 | ADMIN - HOST PLC | Opis użytkownika; biegunowość ustalić |
| CN10.3 | 15 | ADMIN + HOST PLC | Opis użytkownika; biegunowość ustalić |
| CN8.2, CN8.4 | 10 | wspólny wejść CN8 | Zapis użytkownika; sprawdzić ciągłość między pinami |
| CN8.1 | 11 | INVERTER_OK / obecność automatu | Funkcja odpowiada wejściu CN8.1–2 z manuala |
| CN8.3 | 13 | **prawdopodobnie MANUAL/FREE** | Manual opisuje CN8.3–4 jako wejście Manual/Free; niepotwierdzone pomiarem |
| CN9.1 | 12 | NIEUSTALONE (możliwe RUN, lecz bez dowodu) | Nie łączyć z X1 przed ustaleniem |
| CN4.3, CN4.6, CN4.9, CN4.12 | 1 | wspólny styków wyjść P1–P4 | NIE zakładać 0V ani +24V |
| CN4.1 | 2 | Program 1 | Wyjście styku |
| CN4.4 | 3 | Program 2 | Wyjście styku |
| CN4.7 | 4 | Program 3 | Wyjście styku |
| CN4.10 | 5 | Program 4 | Wyjście styku |
| CN5.3, CN5.6, CN5.9, CN5.12 | 1 | wspólny styków P5,P6,pompy,coin | NIE zakładać 0V ani +24V |
| CN5.1 | 6 | Program 5 | Wyjście styku |
| CN5.4 | 8 | Program 6 | Wyjście styku |
| CN5.7 | 7 | Pilotaggio / pompa | Wyjście styku; NIE zasilać silnika bezpośrednio |
| CN5.10 | 9 | Coin / Counter | Impulsy 0,10 EUR wg manuala |

**Wiązka 1:** według oznaczeń jest wspólnym przewodem dla *styków*, nie jest to automatycznie GND, COM wejść PLC czy PE. Zanim zmostkujesz C0/C1: pomiar ciągłości 1 na CN4 i CN5 przy wyłączonym zasilaniu oraz pomiar napięcia względem 0V, PE i innych przewodów przy zabezpieczonym układzie. Nie łączyć różnych źródeł zasilania przez mostek COM.

**CN8:** podany dwukrotnie CN8.2 = przewód 10 jest spójny z parami wejściowymi manuala. Zweryfikować, czy CN8.2 i CN8.4 rzeczywiście są złączone.

## Docelowa mapa WSC-7048R (zmiana w stosunku do istniejącego programu)

| WSC wyjście | Funkcja | Przewód wiązki | Pin oryginału | Grupa proponowana |
|---|---|---:|---|---|
| Y0 | PITSTART_COUNTER / COIN (impulsy) | 9 | CN5.10 | C0 |
| Y1 | PILOTAGGIO / POMPA (styk sterujący) | 7 | CN5.7 | C0 |
| Y2 | Program 1 | 2 | CN4.1 | C0 |
| Y3 | Program 2 | 3 | CN4.4 | C0 |
| Y4 | Program 3 | 4 | CN4.7 | C1 |
| Y5 | Program 4 | 5 | CN4.10 | C1 |
| Y6 | Program 5 | 6 | CN5.1 | C1 |
| Y7 | Program 6 | 8 | CN5.4 | C1 |
| Y20 | Światła robocze (przez stycznik/przekaźnik) | nowy | — | C4 proponowana |
| Y21 | Oświetlenie pomocnicze / rezerwa | nowy | — | C4 proponowana |
| Y22 | Sygnalizacja / rezerwa | nowy | — | C4 proponowana |
| Y23 | RM5 INHIBIT | nowy (RM5 CN5.6) | — | COM5 proponowana |

**BARDZO WAŻNE:** dotychczasowy kod WSB ma Y0 Counter, Y1 Pilotaggio, Y2 RM5 INHIBIT, Y3..Y7/Y10 P1..P6, Y23 mirror INHIBIT oraz Y27 światła. Wariant WSC z przeadresowanymi wyjściami i wejściami jest w `plc/MAIN_WSC7048R_GXDEV_ENTRY.txt` oraz `plc/main_wsc7048r_v1.5.13.csv`. Wymaga kompilacji w GX Developer oraz testów na rzeczywistym WSC. Nie przepinać Y2 do Program 1 przy aktualnym programie. Samo rozłożenie wyjść na grupy C0/C1/C4/COM5 to hipoteza wymagająca weryfikacji z tabliczką połączeń egzemplarza; nazwy COM nie są uniwersalne między WSB, WS i WSC.

### Wejścia (propozycja)

| PLC | Sygnał | Wiązka | Warunek |
|---|---|---|---|
| X0 | AUTOMATE_PRESENT / INVERTER_OK | 11 / CN8.1 | CN8.2 / przewód 10 to drugi biegun wejścia w oryginale; dopasować izolacją |
| X1 | PRACA / RUN | 12 / CN9.1 **tylko po identyfikacji** | Pozostawić niepodłączone, jeśli brak dowodu |
| X2 | diagnostyka (wcześniej X3) | — | M305 = X2 XOR M483 |
| X3 | MANUAL/FREE | 13 / CN8.3 | M302 = X3 XOR M485; przewód 10 na CN8.4 jest wspólnym |
| X4 | diagnostyka (wcześniej X2) | — | M304 = X4 XOR M482 |
| X5 | STOP (przycisk fizyczny) | nowa wiązka | M306 = X5 XOR M484; nie zastępuje awaryjnego STOP |
| X6 | Panel P2 | nowa wiązka | styk przycisku |
| X7 | Panel P3 | nowa wiązka | styk przycisku |
| X10 | Panel P4 | nowa wiązka | styk przycisku |
| X11 | Panel P5 | nowa wiązka | styk przycisku |
| X12 | Panel P6 | nowa wiązka | styk przycisku |
| X13 | Panel P1 (przeniesiony z X5) | nowa wiązka | wybór P1; M441 |
| X14 | ADMIN | 14/15 / CN10.1,CN10.3 | przez właściwy interfejs / izolator; **nie podawać -/+ bezpośrednio** |
| X15–X26 | dodatkowe / rezerwa | — | indywidualna diagnostyka i ekspansja |
| X27 | RM5 Evolution CH1 | RM5 CN5.7 | wyjście NPN/open collector: sprawdzić konfigurację wejścia i common |

**Uwaga adresacyjna:** FX X0..X7, X10..X17, X20..X27; nie istnieją X8,X9,X18,X19. Wspólna zacisków wejściowych **nie jest tym samym** co grupa przekaźników wyjściowych. WSC może mieć więcej niż jeden wspólny zacisk wejść; oznaczenia potwierdzić ze zdjęcia obudowy.

## Elektryka — zasilanie i masy

```text
230 VAC L/N/PE -> rozłącznik + zabezpieczenie -> zasilacz SELV 24 VDC
                                               +24V BUS
                                                 |-- F1 -> PLC DC+
                                                 |-- F2 -> RM5 CN5.2 (+24 V)
                                                 |-- F3 -> cewki 24 VDC przekaźników/styczników, lampki
                                               0V BUS
                                                 |------ PLC DC-
                                                 |------ RM5 CN5.1 (GND)
                                                 |------ powroty obwodów 24 VDC
PE zasilania ------------------------------------ szyna PE
                                                 |------ obudowa metalowa / drzwi / ekran wg projektu EMC
```

- Użyć certyfikowanego zasilacza 24 VDC SELV/PELV o zapasie prądowym dobranym do sumy obciążeń i cewek; bezpośrednio nie zasilać obwodów wyjściowych z zacisku DC+ PLC.
- Rozdzielić zaciski +24 V i 0 V na listwach, osobno zabezpieczyć PLC, RM5 i obciążenia wykonawcze. Dobrać bezpieczniki do przekrojów przewodów, prądów rozruchowych i prądów znamionowych, nie na podstawie przykładowych wartości.
- **PE** (ochronne) nie jest przewodem roboczym 0V ani wspólnym styków PITSTART. Ewentualne pojedyncze połączenie 0V–PE tylko po ustaleniu topologii PELV/EMC zasilacza i urządzenia, bez tworzenia pętli.
- Stosować oddzielne korytka / przegrody dla sieci 230 VAC i SELV 24 V; separacja galwaniczna dla wszystkich niepewnych sygnałów hosta.
- Łączyć wspólne C0/C1 ze stykiem przewodu 1 wyłącznie po potwierdzeniu, że wszystkie osiem styków przełącza **dokładnie jeden zgodny potencjał**, a przekaźniki WSC naprawdę mają oddzielne styki NO w oczekiwanym podziale.
- **Suchy styk = zamknięcie obwodu**, nie „wyjście 24V”. Dla wejścia odbiornika zwieranego do GND używać wspólnego 0V odbiornika tylko po potwierdzeniu i pomiarach. Dla wejścia podawanego +24V używać jego zgodnego +24V. Nie mieszać potencjałów na jednym COM grupy przekaźników.
- X-wejścia: ustalić **sinking/sourcing** i wspólne S/S/COM dla konkretnego WSC. Przewody 10/11/13 i 14/15 mogą mieć obcą polaryzację lub napięcie. Bez znanej referencji zastosować przekaźniki interfejsowe/optoizolację, nie mostek z 0V w ciemno.
- RM5 CH1: typowo NPN aktywne niskim, poprawny powrót wymaga zgodnego obwodu wejścia WSC; przy niezgodnej polaryzacji zastosować interfejs.
- RM5 INHIBIT pin 6: sterowanie należy potwierdzić z instrukcją i konfiguracją; dla wariantu z wejściem INHIBIT aktywowanym +24 V użyć Y23 jako suchego styku podającego +24 V z osobnego bezpiecznika. Wspólny grupy Y23 nie może być równocześnie przeznaczony na inne napięcie.
- Cewki DC: tłumienie przepięć (dioda/TVS) przy cewce z uwzględnieniem czasu odpadania. Cewki AC: odpowiedni RC/MOV; lampy 230V przez właściwy stycznik/przekaźnik, nie przez nieweryfikowany styk PLC.
- STOP programowy **nie zastępuje** E-STOP. Wyłączenie awaryjne realizować sprzętowo w odpowiedniej kategorii bezpieczeństwa, zgodnie z analizą ryzyka.

## Plan weryfikacji przed podłączeniem

1. Potwierdzić pełny model i rewizję WSC-7048R (zdjęcie tabliczki i listew, szczególnie C0/C1/C4/COM5).
2. Odłączyć energię i sprawdzić ciągłości: CN4/CN5 przewód 1, CN8.2/CN8.4 przewód 10. Zidentyfikować każdy tor przewodu 12 (CN9.1).
3. Przy bezpiecznych warunkach mierzyć AC/DC pomiędzy sygnałami i referencjami, bez domniemania wspólnej masy.
4. Zweryfikować, że wejścia hosta akceptują suche styki, jaki potencjał ma przewód 1 i wymagany maksymalny prąd.
5. Uruchomić test I/O na stole z izolowanymi sygnałami, osobno dla Y0..Y7, X0/X13/X14 i X27, sprawdzić spoczynkowe stany INHIBIT i STOP, reset zasilania.
6. Dopiero po zmianie i weryfikacji logiki PLC przełączyć okablowanie na mapę docelową.

## Źródła w repo

- [opis PITSTART](PITSTART.md), [manual PITSTART](manuals/pitstart/Gamma_Pit_PitStart_F_c27-M-PIT-EK_2009-10-13.pdf)
- [obecny schemat WSB](WIRING.md), [manual WSC](manuals/hmi/WSC_HMI_PLC_All_in_one_User_Manual_V1.13.pdf)
- [obecna mapa PLC](../plc/DEVICE_MAP.csv), [RM5](RM5.md)

## Przygotowany wariant programu PLC

- [MAIN_WSC7048R_GXDEV_ENTRY.txt](../plc/MAIN_WSC7048R_GXDEV_ENTRY.txt) — Instruction List do wprowadzania w GX Developer.
- [MAIN_WSC7048R.txt](../plc/MAIN_WSC7048R.txt) — komentowane źródło programu.
- [main_wsc7048r_v1.5.10.csv](../plc/main_wsc7048r_v1.5.10.csv) — wcześniejszy wariant. Aktualny **main_wsc7048r_v1.5.13.csv** zachowuje oryginalny ADMIN.
- [DEVICE_MAP_WSC7048R.csv](../plc/DEVICE_MAP_WSC7048R.csv) — mapa urządzeń wariantu WSC.
- [DEVICE_COMMENTS_WSC7048R.csv](../plc/DEVICE_COMMENTS_WSC7048R.csv) — komentarze urządzeń wariantu WSC.

Zmiany względem źródła V1.5.10: `OUT Y2` -> `OUT M498` (diagnostyka INHIBIT), `OUT Y23` pozostaje aktywnym INHIBIT, programy `Y3..Y7,Y10` -> `Y2..Y7`, światła `Y27` -> `Y20`. Licznik `Y0` i pompa `Y1` bez zmian. `Y21,Y22` pozostają nieużywane.

**Nie dokonano pełnej konwersji projektu natywnego PLC ani próby kompilacji.** Potwierdzić format importu CSV, typ wyjść i adresy dostępne w WSC-7048R. Fizyczny STOP awaryjny poza logiką PLC.

Proponowane mapowanie **nie jest jeszcze zwalidowanym firmware**. 

## Opis pinów PITSTART a logika programu

Przewody i piny PITSTART służą wyłącznie do opisu połączeń oraz mapowania I/O; nie stanowią nowych komend PLC.

| Wejście | Przewód i złącze | Dotychczasowa logika |
|---|---|---|
| X0 | 11 / CN8.1, wspólny 10 / CN8.2 | X0 XOR M480 -> M300 |
| X1 | 12 / CN9.1, funkcja NIEPOTWIERDZONA | X1 XOR M481 -> M301; nie podłączać bez identyfikacji |
| X3 | 13 / CN8.3, prawdopodobnie MANUAL/FREE, wspólny 10 / CN8.4 | X3 XOR M485 -> M302 |
| X14 | 14/15 / CN10.1 / CN10.3 | **LD X14; OUT M303** — ADMIN bez inwersji |

Flagi M480/M481/M485 istniały wcześniej i pozostały bez zmian. Nową inwersję ADMIN wycofano. Nie ma M488 w wykonywanym programie PLC. Nie zmieniono projektu HMI.

Przewody 14/15 opisują polaryzację wiązki, nie uprawniają do podłączenia ich bezpośrednio do X14. Wymagane jest sprawdzenie typu i napięcia sygnału.

### Nowe mapowanie wejść

Przyjęto bezkolizyjne przeniesienie: **X13 Manual/Free → X3, X2 diagnostyczne → X4, X4 STOP → X5, X5 Program 1 → X13, X3 diagnostyczne → X2**. Flagi inwersji przypisane są do funkcji, nie do dawnego numeru wejścia: M485 obsługuje teraz Manual/Free na X3; M484 obsługuje STOP na X5. ADMIN X14/M303 pozostaje bez zmian.

### Aktualna drabinka

- `plc/MAIN_WSC7048R_GXDEV_ENTRY.txt` — ADMIN jak wcześniej (X14 -> M303).
- `plc/main_wsc7048r_v1.5.13.csv` — nowy stan CSV bez inwersji X14.
- `plc/DEVICE_MAP_WSC7048R.csv` — opisy i mapowanie przewodów.

Drabinka nie jest potwierdzona kompilacją ani testem sprzętowym.

## Parametry WSC V1.5.13 i licznik HMI

Zestaw startowy przy każdym uruchomieniu PLC w RUN: `D300=25`, `D550=5`, `D551=35`, `D552=35`, `D553=50`, `D554=35`, `D555=50`, `D556=70`, `M429=0` (Sync time with RUN OFF). Wartości zapisane przez operatora podczas pracy nie są nadpisywane do następnego wejścia w RUN.

- `D596` — **nowy rejestr licznika HMI (read-only)**, 16-bit Unsigned Decimal.
- `M490=0` — wyświetl `D350`, pozostały czas w sekundach (tryb startowy).
- `M490=1` — wyświetl `D580`, dostępne jednostki kredytu po 0,10 EUR.
- `D560` / `D582` — kredyt w centach EUR, `D580` — całe jednostki 0,10 EUR. Nie są to minuty.
- HMI: Numeric Display → PLC `D596`; opcjonalny przełącznik → `M490`.
- `D350`, `D540`, `D541` i `D559` pozostają nietknięte: istniejący widok czasu nie zmieni zachowania automatycznie.

Program HMI `projects/hmi/pitstart.hs` **nie został zmieniony**. Aby zmieniać tryb z ekranu, trzeba dodać do projektu HMI kontrolkę M490 i skierować pole wyświetlacza na D596. Fizyczny ADMIN nadal uruchamia się z X14 bez inwersji.

Aktualny plik importu: [main_wsc7048r_v1.5.13.csv](../plc/main_wsc7048r_v1.5.13.csv). Wariant wymaga kompilacji w GX Developer i testów na WSC-7048R.
