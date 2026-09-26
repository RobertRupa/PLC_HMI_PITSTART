# GX CSV import mapping — V1.4.0

The V1.4.0 CSV contains 9 columns and no header row.

In the import screen shown by GX Converter / list CSV import, assign:

| CSV column | Data Format |
|---|---|
| A | Step number |
| B | Do not Import (Skip) |
| C | Instruction |
| D | I/O(Device) |
| E | Note |
| F | Do not Import (Skip) |
| G | Do not Import (Skip) |
| H | Do not Import (Skip) |
| I | Do not Import (Skip) |

Column E contains ASCII comments so older MELSOFT installations do not have to handle Polish code-page characters.

Rows containing the second or third operand of an instruction use column D as in the current GX export. Their Note cell is intentionally blank.

Time recalculation after program change:
- HMI M401...M406 selects P1...P6.
- Selection writes the matching D551...D556 value into D557.
- D560 monetary credit is NOT changed.
- D350 is recalculated continuously from D560, D557 and D550.
Therefore changing program changes the displayed D540:D541 time on the next PLC scan while preserving the remaining monetary credit.
