# tlm_cmd_merger
This tool reads telemetry and commands from a yaml file generated by airliner. It then writes this data to sqlite database.

# Table of Contents
1. [Dependencies](#dependencies)
2. [How to install](#how_to_install)
3. [Schemas](#schemas)


## Dependencies <a name="dependencies"></a>
`python>=3.5.2`
`PyYAML>=5.3.1`


## How to install <a name="how_to_install"></a>
This is meant to be a development package, so it is intended to be used inside a virtual environment managed by a tool like `venv `

1. Clone the repo 
```
git clone https://github.com/WindhoverLabs/cmd_msg_merger/tree/master
```
2. Start  a virtual environment by running 
```
python3 -m venv venv
``` 
3. Activate the virtual environment 
```
source venv/bin/activate
```
4. Install it
```
pip install -r requiremets.txt
```

5. Run it
```
python3 .src/tlm_cmd_merger.py [PATH_TO_YAML] [PATH_TO_SQLITE] 
```

**NOTE**: Beware that for now this is an internal tool written specifically for [airliner's](https://github.com/WindhoverLabs/airliner)  ground sysytem toolchain. The sqlite database is assumed to have been generated by [juicer](https://github.com/WindhoverLabs/juicer). The yaml file is assumed to have been generated by airliner. So this tool should *not* be executed by itself. As our toolchain evolves and matures we may add capability to run it in isolation, but for now you must run juicer before running this tool.


# Schemas <a name="schemas"></a>

## YAML Schema
The yaml file is expected to have a schema that looks like this:

```
modules:
  AK8963:
    short_name: ak8963
    long_name: TBD
    events:
      AK8963_INIT_INF_EID:
        id: 1
        type: INFORMATION
      AK8963_CMD_NOOP_EID:
        id: 2
        type: INFORMATION
    telemetry:
      AK8963_HK_TLM_MID:
        msgID: 0x0cc1
        struct: AK8963_HkTlm_t
      AK8963_DIAG_TLM_MID:
        msgID: 0x0cc5
        struct: AK8963_DiagPacket_t
    commands:
      AK8963_CMD_MID:
        msgID: 0x1cc4
        commands:
          Noop:
            cc: 0
            struct: CFE_SB_CmdHdr_t
          Reset:
            cc: 1
            struct: CFE_SB_CmdHdr_t
          SendDiag:
            cc: 2
            struct: CFE_SB_CmdHdr_t
          SetCalibration:
            cc: 3
            struct: AK8963_SetCalibrationCmd_t
    perfids:
      AK8963_RECEIVE_PERF_ID:
        id: 102
      AK8963_SEND_PERF_ID:
        id: 103
      AK8963_MAIN_TASK_PERF_ID:
        id: 104
    config:
      AK8963_SB_TIMEOUT:
        name: AK8963_SB_TIMEOUT
        value: CFE_SB_PEND_FOREVER
      AK8963_MISSION_REV:
        name: AK8963_MISSION_REV
        value: 0
    definition: "../apps/ak8963"
```
It is _highly_ recommended that all the entries in the yaml file be filled out. At the moment we do allow
partial records in the database, however, for easier usage(and to avoid things breaking down the road), we recommend
to have all entries filled out in the yaml file to avoid partial records in the database.

## SQLITE Schema
This tool adds all necessary tables to the database(which is generated by juicer). After running `tlm_cmd_merger`
the sqlite database will have the following tables as part of its schema:


"*" = PRIMARY KEY 
"+" = FOREIGN KEY

### bit_fields
|field* | bit_size   | bit_offset |
|---|---|---|
| INTEGER | INTEGER | INTEGER |


### commands
|id* | name   | command_code | message_id | macro | symbol+ | module+ |
|---|---|---|---|---|---|---|
| INTEGER | TEXT | INTEGER | INTEGER |TEXT | INTEGER | INTEGER |

### configurations
|id* | name   | value | macro | module+ |
|---|---|---|---|---|
| INTEGER | TEXT | INTEGER | TEXT | INTEGER

### elfs
| id* | name | module+ | checksum | date | little_endian |
| --- | --- | --- | --- | --- | --- |
|INTEGER | TEXT | INTEGER | TEXT | DATETIME | BOOLEAN |

###  enumerations
| symbol* | value* | name |
| --- | --- | --- |
| INTEGER | INTEGER | TEXT |

### events
| id* | event_id | macro |
| --- | --- | --- |
| INTEGER | INTEGER | TEXT

### fields
| id* | name | symbol+ | byte_offset | type+ | multiplicity | little_endian
| --- | --- | --- | ---| --- | --- | --- |
| INTEGER | TEXT | INTEGER |INTEGER | INTEGER | INTEGER | BOOLEAN |

### modules
| id* | name |
| --- | --- |
| INTEGER | TEXT |

### perf_ids
| id* | name | perf_id | macro | module+ |
| --- | --- | --- | ---| --- |
| INTEGER | TEXT | INTEGER | TEXT | INTEGER |

### symbols
| id* | elf+ | name | byte_size |
| ---| --- |---| --- |
| INTEGER| INTEGER | INTEGER | TEXT | INTEGER|
 
### telemetry
| id* | name | message_id | macro | symbol+ | module+ |
| --- | --- | ---| ---| ---| ---|
| INTEGER | TEXT | INTEGER | TEXT | INTEGER | INTEGER|

**NOTE**: The column `type` in the `fields` table is pointing to another entry in the `symbols` table.

The tables **telemetry**, **perf_ids**, **events**, **configurations**,  **commands** and **modules** are generated by `cmd_tlm_merger` so they do *not* have to exist prior to running the tool. The rest of the tables *must* exist prior to running this tool. These tables should be generated by juicer, which follows the schema described above.

Documentation updated on September 23, 2020.
