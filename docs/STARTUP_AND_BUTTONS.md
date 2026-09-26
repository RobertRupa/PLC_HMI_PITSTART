# Start i parametry — V1.3.1

Na M8002:
- zerowane są D320/D330/D350 i rejestry robocze/liczniki sesji,
- D520 pozostaje bez zmian, jeżeli 1…600; inaczej =10,
- D528 pozostaje bez zmian, jeżeli 1…500; inaczej =100,
- D529 nie jest używany,
- M410 jest SET,
- M414 jest SET,
- programy M420…M425 są resetowane,
- M417 jest resetowane,
- wersja HMI ustawiana jest na V1.3.1.

D330 jest zawsze zerowane przy starcie, aby stara kolejka CREDIT nie została wznowiona.

D350 jest zerowane przy starcie zgodnie z polityką bezpieczeństwa projektu.

## Nowa sesja

Nowa sesja zaczyna się przy pierwszym zaakceptowanym impulsie RM5, gdy D350<=0:

```text
M320 AND D350<=0 -> M417
```

M417 zeruje:
- D320,
- D521:D526,
- D530:D535,

a następnie pierwszy impuls nowej sesji jest normalnie liczony.

STOP nie tworzy nowej sesji, nie zeruje D350 i nie kasuje liczników bieżącej sesji.
