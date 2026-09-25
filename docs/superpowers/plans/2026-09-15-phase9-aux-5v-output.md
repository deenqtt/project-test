# Phase 9 AUX 5V Output Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add the approved current-limited `5V_MAIN` to `AUX_5V` service output without changing existing circuits.

**Architecture:** A new `AUX_5V_OUTPUT` hierarchical sheet contains TPS2553DBVR with 105 kΩ ILIM programming, deterministic always-on enable, local 1 µF input/output capacitors, and a two-pin Phoenix Contact 1715721 output. The parent exposes only `5V_MAIN` and `LOGIC_GND`; `AUX_5V` remains local to the child sheet.

**Tech Stack:** KiCad schematic hierarchy, KiCad MCP, `kicad-cli` validation through MCP.

## Global Constraints

- KiCad modifications use KiCad MCP only.
- Preserve the pre-capture baseline: 0 ERC errors, 4 known warnings, 125 components.
- Do not modify existing sheets or begin PCB work.
- Keep `AUX_5V` isolated from `USB_VBUS`, `VIN_FIELD`, `3V3_LOGIC`, `DI_FIELD_GND`, and `RS485_GND`.
- Use globally unique references and verify flattened annotation.

---

### Task 1: Capture the approved AUX output

**Files:**
- Create: `hardware/aux_5v.kicad_sch`
- Modify: `hardware/kicad_mcp_test.kicad_sch`
- Modify: `hardware/project_symbols.kicad_sym`

**Interfaces:**
- Consumes: `5V_MAIN`, `LOGIC_GND`
- Produces: local `AUX_5V` at J10 pin 1; J10 pin 2 returns to `LOGIC_GND`

- [ ] Snapshot the verified pre-capture project.
- [ ] Create and register the exact TPS2553DBVR project symbol with TI pinout.
- [ ] Create `AUX_5V_OUTPUT` and add parent sheet pins for `5V_MAIN` and `LOGIC_GND`.
- [ ] Place U13, R47, C30, C31, and J10 with approved values and footprints.
- [ ] Wire `5V_MAIN → U13 IN/EN`, `U13 OUT → AUX_5V → J10.1`, `U13 ILIM → R47 → LOGIC_GND`, capacitors to `LOGIC_GND`, and J10.2 to `LOGIC_GND`.
- [ ] Mark U13 FAULT explicitly no-connect and add the approved output/backfeed notes.

### Task 2: Validate and report Phase 9

**Files:**
- Create: `phase9.md`

**Interfaces:**
- Consumes: completed root hierarchy and flattened netlist
- Produces: Phase 9 evidence report

- [ ] Validate child and parent schematic structure.
- [ ] Run fresh root ERC and require 0 errors with only the four known warnings.
- [ ] Generate flattened netlist and require 130 components with no duplicate references.
- [ ] Audit approved path, forbidden nets, EN, FAULT, RILIM, capacitors, connector pinout, USB divider, and unchanged isolation domains.
- [ ] Reload/re-read the hierarchy and repeat annotation/netlist checks.
- [ ] Write `phase9.md` with sections A–Q and final `CONDITIONAL GO` verdict.

