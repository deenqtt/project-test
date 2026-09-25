# PCB-03A2 — DI ISOLATION LOCAL CORRECTION REPORT

Scope executed: exactly the PCB-03A1-approved T2 correction — one new copper-free rule area `DI_ISO_GAP_EXT` + zone refill — on `hardware/kicad_mcp_test.kicad_pcb`. Nothing else touched. Writes via KiCad MCP only (`add_rule_area`, `open_board`, `refill_zones`), one fresh server instance, one writer.

## 1. Pre-write baseline
KiCad GUI 0, MCP writer processes 0, `.lck` 0. `.kicad_pcb` `1ce1972c6f0b6185dc2cb79277f678ecb5157a4cf7e46fa8f7644536f27b17f9`; 17 project-file SHAs identical to the PCB-03A/03A1 records. 136 fp / 1054 tracks / 192 vias; fresh kicad-cli DRC 12 silk_edge only, electrical 0, unconnected 0; netlist 430/430.

## 2. Persistent backup
`hardware/kicad_mcp_test-backups/kicad_mcp_test.PRE-03A2.20260922-162617.kicad_pcb` (`cp -p`, `cmp` byte-identical, SHA `1ce1972c…`; identical to the PRE-C6C-derived C6C result) plus a copy of `.kicad_pro` (`…PRE-03A2….kicad_pro.bak`) for restore context. Earlier backups (PRE-C6, PRE-C6A, PRE-C6C) untouched.

## 3. Disposable reproduction (`/tmp/pcb03a2/cand/REPRO`, fresh copy of the current live)
Sequence: `add_rule_area` (writes the file) → `open_board` (reload) → `refill_zones` (saves). An extra explicit `save_board` was refused by the server's stale-file guard ("contents changed externally") — harmless, because `refill_zones` already saved; the live plan therefore omits `save_board`. Reopened from disk: rule area present with exact geometry/semantics, LOGIC_GND fill 4592.4 → 4584.7 mm², DI minimum 4.250 mm, RS485 6.000 mm, DRC 12 silk / electrical 0, unconnected 0, netlist 430/430, tracks/vias/footprints/pads/drawings/texts identical, plane model trapped set 17 identical, lifelines unchanged — identical to the PCB-03A1 T2 result. → live authorised.

## 4. Rule-area semantics (verified on the disposable copy and again on live after reopen)
Name `DI_ISO_GAP_EXT`; layers F.Cu + B.Cu; polygon (70.0,55.6) (74.0,55.6) (74.0,57.5) (70.0,57.5) mm — exactly the PCB-03A1 T2 geometry; prohibitions: copper pour = **true**, tracks = false, vias = false, footprints = false, pads = false (so the existing LOGIC_GND via (72.825,55.2) and nearby tracks are unaffected and no global routing constraint is introduced). Existing rule areas DI_ISO_GAP and RS485_ISO_GAP unchanged (polygons and flags compared).

## 5. T2 implementation (live)
Gate before write: GUI 0, MCP 0, lck 0, SHA `1ce1972c…`. Ops: add_rule_area (uuid assigned, changed=true) → open_board → refill_zones (5 zones, saved). Result SHA `35a86ad07cf60335912c68746bbf8a2061c7b5df18de864f93c5d0d416d57d35`. Persistence verified by reopening the file with pcbnew (rule area present, fill updated).

## 6. DI spacing before / after (edge-to-edge, same layer, package-internal pads excluded)
Before: 3.818 mm — DI3_SENSE via (68.5,61.3) B.Cu ring ↔ LOGIC_GND B.Cu fill in the pocket x 70–74 / y 55.2–57.5. After: **4.250 mm** — DI_FIELD_GND B.Cu row (47.6,59.6)/(51.2,59.6) ↔ LOGIC_GND fill north of DI_ISO_GAP (y ≤ 54.9); the old pair is now ≥ 5.6 mm. Target ≥ 4.000 mm: PASS. (Regulatory status of the isolation remains NOT ESTABLISHED — design inputs pending.)

## 7. Zone-fill effect
LOGIC_GND B.Cu: 4592.4 → 4584.7 mm² (−7.7 mm²), still 2 outlines (main + F-bridged U3 interior 230.3); pocket copper 8.35 → 0.75 mm² (a 0.4 mm strip y 55.2–55.6 left to keep the via (72.825,55.2) clear). RS485_GND fills 212.1 (F) / 229.9 (B) unchanged. Live vs disposable zone geometry: symmetric-difference area 0.0 mm² (physically identical). Antenna keep-out region (x ≥ 93.95): fill area 0.0 unchanged.

## 8. RS485 regression
Isolated ↔ logic minimum 6.000 mm (RS485_GND fill ↔ LOGIC_GND fill), track/pad minimum 6.3 mm, keepout copper-clean: unchanged.

## 9. Ground-plane regression
Raster model on live: 31 components (30 before; the +1 is the removed pocket becoming a separate 0.75 mm² strip), main 4929.4 mm² (4937.2 before, −7.8), item-free islands [391.8, 230.3, 23.6, 19.5, 8.1] unchanged; trapped set 17 identical (12 U3.41 thermal pads + interior vias, all F-bridged); lifelines K5 (25.2,48.41), Q1.2 (36.55,47.5), R11/R10 (43.2,48.4)/(43.2,50.0), R31 (43.4,45.175), USB shield J1.S1, U4.J1.1/J2.1, (72.7,22.0), (78.3,27.0) all main. No new bottleneck (the pocket was a dead-end pocket between two keepouts).

## 10. DRC / connectivity (fresh, live)
kicad-cli severity-all: electrical 0 (clearance/shorting/items_not_allowed/copper_edge/hole/drill/track_width all 0); silk_edge_clearance 12 — item set identical to the pre-write DRC; unconnected 0; pcbnew unconnected 0; netlist 425/425 + 5 board-only = 430/430; GND unconnected 0.

## 11. Protected-structure regression
K5 via (25.2,48.41)/tie/lane, old via absent, NE + N bridges, BOOT-A (72.2,50.525), C6C D1 vias (62.25,60.95)/(60.5,63.8)/(63.4,61.3), C6C G1 vias (38.3,46.85)/(76.6,36.1)/(37.35,46.85): present; default-off chain pad nets (U3.5→R30→Q1 base/R31, Q1.2 GND, Q1.3 = OE_N = R29.2 = U8 OE) unchanged; antenna keep-out untouched.

## 12. Item-count regression
Tracks 1054 = 1054 (0 differing items), vias 192 = 192 (0), footprints 136; pads (nets/positions/sizes), board-outline/silk drawings, reference/value texts identical; zone definitions: existing identical, +1 rule area.

## 13. Project-file diff
`sha_before` vs `sha_after`: exactly one file changed — `hardware/kicad_mcp_test.kicad_pcb` (`1ce1972c…` → `35a86ad07cf60335…`). `.kicad_pro`, `.kicad_dru`, 9 schematics, 3 symbol libs, lib tables unchanged. Inside the PCB: +1 rule area, refilled zone polygons; 0 track/via/footprint/pad/outline/net/silk changes. End state: 0 `.lck`, 0 GUI, 0 MCP processes.

## 14. Live final renders (`/tmp/pcb03a2/final/`)
01 full board F+B; 02a DI region BEFORE (from the PRE-03A2 backup) / 02b AFTER (live); 03 DI_ISO_GAP_EXT boundary with existing keepouts; 04 critical spacing after; 05 final F.Cu zones; 06 final B.Cu zones; 07 RS485 isolation unchanged; 08 ground-plane continuity. Report copy: `hardware/kicad_mcp_test-backups/PCB-03A2_report.md`.

## 15. Final gate

PCB-03A2: PASS
T2 DI_ISO_GAP_EXT: IMPLEMENTED
DI MINIMUM: 4.250 mm
DI TARGET >= 4.000 mm: PASS
RS485 ISOLATION: PRESERVED
UNCONNECTED: 0
NETLIST: 430/430
ELECTRICAL DRC: 0
GND UNCONNECTED: 0
TRACKS: 1054 UNCHANGED
VIAS: 192 UNCHANGED
FOOTPRINTS: 136 UNCHANGED
GROUND PLANE: PRESERVED
C5L PROTECTED COPPER: UNCHANGED
C6C D1/G1: UNCHANGED
ANTENNA KEEPOUT: PRESERVED
PROJECT MUTATION: .kicad_pcb ONLY
PERSISTENT BACKUP: VERIFIED
NEXT PROPOSED STAGE: PCB-03B MANUFACTURABILITY / ASSEMBLY AUDIT
WAITING FOR HUMAN REVIEW

## 16. Recommended next stage
PCB-03B manufacturability/assembly audit against the PCB-03A1 §7 fabrication inventory (6/8 mil, 0.2 mm drill, 0.15 annular); R3 (DO source vias) stays conditional on the DO load specification; silk cleanup (J8.1 "IO6", TP3/TP16 values) in a later bounded documentation stage. Rollback: `cp -p kicad_mcp_test-backups/kicad_mcp_test.PRE-03A2.20260922-162617.kicad_pcb hardware/kicad_mcp_test.kicad_pcb` → SHA `1ce1972c…`.
