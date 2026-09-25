# Phase 8 USB-C Native USB Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a protected self-powered USB-C native USB service interface to the ESP32-S3 without creating a USB-to-board power path.

**Architecture:** A GCT USB4105-GF-A UFP connector feeds USBLC6-2SC6 ESD protection, then 22 Ω series resistors and frozen GPIO19/20 USB nets. USB VBUS is used only by the ESD reference and a 91 kΩ/130 kΩ GPIO7 sensing divider.

**Tech Stack:** KiCad 9, configured KiCad MCP, KiCad CLI validation/ERC/netlist.

## Global Constraints

- Use KiCad MCP for every KiCad inspection and modification.
- Never directly edit .kicad_sch, .kicad_pcb, or .kicad_pro.
- GPIO19/20 remain USB D−/D+; GPIO7 becomes USB_VBUS_SENSE; GPIO6 remains spare.
- USB_VBUS must not power or connect to any internal power rail.
- Stop after Phase 8; no PCB work.

---

### Task 1: Baseline and checkpoint

**Files:**
- Inspect: `hardware/kicad_mcp_test.kicad_sch`
- Snapshot: `hardware/snapshots/`

- [ ] Run root hierarchy, ERC, ESP32 label, and netlist inspections through KiCad MCP.
- [ ] Confirm baseline 0 errors / 7 warnings and GPIO19/20 USB mapping.
- [ ] Create a pre-Phase-8 MCP project snapshot.

### Task 2: Update approved GPIO7 service assignment

**Files:**
- Modify: `hardware/esp32_core.kicad_sch`
- Modify: `hardware/sensors_service.kicad_sch`
- Modify: `phase4.md`

- [ ] Replace `SPARE_GPIO3` labels with `USB_VBUS_SENSE` using KiCad MCP.
- [ ] Verify U3 GPIO7 and J8 former spare pin share the new global net.
- [ ] Update the Markdown GPIO map while preserving GPIO6 as `SPARE_GPIO2`.

### Task 3: Create USB-C hierarchy and circuit

**Files:**
- Create: `hardware/usb_c.kicad_sch`
- Modify: `hardware/kicad_mcp_test.kicad_sch`

- [ ] Create child schematic and root sheet `USB_C` via KiCad MCP.
- [ ] Place GCT USB4105-GF-A connector with verified stock symbol/footprint.
- [ ] Place two 5.1 kΩ 1% CC resistors to LOGIC_GND.
- [ ] Place USBLC6-2SC6 using verified pinout and SOT23-6 footprint.
- [ ] Place two 22 Ω 1% series resistors between ESD and MCU nets.
- [ ] Place 91 kΩ/130 kΩ 1% VBUS divider to GPIO7.
- [ ] Join shield directly to LOGIC_GND and no-connect SBU pins.
- [ ] Add concise VBUS-isolation and later-layout notes.

### Task 4: Validate and report

**Files:**
- Create: `phase8.md`

- [ ] Validate root and child schematic structure with KiCad MCP/KiCad CLI.
- [ ] Run fresh ERC without suppressions and record exact before/after.
- [ ] Generate netlist and prove USB_VBUS is absent from 5V_MAIN, 3V3_LOGIC, VIN_FIELD, and AUX_5V.
- [ ] Verify GPIO19/20 polarity, GPIO7 sense, CC resistors, ESD paths, ground/shield, references, wires, and geometry.
- [ ] Create final MCP snapshot.
- [ ] Write the requested A–N report and stop.
