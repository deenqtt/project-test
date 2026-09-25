# PCB-02C6B — FINAL TWO AIRWIRES LOCAL-REWORK STUDY REPORT

Read-only study on `hardware/kicad_mcp_test.kicad_pcb` (post-C6A, SHA `cf1699628b84f7072bcb485d38b69af4345601a9748963b2ab6efc529387e7ae`). All experiments on disposable copies under `/tmp/c6b/cand/{D1,G1,BOTH}` (fresh copies of live + `.kicad_pro/.kicad_dru`), applied through KiCad MCP (`delete_trace` by UUID, `route_trace`, `add_via`, `refill_zones`), audited with kicad-cli DRC, pcbnew read-only inspection, analytic clearance checker (per-netclass incl. the ISO1212 .dru package rules), a grid maze-router as an existence oracle, and a 0.05 mm raster model of the B.Cu LOGIC_GND fill. Live never opened for writing. Artefacts: `/tmp/c6b/e/cands.py` (geometry-as-code manifests), `e/manifest.md`, `e/margins_both.json`, `drc/*.json`, `final/*.png`; persistent copies of report + manifest in `hardware/kicad_mcp_test-backups/`.

## 1. Live integrity / preflight
Start and end: GUI 0, `.lck` 0, no MCP processes; live SHA `cf1699628b84f707…` unchanged; 17 project files identical to the C6A end record; 136 fp / 1024 tracks / 188 vias; kicad-cli DRC 12 silk_edge only, unconnected 2 (DI4_SENSE, SPARE_GPIO1); PRE-C6 and PRE-C6A backups intact. Live never modified.

## 2. DI4_SENSE electrical topology
Isolated-input channel 4 field side (ISO1212 U7): field terminal → R24 (330 R pulse-proof) → node DI4_SENSE = {R24.2, R28.1 (562 R → DI4_IN → U7.10 IN2), C28.1 (10 nF → DI_FIELD_GND), D7.2 (TVS → DI_FIELD_GND)} → U7.11 SENSE2. Everything except the U7.11 hop is routed (4 pads, one cluster). Netclass DI_FIELD (0.4 mm clearance, 0.3 mm width) for all field nets; .dru: DI_FIELD tracks inside the U7 courtyard may be 0.25 mm from U7 field pads 9–16 and from other DI_FIELD tracks in the courtyard (tracks only — vias are not covered). Field side = y ≥ 59.0 (pad row) / south of DI_ISO_GAP (keepout y 55.2–58.8). Both endpoints on the field side; no barrier involvement.

## 3. DI4_SENSE geometric obstruction (recalculated on post-C6A geometry)
U7.11 (61.85–62.25 × 59.0–60.2) sits between U7.10 DI4_IN (61.21–61.61) and U7.12 NC (62.48–62.88). Its only exit is south, into a pocket closed by: **W** DI4_IN F track x 61.413 w0.3 (y 59.6→61.9, right edge 61.563) + R28.2 (60.92–61.88, 61.77–62.58); **E** DI_FIELD_GND F stub U7.14→(63.45,61.0)→(63.35,61.2)→via (63.2,61.2)→(63.2,62.225)→C27.2 (62.73–63.67); **S** the R28.2/C27.2 gap 0.85 mm (needs 1.1 mm for a 0.3 track, 0.95 mm for 0.15). Analytically: below the courtyard (y > 60.445) a DI4_SENSE track centre needs x ≥ 62.113 (DI4_IN) **and** ≥ 0.55 mm from the U7.12 corner (62.48,60.2) → no solution for y 60.445–60.61; a via near the pad needs x ≥ 62.263 (DI4_IN), ≥ 0.7 mm from the U7.12 corner (→ y ≥ 60.87), ≥ 1.0 mm from GND via (63.2,61.2) (→ y ≤ 60.92 at x 62.263… empty at x 62.3) and its B side ≥ 0.4 from the DI_FIELD_GND B row y 60.3 (→ y ≥ 61.15) → empty set. Even if a via existed, its B side is enclosed by the DI_FIELD_GND B row (y 60.3, x 51.9→63.2), GND vias (59.6,61.2)/(63.2,61.2) and the DI3_SENSE B row y 62.2 (x 60.5→68.0) + vertical (60.5, 62.2→64.0). Router: NO PATH at w 0.3 and 0.15; NO PATH with any single one of {DI3_SENSE row moved, GND row+via moved, DI4_IN narrowed}; NO PATH with GND+DI3 only. The U6/DI2 counterpart works only because U6's GND B row jogs to y 59.6 under pads 9–14 and DI1_SENSE/DI2_SENSE rows do not overlap in x.

## 4. DI4 candidate set
| id | description | result |
|---|---|---|
| D0 | additive only (C6) | NO PATH (proven again) |
| D-a | move DI3_SENSE B row only | NO PATH (F exit still impossible) |
| D-b | jog DI_FIELD_GND B row + move GND via only | NO PATH (F exit still impossible) |
| D-c | narrow DI4_IN F stub to 0.15 only | NO PATH (B pocket closed) |
| D-d | D-a + D-b (no DI4_IN change) | NO PATH (F exit window needs DI4_IN at 0.15) |
| **D1** | D-a + D-b + D-c, DI4_SENSE: 0.15 F stub → via (62.25,60.95) → B → via (60.5,63.8) on the existing DI4_SENSE F track between C28.1/R28.1 | **VALID** (disposable copy: DI4 airwire gone, DRC electrical 0, no new airwire) |
Variants ruled out: moving DI4_IN west (U7.9 GND stub blocks, 0.03–0.17 mm short), DI4_IN on B (no legal via near U7.10), exit east under U7.12/13 (U7.14 GND stub), any .dru/netclass change, footprint moves.

## 5. DI4 candidate comparison (only D1 is valid; factual data)
Removed: 8 tracks + 1 via (DI4_IN 1 F; DI_FIELD_GND 2 B + 1 via; DI3_SENSE 5 B). Added: 17 tracks + 3 vias (DI4_IN 1 F w0.15; DI_FIELD_GND 1 F + 5 B + 1 via; DI3_SENSE 4 B; DI4_SENSE 3 F w0.15 + 3 B w0.3 + 2 vias). Copper area re-laid ≈ 12 mm² of B track, all inside x 51.9–68.5 / y 59.6–66.3 (field side). Nets touched: DI4_IN, DI_FIELD_GND, DI3_SENSE (+ target). Min margin (real geometry, combined board): DI4_SENSE +0.062 (via ↔ DI4_IN 0.15 stub 0.462/0.40), U7.12 corner +0.085; DI_FIELD_GND +0.05 (inherited: replaced row ↔ DI1_SENSE via (56.5,61.2) 0.45 — identical to live); DI3_SENSE +0.35. Protected C5L copper: untouched. Ground geometry: LOGIC_GND unaffected (field-side DI_FIELD_GND only, track-routed, 12/12 pads connected). Isolation-adjacent: GND row jog at y 59.7 is 0.75 mm south of the DI_ISO_GAP edge (58.8); no copper enters the keepout. Route length added 9.3 mm (DI4_SENSE), 2 layer transitions.

## 6. Preferred DI4 candidate: D1
Exact geometry in §20. Note: 0.15 mm width used for two short field-side stubs (DI4_IN 2.3 mm, DI4_SENSE 1.1 mm) — board min track 0.15 mm, sense/input currents are sub-mA; DI_FIELD 0.3 mm is the class default, not a minimum. The DI3_SENSE B row at y 65.0 passes through the existing DI3_SENSE via (64.1,65.0) (own net) and keeps R23.2 ↔ U7.16 ↔ C27.1/R27.1/D7.1 in one cluster (verified).

## 7. SPARE_GPIO1 functional topology (schematic + PCB)
U3.5 IO5 (ESP32-S3) → net SPARE_GPIO1 → R30.1; R30 10 k → R30.2 = DO_BUF_EN_BASE = Q1.1 (MMBT3904 base) with R31 47 k base→LOGIC_GND; Q1.2 emitter → LOGIC_GND (F track → via (36.55,47.5), sole ground path); Q1.3 collector = DO_BUF_OE_N = U8 (SN74AHCT125) pins 1/4/10/13 (OE, active-low) with R29 10 k → 5V_MAIN pull-up. Schematic text "5 V AHCT BUFFER — HARDWARE DEFAULT OFF".

## 8. Hardware-default-off verification
At reset/boot IO5 is high-Z/low → R31 holds Q1 off → R29 pulls OE high → all four buffers disabled → DO outputs off until firmware drives IO5 high. G1 changes no component, pad net, pull resistor, or transistor connection; Q1.2's ground via and its F link are untouched; Q1.3's OE_N link is re-laid but still a plain copper connection to the same net; netlist 425/425 on the disposable result. Behaviour preserved.

## 9. SPARE_GPIO1 geometric obstruction
R30.1 (38.52–39.48 × 46.42–47.23) F pocket: N R30.2 + DO_BUF_EN_BASE F row y 45.05/45.175 → (41.3,45.975→46.825) → R31.1; E DO4_CTRL via (40.975,47.45); S DO3_CTRL F row y 48.4; W OE_N via (37.6,47.4). A via under the pad lands in the B pocket (x 37.9–42.08, y 45.5–48.1) fenced by DO4_CTRL B row y 45.35 + (40.975,45.825→47.45) (N/E), OE_N B (42.2,44.4→47.9) (E), OE_N B row y 48.2 (S), OE_N via/stub (37.6,47.4…) + LOGIC_GND via (36.55,47.5) (W). Its only exit (x 34.2–37.3) traps via (36.55,47.5) = Q1 emitter ground; and every eastward path south of the pocket runs through the corridor y 48.3–50.7 which is the sole plane lifeline of R11.2/R10.2 grounds (vias (43.2,48.4)/(43.2,50.0)). Router: NO PATH with that exit forbidden; NO PATH F-only. The lane x 42.6–44.1 (north of the pocket) is also the sole plane neck of the mid-board B region (x 42–75, y 15–42, ≈600 mm²) → a track cannot occupy it lengthwise.

## 10. GPIO1 candidate set
| id | description | result |
|---|---|---|
| G0 | additive (C6) | NO PATH / traps (36.55,47.5) |
| G-B | move Q1.2 ground via to (35.0,47.75) | opens west exit but the eastward corridor still cuts the R11/R10 ground lifeline → rejected |
| G-T | move OE_N Q1.3 via to (40.3,46.4) / east side | OE_N still needs the south loop (DO4_CTRL vertical x 40.975 blocks any row between y 45.7 and 48.075) → no gain |
| G-a | remove OE_N south loop only | Q1.3 has no alternative link: every B path to OE_N row y 44.72 crosses the DO4_CTRL feed row y 45.35 (router "path" only because the test ignored that crossing) → invalid alone |
| G-b | re-lay DO4_CTRL feed only | NO PATH (OE_N wall remains) |
| G1-v1 | G-a + G-b, GPIO1 through the lane + band (C6 path) | plane model: mid-board region islanded (558–590 mm²) → rejected |
| G1-v2 | as v1 but crossing above GND via (43.4,45.175) | lane neck still cut → rejected |
| **G1** | G-a + G-b; GPIO1: pocket → lane west side (x 42.7/42.9) → north along the 3V3 B wall (x 42.05) → east under the C6A GPIO2 row (y 17.0) → down the U4 J1 column (x 46.85) → east under the ETH_INT_N row (y 21.35) → parallel to the ETH_INT_N diagonal → (76.6,36.1) via → F under the module to U3.5 | **VALID** (disposable: airwire gone, DRC 0, plane intact) |
Also ruled out: F hop across the neck (ONEWIRE F row y 41.0 and CTRL2 F row y 41.6 block), stitching via for the mid region (no F GND copper there; and no stitching campaigns), moving R30/Q1/R11 (footprint moves), moving DO3/DO4/CTRL rows (C5L).

## 11. GPIO1 candidate comparison (valid = G1 only)
Removed: 10 tracks + 1 via (DO4_CTRL 3 B; OE_N 2 F + 5 B + 1 via). Added: 31 tracks + 3 vias (DO4_CTRL 5 B; OE_N 2 F + 1 B + 1 via; SPARE_GPIO1 1 F + 21 B + 3 F + 2 vias, 95.2 mm). Nets touched: DO4_CTRL (U8-side feed only; the C5L CTRL4 F lane, via (44.3,44.8), (40.975,47.45) via and F link to R11.1 untouched), DO_BUF_OE_N (Q1.3 link only). Min margins (real geometry): DO4_CTRL +0.005/+0.025 — both **inherited** (the re-laid row starts on the same y 45.35 line as today: 0.205 to OE_N via (34.02,44.72) and 0.225 to DO4_DRIVE via (34.02,46.0), identical to the live row); OE_N +0.15; SPARE_GPIO1 +0.05 (OE_N (42.2,42.75→44.4) 0.25; 3V3 B wall 0.325/0.25; U4.J1.5/J1.6 0.267; ETH_CS_N diag 0.264; ETH_INT_N via 0.279). Protected C5L: untouched. Ground geometry: B fill −82 mm² of clearance corridors; plane model on real copper: trapped set identical to live (17 = U3 thermal pads + 5 interior vias, all F-bridged), lifelines (36.55,47.5), (43.2,48.4), (43.2,50.0), (43.4,45.175), K5 (25.2,48.41), USB shield J1.S1, U4.J1.1/J2.1 all in main; largest new item-free island 19.5 mm² (NE strip pocket x 71–79, y 36.7–39; no GND items; removed by KiCad's island rule). 2 layer transitions. Length is long (95 mm) because R30 sits at x 39 and U3.5 at x 87 and the only plane-safe corridor is the region boundary; a shorter lane/band path exists but islands the mid-board plane.

## 12. Preferred GPIO1 candidate: G1
Exact geometry in §20. Note the C6A SPARE_GPIO2 route is untouched; GPIO1's F entry (row y 36.1 → x 87.38 into U3.5 from the south) clears GPIO2's F copper by ≥ 1.4 mm.

## 13. Combined disposable test (`/tmp/c6b/cand/BOTH`)
Fresh copy of live → D1 + G1 in one MCP session (76 ops: 20 deletions, 54 additions, refill): all ops succeeded; SHA `7848ecf255971ece…`. Item diff vs live: removed 18 tracks + 2 vias, added 48 tracks + 6 vias = exactly the manifest; footprints, pad nets, zones/rule areas identical; 136 fp / 1054 tracks / 192 vias.

## 14. Combined DRC
kicad-cli severity-all: clearance 0, shorting 0, items_not_allowed 0, copper_edge 0, hole_clearance 0, drill 0, tracks_crossing 0 → electrical 0; silk_edge_clearance 12 (inherited). Track-width minimum (0.15) satisfied.

## 15. Combined connectivity
**unconnected = 0** (kicad-cli and pcbnew connectivity). All touched nets single-cluster: DI4_SENSE 5/5, DI4_IN 2/2, DI_FIELD_GND 12/12, DI3_SENSE 5/5, DO4_CTRL 3/3, DO_BUF_OE_N 6/6, SPARE_GPIO1 2/2; netlist 425/425 (+5 board-only) = 430/430. CTRL/DI logic-side 8/8 untouched.

## 16. Combined GND-plane audit
KiCad fill: LOGIC_GND B = 2 outlines (4592.6 mm² main + 230.3 mm² F-bridged U3 interior; live 4674.7 + 230.3). No third outline → no island holding a GND item. Raster model on the real combined copper: trapped set identical to live (17), no unfilled item, lifelines above all main. GND unconnected 0.

## 17. Isolation audit
Rule areas DI_ISO_GAP / RS485_ISO_GAP: 0 copper hits by any track/via of the touched nets. D1 copper is entirely on the field side (min y 59.6 for F, 59.55 for the B jog vs keepout edge 58.8). No field-side net north of y 55.2; no logic net inside the DI field region. No joins between DI_FIELD_GND / RS485_GND and LOGIC_GND. The ISO1212 .dru package exceptions were used only for the DI4_SENSE stub inside the U7 courtyard vs U7 field pads/DI4_IN, as designed.

## 18. Antenna audit
Analytic keepout x ≥ 93.45, y 30.5–49.5: 0 hits; max x of new copper 87.5 (GPIO1 vertical x 87.38 into U3.5).

## 19. Protected C5L audit
K5 via (25.2,48.41) 0.6/0.3, K5 F tie w0.4, CTRL1 B lane y 47.775, old via absent, NE/N bridges + vias, BOOT-A via (72.2,50.525): all present on every disposable copy; none of the 20 removed items is C5L copper (they are the Phase-2A DI field GND/DI3 rows and the Phase-2 DO-block U8-side feeds). C5L CTRL4 F lane, via (44.3,44.8), DO3/DO4 B rows: untouched.

## 20. Exact proposed future mutation manifest (PCB-02C6C; MCP only; batches: C1 = D1 removals+adds, C2 = G1 removals+adds, C3 = refill; DRC/connectivity/plane gate after each)

### D1 — remove
01. delete_trace F.Cu (61.413,59.6)->(61.413,61.9) net `/DIGITAL_INPUTS_ISOLATED/DI4_IN` — DI4_IN F stub replaced by 0.15 mm width (same geometry) — creates the only legal U7.11 exit window (0.4 mm class clearance to DI4_IN)
02. delete_trace via (63.2,61.2) net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND` — DI_FIELD_GND tie via moved 0.22 mm SE so a DI4_SENSE via can sit 1.0 mm away (via-to-via 0.4 class clearance)
03. delete_trace B.Cu (63.2,61.2)->(63.2,60.3) net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND` — B stub of the moved via (replaced by new via stub)
04. delete_trace B.Cu (51.9,60.3)->(63.2,60.3) net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND` — DI_FIELD_GND B row replaced by the same row with a U6-style jog to y 59.7 under U7 pads 11–13 (clears the DI4_SENSE via)
05. delete_trace B.Cu (68.0,62.2)->(68.5,61.7) net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE` — DI3_SENSE B row (y 62.2) sealed the DI4 pocket; row moved to y 65.0 (through existing via (64.1,65.0))
06. delete_trace B.Cu (60.5,62.2)->(68.0,62.2) net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE` — DI3_SENSE B row y 62.2 (see above)
07. delete_trace B.Cu (60.5,64.0)->(60.5,62.2) net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE` — DI3_SENSE B vertical x 60.5 (part of the old row path)
08. delete_trace B.Cu (59.2,65.3)->(60.5,64.0) net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE` — DI3_SENSE B diagonal to (60.5,64.0) (part of the old row path)
09. delete_trace B.Cu (64.1,62.2)->(64.1,65.0) net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE` — DI3_SENSE B branch (64.1,62.2)->(64.1,65.0): its via now sits on the new y 65.0 row

### D1 — add
10. route_trace F.Cu w0.15 net `/DIGITAL_INPUTS_ISOLATED/DI4_IN`: (61.413,59.6) -> (61.413,61.9)
    (net /DIGITAL_INPUTS_ISOLATED/DI4_IN: 2.3 mm added, 0 via(s))
11. route_trace F.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND`: (63.35,61.2) -> (63.4,61.3)
12. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND`: (63.4,61.3) -> (63.4,60.1)
13. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND`: (63.4,60.1) -> (63.0,59.7)
14. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND`: (63.0,59.7) -> (61.8,59.7)
15. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND`: (61.8,59.7) -> (61.2,60.3)
16. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND`: (61.2,60.3) -> (51.9,60.3)
17. add_via 0.6/0.3 net `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND` at (63.4,61.3)
    (net /DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND: 13.2 mm added, 1 via(s))
18. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE`: (68.5,61.7) -> (68.5,64.3)
19. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE`: (68.5,64.3) -> (67.8,65.0)
20. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE`: (67.8,65.0) -> (59.5,65.0)
21. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI3_SENSE`: (59.5,65.0) -> (59.2,65.3)
    (net /DIGITAL_INPUTS_ISOLATED/DI3_SENSE: 12.3 mm added, 0 via(s))
22. route_trace F.Cu w0.15 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE`: (62.047,59.6) -> (62.047,60.445)
23. route_trace F.Cu w0.15 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE`: (62.047,60.445) -> (62.25,60.75)
24. route_trace F.Cu w0.15 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE`: (62.25,60.75) -> (62.25,60.95)
25. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE`: (62.25,60.95) -> (62.25,63.0)
26. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE`: (62.25,63.0) -> (61.45,63.8)
27. route_trace B.Cu w0.3 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE`: (61.45,63.8) -> (60.5,63.8)
28. add_via 0.6/0.3 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE` at (62.25,60.95)
29. add_via 0.6/0.3 net `/DIGITAL_INPUTS_ISOLATED/DI4_SENSE` at (60.5,63.8)
    (net /DIGITAL_INPUTS_ISOLATED/DI4_SENSE: 5.5 mm added, 2 via(s))

### G1 — remove
30. delete_trace B.Cu (33.259999,45.35)->(40.5,45.35) net `DO4_CTRL` — DO4_CTRL diagonal (part of the old feed)
31. delete_trace B.Cu (40.5,45.35)->(40.975,45.825) net `DO4_CTRL` — DO4_CTRL diagonal (part of the old feed)
32. delete_trace B.Cu (40.975,45.825)->(40.975,47.45) net `DO4_CTRL` — DO4_CTRL vertical to via (40.975,47.45) (part of the old feed; via and F link to R11.1 are kept)
33. delete_trace via (37.6,47.4) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — DO_BUF_OE_N Q1.3 via moved to (37.35,46.85); its B south loop removed (loop fenced R30.1 on E and S)
34. delete_trace F.Cu (37.138,46.0)->(37.138,46.9) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N F stub Q1.3 -> old via (replaced by stub to new via)
35. delete_trace F.Cu (37.138,46.9)->(37.6,47.4) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N F stub (replaced)
36. delete_trace B.Cu (37.6,47.9)->(37.6,47.4) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N B south loop (removed)
37. delete_trace B.Cu (37.9,48.2)->(37.6,47.9) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N B south loop (removed)
38. delete_trace B.Cu (41.9,48.2)->(37.9,48.2) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N B south row y 48.2 (removed)
39. delete_trace B.Cu (42.2,47.9)->(41.9,48.2) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N B south loop corner (removed)
40. delete_trace B.Cu (42.2,44.4)->(42.2,47.9) net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` — OE_N B east wall x 42.2 (44.4->47.9) (removed; (42.2,42.75->44.4) kept)

### G1 — add
41. route_trace B.Cu w0.25 net `DO4_CTRL`: (33.26,45.35) -> (35.8,45.35)
42. route_trace B.Cu w0.25 net `DO4_CTRL`: (35.8,45.35) -> (35.8,48.4)
43. route_trace B.Cu w0.25 net `DO4_CTRL`: (35.8,48.4) -> (40.3,48.4)
44. route_trace B.Cu w0.25 net `DO4_CTRL`: (40.3,48.4) -> (40.975,47.725)
45. route_trace B.Cu w0.25 net `DO4_CTRL`: (40.975,47.725) -> (40.975,47.45)
    (net DO4_CTRL: 11.3 mm added, 0 via(s))
46. route_trace F.Cu w0.25 net `/DIGITAL_OUTPUTS/DO_BUF_OE_N`: (37.138,46.0) -> (37.138,46.6)
47. route_trace F.Cu w0.25 net `/DIGITAL_OUTPUTS/DO_BUF_OE_N`: (37.138,46.6) -> (37.35,46.85)
48. route_trace B.Cu w0.25 net `/DIGITAL_OUTPUTS/DO_BUF_OE_N`: (37.35,46.85) -> (37.35,44.72)
49. add_via 0.6/0.3 net `/DIGITAL_OUTPUTS/DO_BUF_OE_N` at (37.35,46.85)
    (net /DIGITAL_OUTPUTS/DO_BUF_OE_N: 3.1 mm added, 1 via(s))
50. route_trace F.Cu w0.25 net `SPARE_GPIO1`: (39.0,46.825) -> (38.3,46.85)
51. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (38.3,46.85) -> (38.55,46.6)
52. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (38.55,46.6) -> (38.55,46.0)
53. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (38.55,46.0) -> (41.6,46.0)
54. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (41.6,46.0) -> (42.7,45.6)
55. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.7,45.6) -> (42.7,43.45)
56. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.7,43.45) -> (42.9,43.25)
57. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.9,43.25) -> (42.9,42.0)
58. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.9,42.0) -> (42.05,41.15)
59. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.05,41.15) -> (42.05,17.2)
60. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.05,17.2) -> (42.25,17.0)
61. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (42.25,17.0) -> (46.7,17.0)
62. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (46.7,17.0) -> (46.85,17.15)
63. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (46.85,17.15) -> (46.85,21.15)
64. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (46.85,21.15) -> (47.05,21.35)
65. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (47.05,21.35) -> (60.65,21.35)
66. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (60.65,21.35) -> (75.4,36.1)
67. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (75.4,36.1) -> (75.7,36.4)
68. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (75.7,36.4) -> (76.3,36.4)
69. route_trace B.Cu w0.25 net `SPARE_GPIO1`: (76.3,36.4) -> (76.6,36.1)
70. route_trace F.Cu w0.25 net `SPARE_GPIO1`: (76.6,36.1) -> (86.9,36.1)
71. route_trace F.Cu w0.25 net `SPARE_GPIO1`: (86.9,36.1) -> (87.38,35.62)
72. route_trace F.Cu w0.25 net `SPARE_GPIO1`: (87.38,35.62) -> (87.38,31.25)
73. add_via 0.6/0.3 net `SPARE_GPIO1` at (38.3,46.85)
74. add_via 0.6/0.3 net `SPARE_GPIO1` at (76.6,36.1)
    (net SPARE_GPIO1: 95.2 mm added, 2 via(s))

Total ops: 74 (+ open_board, refill_zones per batch)


## 21. Rollback strategy
Persistent PRE-C6C backup to be taken before any write (timestamped, in `kicad_mcp_test-backups/`); rollback = `cp -p` of that file back; SHA must read `cf1699628b84f707…` (the existing PRE-C6A/PRE-C6 backups restore the earlier `2226d0c3…` state instead). Per-batch abort: any electrical DRC, new airwire, new trapped GND item, or plane island > 25 mm² with GND items → restore backup.

## 22. Live SHA / end-state proof
Live `.kicad_pcb` `cf1699628b84f7072bcb485d38b69af4345601a9748963b2ab6efc529387e7ae` at start and end; 17/17 project-file SHAs identical; 136 / 1024 / 188; DRC 12 silk_edge, unconnected 2; 0 lck, 0 GUI, 0 MCP processes.

## 23. Recommendation for PCB-02C6C
Implement D1 then G1 exactly as §20 (disposable reproduction first, then live batch-by-batch with gates). Expected result: unconnected 0, electrical DRC 0, tracks 1054, vias 192. Human decisions to confirm before C6C: (a) accept 0.15 mm width for the two short field-side stubs; (b) accept the 95 mm SPARE_GPIO1 detour (plane-safe) instead of a shorter path that would island the mid-board plane; (c) accept re-laying the Phase-2 DO-block feeds (DO4_CTRL U8-side row, OE_N Q1.3 link).

PCB-02C6B STUDY: PASS
LIVE PCB MODIFIED: NO
DI4_SENSE SOLUTION: FOUND
SPARE_GPIO1 SOLUTION: FOUND
COMBINED DISPOSABLE TEST: PASS
HYPOTHETICAL UNCONNECTED: 0
ELECTRICAL DRC: 0
GND PLANE: PRESERVED
DI ISOLATION: PRESERVED
ANTENNA KEEPOUT: PRESERVED
C5L PROTECTED COPPER: PRESERVED
NEXT PROPOSED STAGE: PCB-02C6C IMPLEMENTATION
WAITING FOR HUMAN REVIEW
