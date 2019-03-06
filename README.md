# block-ark

Block with all sorts of bus interfaces

## Bus Interfaces

| type | Initiator | Target | mem |
|-|-|-|-|
| AMBA4.AXI4-Lite  | i2  | t1  | ✓ |
| AMBA4.AXI4       | i4  | t3  | ✓ |
| AMBA4.AHBLite    | i6  | t5  | ✓ |
| AMBA3.APB        | i8  | t7  | ✓ |
| MEM.SPRAM        | i10 | t9  | ✓ |
| MEM.DPRAM        | i12 | t11 | ✓ |
| AMBA4.AXI4Stream | i14 | t13 |   |
| TileLink         |     |     | ✓ |
