# Oprogramowanie serwisowe

Poniższe linki prowadzą do kopii instalatorów i dokumentacji używanych przy obsłudze urządzeń związanych z projektem.

## RM5 Evolution

### Clone5 Professional

Program do konfiguracji i diagnostyki RM5 Evolution.

- [Clone5 Professional — Google Drive](https://drive.google.com/file/d/1hJBpGC8fXR5bTwJC6GsIhJDk0oNyu0VW/view?usp=drive_link)

### Clone5

Starsza wersja programu do konfiguracji RM5.

- [Clone5 — Google Drive](https://drive.google.com/file/d/1Y16ktjU0Uoaz9X2j15VbVoOdI_D4C2Ow/view?usp=drive_link)

Szczegóły konfiguracji RM5, połączenia TTL i wymogu COM1: [RM5.md](RM5.md).

## EuroKey / Unio

### Unio

Program używany do konfiguracji urządzeń EuroKey.

- [Unio — Google Drive](https://drive.google.com/file/d/1zQiwiUUEiSnq7PA96H1h-11UYMGSpPD7/view?usp=drive_link)

### Dokumentacja Unio

- [Unio — dokumentacja PDF / pakiet dokumentacji — Google Drive](https://drive.google.com/file/d/1i7EBozYrq4U4nZQtFW9QgaZ4W6pq057S/view?usp=drive_link)

## Uwaga

Linki powyżej wskazują na zewnętrzne pliki Google Drive. Repozytorium nie zawiera tych instalatorów. Przed użyciem na innym stanowisku warto zachować lokalną kopię wersji, która została sprawdzona z konkretnym urządzeniem.

## HMI SEEKU / Winsun

Producent dla rodziny WSB opisuje oprogramowanie **HMI Studio / HMI_Setup 5.1**.

- [HMI_Setup V5.1 — pakiet producenta](https://www.winsunzk.cn/upload/application/upload_594f4ede25a16934f8723e5d2ff08091.zip)
- [lokalny manual WSB V1.79](manuals/hmi/WSB_HMI_PLC_All_in_one_User_Manual_V1.79.pdf)
- [lokalny manual WSC V1.13](manuals/hmi/WSC_HMI_PLC_All_in_one_User_Manual_V1.13.pdf)

Aktualny plik `projects/hmi/pitstart.hs` ma w nagłówku identyfikator **KinSealStudio V1.0.2** i odwołania do zasobów `WSZKHMI5.1En`. Te nazwy dotyczą środowiska/profilu projektu; manual WSB pozostaje podstawowym źródłem konfiguracji komunikacji dla WSB7020R.
