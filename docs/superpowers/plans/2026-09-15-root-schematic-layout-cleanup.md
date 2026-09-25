# Root Schematic Layout Cleanup Implementation Plan

> **For agentic workers:** Execute this plan through KiCad MCP only for schematic changes.

**Goal:** Improve readability of the root `kicad_mcp_test.kicad_sch` overview without changing symbols, values, nets, wiring, hierarchy names, or electrical connectivity.

**Architecture:** Treat the root sheet as a system block diagram. Keep circuit details in child sheets; arrange root hierarchical sheets in a balanced functional grid, with readable non-overlapping sheet labels and notes. Validate before and after with ERC, flattened netlist, sheet list, and component/reference counts.

**Tech Stack:** KiCad schematic, KiCad MCP, KiCad CLI validation, Markdown report.

## Global Constraints

- KiCad schematic modifications only through KiCad MCP.
- Do not modify components, values, footprints, references, hierarchy names/files, nets, or wires.
- Do not modify child circuit sheets.
- Stop if ERC errors/warnings, flattened netlist, sheet count, or component/reference counts change unexpectedly.

### Task 1: Establish root baseline

- [ ] Read root sheet properties, sheet boxes, texts, labels, and wires through MCP.
- [ ] Run root ERC and generate flattened netlist.
- [ ] Record component count, unique references, sheet count, and critical net signatures.

### Task 2: Apply root-only layout cleanup

- [ ] Move only hierarchical sheet blocks and free-form documentation text using MCP.
- [ ] Use coordinates snapped to the existing schematic grid.
- [ ] Preserve every sheet name, sheet file, sheet pin, label, component, wire, and net.
- [ ] Do not edit child sheets or any circuit object.

### Task 3: Validate and report

- [ ] Re-run root structural validation, ERC, flattened netlist, hierarchy inspection, and count/reference audits.
- [ ] Confirm the only differences are root layout coordinates and intended root text positions.
- [ ] Write the research summary and before/after validation to `phase10.md`.

