# Changelog

## V1.5.2

- RM5 INHIBIT przechodzi w stan wysoki, gdy kolejny pełny impuls RM5 nie mieści się już poniżej D549,
- bieżąca paczka RM5 rezerwuje wolne miejsce przez M437,
- dodano D594 z liczbą wolnych impulsów Y0,
- dodano T204/M438 jako niezależne okno resetu stanu sesji po wejściu PLC w RUN,
- podczas resetu startowego zerowane są kredyt, czas, kolejki, liczniki sesji i stany M435…M437,
- wersja PLC V1.5.2.


## V1.5.1

- D549 pozostaje w zakresie 1…999,
- M435 uwzględnia kredyt w D560 oraz impulsy oczekujące w D330,
- paczki RM5 są dopisywane do D330 zamiast zastępować istniejącą kolejkę,
- liczba nowych impulsów Y0 jest ograniczana do wolnego miejsca poniżej D549,
- dodano M436 RM5_TOPUP_BUSY,
- podczas M436=1 zużycie kredytu jest wstrzymane do końca obsługi zaakceptowanego doładowania,
- późniejsze doładowanie nie zmienia D558 ani aktywnego programu,
- pierwszy skan zeruje D588…D592 oraz M436,
- M430 pozostaje zgodne z: PRACA OR /SYNC OR /PILOTAGGIO.


## V1.5.0

- zwiększono zakres D549 MAX CREDIT do 1…999 jednostek po 0,10 EUR,
- zmieniono wewnętrzną reprezentację D560 na centy, co pozwala obsłużyć D549=999 bez przepełnienia rejestru 16-bit,
- dodano M435 MAX_CREDIT_REACHED,
- po osiągnięciu limitu kredytu Y2/Y23 aktywują RM5 INHIBIT, a X27 nie jest dalej przyjmowane,
- doładowanie aktywnej sesji nie zmienia wybranego programu,
- Auto Start wymaga D558=0 i nie może przełączyć programu po późniejszym doładowaniu,
- zmieniono bramkę odliczania: przy M429=1 PRACA/X1 zatrzymuje czas tylko gdy M410/Pilotaggio jest włączone,
- potwierdzono zerowanie kredytu, czasu i liczników bieżącej sesji na pierwszym skanie PLC,
- zaktualizowano dokumentację oraz ekran Admin.


## V1.4.9

- rozdzielono rejestry sterowania ekranem i statusu HMI,
- `D585` pozostaje indeksem ekranu zadawanym przez PLC,
- `D586` jest indeksem aktualnego ekranu zapisywanym przez HMI,
- `D587` jest 0/1 mirrorem `M419` dla sygnału RM5/wake,
- dodano pliki projektów PLC i HMI,
- dodano zrzuty ekranów HMI oraz konfiguracji WSStudio,
- dodano zrzuty konfiguracji RM5 z Clone5 Professional,
- uzupełniono dokumentację konfiguracji HMI i RM5.


## V1.4.8

- ustawiono domyślnie `M429=1` — Sync countdown with PRACA,
- ustawiono domyślnie `M431=1` — Auto Start Program,
- zmieniono `D585` z prostego mirroru Admin na `HMI_SCREEN_INDEX`,
- `D585=0` — Main/Work,
- `D585=1` — Admin, gdy X14=1,
- `D585=2` — Stanowisko nieczynne, gdy Admin OFF i X0=0,
- `D585=3` — Stanowisko wolne, gdy Admin OFF, X0=1 i M427=0,
- priorytet ekranów: Admin > Nieczynne > Wolne > Main,
- zaktualizowano dokumentację HMI, I/O i pliki importu,
- wersja PLC/HMI V1.4.8.


## V1.4.7

- przeniesiono główne fizyczne RM5 INHIBIT z Y23 na Y2,
- Y23 pozostaje aktywnym mirrorem INHIBIT dla przyszłego PLC z większą liczbą wyjść,
- przesunięto wyjścia programów: P1=Y3, P2=Y4, P3=Y5, P4=Y6, P5=Y7, P6=Y10,
- udokumentowano ósemkową adresację FX: po Y7 występuje Y10, adres Y8 nie istnieje,
- dodano wymaganie konfiguracji RM5: wszystkie używane kanały/nominały mają wysyłać impulsy przez CH1 podłączony do X27,
- rozszerzono opis kodowania wartości monet liczbą impulsów CH1 i zależności od D300,
- zaktualizowano README, opis funkcjonalności, mapę I/O, komentarze urządzeń i dokumentację RM5,
- wersja PLC/HMI V1.4.7.


## V1.4.6

- dodano `D585 = ADMIN_SCREEN_REQUEST_WORD`, 0/1 mirror X14/M303,
- dodano `D586 = HMI_WAKE_REQUEST_WORD`, 0/1 mirror M419,
- rejestry są przeznaczone dla HMI, które nie może używać bitów M jako triggerów ekranów,
- wersja PLC/HMI V1.4.6.


## V1.4.5

- dodano fizyczny przełącznik ADMIN na X14,
- dodano `M303 = ADMIN_SCREEN_REQUEST`,
- `M303` bezpośrednio śledzi stan X14,
- HMI może użyć M303=1 do ekranu Admin i M303=0 do Home,
- wersja PLC/HMI V1.4.5.


## V1.4.4

- zastąpiono Auto Start Timer funkcją Auto Start Program,
- `M431 = AUTO_START_PROGRAM_ENABLE`,
- dodano `D584 = AUTO_START_PROGRAM_NO` z zakresem 1…6,
- `M432` jest jednocyklowym AUTO_START_TRIGGER,
- `M433` zapamiętuje wykonanie/anulowanie Auto Start w bieżącej sesji,
- `M434` sygnalizuje poprawny numer programu,
- dodano M460…M465 Auto Select i M470…M475 Effective Select,
- Auto Start następuje dopiero po pełnym zakończeniu kolejki Y0/CREDIT,
- ręcznie uruchomiony program nie jest później nadpisywany przez Auto Start,
- STOP podczas oczekiwania anuluje Auto Start dla bieżącej sesji,
- odliczanie czasu nie jest już opóźniane przez M431,
- wersja PLC/HMI V1.4.4.


## V1.4.3

- dodano `M431 = AUTO_START_TIMER_ENABLE`,
- dodano `M432 = AUTO_TIMER_ACTIVE`,
- dodano `M433 = COUNTDOWN_ENABLE`,
- przy M431=ON odliczanie rozpoczyna się dopiero po całkowitym zakończeniu kolejki Y0/CREDIT,
- jeśli CREDIT zakończą się przed wyborem programu, Auto Timer wystartuje po późniejszym wyborze programu,
- STOP, utrata X0 i brak kredytu resetują Auto Timer,
- synchronizacja M429/PRACA nadal obowiązuje,
- wersja PLC/HMI V1.4.3.


## V1.4.2

- dodano `D559 = TIME_BAR_MAX` dla paska czasu HMI,
- D559 jest zerowane przy nowej sesji i zmianie programu,
- D559 rośnie, gdy doładowanie zwiększa dostępny czas,
- D559 nie maleje podczas odliczania D350,
- wersja PLC/HMI V1.4.2.


## V1.4.1

- Y23/RM5_INHIBIT zależy wyłącznie od X0,
- X0=0 -> Y23=1,
- X0=1 -> Y23=0,
- M413/M415 nadal filtrują programowe przyjęcie impulsu X27, ale nie sterują INHIBIT,
- wersja PLC/HMI V1.4.1.


## V1.4.0

- przebudowano model na kredyt pieniężny zgodny z PitStart,
- Y0/Counter = jednostka 0,10 EUR,
- D300 = wartość RM5 CH1 w jednostkach 0,10 EUR,
- D549 = maksymalny kredyt,
- D550 = wspólna cena bazowa,
- D551…D556 = czas P1…P6 dla ceny bazowej,
- D560 = pozostały kredyt z rozdzielczością 1/100 jednostki Counter,
- D350/D540/D541 są obliczane z kredytu i taryfy wybranego programu,
- zmiana programu zachowuje kredyt i zmienia wynikowy czas,
- dodano M429 do opcjonalnej synchronizacji zużycia kredytu z X1/M301 PRACA,
- D528 pozostaje globalną korekcją zegara x100,
- dodano M419 HMI_WAKE_REQUEST na około 3 s po monecie,
- dodano D565 HMI_WAKE_EVENT_COUNTER,
- dodano D582 = pozostały kredyt w centach,
- wersja PLC/HMI V1.4.0.

## V1.3.2

- D300 działał jako Coin multiplier,
- czas był naliczany podczas wysyłki impulsów Y0.

## V1.3.0

- dodano M414 WORK_LIGHTS_ENABLE,
- dodano pierwszą wersję korekcji zegara i MM:SS.

## V1.2.0

- przygotowano czysty program FX3UC,
- dodano P1…P6, STOP i bezpieczny restart.
