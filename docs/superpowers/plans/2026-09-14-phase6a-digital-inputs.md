# Phase 6A Isolated Digital Inputs Implementation Plan

**Scope:** Add only the four-channel ISO1212 isolated 12–24 V digital-input subsystem.

1. Confirm official ISO1212, transient-protection, connector, symbol, and footprint data.
2. Snapshot the existing KiCad project and preserve the measured ERC baseline.
3. Create `hardware/digital_inputs_isolated.kicad_sch` and link it as `DIGITAL_INPUTS_ISOLATED` using KiCad MCP.
4. Capture two ISO1212 channels per IC with four protected field inputs, logic-side decoupling, frozen logic net names, and separated ground domains.
5. Validate structure, ERC, connectivity, geometry, references, netlist, and galvanic isolation.
6. Save a post-capture snapshot and document Phase 6A findings in `phase6.md`.

No PCB work and no changes to existing Ethernet, RS485, or later subsystems are included.
