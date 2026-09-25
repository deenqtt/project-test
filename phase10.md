# Phase 10 — Root Schematic Readability Review

## A. Research: practical KiCad schematic-layout rules

- [VERIFIED FROM KICAD DOCUMENTATION] Use hierarchical sheets to keep the root page as a high-level overview and place circuit detail inside child sheets. KiCad states that hierarchical design improves schematic legibility.
- [VERIFIED FROM KICAD DOCUMENTATION] Use hierarchical labels and sheet pins only for parent/child interfaces; use global labels for intentional cross-sheet nets. Do not move an electrical anchor merely to improve text appearance.
- [VERIFIED FROM KICAD DOCUMENTATION] KiCad recommends the standard 50 mil grid for symbol placement and wiring because the standard symbol library follows that grid.
- [ENGINEERING PRACTICE] Arrange functional blocks left-to-right or top-to-bottom, keep similar blocks aligned, maintain consistent whitespace, and reserve the title block area.
- [ENGINEERING PRACTICE] Keep long notes outside circuit geometry, use one clear title per functional section, and avoid sheet rectangles extending beyond the drawing page.

Sources: [KiCad Schematic Editor 9.0 documentation](https://docs.kicad.org/9.0/en/eeschema/eeschema.html), [KiCad hierarchical schematics documentation](https://github.com/KiCad/kicad-doc/blob/master/src/eeschema/eeschema_hierarchical_schematics.adoc), [KiCad 8 grid guidance](https://docs.kicad.org/8.0/ca/eeschema/eeschema.html).

## B. Root-sheet audit

- [VERIFIED FROM KICAD] Root file: `hardware/kicad_mcp_test.kicad_sch`.
- [VERIFIED FROM KICAD] Root contains the Phase 3A/3B power overview plus eight functional sheet blocks:
  `DIGITAL_INPUTS_ISOLATED`, `DIGITAL_OUTPUTS`, `ETHERNET_WIZ850IO`, `ESP32_CORE`, `RS485_ISOLATED`, `SENSORS_SERVICE`, `USB_C`, and `AUX_5V_OUTPUT`.
- [VERIFIED FROM KICAD] Current root layout has `SENSORS_SERVICE` starting at approximately `x=296.325 mm`, outside the practical A4 drawing area.
- [VERIFIED FROM KICAD] `RS485_ISOLATED` and `AUX_5V_OUTPUT` occupy the lower-right area close to the title block. This makes the overview visually crowded even though the electrical connectivity is valid.
- [VERIFIED FROM KICAD] Sheet block widths/heights are inconsistent, and several long `Sheetfile` strings visually extend into nearby space.

## C. Cleanup attempt and safety result

- [VERIFIED FROM KICAD MCP] A one-sheet test was attempted on `SENSORS_SERVICE`, which has no sheet pins or wires attached at the root.
- [VERIFIED FROM KICAD MCP] The available MCP interface has add/remove hierarchical-sheet operations, but no direct “move sheet while preserving UUID and instance mapping” operation.
- [VERIFIED FROM KICAD MCP] Removing and re-adding the sheet generated a new UUID and caused unrelated Phase 9 child references to appear as `?` in the flattened netlist.
- [SAFETY DECISION] The movement was rolled back. No multi-sheet cleanup was attempted.
- [VERIFIED FROM KICAD] The root schematic and `aux_5v.kicad_sch` were restored from the verified Phase 9 snapshots. No component, value, footprint, net, wire, hierarchy name, or connector pinout was intentionally changed.

## D. Final verification after rollback

- [VERIFIED FROM KICAD] Root structural validation: valid; KiCad CLI PDF export exit 0.
- [VERIFIED FROM KICAD] AUX child structural validation: valid; KiCad CLI PDF export exit 0.
- [VERIFIED FROM KICAD] Fresh root ERC: **0 errors / 4 known warnings**.
- [VERIFIED FROM KICAD] The four warnings are unchanged legacy warnings: three RS485 off-grid sheet-pin warnings and one intentional `DO_FIELD_GND` / `LOGIC_GND` alias warning.
- [VERIFIED FROM KICAD] Flattened netlist: **130 components / 105 nets**.
- [VERIFIED FROM KICAD] Component and net signatures are identical to the Phase 9 verified netlist; no component or net differences were found.
- [VERIFIED FROM KICAD] Phase 9 references remain persistent and unique, including `U13`, `R47`, `C30`, `C31`, and `J10`.

## E. Result

**NO FINAL LAYOUT CHANGE RETAINED.**

The requested visual cleanup cannot be performed safely with the currently exposed KiCad MCP operations. The root is back at its verified electrical baseline. The main remaining visual problems are sheet-block coordinates and title-block crowding, which require a true sheet-move operation that preserves each existing sheet UUID, sheet pins, wires, and `sheet_instances` mapping.

## F. Recommended next action

Use KiCad GUI to drag only the root sheet rectangles on the 50 mil grid, preserving their existing UUIDs and pins, then rerun ERC and netlist validation. A safe MCP continuation requires adding a server operation equivalent to “move hierarchical sheet by UUID,” not remove/re-add.

No PCB work or circuit redesign was started.

