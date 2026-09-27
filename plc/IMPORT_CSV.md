# GX CSV import mapping — V1.4.7

The canonical import file is `main_v1.4.7.csv`. It contains no Note/P-I/Line statement data.

Assign columns in the GX import dialog:

| CSV column | Data Format |
|---|---|
| A | Step number |
| B | Do not Import (Skip) |
| C | Instruction |
| D | I/O(Device) |
| E | Do not Import (Skip) |
| F | Do not Import (Skip) |
| G | Do not Import (Skip) |
| H | Do not Import (Skip) |
| I | Do not Import (Skip) |

Device comments are kept separately in:
- `DEVICE_COMMENTS.csv`
- `DEVICE_COMMENTS.txt`

The separate comments table avoids the importer rejecting Note data.
