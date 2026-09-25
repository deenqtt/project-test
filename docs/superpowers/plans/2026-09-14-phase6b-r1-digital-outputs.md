# Phase 6B-R1 Digital Outputs Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Capture and verify four startup-safe 12–24 V protected low-side outputs.

**Architecture:** A 5 V SN74AHCT125 translates four ESP32 controls to four ZXMS6005N8Q inputs. Shared active-low OE defaults disabled through a 5 V pull-up and is enabled by an NPN controlled from SPARE_GPIO1; downstream pull-downs guarantee OFF while outputs are high impedance.

**Tech Stack:** KiCad 9 schematic hierarchy, configured KiCad MCP, official TI/Diodes Incorporated/Phoenix Contact documentation.

## Global Constraints

- Modify KiCad data only through KiCad MCP.
- Do not alter Ethernet, RS485, digital-input, PCB, or frozen DO GPIO circuits.
- Keep `DO_FIELD_GND` common with `LOGIC_GND`, and isolated from `DI_FIELD_GND` and `RS485_GND`.
- Stop after Phase 6B.

---

### Task 1: Pre-capture checkpoint and library verification

**Files:**
- Inspect: `hardware/kicad_mcp_test.kicad_sch`
- Create through MCP: `hardware/project_symbols.kicad_sym`

**Interfaces:**
- Consumes: Existing root hierarchy and frozen global nets.
- Produces: Verified symbols and footprints for capture.

- [ ] Snapshot the project and confirm ERC is 0 errors / 12 warnings.
- [ ] Verify stock SN74AHCT125, connector, transistor, diode symbols and exact footprints.
- [ ] Create/register a project-local ZXMS6005N8Q symbol with pins 1–3 Source, 4 IN, 5–8 Drain.
- [ ] Validate the custom symbol library.

### Task 2: Capture the digital-output sheet

**Files:**
- Create through MCP: `hardware/digital_outputs.kicad_sch`
- Modify through MCP: `hardware/kicad_mcp_test.kicad_sch`

**Interfaces:**
- Consumes: `5V_MAIN`, `LOGIC_GND`, `SPARE_GPIO1`, and `DO1_CTRL`–`DO4_CTRL`.
- Produces: `DO_FIELD_V+`, `DO1_OUT`–`DO4_OUT`, and `DO_FIELD_GND`.

- [ ] Add the `DIGITAL_OUTPUTS` hierarchical sheet.
- [ ] Place the AHCT buffer, 100 nF decoupling, OE network, four 10 kΩ downstream pull-downs, four ZXMS devices, four optional flyback diodes, connector, and test points.
- [ ] Connect all nets with `DO_FIELD_GND` explicitly joined to `LOGIC_GND`.
- [ ] Add sourcing metadata and engineering/layout notes.

### Task 3: Validate connectivity and isolation

**Files:**
- Validate: `hardware/digital_outputs.kicad_sch`
- Validate: `hardware/kicad_mcp_test.kicad_sch`
- Update: `phase6b.md`

**Interfaces:**
- Consumes: Completed Phase 6B schematic.
- Produces: ERC, geometry, netlist, and domain-audit evidence.

- [ ] Run structural validation and fresh ERC.
- [ ] Generate the netlist and verify every control, buffer, switch, output, field-supply, and ground connection.
- [ ] Run orphan-wire, floating-label, off-grid, overlap, and wire-crossing checks.
- [ ] Prove `DO_FIELD_GND == LOGIC_GND`, `DO_FIELD_GND != DI_FIELD_GND`, and `DO_FIELD_GND != RS485_GND`.
- [ ] Save a completion snapshot and record the final A–M report.
