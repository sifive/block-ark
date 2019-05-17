# block-ark

Block with all sorts of bus interfaces from
[duh-bus](https://github.com/sifive/duh-bus)

## Scala Compile Test
requires wake 0.14 and wit 0.5
```
wit init workspace -a git@github.com:sifive/block-ark.git
cd workspace
wake --init .
duh-export-scala block-ark/ark.full.json5 -o block-ark/src/
wake -v compileScalaModule arkScalaModule
```

## Bus Interfaces

| type | Initiator     | Target | mem |
|-|-|-|-|
| AMBA4.AXI4-Lite      | i2  | t1  | ✓ |
| AMBA4.AXI4           | i4  | t3  | ✓ |
| AMBA4.AHBLite        | i6  | t5  | ✓ |
| AMBA3.APB            | i8  | t7  | ✓ |
| MEM.SPRAM            | i10 | t9  | ✓ |
| MEM.DPRAM            | i12 | t11 | ✓ |
| AMBA4.AXI4Stream     | i14 | t13 |   |
| sifive.basic.channel | i16 | t15 |   |
| AMBA3.AXI            |     |     | ✓ |
| TileLink             |     |     | ✓ |
| Bundle               |     |     |   |

## Walkthrough example import using duh

First ensure [duh](https://github.com/sifive/duh) is installed and clone
the block-ark repository

```console
% git clone https://github.com/sifive/block-ark.git
% cd block-ark
```

#### Infer mappings to known bus definitions

Run `duh-portinf` to infer initial bus interface matches present in the
ark design.

```console
% duh-portinf -o ark-busprop.json ark.json5
```

The resulting `ark-busprop.json` file contains inferred bus interfaces
present in the ark design:

```json
"component": {
    "vendor": "sifive",
    "library": "blocks",
    "name": "ark",
    "version": "0.1.0",
    "busInterfaces": [
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_0-mapping_0-prefix_t3_axi4-slave-AXI4_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_1-mapping_0-prefix_i4_axi4-master-AXI4_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_2-mapping_0-prefix_i2_axi4lite-master-AXI4-Lite_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_3-mapping_0-prefix_t1_axi4lite-slave-AXI4-Lite_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_4-mapping_0-prefix_t5_ahblite_H-slave-AHBLite_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_5-mapping_0-prefix_i6_ahblite_H-master-AHB_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_6-mapping_0-prefix_i8_apb_P-master-APB4_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_7-mapping_0-prefix_t7_apb_P-slave-APB4_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_8-mapping_0-prefix_i12_dpram-master-DPRAM_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_9-mapping_0-prefix_t11_dpram-slave-DPRAM_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_10-mapping_0-prefix_i10_spram-master-SPRAM_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_11-mapping_0-prefix_t9_spram-slave-SPRAM_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_12-mapping_0-prefix_i14_stream_T-master-AXI4Stream_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_13-mapping_0-prefix_t13_stream_T-slave-AXI4Stream_rtl"},
        {"$ref": "#/definitions/busDefinitions/busint-portgroup_14-mapping_0-prefix_sys-slave-JTAG_rtl"}
    ],   
    "addressSpaces": []
}
```

Each entry in `obj["component"]["busInterfaces"]` is a JSON reference ($ref) to
a dictionary defining a single bus interface for a select portgroup.  The
naming of these references follows the format:

```
busint-portgroup_{portgroup index}-mapping_{mapping index}-prefix_{portgroup prefix}-{master,slave}-{bus abstract name}
```

Ports within portgroup\_0 share the prefix `t3_axi4-slave` and we can see from
the reference name alone that the correct bus definition was inferred for this
portgroup.  We can further examine the mapping of physical ports within the
portgroup to the bus definition logical names:

```json
"busint-portgroup_0-mapping_0-prefix_t3_axi4-slave-AXI4_rtl": {
    "name": "t3_axi4",
    "interfaceMode": "slave",
    "busType": {
        "vendor": "amba.com",
        "library": "AMBA4",
        "name": "AXI4",
        "version": "r0p0_0"
    },
    "abstractionTypes": [
        {
            "viewRef": "RTLview",
            "portMaps": {
                "ARLOCK": "t3_axi4_ARLOCK",
                "BREADY": "t3_axi4_BREADY",
                "WVALID": "t3_axi4_WVALID",
                ...
                "AWREGION": "t3_axi4_AWREGION",
                "AWPROT": "t3_axi4_AWPROT",
                "AWCACHE": "t3_axi4_AWCACHE_0",
                "ARUSER": [
                    "t3_axi4_ARAPCMD"
                ],
                "AWUSER": [
                    "t3_axi4_AWAPCMD",
                    "t3_axi4_AWALLSTRB",
                    "t3_axi4_AWCOBUF"
                ]
            }
        }
    ]
}
```

Notice that four user signals `t3_axi4_ARAPCMD`, `t3_axi4_AWAPCMD`,
`t3_axi4_AWALLSTRB`, and `t3_axi4_AWCOBUF` for this AXI4 bus interface are
correctly mapped to user port groups `"ARUSER"` and `"AWUSER"`.

Ports within portgroup\_14 share the prefix `sys`, and they are unexpectedly
mapped to the JTAG bus definition.  Examining the mapping we can see that this
portgroup does not map well to JTAG:

```json
"busint-portgroup_14-mapping_0-prefix_sys-slave-JTAG_rtl": {
    "name": "sys",
    "interfaceMode": "slave",
    "busType": {
        "vendor": "sifive.com",
        "library": "TEST",
        "name": "JTAG",
        "version": "0.1.0"
    },
    "abstractionTypes": [
        {
            "viewRef": "RTLview",
            "portMaps": {
                "TDI": [
                    "sys_int0",
                    "sys_int1",
                    "sys_int2",
                    "sys_int3",
                    "sys_int4",
                    "sys_int5",
                    "sys_int6",
                    "sys_int7"
                ],
                "TCK": "sys_clock_enable",
                "TDO": "sys_power_status",
                "__UMAP__": [
                    "sys_power_enable"
                ]
            }
        }
    ]
}
```

Furthermore, mappings to alternate bus definitions
(`obj["component"]["busInterfaceAlts"]`) for this portgroup are not
correct either:

```json
"busInterfaceAlts": [
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_0-mapping_1-prefix_t3_axi4-slave-ACE-Lite_rtl"},
    ...
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_14-mapping_1-prefix_sys-slave-AXI4Stream_rtl"},
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_14-mapping_2-prefix_sys-master-AXI4Stream_rtl"},
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_14-mapping_3-prefix_sys-slave-ATB_rtl"},
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_14-mapping_4-prefix_sys-master-ATB_rtl"}
]
```

We can create a user-edited copy of `ark-busprop.json` in which we remove the
reference to the chosen JTAG mapping for portgroup\_14:

```console
% cp ark-busprop.json ark-busprop.edit.json
# remove incorrect bus mapping for portgroup_14
# and validate the edited file
% duh validate ark-busprop.edit.json
```

#### Group remaining ports into bundles

Next run `duh-portbundler` to structure the remaining ports that were not
previously assigned to a bus interface into structured bundles.

```console
% duh-portbundler -o ark-final.json ark-busprop.edit.json
```

The resulting `ark-final.json` file contains the previously inferred bus
interfaces and an additional two structured bundles under
`obj["component"]["busInterfaces"]`:

```json
"busInterfaces": [
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_0-mapping_0-prefix_t3_axi4-slave-AXI4_rtl"},
    ...
    {"$ref": "#/definitions/busDefinitions/busint-portgroup_13-mapping_0-prefix_t13_stream_T-slave-AXI4Stream_rtl"},
    {"$ref": "#/definitions/bundleDefinitions/bundle-sys"},
    {"$ref": "#/definitions/bundleDefinitions/bundle-root"}
]
```

Ports with the shared prefix `sys` are grouped together in a single structured
bundle:

```json
"bundle-sys": {
    "name": "sys",
    "interfaceMode": null,
    "busType": "bundle",
    "abstractionTypes": [
        {
            "viewRef": "RTLview",
            "portMaps": {
                "int": [
                    "sys_int0",
                    "sys_int1",
                    "sys_int2",
                    "sys_int3",
                    "sys_int4",
                    "sys_int5",
                    "sys_int6",
                    "sys_int7"
                ],
                "power": {
                    "enable": "sys_power_enable",
                    "status": "sys_power_status"
                },
                "clock_enable": "sys_clock_enable"
            }
        }
    ]
}
```

`sys_intX` signals are also grouped together into a single vector.

The remaining `clock` and `reset_n` ports are grouped together in a separate
"root" bundle:

```json
"bundle-root": {
    "name": "root",
    "interfaceMode": null,
    "busType": "bundle",
    "abstractionTypes": [
        {
            "viewRef": "RTLview",
            "portMaps": {
                "clock": "clock",
                "reset_n": "reset_n"
            }
        }
    ]
}
```
