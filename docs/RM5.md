# Comestero RM5 Evolution — V1.5.5

## Założenie projektu

PLC korzysta tylko z jednego wejścia impulsowego z akceptora:

- RM5 pin 7 / CH1 -> X27.

Dlatego **wszystkie używane kanały/nominały RM5 muszą być skonfigurowane tak, aby impulsy trafiały na wyjście CH1**.

PLC nie odczytuje bezpośrednio CH2…CH6.

Jeżeli różne nominały mają być rozróżniane wartością, RM5 powinien zakodować je odpowiednią liczbą impulsów CH1. PLC zlicza impulsy na X27 i każdy impuls wycenia według `D300`.

Przykład:
- D300=1 -> jeden impuls CH1 = 0,10 EUR,
- moneta 0,50 EUR powinna wtedy dać 5 impulsów CH1,
- moneta 1,00 EUR -> 10 impulsów CH1,
- moneta 2,00 EUR -> 20 impulsów CH1.

Jeżeli D300=10, jeden impuls CH1 jest wart 1,00 EUR. W takim ustawieniu pojedynczy impuls nie może sam rozróżnić monet o innych wartościach.

## Połączenie RM5

Przy zasilaniu 24 VDC:

- pin 1 GND -> 0 V,
- pin 2 zasilanie -> +24 VDC,
- pin 7 CH1 -> X27,
- pin 6 INHIBIT <- Y2 (główne fizyczne wyjście),
- Y23 pozostaje programowym mirrorem INHIBIT dla przyszłego sterownika z większą liczbą fizycznych wyjść.

## INHIBIT

Aktualna logika:

```text
X0 = 0 -> Y2 = 1 i Y23 = 1 -> RM5 zablokowany
M435 = 1 -> Y2 = 1 i Y23 = 1 -> osiągnięty/zarezerwowany limit kredytu
X0 = 1 i M435 = 0 -> Y2 = 0 i Y23 = 0 -> RM5 odblokowany
```

Instruction List:

```text
LDI X0
OR M435
OUT Y2

LDI X0
OR M435
OUT Y23
```

Na obecnym sterowniku do pinu 6 RM5 należy używać **Y2**. Y23 jest zachowane wyłącznie jako zgodny logicznie zapas pod przyszły PLC.

## Połączenie serwisowe TTL z Clone5 Professional

Do programowania RM5 używane jest połączenie szeregowe **TTL**. Nie jest to klasyczny port RS-232 z poziomami napięć ±12 V.

Połączenie:

```text
USB-UART TTL TX  -> RM5 RX
USB-UART TTL RX  -> RM5 TX
USB-UART TTL GND -> RM5 GND
```

RM5 powinien być zasilany normalnie z instalacji. Linie komunikacyjne wymagają wspólnej masy. Nie podawać 24 V na TX/RX interfejsu TTL.

### COM1

W użytej konfiguracji **Clone5 Professional wykrywa RM5 po ustawieniu interfejsu jako COM1**. Jeżeli adapter USB-UART pojawi się jako COM3, COM5 itd., trzeba zmienić numer portu w Windows:

```text
Menedżer urządzeń
  -> Porty (COM i LPT)
  -> właściwości adaptera USB-UART
  -> Ustawienia portu
  -> Zaawansowane
  -> Numer portu COM: COM1
```

Po zmianie numeru portu zamknąć i uruchomić ponownie Clone5 Professional. Jeżeli COM1 jest zajęty przez nieużywane urządzenie, najpierw zwolnić ten numer.

## Dane kalibracyjne

Wartości pokazane w zakładce kalibracji dotyczą **konkretnego egzemplarza RM5**. Nie należy ich kopiować do innego akceptora.

Kalibracja opisuje charakterystykę czujników i tolerancje zaprogramowane dla danego mechanizmu. Przy wymianie RM5, płyty elektroniki albo głowicy pomiarowej należy zachować dane właściwe dla tego urządzenia lub przeprowadzić poprawną procedurę kalibracji.

W repo zrzut kalibracji służy jako dokumentacja tego egzemplarza, a nie jako zestaw wartości wzorcowych.


## Konfiguracja Clone5 / RM5

W Clone5 należy skonfigurować akceptor tak, aby:

1. wszystkie używane nominały były aktywne,
2. wszystkie te nominały generowały sygnał na fizycznym wyjściu CH1,
3. liczba impulsów CH1 odpowiadała wartości monety w przyjętej jednostce projektu,
4. INHIBIT był aktywny z wejścia inhibit RM5.

Projekt nie zakłada osobnych wejść PLC dla CH2…CH6.

## Wartość impulsu

`D300` = wartość pojedynczego impulsu CH1 w jednostkach 0,10 EUR.

```text
D300 = 1  -> 0,10 EUR / impuls
D300 = 5  -> 0,50 EUR / impuls
D300 = 10 -> 1,00 EUR / impuls
```

Po zakończeniu paczki:

```text
COUNTER_PULSES = RM5_CH1_PULSES * D300
```

Każdy impuls Y0/Counter odpowiada 0,10 EUR.

## Akceptacja programowa

Impuls X27 jest przyjmowany tylko przy:

- M300=1,
- M413=1,
- M415=1.

Y2/Y23 INHIBIT zależą od X0 oraz M435. M435 jest ustawiane także wtedy, gdy kolejny pełny impuls RM5 nie mieści się już w wolnym limicie lub bieżąca paczka RM5 wypełniła pozostałe miejsce.

## Diagnostyka

- D320 — impulsy bieżącej paczki RM5,
- D521 — impulsy ostatniej paczki,
- D522 — liczba impulsów Y0 wyliczona z ostatniej paczki,
- D330 — kolejka Y0,
- D524 — impulsy RM5 sesji,
- D533 — impulsy Y0 faktycznie wysłane,
- D565 — licznik zaakceptowanych zdarzeń RM5,
- M419 / D587 — żądanie wake HMI przez około 3 s.


## Clone5 Professional — zrzuty konfiguracji

Pliki są przechowywane w `docs/images/rm5/`.

### Control / diagnostyka

![Clone5 Control](images/rm5/clone5_control.png)

Podgląd stanu wejść, INHIBIT, Anti-Fishing, Cash Sensor i diagnostyki RM5.

### Kanały 1–10

![Clone5 Channels](images/rm5/clone5_channels_1_10.png)

Ustawienia kanałów monet. W tym projekcie PLC używa wyłącznie fizycznego wyjścia CH1.

### Configuration

![Clone5 Configuration](images/rm5/clone5_configuration.png)

Istotne dla projektu:
- tryb `00 - Validator`,
- aktywne `Inhibition Id`,
- czas impulsu kredytowego dopasowany do wejścia PLC,
- wartości monet wyprowadzane jako impulsy CH1.

### Calibration

![Clone5 Calibration](images/rm5/clone5_calibration.png)

**Nie kopiować tych wartości do innego RM5.** Są to dane konkretnego egzemplarza.

### Połączenie / port COM

![Clone5 COM1](images/rm5/clone5_com1.png)

Interfejs programujący pracuje po TTL. W tej konfiguracji Clone5 Professional wymaga przypisania adapterowi numeru `COM1`.

## Limit kredytu

`D549` ma zakres 1…999 jednostek po 0,10 EUR.

```text
D549 = 50  -> 5,00 EUR
D549 = 999 -> 99,90 EUR
```

Do limitu wliczany jest:
- kredyt w D560,
- kredyt odpowiadający impulsom oczekującym w D330.

Po wypełnieniu limitu M435 blokuje RM5 przez Y2/Y23. Nowa paczka jest dodatkowo ograniczana do liczby impulsów, które mieszczą się jeszcze poniżej D549.
