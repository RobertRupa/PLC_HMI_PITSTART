# PLC text export

This directory contains a text representation of the current PLC logic.

## Files

- `MAIN.txt` — mnemonic / instruction-list representation of the current ladder logic.
- `DEVICE_MAP.csv` — compact device/address map for review and spreadsheet use.

## Recommended workflow

1. Keep the complete native **GX Developer project** as the authoritative backup.
2. Keep `MAIN.txt` in Git for readable diffs and code review.
3. When transferring to GX Developer, use the mnemonic list as the reference for import/paste/rebuild. Exact text-import steps depend on the installed GX Developer/GX Converter version.
4. After import or manual entry, compile/convert the project in GX Developer and compare online monitoring against the documented I/O map before enabling the machine.

## Important startup behavior

The first-scan block uses `M8002` to force:
- `D300 = 10`,
- `D320 = 0`,
- `D330 = 0`,
- `D350 = 0`,
- `D351 = 0`,
- `D520 = 10` seconds/pulse,
- `D521…D526 = 0`,
- all program-selection bits OFF,
- Pilotaggio OFF.

Clearing `D330` prevents the PLC from resuming an old CREDIT pulse queue after a restart.

## RM5

- CH1 input: `X27`
- INHIBIT: `Y23`
- channel-1 multiplier/value: `D300 = 10`

## Local buttons

- STOP: `X4`
- P1: `X5`
- P2: `X6`
- P3: `X7`
- P4: `X10`
- P5: `X11`
- P6: `X12`

HMI commands use `M400…M406` and are ORed with the mechanical buttons.

## PLC version for HMI

The PLC exposes version `V1.1.0` starting at `D500`.

- `D500…D507` — ASCII version string area
- `D516` — major
- `D517` — minor
- `D518` — patch

WSStudio should use an ASCII/String display starting at `D500`, e.g. 16 characters. If character pairs appear reversed, swap the byte order of the HEX constants in the initialization block.

## HMI pulse counter and time conversion

- `D520` — seconds per RM5 pulse, HMI RW, default 10, valid 1…600
- `D521` — last packet pulse count
- `D522:D523` — last packet converted seconds
- `D524` — session pulse counter
- `D525:D526` — session equivalent seconds
- `M411` — momentary reset for session counter
- `M413` — time parameter valid

Every accepted RM5 pulse adds `D520` seconds to `D350`. Remaining time is saturated at 32000 s.

