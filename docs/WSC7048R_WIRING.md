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
| CN5.7 | 7 | Pump Control | Wyjście styku; NIE zasilać silnika bezpośrednio |
| CN5.10 | 9 | Coin / Counter | Impulsy 0,10 EUR wg manuala |

**Wiązka 1:** według oznaczeń jest wspólnym przewodem dla *styków*, nie jest to automatycznie GND, COM wejść PLC czy PE. Zanim zmostkujesz C0/C1: pomiar ciągłości 1 na CN4 i CN5 przy wyłączonym zasilaniu oraz pomiar napięcia względem 0V, PE i innych przewodów przy zabezpieczonym układzie. Nie łączyć różnych źródeł zasilania przez mostek COM.

**CN8:** podany dwukrotnie CN8.2 = przewód 10 jest spójny z parami wejściowymi manuala. Zweryfikować, czy CN8.2 i CN8.4 rzeczywiście są złączone.

## Docelowa mapa WSC-7048R (zmiana w stosunku do istniejącego programu)

| WSC wyjście | Funkcja | Przewód wiązki | Pin oryginału | Grupa proponowana |
|---|---|---:|---|---|
| Y0 | PITSTART_COUNTER / COIN (impulsy) | 9 | CN5.10 | C0 |
| Y1 | Pump Control (styk sterujący) | 7 | CN5.7 | C0 |
| Y2 | Program 1 | 2 | CN4.1 | C0 |
| Y3 | Program 2 | 3 | CN4.4 | C0 |
| Y4 | Program 3 | 4 | CN4.7 | C1 |
| Y5 | Program 4 | 5 | CN4.10 | C1 |
| Y6 | Program 5 | 6 | CN5.1 | C1 |
| Y7 | Program 6 | 8 | CN5.4 | C1 |
| Y20 | Work Lights (przez stycznik/przekaźnik) | nowy | — | C4 proponowana |
| Y21 | Oświetlenie pomocnicze / rezerwa | nowy | — | C4 proponowana |
| Y22 | Sygnalizacja / rezerwa | nowy | — | C4 proponowana |
| Y23 | RM5 INHIBIT | nowy (RM5 CN5.6) | — | COM5 proponowana |

**BARDZO WAŻNE:** dotychczasowy kod WSB ma Y0 Counter, Y1 Pump Control, Y2 RM5 INHIBIT, Y3..Y7/Y10 P1..P6, Y23 mirror INHIBIT oraz Y27 światła. Wariant WSC z przeadresowanymi wyjściami i wejściami jest w `plc/MAIN_WSC7048R_GXDEV_ENTRY.txt` oraz `plc/main_wsc7048r_v1.5.20.csv`. Wymaga kompilacji w GX Developer oraz testów na rzeczywistym WSC. Nie przepinać Y2 do Program 1 przy aktualnym programie. Samo rozłożenie wyjść na grupy C0/C1/C4/COM5 to hipoteza wymagająca weryfikacji z tabliczką połączeń egzemplarza; nazwy COM nie są uniwersalne między WSB, WS i WSC.

### Wejścia — aktualne WSC-7048R V1.5.20

| PLC | Funkcja | Przewód / złącze PITSTART | Logika |
|---|---|---|---|
| X0 | INVERTER_OK | 11 / CN8.1 (wspólny 10 / CN8.2) | M300 = X0 XOR M480 |
| X1 | PRACA / RUN (niepotwierdzone) | 12 / CN9.1 — nie łączyć bez identyfikacji | M301 = X1 XOR M481 |
| X2 | MANUAL/FREE | 13 / CN8.3 (wspólny 10 / CN8.4) | M302 = X2 XOR M485 |
| X3 | Diagnostyka | — | M304 = X3 XOR M482 |
| X4 | Diagnostyka | — | M305 = X4 XOR M483 |
| X14 | ADMIN | 14 i 15 / CN10.1 i CN10.3 | M303 = X14; wymagany właściwy interfejs |
| **X15** | **STOP (logiczny, nie awaryjny)** | nowy przycisk | M306 = X15 XOR M484 |
| **X16** | **Przycisk P1** | nowy przycisk | M441 = X16 OR M401 |
| **X17** | **Przycisk P2** | nowy przycisk | M442 = X17 OR M402 |
| **X20** | **Przycisk P3** | nowy przycisk | M443 = X20 OR M403 |
| **X21** | **Przycisk P4** | nowy przycisk | M444 = X21 OR M404 |
| **X22** | **Przycisk P5** | nowy przycisk | M445 = X22 OR M405 |
| **X23** | **Przycisk P6** | nowy przycisk | M446 = X23 OR M406 |
| X27 | RM5 CH1 | złącze RM5 CN5.7 (nie złącze PITSTART) | impulsy RM5 |

X5 nie służy już do STOP. X6, X7, X10–X13 oraz X24–X26 nie są używane w tym wariancie do przycisków.

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
- [main_wsc7048r_v1.5.10.csv](../plc/main_wsc7048r_v1.5.10.csv) — wcześniejszy wariant. Aktualny **main_wsc7048r_v1.5.20.csv** zachowuje oryginalny ADMIN.
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
| X2 | 13 / CN8.3, prawdopodobnie MANUAL/FREE, wspólny 10 / CN8.4 | X2 XOR M485 -> M302 |
| X14 | 14/15 / CN10.1 / CN10.3 | **LD X14; OUT M303** — ADMIN bez inwersji |

Flagi M480/M481/M485 istniały wcześniej i pozostały bez zmian. Nową inwersję ADMIN wycofano. Nie ma M488 w wykonywanym programie PLC. Nie zmieniono projektu HMI.

Przewody 14/15 opisują polaryzację wiązki, nie uprawniają do podłączenia ich bezpośrednio do X14. Wymagane jest sprawdzenie typu i napięcia sygnału.

### Nowe mapowanie wejść

Wariant V1.5.20: X2=MANUAL/FREE (M485), X3=diagnostyka (M482), X4=diagnostyka (M483), X15=STOP (M484), P1–P6 na X16, X17, X20, X21, X22, X23. X14 ADMIN i X27 RM5 bez zmian. Adresacja Mitsubishi FX jest ósemkowa: X17 -> X20.

### Aktualna drabinka

- `plc/MAIN_WSC7048R_GXDEV_ENTRY.txt` — ADMIN jak wcześniej (X14 -> M303).
- `plc/main_wsc7048r_v1.5.20.csv` — nowy stan CSV bez inwersji X14.
- `plc/DEVICE_MAP_WSC7048R.csv` — opisy i mapowanie przewodów.

Drabinka nie jest potwierdzona kompilacją ani testem sprzętowym.

## Parametry WSC V1.5.20 i licznik HMI

Zestaw startowy przy każdym uruchomieniu PLC w RUN: `D300=10`, `D549=50`, `D550=50`, `D551=35`, `D552=35`, `D553=50`, `D554=35`, `D555=50`, `D556=70`, `M429=0` (Sync time with RUN OFF). Wartości zapisane przez operatora podczas pracy nie są nadpisywane do następnego wejścia w RUN.

- `D596` — **nowy rejestr licznika HMI (read-only)**, 16-bit Unsigned Decimal.
- `M490=0` — wyświetl `D350`, pozostały czas w sekundach (tryb startowy).
- `M490=1` — wyświetl `D580`, dostępne jednostki kredytu po 0,10 EUR.
- `D560` / `D582` — kredyt w centach EUR, `D580` — całe jednostki 0,10 EUR. Nie są to minuty.
- HMI: Numeric Display → PLC `D596`; opcjonalny przełącznik → `M490`.
- `D350`, `D540`, `D541` i `D559` pozostają nietknięte: istniejący widok czasu nie zmieni zachowania automatycznie.

Program HMI `projects/hmi/pitstart.hs` **nie został zmieniony**. Aby zmieniać tryb z ekranu, trzeba dodać do projektu HMI kontrolkę M490 i skierować pole wyświetlacza na D596. Fizyczny ADMIN nadal uruchamia się z X14 bez inwersji.

Aktualny plik importu: [main_wsc7048r_v1.5.20.csv](../plc/main_wsc7048r_v1.5.20.csv). Wariant wymaga kompilacji w GX Developer i testów na WSC-7048R.

### Dwa rejestry 0/1 dla liczników HMI

| M427 | M490 | D597 — czas widoczny | D598 — kredyty widoczne |
|---|---|---:|---:|
| 0 | 0 lub 1 | 0 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

Obydwa rejestry są 16-bit RO i mają wartość wyłącznie 0 lub 1. `D596` dalej udostępnia wartość do wyświetlenia (czas lub kredyty); `D597` i `D598` sterują widocznością dwóch niezależnych obiektów HMI. Aby uzyskać osobne wartości bez przełączania, można podłączyć licznik czasu bezpośrednio do D350, a licznik kredytu do D580.

Nowy plik importu: [main_wsc7048r_v1.5.20.csv](../plc/main_wsc7048r_v1.5.20.csv). ADMIN X14 pozostaje bez zmian.

### Nowy zestaw domyślny RM5 — 5 impulsów = 5 EUR

W wersji V1.5.20 jeden impuls RM5 CH1 odpowiada 1,00 EUR (`D300=10`), a pięć impulsów daje 5,00 EUR. `D549=50` ustala limit kredytu na 5,00 EUR; `D550=50` oznacza cenę bazową 5,00 EUR. `D551=35` oznacza 35 sekund programu P1 za cenę bazową. Pozostałe czasy pozostają 35/50/35/50/70 s. `M429=0` — Sync time with RUN OFF. D596/D597/D598 i ADMIN X14 bez zmian. Uwaga: limit dokładnie 5 EUR może blokować doładowanie w trakcie sesji, gdy brak miejsca na następny pełny impuls. Wartości startowe są narzucane po wejściu PLC w RUN.

## Zmiana w V1.5.20: światła i Touch Control

- `Y20` (światła) = 1 wyłącznie gdy `D585=0` (ekran Home 000 wskazany przez PLC). Dla `D585=1` (Admin), `D585=2`, `D585=3` światła są wyłączone.
- Usunięto z drabinki `SET M414` i poprzednie `LD M414; AND M412; OUT Y20`. `M414` nie ma już wpływu na `Y20`; poprzedniej kontrolki Work Lights nie należy pozostawiać na ekranie Admin.
- **Touch Control**: `LB555` jest adresem lokalnym HMI, którego PLC FX nie steruje. Żeby kontrolki dotykowe były zawsze widoczne, w edytorze `pitstart.hs` należy usunąć przycisk przełączający `LB555` oraz usunąć warunki ukrywania przycisków/obiektów zależne od `LB555` (ustawić widoczność Always/bezwarunkową), po czym ponownie skompilować i wgrać projekt HMI. Tej modyfikacji HMI **jeszcze nie wykonano**, ponieważ repo przechowuje natywny binarny format projektu.
- `D585` jest ekranem **zadanym przez PLC**, a niekoniecznie ekranem faktycznie oglądanym przez operatora. Jeśli światła mają reagować na rzeczywistą stronę HMI, należy rozważyć `D586` (HMI Screen Index) po potwierdzeniu aktualizacji tego rejestru przez panel.
- Przed włączeniem na urządzeniu sprawdzić stan wyjścia Y20 przy starcie PLC i zmianach ekranów 000/001/002/003.


## Zbiorcza tabela montażowa (WSC-7048R V1.5.20)

| Funkcja | I/O na HMI/PLC | Nr przewodu | Nr złącza i pin CNx | Opis PITSTART |
|---|---|---|---|---|
| Inverter OK / obecność automatu | X0 | 11 | CN8.1 | Sygnał obecności/zezwolenia; drugi biegun CN8.2 przewód 10 |
| RUN / praca (niepotwierdzone) | X1 | 12 | CN9.1 | Funkcja nieustalona, wymaga pomiaru; bez podłączenia |
| Manual / Free | X2 | 13 | CN8.3 | Wejście Manual/Free wg manuala CN8.3–4; CN8.4 przewód 10 |
| Diagnostyka | X3 | — | — | Brak przypisanego przewodu PITSTART |
| Diagnostyka | X4 | — | — | Brak przypisanego przewodu PITSTART |
| ADMIN | X14 | 14 (−), 15 (+) | CN10.1, CN10.3 | ADMIN HOST PLC; wymagany interfejs, nie łączyć wprost bez pomiaru |
| STOP (przycisk) | **X15** | nowa wiązka | — | Przycisk zewnętrzny; nie jest sygnałem ze złącza PITSTART |
| Program 1 — przycisk | **X16** | nowa wiązka | — | Przycisk panelu; odpowiada wyjściu PITSTART P1 na CN4.1 |
| Program 2 — przycisk | **X17** | nowa wiązka | — | Przycisk panelu; odpowiada wyjściu PITSTART P2 na CN4.4 |
| Program 3 — przycisk | **X20** | nowa wiązka | — | Przycisk panelu; odpowiada wyjściu PITSTART P3 na CN4.7 |
| Program 4 — przycisk | **X21** | nowa wiązka | — | Przycisk panelu; odpowiada wyjściu PITSTART P4 na CN4.10 |
| Program 5 — przycisk | **X22** | nowa wiązka | — | Przycisk panelu; odpowiada wyjściu PITSTART P5 na CN5.1 |
| Program 6 — przycisk | **X23** | nowa wiązka | — | Przycisk panelu; odpowiada wyjściu PITSTART P6 na CN5.4 |
| RM5 CH1 — impuls monety | X27 | osobny przewód RM5 | RM5 CN5.7 | Wejście monetnika, **nie** PITSTART CN5.7 |
| Counter / Coin | Y0 | 9 | CN5.10 | Styk licznika, wspólny przewód 1 na CN5.12 |
| Pump Control | **Y1** | 7 | CN5.7 | Styk sterowania pompą, wspólny przewód 1 na CN5.9 |
| Program 1 — wyjście | Y2 | 2 | CN4.1 | Styk P1, wspólny przewód 1 na CN4.3 |
| Program 2 — wyjście | Y3 | 3 | CN4.4 | Styk P2, wspólny przewód 1 na CN4.6 |
| Program 3 — wyjście | Y4 | 4 | CN4.7 | Styk P3, wspólny przewód 1 na CN4.9 |
| Program 4 — wyjście | Y5 | 5 | CN4.10 | Styk P4, wspólny przewód 1 na CN4.12 |
| Program 5 — wyjście | Y6 | 6 | CN5.1 | Styk P5, wspólny przewód 1 na CN5.3 |
| Program 6 — wyjście | Y7 | 8 | CN5.4 | Styk P6, wspólny przewód 1 na CN5.6 |
| Work Lights | **Y20** | nowa wiązka | — | Światła niezależne od PITSTART; Y20 ON przy D585=0 (Home) |
| Rezerwa | Y21, Y22 | — | — | Wyjścia nieużywane |
| RM5 INHIBIT | Y23 | osobny przewód RM5 | złącze RM5, pin do potwierdzenia | Inhibit monetnika, **nie** przewód wspólny PITSTART CN5.6 |
| PITSTART common — wyjścia P1–P4 | C0/C1 (propozycja) | 1 | CN4.3, CN4.6, CN4.9, CN4.12 | Wspólny styku NO; nie zakładać 0 V/+24 V |
| PITSTART common — wyjścia P5–P6/pompa/counter | C0/C1 (propozycja) | 1 | CN5.3, CN5.6, CN5.9, CN5.12 | Wspólny styku NO; nie zakładać 0 V/+24 V |
| PITSTART common — wejścia CN8 | COM wejść przez interfejs | 10 | CN8.2, CN8.4 | Wspólny obwodów wejść; odrębny od przewodu 1 |

Uwaga: kolumna I/O podaje **adres logiczny w PLC/HMI**, nie numer fizycznego styku listwy WSC. Fizyczne zaciski WSC i grupy COM sprawdzić w dokumentacji konkretnej rewizji. Złącza RM5 i PITSTART mają własne oznaczenia CN5 — są to **różne złącza**.

## V1.5.20 — opcjonalne kasowanie kredytów po STOP

| Adres | Funkcja | Dostęp HMI | Domyślnie |
|---|---|---|---|
| M491 | RESET_CREDIT_ON_STOP: 0 zachowaj kredyty / 1 skasuj po STOP | RW toggle Admin | OFF (0) przy wejściu w RUN |
| M492 | STOP_CREDIT_RESET_PULSE — jednoskanowy impuls resetu | RO diagnostyka | 0 |
| M400 | HMI STOP przycisk chwilowy | RW momentary | — |
| X15 | Fizyczny przycisk STOP | wejście PLC | — |
| D560 | Pozostały kredyt w centach | RO | — |

Przy M491=1 narastające M440 = X15 po inwersji M484 OR M400 wyzwala pojedynczy reset. PLC kasuje nie tylko D560, ale także kolejkę Y0 i aktywną paczkę RM5, żeby kredyt nie wrócił po zatrzymaniu. W trakcie przyjmowania monet takie kasowanie oznacza utratę niezliczonych jeszcze impulsów bez zwrotu monet; wymaga świadomej decyzji operatora i testów bezpieczeństwa/logiki rozliczania. M491 jest ustawiany na 0 po uruchomieniu PLC w RUN. Projekt binarny HMI pitstart.hs nie jest zmodyfikowany.
