# PCB-03A1 — CORRECTIVE FEASIBILITY STUDY

Read-only study on `hardware/kicad_mcp_test.kicad_pcb` (post-C6C, SHA `1ce1972c6f0b6185dc2cb79277f678ecb5157a4cf7e46fa8f7644536f27b17f9`). Experiments only on disposable copies `/tmp/pcb03a1/cand/{T1,T2,T3,COMB}` via KiCad MCP; audits with kicad-cli DRC, pcbnew read-only inspection, shapely geometry model and the 0.05 mm B.Cu fill raster model. Renders `/tmp/pcb03a1/final/*.png` (live ones labelled LIVE, candidates "DISPOSABLE STUDY COPY — NOT LIVE").

## 1. Live baseline / integrity
Start and end: 17 project-file SHAs identical to the PCB-03A start record; `.kicad_pcb` `1ce1972c6f0b6185…`; 136 fp / 1054 tracks / 192 vias; kicad-cli DRC 12 silk_edge only; unconnected 0; 0 `.lck`, 0 KiCad GUI, 0 MCP processes at end. Live never opened for writing.

## 2. R1 — ground-plane neck investigation
**Reconstruction (B.Cu LOGIC_GND, raster model, opening 0.6 mm):** main fill 4937 mm² splits into three parts: main body 3733 mm²; **region A** 563 mm² (x 42.5–78.6, y 17.4–44.4, the mid-board band under/around the ETH diagonals and the C5L rows); **region B** 509 mm² (x 19.8–81.9, y 4.2–32.2, the corridor south of the I2C/VBUS_DET_N B bus + U4 underside + NE area).
- Neck A: x 43.35–43.85, y 42.05–43.50; fill width 0.60 mm, ≈1.5 mm long; formed by the 3V3 B vertical x 41.2 (w0.8), OE_N via (42.2,42.75)/(42.2,42.75→44.4), DO4_CTRL corner (44.3,42.2)→(44.6,41.9) and the GPIO1 lane track x 42.7/42.9 (C6C). GND items depending on region A: **none** (no pad/via touches it).
- Neck B: x 20.45–20.85, y 2.65–4.35; fill width 0.45 mm, ≈1.7 mm long; it is the channel between J8.1 (SPARE_GPIO2, PTH) and J8.2 (USB_VBUS_DET_N) header pads, bounded by the GPIO2 B descent (19.3,3.5→7.5) and the VBUS_DET_N B vertical (21.84,3.5→5.62). GND items in region B: U4.J1.1, U4.J1.2, U4.J2.1 (WIZ850io GND pins), vias (71.8,15.375), (72.5,10.875), (72.7,22.0), (74.4,11.05), (78.3,27.0) and two of the four J1.S1 shield pads (the other two S1 pads at y 2.6 are in the main body). Additional copper ties of region B to the main body: F.Cu N-bridge (78.3,27.0)→(81.1,27.0) w0.5 into the U3 interior island, which reaches the main body through the NE bridge (86.2,24.6)→(87.95,25.2) w0.5 — i.e. two 0.5 mm F links in series, plus the 0.45 mm fill neck.
**Signal/return mapping:** ETH SCK/MOSI (F.Cu diagonals) run over region B for their whole U4→(72.85/73.3, 27) length — the same region that holds the U4 GND pins, so their return current stays inside region B; only the last 12 mm (verticals x 72.85/73.3, y 27→39) lie over region A / the NE strip, and the U3-side ground reaches region B through the N bridge (one 0.5 mm F link). ETH MISO/CS/INT are B.Cu tracks (no plane reference by construction). ONEWIRE F row y 41.0 (42.3→69.2) runs over region A; its return exits through neck A — a slow 1-Wire signal (µs edges). USB_VBUS_DET_N is a B.Cu track (DC-level sense). USB D± (F, 90–94 % over the main body) do not use either neck. Conclusion: no high-edge-rate signal forces its return through neck A; neck B carries the DC return of the Ethernet module (≈150–200 mA class) and half of the USB shield's plane contact; region A carries no GND item.
**Correction attempts:** T1 (single F.Cu GND tie J9.2 pad → via (38.34,6.45) into region B) — fails DRC (crosses the I2C_SDA F track y 6.3 and the R36 pads; the whole J9/R36/R37/TP17/TP18 area is I2C F wiring), no other GND pad has a legal F path across the bus (ONEWIRE F row y 5.5 blocks x 12.9–36.3; 3V3 F row y 5.0 blocks x 45–70; ETH_RST_N x 70.9). Re-laying the GPIO2 descent or the I2C bus would be a corridor rework (last-resort class). **Decision: ACCEPT AS-IS** — DC continuity is provided by three parallel paths (0.45 mm fill neck + two 0.5 mm F bridges), no fast signal depends on the necks, the USB shield has direct main-body pads, and the only candidate correction is a corridor rework disproportionate to the evidence. Recorded for the design-rule input: if an ESD/EMC test later shows Ethernet-module ground bounce, the smallest change is a GND tie through the J9/R36 I2C area (requires moving the SDA F stub).

## 3. R2 — DI isolation investigation
Objects: (1) DI3_SENSE via (68.5,61.3), 0.6 mm, B.Cu ring (field side); (2) LOGIC_GND B.Cu fill in the pocket x 70–74 / y 55.2–57.5 (8.35 mm²) that lies between the DI_ISO_GAP rule area (ends at x 70) and the RS485_ISO_GAP rule area (starts at y 57.5). Edge-to-edge 3.818 mm on B.Cu (zone clearance 0.3 mm contributes: the fill edge is 0.3 mm from the keepout/track edges). Classification: the pocket is **adjacent to** the intended DI barrier (outside the keepout polygon) but geometrically closer than the ISO1212 package pitch (4.0 mm pad-row spacing) and closer than the keepout's own 3.6 mm band + clearances; it reduces the functional creepage on B.Cu between field copper and logic ground. Project rules: DI_FIELD clearance 0.4 mm (not an isolation rule); no project rule states 4.0 mm — the intent is expressed by the keepout geometry (≈3.6–4.5 mm). CURRENT GEOMETRIC MINIMUM 3.818 mm (B.Cu). REGULATORY STATUS: NOT ESTABLISHED (working voltage, pollution degree, material CTI, standard unknown).
**Correction (T2, disposable):** one additional rule area `DI_ISO_GAP_EXT`, layers F.Cu+B.Cu, polygon (70,55.6)-(74,55.6)-(74,57.5)-(70,57.5), "no copper pour" only (tracks/vias allowed so the existing LOGIC_GND via (72.825,55.2) and nearby tracks are untouched), then refill. Result: pocket fill 8.35 → 0.75 mm² (a 0.4 mm strip y 55.2–55.6 remains, ≥ 5.6 mm from any field copper); **DI field↔logic minimum 3.818 → 4.250 mm** (now the DI_FIELD_GND B row y 59.6 vs the fill north of the keepout); RS485 minimum unchanged 6.0 mm; LOGIC_GND fill 4592.4 → 4584.7 mm² (−7.7 mm²), 2 outlines, trapped set identical (17), lifelines unchanged; DRC 12 silk only; unconnected 0; tracks/vias/footprints/pads unchanged. Note: the MCP `add_rule_area` tool writes the file itself; the batch must be add_rule_area → open_board → refill_zones (a refill from a board opened *before* the rule-area write overwrites it — observed once, corrected). **Decision: CORRECT BEFORE 03B** (restores the ≥ 4.0 mm intent with a copper-free zone change only; no routing touched).

## 4. R3 — DO ground-return investigation
Per channel (U9/U10/U11/U12, ZXMS6004N8Q SOIC-8 rot −90): source pads 1/2/3 at y 63.525 (x 13.965/15.235/16.505, +7.3 mm per channel), 0.5 mm F.Cu link between the three pads, one 0.5 mm F stub from pad 2 down to a single via 0.6/0.3 at (x, 64.925) into the B plane (main body, directly beneath); downstream: plane → J6.6 GND PTH ≈ 12–25 mm away. Drain/load path: DOx_OUT 1.0 mm F.Cu 22–25 mm to J6. Geometrically the via is the narrowest element of the load return (0.3 mm drill ≈ 0.14 mm² plated cross-section vs 1.0 mm × 35 µm trace); whether it is *insufficient* depends on the per-channel load and duty — unknown (ZXMS6004 self-limits at its internal current limit; connector, copper weight, ambient unknown). **T3 (disposable, U9):** two additional vias at (13.965,64.925) and (16.505,64.925) with 0.5 mm F stubs from pads 3 and 1 — DRC 0, unconnected 0, nearest foreign copper 0.72 mm (DO1_DRIVE pad U9.4), plane continuity unaffected; the identical pattern fits U10–U12 (same footprint, same free space below the source row; DOx_DRIVE pad is the only neighbour at ≥0.7 mm). **Decision: DEFER PENDING MAXIMUM DO LOAD CURRENT** (feasible, 6 vias + 6 stubs total, zero risk; implement if the specified continuous load exceeds ~1 A per channel).

## 5. R4 — U1 SW node investigation
Routed SW: pin 8 (28.875,26.095) → (28.875,24.6) → (32.775,24.6) → (32.775,26.43) [C4.2 BOOT cap on the node] → (33.845,27.5) → (33.845,31.5) L1.1; 12.7 mm × 1.0 mm (≈12.7 mm² + pads); direct pin-8→L1.1 distance 7.34 mm. Why the hook: the direct diagonal crosses the BOOT trace (pin 7 → C4.1) and the VCC cap C5/VCC trace (pin 6 → C5.1); C4 must sit on the SW node, so SW goes around the north of C4. FB: nearest FB_5V copper ≥ 3.5 mm from any SW copper (FB runs south from pin 5, SW north/east). GND: B plane continuous under the whole SW hook (no B tracks under x 28.9–33.8 / y 24.6–31.5 except the plane). A shortened alternative (e.g. y 25.1 instead of 24.6) saves ≈1 mm but violates the 0.3 mm POWER_5V clearance to C4.1/BOOT (0.175–0.275 mm); a materially shorter path needs C4/C5 to move (not allowed). Loop area change would be < 10 %. **Decision: ACCEPT AS-IS.**

## 6. R5 — U2 C8 return investigation
HF input loop: C8.1 (21.05,34.9) → F 1.0/0.6 mm → U2.1 VIN (21.863,37.55): 3.8 mm; C8.2 (22.95,34.9) → F 0.8 mm 1.4 mm → via (22.95,33.5) → B.Cu (plane + explicit 0.8 mm B track (22.95,33.5)→(20.4,38.5), 5.6 mm) → via (20.4,38.5) → F 0.4 mm 1.5 mm → U2.2 GND. Loop ≈ 4 × 2.5 mm through two vias with the B plane directly underneath (no plane gap between the two vias — checked: continuous fill). A direct F.Cu return C8.2 → pin 2 is blocked by the 5V_MAIN F row y 36.44 (1.0 mm) and C8.1; the alternative (relocating C8 beside pins 1/2) is a footprint move. The plane path is short and unbroken; TLV62569 (1.5 MHz) tolerates it. **Decision: ACCEPT AS-IS.**

## 7. R7 / R8 — fabrication feature inventory (live, exact)
- Copper pairs < 0.21 mm (different nets, same layer): 52 = 23 PACKAGE-INTERNAL (all J1 USB4105 signal pads A5–A8/B5–B8, min 0.198 mm by polygon approximation, covered by the .dru 0.20 mm package rule, DRC-clean) + 29 BOARD-ROUTING at 0.200–0.210 mm: R45.2 ↔ ETH_RST_N (0.200), ETH_MOSI ↔ ETH_SCK parallel diagonals/verticals (0.200–0.2025, 6 pairs), BOOT_N ↔ RS485_TXD via (0.205), I2C_SCL ↔ GND via (87.95,25.2) (0.205), OE_N via ↔ DO4_CTRL row (0.205, C6C-inherited), DI3_LOGIC via ↔ DI4_LOGIC (0.205), I2C/VBUS_DET_N B bus rows (0.207–0.21, 12 pairs), U6.6/U7.6 NC pads ↔ DI2/DI4_LOGIC exits (0.2077), R33.1 ↔ DO1_CTRL via (0.2084). None below the 0.20 mm board rule.
- Tracks ≤ 0.20 mm: 0.15 mm ×4 (DI4_IN stub 2.3 mm; DI4_SENSE stubs 0.85+0.36+0.2 mm — C6C), 0.20 mm ×12 (DO4_CTRL C5L lane 5 segments; USB J1 breakout stubs CC1/CC2/D+/D− 7 segments ≤ 1.35 mm).
- Vias: 192 × 0.6/0.3 (annular 0.15). PTH: drills 0.2 (12, U3 thermal-via pads 0.6), 0.33 (6, U1 EP vias 0.63), 0.6, 0.8, 0.914, 1.0, 1.1, 1.3 mm; min PTH annular 0.15.
- Board rules: min clearance 0.2, min track 0.15, min via 0.5, min through drill 0.2, hole clearance 0.25, hole-to-hole 0.25, edge clearance 0.5, min annular 0.1.
**Minimum manufacturing requirements for PCB-03B:** track ≥ 0.15 mm (6 mil), spacing ≥ 0.20 mm (8 mil), mechanical drill ≥ 0.20 mm, finished via 0.3 mm drill / 0.6 mm pad (0.15 annular), PTH annular ≥ 0.15, copper-to-edge 0.5 mm, 2-layer 1.6 mm. Fab must confirm these (D7).

## 8. R9 — label findings (read-only, netlist-verified)
- J8 pin 1: net SPARE_GPIO2 = ESP32-S3 pin 6 = **IO6**; silk reads "IO2" (text at (18.2–20.4, 5.2–6.6)) → corrected label "IO6". J8 pins 2–6 silk "VBUSD/DQ/3V3/GND/NC" match nets USB_VBUS_DET_N / ONEWIRE_DATA / 3V3_LOGIC / LOGIC_GND / NC.
- TP3: value "PWR_GND", net LOGIC_GND; adjacent silk reads "GND" → silk correct, schematic value misleading (single-ground design). TP16: value "DO_FIELD_GND", net LOGIC_GND; adjacent silk "GND" → same. Recommended value/label text: "GND" (or keep names but document that all grounds are one net). Candidates for a later bounded documentation cleanup; not this stage.

## 9. R12 — silk-edge findings (12, all "Silkscreen clipped by board edge")
J6 ×2 (9.43,80.72)/(40.15,70.68); J4 ×2 (74.92,80.72)/(90.41,70.68); J5 ×2 (44.42,80.72)/(70.06,70.68); J2 ×2 (−0.12,9.34)/(9.92,19.74); J10 ×2 (−0.12,41.34)/(9.92,51.74); U3 ×2 (100.1,30.8)/(100.1,49.2). All are footprint courtyard/outline silk of edge-mounted connectors and the ESP32 module drawn up to/over the board edge: intentional placement (terminal blocks flush with the edge, module antenna at the edge). Classification: documentation-only / fab will clip silk at the edge; candidate cleanup = none required (optionally hide the outer outline segments). No fabrication concern.

## 10. USB D− / ETH_RST_N sanity check
USB: D+ chain 28.3 mm (F only, 0 vias), D− chain 29.6 mm (25.4 F + 4.2 B, 2 vias at (78.3,12.9)/(77.5,17.0)); skew 1.3 mm; 0.25 mm tracks, 0.25 mm min spacing; both ≥ 90 % over the main plane; no stubs. Full-Speed USB only (ESP32-S3 native) — the asymmetry and skew are electrically irrelevant at 12 Mb/s; impedance not claimed (D2). Decision: ACCEPT AS-IS.
ETH_RST_N: 67.7 mm F.Cu from U4.J2.5 (68.32,17.66) via x 70.9 north/east along the USB area to x 83.35 and down to U3.12 (78.49,31.25), series R14 1 k at the ESP32 end; 50 % over the main plane, 17 fill transitions; connectivity 2/2 + R14. Reset line (static level) — length/reference irrelevant. Decision: ACCEPT AS-IS.

## 11. Decision matrix
| Finding | Evidence | Decision | Proposed action |
|---|---|---|---|
| R1 plane necks | region A no GND items; region B tied by 0.45 mm fill + 2 F bridges; no fast signal depends on necks; only correction is a corridor rework (T1 tie fails) | ACCEPT AS-IS | none (record for EMC test) |
| R2 DI spacing 3.82 mm | pocket fill between keepouts; T2 restores 4.25 mm with one copper-free rule area, no routing touched | CORRECT BEFORE 03B | add rule area DI_ISO_GAP_EXT + refill |
| R3 DO source via | single 0.3 mm via/channel; T3 shows 2 extra vias/channel fit with 0.72 mm clearance | DEFER PENDING DESIGN INPUT (max DO load) | implement T3 pattern ×4 if load > ~1 A/channel |
| R4 U1 SW hook | 12.7 vs 7.3 mm, forced by C4/C5 placement, FB ≥ 3.5 mm, plane beneath | ACCEPT AS-IS | none |
| R5 U2 C8 return | 2-via loop ≈ 4×2.5 mm over continuous plane; direct F link blocked by 5 V row | ACCEPT AS-IS | none |
| R7 clearances 0.20–0.21 | all ≥ 0.20 board rule; 23 J1 package-internal | DEFER PENDING DESIGN INPUT (fab spec) | 03B fab check |
| R8 0.15/0.20 tracks | 4 + 12 short segments | DEFER PENDING DESIGN INPUT (fab spec) | 03B fab check |
| R9 labels | J8.1 silk IO2 vs IO6; TP3/TP16 values vs net | ACCEPT AS-IS (documentation cleanup later) | later silk/value edit "IO6", "GND" |
| R12 silk edge | 12 edge-clipped connector/module outlines | ACCEPT AS-IS | none |
| USB D− asym. / ETH_RST_N | FS USB; static reset | ACCEPT AS-IS | none |

## 12. Disposable correction candidates
T1 (R1 GND tie) — FAILED (tracks_crossing + shorting with I2C_SDA F track; not pursued). T2 (R2 rule area) — PASS. T3 (R3 vias, U9) — PASS (feasibility; deferred). COMB — see §13.

## 13. Combined candidate (only recommended corrections) = T2
`/tmp/pcb03a1/cand/COMB`: fresh copy of live + rule area DI_ISO_GAP_EXT + refill. SHA `4d7fc4900348b9c1…`.

## 14. DRC / connectivity results (COMB)
kicad-cli: clearance/shorting/items_not_allowed/copper_edge/hole/drill/track_width 0 → electrical 0; silk_edge 12 (unchanged set); unconnected 0; netlist 425/425 (+5) = 430/430; pcbnew unconnected 0.

## 15. Regression results (COMB vs live)
Tracks 1054 = 1054 (0 diff), vias 192 = 192 (0 diff), footprints/pads identical, existing zones/rule areas identical + 1 new rule area; LOGIC_GND fill 4584.7 + 230.3 mm² (2 outlines); plane model trapped set 17 identical, lifelines (K5, Q1.2, R10/R11, USB shield, U4 GND pins) all main; DI min 4.25 mm, RS485 min 6.0 mm; antenna keep-out untouched; K5/BOOT-A/bridges/D1/G1 vias present; default-off chain pads unchanged (netlist identical).

## 16. Exact proposed implementation manifest (PCB-03A2)
REMOVED ITEMS: none. ADDED ITEMS: 1 rule area — name `DI_ISO_GAP_EXT`, layers F.Cu + B.Cu, polygon (70.0,55.6) (74.0,55.6) (74.0,57.5) (70.0,57.5) mm, disallow copper pour = true, tracks = false, vias = false, footprints/pads = false (MCP `add_rule_area`, then `open_board` + `refill_zones`). MOVED VIAS: none. REROUTED NETS: none. ZONE EFFECT: LOGIC_GND B.Cu fill −7.7 mm² inside the pocket (8.35 → 0.75 mm²), no other fill change; RS485_GND fills unchanged. Optional (only if D1 load input requires): R3 pattern — per channel U9–U12 add F.Cu LOGIC_GND stubs 0.5 mm from source pads 1 and 3 to vias 0.6/0.3 at (x_pad, 64.925): U9 (13.965/16.505), U10 (21.265/23.805), U11 (28.565/31.105), U12 (35.865/38.405) — not part of the recommended set.

## 17. Remaining design inputs
D1 loads/copper weight (decides R3), D2 USB stackup (impedance statement), D3 SPI clock, D4 isolation standard/working voltage (R2 gives 4.25 mm geometric), D5 ESP32-S3 strap defaults, D6 dissipations, D7 fab capability (R7/R8 inventory §7), D8 enclosure clearance at the antenna edge.

## 18. Recommendation before PCB-03B
Implement the single R2 correction (PCB-03A2: one rule area + refill, no routing), then proceed to PCB-03B with the §7 fabrication inventory; keep R3 as a conditional change tied to the DO load specification.

## 19. Live-project integrity proof
`sha_end.txt` == PCB-03A `sha_start.txt` (17/17 files); `.kicad_pcb` `1ce1972c6f0b6185…`; 1054/192/136 unchanged; DRC and unconnected unchanged; 0 `.lck`; 0 KiCad/MCP processes.

PCB-03A1 STUDY: PASS
LIVE PCB MODIFIED: NO
CORRECTIONS REQUIRED BEFORE 03B: 1
DISPOSABLE COMBINED CANDIDATE: PASS
LIVE IMPLEMENTATION: NOT AUTHORIZED
NEXT PROPOSED STAGE: PCB-03A2 IMPLEMENTATION
WAITING FOR HUMAN REVIEW
