# Phase 6B-R1 Digital Output Interface Design

## Scope

Complete only the four-channel non-isolated 12–24 V low-side output schematic. Preserve Ethernet, RS485, isolated digital inputs, PCB files, and frozen DO GPIO assignments.

## Logic architecture

`DO1_CTRL`–`DO4_CTRL` feed the four A inputs of an `SN74AHCT125PWR` powered from `5V_MAIN`. Its Y outputs drive four `ZXMS6005N8Q-13` inputs. Each ZXMS input has 10 kΩ to `LOGIC_GND`, ensuring OFF whenever the AHCT output is high impedance.

All four active-low OE pins share `DO_BUF_OE_N`. A 10 kΩ pull-up to `5V_MAIN` disables the buffer by default. `MMBT3904-7-F`, driven from existing `SPARE_GPIO1` through 10 kΩ with 47 kΩ base pull-down, pulls OE low only after firmware deliberately drives GPIO5 HIGH. This consumes the previously reserved spare without changing DO1–DO4 GPIO mapping.

## Field architecture

Each ZXMS is a protected low-side switch: Sources connect to `DO_FIELD_GND`, Drains connect to `DOx_OUT`. `DO_FIELD_GND` is deliberately the same DC domain as `LOGIC_GND/PWR_GND`; it remains separate from `DI_FIELD_GND` and `RS485_GND`.

The six-position Phoenix Contact 1706303 connector exposes `DO_FIELD_V+`, four outputs, and `DO_FIELD_GND`. `DO_FIELD_V+` is external only. Four optional `B360Q-13-F` flyback diodes connect anode to `DOx_OUT`, cathode to `DO_FIELD_V+`.

## Verification criteria

The sheet must validate in KiCad, introduce no ERC errors, resolve the four staged DO control warnings, preserve all isolated grounds, contain no orphan/floating/off-grid/overlapping items, and show complete buffer, switch, flyback, and connector paths in the generated netlist.
