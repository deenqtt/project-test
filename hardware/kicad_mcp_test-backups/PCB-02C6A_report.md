# PCB-02C6A — FOUR ADDITIVE CONNECTIONS IMPLEMENTATION REPORT

Project `hardware/kicad_mcp_test.kicad_pcb` (KiCad 9.0.9). Write stage; geometry source = the validated PCB-02C6 manifest (`/tmp/c6/e/c6a.py`, persistent copy `hardware/kicad_mcp_test-backups/PCB-02C6A_manifest_c6a.py`). All PCB writes through KiCad MCP (`route_trace`, `add_via`, `refill_zones`; fresh server instance per batch, one writer). No text edits, no server changes. DI4_SENSE and SPARE_GPIO1 not touched.

## 1. Preflight
GUI processes 0, `.lck` 0; live SHA `2226d0c333d513f57f18effbb9a015a3ea746f1a00f7d89d933ec515458890dc`; 17 project-file SHAs recorded (`/tmp/c6a/sha_before.txt`) and identical line-by-line to the C6 baseline/end record.

## 2. Persistent backup verification
PRE-C6 `kicad_mcp_test-backups/kicad_mcp_test.PRE-C6.20260921-171851.kicad_pcb`: byte-identical to live at start (cmp), SHA `2226d0c3…`, untouched. New PRE-C6A `kicad_mcp_test-backups/kicad_mcp_test.PRE-C6A.20260921-230535.kicad_pcb`: `cp -p`, cmp byte-identical to live, SHA `2226d0c3…`. Both verified intact again at stage end (identical to each other). `/tmp` used only for disposable copies, plans, DRC JSONs, renders.

## 3. Baseline SHA / counts
`2226d0c3…`; 136 fp / 989 tracks / 183 vias; DRC 12 silk_edge only, 0 electrical; unconnected 6; netlist 425/425 (+5 board-only Q2/R48) = 430/430.

## 4. Implementation batches B1–B5
Disposable reproduction first (`/tmp/c6a/cand/REPRO`, fresh copy of live): B1–B5 clean, unconnected 6→5→4→3→2, electrical 0 each; final copper = manifest exactly (40/40 items), +35 tracks/+5 vias, footprints/pads/zones/drawings/ref-texts identical, plane model identical to the approved C6 model. Live plans verified op-identical to reproduction plans (only `boardPath` differs). Abort-on-fail gate (procs 0, lck 0, SHA == previous) run before every live batch.

| batch | net | MCP ops (open+ops+refill) | electrical DRC | other | unconnected | SHA after |
|---|---|---|---|---|---|---|
| baseline | — | — | 0 | silk_edge 12 | 6 | `2226d0c333d513f5` |
| B1 | USB_VBUS | 8/8 ok | 0 | silk_edge 12 | 5 | `e7719bea146dde72` |
| B2 | TP12 / 3V3_LOGIC | 6/6 ok | 0 | silk_edge 12 | 4 | `09d51007823a8f99` |
| B3 | UART0_RX | 17/17 ok | 0 | silk_edge 12 | 3 | `654f8c5da5db7c8b` |
| B4 | SPARE_GPIO2 | 17/17 ok | 0 | silk_edge 12 | 2 | `cf1699628b84f707` |
| B5 | final refill | 2/2 ok | 0 | silk_edge 12 | 2 | `cf1699628b84f707` (stable) |

Incident: the shell running B5 was killed by the host (exit 137) after the MCP refill had saved (file mtime 23:15, SHA unchanged at `cf1699…`); an orphan MCP server process was terminated (`kill <pid>`), gate re-run (procs 0, lck 0, SHA `cf1699…`), and B5 re-executed cleanly: SHA unchanged → fill deterministic. Runner note: first reproduction attempt aborted at `open_board` (arg name `boardPath`); nothing was written (SHA unchanged), plan fixed, reproduction restarted from the pristine copy.

## 5. Exact tracks / vias added per net (all vias 0.6/0.3)

### USB_VBUS — net `/USB_C/USB_VBUS`
01. route_trace F.Cu w0.4: (74.1,7.355) -> (74.1,8.7)
02. route_trace F.Cu w0.4: (74.1,8.7) -> (73.95,8.85)
03. route_trace B.Cu w0.4: (73.95,8.85) -> (75.8,10.7)
04. route_trace B.Cu w0.4: (75.8,10.7) -> (76.1,11.0)
05. route_trace B.Cu w0.4: (76.1,11.0) -> (76.5,11.0)
06. add_via 0.6/0.3 at (73.95,8.85)

### TP12 — net `3V3_LOGIC`
07. route_trace F.Cu w0.5: (72.0,9.7) -> (71.75,9.95)
08. route_trace B.Cu w0.5: (71.75,9.95) -> (71.66,10.04)
09. route_trace B.Cu w0.5: (71.66,10.04) -> (68.32,10.04)
10. add_via 0.6/0.3 at (71.75,9.95)

### UART0_RX — net `/ESP32_CORE/UART0_RX`
11. route_trace F.Cu w0.25: (87.35,48.75) -> (87.35,47.75)
12. route_trace F.Cu w0.25: (87.35,47.75) -> (87.0,47.4)
13. route_trace F.Cu w0.25: (87.0,47.4) -> (86.35,47.4)
14. route_trace F.Cu w0.25: (86.35,47.4) -> (86.1,47.15)
15. route_trace F.Cu w0.25: (86.1,47.15) -> (86.1,46.2)
16. route_trace F.Cu w0.25: (86.1,46.2) -> (86.5,45.8)
17. route_trace F.Cu w0.25: (86.5,45.8) -> (88.8,45.8)
18. route_trace F.Cu w0.25: (88.8,45.8) -> (89.4,46.4)
19. route_trace F.Cu w0.25: (89.4,46.4) -> (90.9,46.4)
20. route_trace F.Cu w0.25: (90.9,46.4) -> (91.3,46.8)
21. route_trace B.Cu w0.25: (91.3,46.8) -> (91.3,50.75)
22. route_trace F.Cu w0.25: (91.3,50.75) -> (91.05,51.0)
23. route_trace F.Cu w0.25: (91.05,51.0) -> (90.3,51.0)
24. add_via 0.6/0.3 at (91.3,46.8)
25. add_via 0.6/0.3 at (91.3,50.75)

### SPARE_GPIO2 — net `SPARE_GPIO2`
26. route_trace B.Cu w0.25: (19.3,3.5) -> (19.3,7.5)
27. route_trace B.Cu w0.25: (19.3,7.5) -> (46.0,7.5)
28. route_trace B.Cu w0.25: (46.0,7.5) -> (46.0,16.4)
29. route_trace B.Cu w0.25: (46.0,16.4) -> (59.1,16.4)
30. route_trace B.Cu w0.25: (59.1,16.4) -> (75.3,32.6)
31. route_trace B.Cu w0.25: (75.3,32.6) -> (78.3,32.6)
32. route_trace F.Cu w0.25: (78.3,32.6) -> (79.4,33.7)
33. route_trace F.Cu w0.25: (79.4,33.7) -> (81.7,33.7)
34. route_trace F.Cu w0.25: (81.7,33.7) -> (82.1,34.1)
35. route_trace F.Cu w0.25: (82.1,34.1) -> (85.0,34.1)
36. route_trace F.Cu w0.25: (85.0,34.1) -> (85.8,33.3)
37. route_trace F.Cu w0.25: (85.8,33.3) -> (85.8,32.6)
38. route_trace F.Cu w0.25: (85.8,32.6) -> (86.11,32.3)
39. route_trace F.Cu w0.25: (86.11,32.3) -> (86.11,31.25)
40. add_via 0.6/0.3 at (78.3,32.6)

Total ops: 40


Totals: 35 tracks + 5 vias = 40 items; live now 136 fp / 1024 tracks / 188 vias. Item-by-item comparison live vs PRE-C6A backup: removed 0 tracks / 0 vias; added 35 / 5; all 40 manifest items found with exact coordinates, layer, width, net.

## 6. UART0_RX result
U3.36 and TP10.1 in one cluster (KiCad connectivity; not in unconnected list). Path as planned: F loop north of the DO1/DO2 F hooks under the module, via (91.3,46.8), B.Cu drop x 91.3 between the 3V3 B track (x 90.2) and the LOGIC_GND via (93.26,48.75), via (91.3,50.75), F into TP10. UART0_TX copper (3 F tracks U3.37→TP11) unchanged (item-identical to backup). Length 14.3 mm, 2 vias.

## 7. USB_VBUS result + safety audit
J1.A9/B4 pad now tied: F (74.1,7.355)→(74.1,8.7)→(73.95,8.85) w0.4, via, B to the existing corner (76.5,11.0). Net /USB_C/USB_VBUS pads after: J1.A4, J1.A9, J1.B4, J1.B9, R45.1, D1.5 — one cluster, 6/6. Foreign nets touching any USB_VBUS copper: NONE. 5V_MAIN, 3V3_LOGIC, VIN_FUSED, AUX_5V not members. Net remains sense-only (D1.5 clamp, R45 39k → Q2 gate); no power-feed rail created; no powered-off back-feed path (no on-board source on this net). Correct J1 VBUS members confirmed by pad numbers A9/B4 (footprint pad at (74.1,7.355)).

## 8. TP12 result
TP12.1 → F (72.0,9.7)→(71.75,9.95) w0.5, via, B (71.75,9.95)→(71.66,10.04)→(68.32,10.04) into the PTH U4.J2.2 (3V3). 3V3_LOGIC 33/33 pads one cluster; the main rail copper (81 tracks/13 vias before) unchanged — branch stub only.

## 9. SPARE_GPIO2 result
J8.1 → B.Cu (19.3,3.5)→(19.3,7.5)→(46.0,7.5)→(46.0,16.4)→(59.1,16.4)→(75.3,32.6)→(78.3,32.6) (south of the I2C/VBUS_DET_N B bus, through U4 header gap J1.4/J1.5, parallel to ETH_CS_N on its NE side), via (78.3,32.6), F under the module into U3.6 from the south. 89.2 mm, 1 via. J8.1 and U3.6 one cluster. The rejected top-corridor candidate was NOT used. Plane model (0.05 mm raster, real live copper): identical to the approved C6 model — main + F-bridged U3 interior; trapped set = 17 (12 U3.41 thermal pads + 5 interior vias, as in C5L); J1.S1 (USB shield), U4.J1.1, U4.J2.1, vias (72.7,22.0), (74.4,11.05), (72.5,10.875), (71.8,15.375) all in main; (36.55,47.5) and (43.2,48.4) in main; K5 via (25.2,48.41) in main. Silk/schematic untouched (IO2 label vs IO6 pin noted in C6).

## 10. Clearance / margin audit (real live geometry, analytic per-netclass incl. .dru exceptions)
| route | min margin | FAIL | CRIT | TIGHT | ACC | ROB |
|---|---|---|---|---|---|---|
| USB_VBUS | +0.100 | 0 | 0 | 0 | 3 | 13 |
| TP12 | +0.175 | 0 | 0 | 0 | 0 | 7 |
| UART0_RX | +0.064 | 0 | 0 | 0 | 3 | 20 |
| SPARE_GPIO2 | +0.075 | 0 | 0 | 0 | 5 | 24 |

Identical to the C6 approved audit (min +0.064 UART0_RX via ↔ DO1_CTRL F hook; USB_VBUS stub ↔ J1 GND/CC2 pads 0.30; SPARE_GPIO2 F ↔ RXD/TXD/SCL/SDA vias 0.275, via ↔ ETH_MISO 0.275; TP12 ≥ 0.425). No netclass change, no .dru exception. kicad-cli clearance/hole/edge/drill: 0.

## 11. Ground-plane audit
kicad-cli: GND in unconnected list 0. LOGIC_GND B fill (KiCad): 2 outlines = 4674.7 mm² main + 230.3 mm² U3 interior (F-bridged, unchanged); before C6A 4762.2 + 230.3 — the 87.5 mm² loss is the new B tracks' clearance corridors (SPARE_GPIO2, TP12, USB_VBUS); no third outline → no island holding GND items (island removal would have kept it). Model: no new trapped/unfilled GND item; U3 NE/N bridges and interior unchanged; K5 via in main; USB shield J1.S1 and U4 GND pins in main.

## 12. Isolation audit
DI_ISO_GAP / RS485_ISO_GAP rule areas: 0 hits by any copper of the four nets (new or pre-existing). No route within the isolated regions; no GND-domain joins.

## 13. Antenna audit
Analytic keepout x ≥ 93.45, y 30.5–49.5: 0 hits; max x of new copper 91.6 (UART0_RX via (91.3,50.75), outside the module y span).

## 14. Protected C5L copper audit
Removed 0 items. K5 via (25.2,48.41) LOGIC_GND 0.6/0.3 ✓; K5 F tie (25.75,47.95)→(25.2,48.41) w0.4 ✓; CTRL1 B lane y 47.775 ✓; old via (25.04,47.95) absent ✓; NE bridge via (86.2,24.6) + F w0.5 ✓; N bridge vias (78.3,27.0)/(81.1,27.0) + F w0.5 ✓; BOOT-A via (72.2,50.525) ✓; all pre-existing 989 tracks/183 vias item-identical; footprints/pads/zones/drawings/reference texts identical. Closest new copper to C5L copper: UART0_RX ↔ DO1_CTRL hook 0.264 mm (req 0.20).

## 15. DRC (kicad-cli 9.0.9, severity-all, `/tmp/c6a/drc/live_B5.json`)
clearance 0, shorting_items 0, items_not_allowed 0, copper_edge_clearance 0, hole_clearance 0, drill 0, tracks_crossing 0 → electrical 0; silk_edge_clearance 12 (inherited, unchanged).

## 16. Connectivity
Unconnected 6 → 2. Per-net clusters: USB_VBUS 6 pads/1, UART0_RX 2/1, 3V3_LOGIC 33/1, SPARE_GPIO2 2/1. Netlist 425/425 (+5) = 430/430. CTRL/DI 8/8 still connected.

## 17. Remaining two airwires (expected)
DI4_SENSE: U7.11 ↔ F stub at R28.1 (61.4,64.22). SPARE_GPIO1: R30.1 ↔ U3.5. Both REQUIRES_LOCAL_REWORK per C6; untouched here.

## 18. Project mutation / SHA audit
`diff sha_before sha_after`: exactly one file changed — `hardware/kicad_mcp_test.kicad_pcb` → `cf1699628b84f7072bcb485d38b69af4345601a9748963b2ab6efc529387e7ae`. `.kicad_pro`, `.kicad_dru`, 9 schematics, 3 symbol libs, lib tables unchanged. `(setup …)` block of the board textually identical to the backup (a pcbnew-SWIG "design settings differ" reading appears only because the backup copy sits in a folder without a `.kicad_pro`; not a mutation). Additions: PRE-C6A backup, this report and renders. 0 `.lck`, 0 GUI / MCP processes at end.

## 19. Renders (`/tmp/c6a/final/`, all from the LIVE board, labelled "LIVE FINAL (PCB-02C6A)")
01 full board F+B; 02 UART0_RX; 03 USB_VBUS; 04 TP12 stub; 05 SPARE_GPIO2 full; 06 SPARE_GPIO2 module end; 07 SPARE_GPIO2 B.Cu plane corridor; 08 isolation overview; 09 antenna keepout; 10 remaining two airwires; 11 K5/CTRL1 intact; 12 LOGIC_GND B plane. Persistent copy of the report: `hardware/kicad_mcp_test-backups/PCB-02C6A_report.md`.

## 20. Rollback instructions
`cp -p hardware/kicad_mcp_test-backups/kicad_mcp_test.PRE-C6A.20260921-230535.kicad_pcb hardware/kicad_mcp_test.kicad_pcb` (with GUI 0, lck 0), then `sha256sum` must read `2226d0c333d513f57f18effbb9a015a3ea746f1a00f7d89d933ec515458890dc`; kicad-cli DRC must show unconnected 6 / electrical 0. PRE-C6 backup is an identical alternative.

## 21. Final gate

PCB-02C6A: PASS
ADDITIVE ROUTES: 4/4 IMPLEMENTED
UART0_RX: CONNECTED
USB_VBUS: CONNECTED — SENSE ONLY
TP12 / 3V3: CONNECTED
SPARE_GPIO2: CONNECTED
UNCONNECTED: 2
REMAINING: DI4_SENSE, SPARE_GPIO1
ELECTRICAL DRC: 0
GND UNCONNECTED: 0
GROUND PLANE: PRESERVED
C5L PROTECTED COPPER: UNCHANGED
ISOLATION: PRESERVED
ANTENNA KEEPOUT: PRESERVED
PROJECT MUTATION: .kicad_pcb ONLY
PERSISTENT BACKUP: VERIFIED
NEXT DECISION STAGE: PCB-02C6B
WAITING FOR HUMAN REVIEW
