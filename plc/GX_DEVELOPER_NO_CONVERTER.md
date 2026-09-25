# Wprowadzenie programu do GX Developer bez GX Converter

Docelowy PLC projektu: **FX3UC**. Plik `MAIN_GXDEV_ENTRY.txt` zawiera czystą listę instrukcji bez numerów kroków. GX Developer wylicza kroki sam.

## Najbezpieczniejsza procedura

1. Zrób kopię obecnego projektu testowego: **Project -> Save as**.
2. Otwórz `Program -> MAIN`.
3. Przełącz na **Instruction List**: `View -> Instruction list` albo `Alt+F1`.
4. Usuń testowy program MAIN. Zostaw pusty program/END albo utwórz nowy MAIN.
5. Ustaw **Insert mode** klawiszem `Insert`.
6. Wprowadzaj kolejne wiersze z `MAIN_GXDEV_ENTRY.txt`. GX Developer przyjmuje instrukcję w postaci np. `LD X0`, `MOV K10 D300`; po każdym wierszu zatwierdź Enter.
7. Po zakończeniu naciśnij **F4 / Convert**.
8. Przełącz `Alt+F1` do Ladder i sprawdź, czy nie ma żółtych/błędnych bloków.
9. Wykonaj **Tools/Diagnostics -> Check program** (nazwa zależy od wersji GX Developer).
10. Dopiero po sprawdzeniu wykonaj zapis do PLC.

GX Developer oficjalnie obsługuje wprowadzanie programu bezpośrednio w Instruction List; GX Converter nie jest do tego wymagany.

## Co różni V1.2.0 od wcześniejszego pliku V1.1.1

- bezpieczne impulsy wyboru programu `M451..M456` zamiast wielu SET/RST po jednym warunku IL,
- utrata `X0/AUTOMATE_PRESENT` natychmiast kasuje wybór programu,
- wyjścia P1..P6 są dodatkowo blokowane przez `M300`,
- `D350` odlicza czas tylko podczas aktywnego programu (`M412=1`), więc STOP pauzuje czas,
- `M410/PILOTAGGIO_ENABLE` startuje jako ON, aby sterowanie mechaniczne działało również bez HMI,
- `D520` nie jest bezwarunkowo nadpisywane przy restarcie; jeśli ma poprawną wartość 1..600, zostaje zachowane,
- `M411` sam się kasuje po resecie licznika sesji,
- wersja PLC/HMI: `V1.2.0`.

## Manual / Free

W tej wersji `X13 -> M302` jest **statusem tylko do odczytu**, zgodnie z aktualnym repo. Nie uruchamia programów bez kredytu. Funkcję pełnego trybu FREE można dodać osobno po potwierdzeniu oczekiwanego zachowania.
