# Start i parametry — V1.3.0

Na M8002:
- zerowane są D320/D330/D350 i rejestry robocze/liczniki sesji,
- D520 pozostaje bez zmian, jeżeli 1…600; inaczej =10,
- D528 pozostaje bez zmian, jeżeli 1…1000; inaczej =100,
- D529 pozostaje bez zmian, jeżeli 1…1000; inaczej =100,
- M410 jest SET,
- M414 jest SET,
- programy M420…M425 są resetowane,
- wersja HMI ustawiana jest na V1.3.0.

D330 jest zawsze zerowane przy starcie, aby stara kolejka CREDIT nie została wznowiona.

D350 jest zerowane przy starcie zgodnie z dotychczasową polityką bezpieczeństwa projektu.
