# PCB-02C6 — FINAL AIRWIRE CLASSIFICATION & CLOSURE PLAN

Project `hardware/kicad_mcp_test.kicad_pcb` (Rev-A Industrial IoT Edge Node, KiCad 9.0.9). Stage type: READ-ONLY classification + route planning. **No live routing performed.** All analysis: kicad-cli (DRC, netlist, SVG), pcbnew SWIG read-only, shapely/scipy geometry models, a grid maze-router used only as an existence/feasibility oracle. Study artefacts: `/tmp/c6/` (scripts `e/geo.py`, `e/chk.py`, `e/router.py`, `e/plane.py`, manifest `e/c6a.py`, `e/margins.json`, renders `final/*.png`). Note: the host rebooted twice during this stage (`/tmp` wiped both times; everything below was regenerated from the live files afterwards).

## 1. Preflight
KiCad GUI processes 0, `.lck` 0 (checked at stage start, after the second reboot, and at stage end). Live `.kicad_pcb` SHA-256 `2226d0c333d513f57f18effbb9a015a3ea746f1a00f7d89d933ec515458890dc`; `.kicad_pro` `21fd0939…`, `.kicad_dru` `c511468f…`; 9 schematics, 3 symbol libs, 2 lib tables recorded in `/tmp/c6/sha_start.txt` (17 files).

## 2. Persistent backup result
Created `hardware/kicad_mcp_test-backups/kicad_mcp_test.PRE-C6.20260921-171851.kicad_pcb` by file copy (`cp -p`; no KiCad write path involved, no existing backup overwritten). `cmp` byte-identical to live; SHA-256 `2226d0c3…` == live. Re-verified byte-identical after the second reboot. `/tmp` is no longer the only rollback copy.

## 3. Live SHA baseline
Start `2226d0c333d513f5…`; end `2226d0c333d513f5…` (§21). Unchanged.

## 4. C5L regression verification (object inspection, not pixels)
136 footprints, 989 tracks, 183 vias; netlist 425/425 annotated nodes + 5 board-only nodes (Q2/R48, schematic `Q?`/`R?`, pre-existing) = 430/430; kicad-cli DRC: 12 silk_edge_clearance only, **0 electrical**, unconnected **6**; GND in unconnected list: 0. Objects present: K5 via (25.2,48.41) LOGIC_GND 0.6/0.3; K5 F tie (25.75,47.95)→(25.2,48.41) w0.4; old via (25.04,47.95) absent; CTRL1 B lane DO1_CTRL y 47.775 (20.55→33.0); U3 NE bridge via (86.2,24.6) + N bridge vias (78.3,27.0)/(81.1,27.0) LOGIC_GND; BOOT-A via (72.2,50.525) BOOT_N; LOGIC_GND B fill = main 4762.2 mm² + U3 interior 230.3 mm² (F-bridged) — identical to C5L. CTRL/DI 8/8 connected (none of the 8 nets in the unconnected list). Protected copper unchanged (SHA identical).

## 5. Exact six-airwire inventory (kicad-cli unconnected_items)
| # | net | item A | item B |
|---|---|---|---|
| 1 | /DIGITAL_INPUTS_ISOLATED/DI4_SENSE | U7.11 F.Cu (62.047,59.6) | F.Cu track 0.42 mm at (61.4,64.22) |
| 2 | /ESP32_CORE/UART0_RX | U3.36 F.Cu (87.38,48.75) | TP10.1 F.Cu (90.3,51.0) |
| 3 | /USB_C/USB_VBUS | B.Cu track 3.39 mm at (76.5,11.0) | J1.A9 F.Cu (74.1,7.355) |
| 4 | 3V3_LOGIC | TP12.1 F.Cu (72.0,9.7) | F.Cu track 5.04 mm at (69.93,5.0…10.04) |
| 5 | SPARE_GPIO1 | R30.1 F.Cu (39.0,46.825) | U3.5 F.Cu (87.38,31.25) |
| 6 | SPARE_GPIO2 | J8.1 PTH (19.3,3.5) | U3.6 F.Cu (86.11,31.25) |

## 6. Endpoint / member details (schematic netlist + PCB connectivity clusters)
- **DI4_SENSE** (sheet DIGITAL_INPUTS_ISOLATED; netclass DI_FIELD 0.4/0.3): U7.11 SENSE2 (input, field side), R28.1 562R, R24.2 330R pulse-proof, C28.1 10nF, D7.2 TVS. PCB: cluster {D7.2, R24.2, R28.1, C28.1} routed (8 F tracks); U7.11 alone.
- **UART0_RX** (ESP32_CORE; Default 0.2/0.25): U3.36 RXD0, TP10.1 (silk "RX", "DEBUG TEST PADS" group with TP7–TP9/TP11 JTAG/TX). PCB: both pads bare. Sibling UART0_TX U3.37→TP11 is routed (F, 4 items).
- **USB_VBUS** (USB_C; Default): J1.A4/A9/B4/B9 (footprint merges A4+B9 at (78.9,7.355) and A9+B4 at (74.1,7.355)), D1.5 USBLC6 VBUS pin, R45.1 39k. PCB: A4/B9 pad → F → via (78.9,8.6) → B → via (76.5,13.5)→D1.5 and via (81.6,9.4)→R45.1; A9/B4 pad bare. Net has NO other members: no regulator, no rail, no J8 pin.
- **3V3_LOGIC** (POWER_3V3 0.25/0.8): 33 pads; 32 in one cluster (126 items); TP12.1 (value "ETH_3V3", silk "3V3" at x 69.8–71.1) alone. Nearest same-net copper: F track (69.93,5.0)→(69.93,10.04) w0.8 and PTH U4.J2.2 (68.32,10.04) (F+B).
- **SPARE_GPIO1** (Default): U3.5 IO5 ↔ R30.1 10k. R30.2 = /DIGITAL_OUTPUTS/DO_BUF_EN_BASE = Q1.1 (MMBT3904 base) with R31 47k base pull-down; Q1.3 = DO_BUF_OE_N = U8 pins 1/4/10/13 (SN74AHCT125 OE) with R29 10k pull-up. Schematic note: "5 V AHCT BUFFER — HARDWARE DEFAULT OFF".
- **SPARE_GPIO2** (Default): U3.6 IO6 ↔ J8.1. J8 = SERVICE header, Pin_Map property "1 SPARE_GPIO2; 2 USB_VBUS_DET_N (…input/test only); 3 ONEWIRE_DATA; 4 3V3_LOGIC; 5 LOGIC_GND; 6 NC"; sheet text "SPARE GPIO2/3 + 1W SERVICE". Silk label at J8.1 reads "IO2" (ESP32 pin is IO6 — label/pin mismatch, documentation only).

## 7. DI4_SENSE forensic result
C5L's 8/8 refers to the LOGIC side (U7.4/U7.5 → U3.30/U3.31) and is correct. DI4_SENSE is the isolated FIELD-side sense node of channel 4 (ISO1212 SENSE2). The passive network (R24 field resistor → R28/C28/D7 → U7 SENSE2) is routed except the last hop into U7.11. Without it channel DI4 has no input current path → **non-functional channel** → REQUIRED_FUNCTIONAL. Both endpoints are on the field side (U7 field pads y 59.6, all copper south of the DI_ISO_GAP keepout edge y 58.8); a route never crosses the barrier. Route feasibility: **none additive** (§15).

## 8. UART0_RX forensic result
Dedicated debug test pad (TP10, silk "RX") paired with TP11 "TX" which IS routed; the ESP32-S3 UART0 is the console/programming UART (native USB D+/D− go to GPIO19/20 separately). A half-routed UART pair is an inconsistency, not an intent. → REQUIRED_TESTPOINT. Additive route exists (§15).

## 9. USB_VBUS forensic result
USB_VBUS is **not** a board rail: members are only the four receptacle VBUS contacts, the ESD array VBUS pin (D1.5) and the 39 k top of the VBUS-detect divider (R45 → USB_VBUS_DET_GATE → Q2 2N7002 gate, R48 470k to GND; Q2 drain = USB_VBUS_DET_N → R40 1k → U3.7 and J8.2; silk "VBUSD = SENSE ONLY"). The airwire is the receptacle's second VBUS pad pair (A9/B4) which the footprint keeps as a separate pad; the schematic ties all four VBUS pins. Connecting it adds parallel contacts of the same connector to the same sense node — no new power path, no back-feed, no injection (nothing on this net can source current except the plug itself, and it already does through A4/B9). → REQUIRED_FUNCTIONAL (connector-pin parity / plug-orientation robustness). Additive route exists (§15).

## 10. TP12 / 3V3 forensic result
3V3 rail complete (32/33 pads in one cluster). TP12 is the only disconnected member; schematic includes it as a test point; silk "3V3" next to it. → REQUIRED_TESTPOINT. Shortest local tie: F.Cu stub + 1 via + B.Cu to the PTH pad U4.J2.2 (3V3) — ETH_RST_N (F, x 70.9, y 1.4–17.6) walls off the F path to the 3V3 track. No change to the rail topology (branch stub only). Additive route exists (§15).

## 11. SPARE_GPIO1 forensic result
Despite the name, SPARE_GPIO1 is the ESP32 control line of the DO buffer enable (IO5 → R30 10k → Q1 base → Q1 collector = DO_BUF_OE_N → all four AHCT125 OE pins). "Hardware default off" means the buffer stays disabled until firmware drives IO5 high; without this connection the four DO channels can never be enabled. → REQUIRED_FUNCTIONAL (net-name anomaly noted; no schematic change proposed). Route feasibility: **none additive** (§15).

## 12. SPARE_GPIO2 forensic result
Real schematic connection ESP32 IO6 → service header J8 pin 1; header pin map documents it; silk says "IO2" (mismatch is silkscreen documentation only — no silk change authorised or proposed). → REQUIRED_FUNCTIONAL (service exposure). Additive route exists (§15) — 89 mm, 1 via, plane-safe.

## 13. Classification table
| # | Net | Endpoint A | Endpoint B | Other members | Classification | Must route? | Reason |
|---|---|---|---|---|---|---|---|
| 1 | DI4_SENSE | U7.11 (62.047,59.6) F | R28.1 stub (61.4,64.22) F | D7.2, R24.2, C28.1 (routed) | REQUIRED_FUNCTIONAL | YES | field-side sense input of DI4; channel dead without it |
| 2 | UART0_RX | U3.36 (87.38,48.75) F | TP10.1 (90.3,51.0) F | — | REQUIRED_TESTPOINT | YES | console UART debug pad, TX half already routed |
| 3 | USB_VBUS | J1.A9/B4 (74.1,7.355) F | B.Cu net copper at (76.5,11.0) | J1.A4/B9, D1.5, R45.1 (routed) | REQUIRED_FUNCTIONAL | YES | 2nd VBUS contact pair of the receptacle; sense-only net |
| 4 | 3V3_LOGIC (TP12) | TP12.1 (72.0,9.7) F | U4.J2.2 (68.32,10.04) PTH / F track x 69.93 | 31 others (routed) | REQUIRED_TESTPOINT | YES | test point on complete rail |
| 5 | SPARE_GPIO1 | R30.1 (39.0,46.825) F | U3.5 (87.38,31.25) F | — | REQUIRED_FUNCTIONAL | YES | DO buffer enable control (Q1 base) |
| 6 | SPARE_GPIO2 | J8.1 (19.3,3.5) PTH | U3.6 (86.11,31.25) F | — | REQUIRED_FUNCTIONAL | YES | service header GPIO per pin map |

## 14. Required-vs-intentional count
REQUIRED_FUNCTIONAL 4, REQUIRED_TESTPOINT 2, INTENTIONAL_NC 0, SCHEMATIC_ANOMALY 0 (two naming/silk anomalies noted, electrically unambiguous), AMBIGUOUS 0. Additively routable now: **4/6**. REQUIRES_LOCAL_REWORK: **2/6** (DI4_SENSE, SPARE_GPIO1).

## 15. Candidate route for each required item (geometry in `/tmp/c6/e/c6a.py`; all vias 0.6/0.3)
**USB_VBUS** — src J1.A9/B4 (74.1,7.355) F; dst existing B corner (76.5,11.0); F (74.1,7.355)→(74.1,8.7)→(73.95,8.85) w0.4, via (73.95,8.85), B (73.95,8.85)→(75.8,10.7)→(76.1,11.0)→(76.5,11.0) w0.4. Netclass Default (0.25) — 0.4 used to match the existing VBUS copper. 1 via, 5.0 mm. Closest: J1.A12/B1 GND pads and J1.B5 CC2 pad 0.30 (req 0.20), CC2 track 0.325. No C5L copper, no pour issue, no keepout.
**TP12** — src TP12.1 (72.0,9.7) F; dst U4.J2.2 PTH (68.32,10.04) on B; F (72.0,9.7)→(71.75,9.95) w0.5, via (71.75,9.95), B (71.75,9.95)→(71.66,10.04)→(68.32,10.04) w0.5. 1 via, 3.8 mm. Closest: ETH_RST_N F 0.425 (req 0.25), LOGIC_GND via (72.5,10.875) 0.59. Rail topology untouched.
**UART0_RX** — src U3.36 (87.35,48.75) F; dst TP10.1 (90.3,51.0) F; F (87.35,48.75)→(87.35,47.75)→(87.0,47.4)→(86.35,47.4)→(86.1,47.15)→(86.1,46.2)→(86.5,45.8)→(88.8,45.8)→(89.4,46.4)→(90.9,46.4)→(91.3,46.8) w0.25 (loops north of the C5L DO1/DO2 F hooks, under the module), via (91.3,46.8), B (91.3,46.8)→(91.3,50.75) (between 3V3 B x 90.2 and the LOGIC_GND via (93.26,48.75)), via (91.3,50.75), F (91.3,50.75)→(91.05,51.0)→(90.3,51.0). 2 vias, 14.3 mm. Closest: DO1_CTRL F hook 0.264/0.296 and DO1 via (86.8,46.5) 0.275 (req 0.20), DO2 via 0.313, 3V3 F 0.375 (req 0.25), UART0_TX row 0.325, TP11 0.55. Direct path is boxed by JTAG_TMS/TP9 (west) and UART0_TX (east) with a 0.38 mm slot — hence the loop. Max x 91.6 < antenna zone. Touches no C5L copper.
**SPARE_GPIO2** — src J8.1 PTH (19.3,3.5), dst U3.6 (86.11,31.25) F; B (19.3,3.5)→(19.3,7.5)→(46.0,7.5)→(46.0,16.4)→(59.1,16.4)→(75.3,32.6)→(78.3,32.6) w0.25 (south of the I2C/VBUS_DET_N B bus, through the U4 header gap J1.4/J1.5, then parallel to and 0.62 mm NE of the ETH_CS_N diagonal), via (78.3,32.6), F (78.3,32.6)→(79.4,33.7)→(81.7,33.7)→(82.1,34.1)→(85.0,34.1)→(85.8,33.3)→(85.8,32.6)→(86.11,32.3)→(86.11,31.25) w0.25 (under the module between the pad-row vias and the thermal pad, into U3.6 from the south). 1 via, 89.2 mm. Closest: RXD/TXD/SCL/SDA vias 0.275 (req 0.20), ETH_MISO B 0.275, 3V3 B via (45.0,13.0) 0.375 (req 0.25), U4.J1.4/J1.5 pads 0.375. Crosses no keepout; enters U3 interior west of x 79 (MISO), nowhere near the antenna. Plane-safe (§16).
**DI4_SENSE** — REQUIRES_LOCAL_REWORK. U7.11 sits in a closed pocket: west = DI4_IN F track U7.10→R28.2 (x 61.413) + R28.2 pad; east = DI_FIELD_GND F stub U7.14→(63.2,61.2) via→C27.2; south = the R28.2/C27.2 gap of 0.85 mm (DI_FIELD class needs 0.3+0.4+0.4 = 1.1; even 0.15 mm track needs 0.95). B.Cu escape (the U6/DI2_SENSE pattern) impossible: DI_FIELD_GND B row y 60.3 runs straight under U7 (no jog like U6's (47.6,59.6)→(51.2,59.6)), DI_FIELD_GND via (63.2,61.2), and the DI3_SENSE B row y 62.2 (60.5→68.0) + vertical (60.5, 62.2→64.0) seal the pocket. Best attempts: F corridor margin −0.125 (w0.3) / −0.050 (w0.15); B via (62.28,60.95) −0.200 vs GND row, (62.28,61.3) −0.074 vs GND via. Maze router: NO PATH at w0.3 and w0.15. Obstructing objects (all Phase-2A protected DI copper): R28.2 pad (DI4_IN), C27.2 pad (DI_FIELD_GND), F DI4_IN (61.413,59.6)→(61.413,61.9), F DI_FIELD_GND (63.953,60.45)→(63.45,61.0)→(63.35,61.2) + via (63.2,61.2) + (63.2,61.2)→(63.2,62.225), B DI_FIELD_GND (51.9,60.3)→(63.2,60.3), B DI3_SENSE (60.5,62.2)→(68.0,62.2) and (60.5,64.0)→(60.5,62.2).
**SPARE_GPIO1** — REQUIRES_LOCAL_REWORK. R30.1 is boxed on F (DO_BUF_EN_BASE F (39.0,45.175)→(40.5,45.175)→(41.3,45.975)→(41.3,46.825)→(42.5,46.825), DO4_CTRL via (40.975,47.45), DO3_CTRL F row y 48.4, OE_N via (37.6,47.4)). A via under the pad lands in a B pocket (DO4_CTRL B row y 45.35 north, DO_BUF_OE_N B x 42.2 east, DO_BUF_OE_N B row y 48.2 south, DO_BUF_OE_N via/stub (37.6,47.4)→(37.9,48.2) west) whose only exit (x 34.2–37.3) is shared with the LOGIC_GND via (36.55,47.5) = the only ground of Q1.2 (emitter) — any track through that exit turns that via into a 0.7 mm² B island (plane model), i.e. floats the Q1 emitter. Maze router with that exit forbidden: NO PATH; F-only: NO PATH. Obstructing objects (Phase-2 DO block, protected): DO_BUF_OE_N B (42.2,42.75)→(42.2,47.9)→(41.9,48.2)→(37.9,48.2)→(37.6,47.9)→via (37.6,47.4); DO4_CTRL B (33.26,45.35)→(40.5,45.35)→(40.975,45.825)→via (40.975,47.45); LOGIC_GND via (36.55,47.5) + F (35.262,46.95)→(36.55,47.5); DO4_DRIVE via (34.02,46.0) + B x 34.02. Note: a plane-cutting route (auto-router result, 74 mm, 2 vias) exists but was rejected for trapping (36.55,47.5) and (43.2,48.4).

## 16. Clearance / margin analysis (analytic, per-netclass, incl. .dru exceptions, mutual clearance between the four new routes, board edge 0.5)
| route | min margin | FAIL | CRIT | TIGHT | ACC | ROB | length | vias |
|---|---|---|---|---|---|---|---|---|
| USB_VBUS | +0.100 | 0 | 0 | 0 | 3 | 13 | 5.0 mm | 1 |
| TP12 | +0.175 | 0 | 0 | 0 | 0 | 7 | 3.8 mm | 1 |
| UART0_RX | +0.064 | 0 | 0 | 0 | 3 | 20 | 14.3 mm | 2 |
| SPARE_GPIO2 | +0.075 | 0 | 0 | 0 | 5 | 24 | 89.2 mm | 1 |

Non-ROBUST pairs (all ≥ +0.05): USB_VBUS F stub ↔ J1.A12/B1/B5 pads 0.30; UART0_RX via (91.3,46.8) ↔ DO1_CTRL F hook 0.264/0.296, F loop ↔ DO1 via 0.275; SPARE_GPIO2 F ↔ RXD/TXD/SCL/SDA vias 0.275, via ↔ ETH_MISO B 0.275. Hole-to-hole ≥ 0.89 (TP12 via ↔ GND via (72.5,10.875)). No new item within 0.5 mm of the board edge. Grid-router margins were +0.03…+0.05; the hand-cleaned manifest raised every pair to ≥ +0.055.
**LOGIC_GND B plane model** (0.05 mm raster, zone clearance 0.3, min width 0.2): live = main 5108 mm² (model) + U3 interior 230.3 + no-item islands; 17 GND items outside main (12 U3.41 thermal pads + 5 vias, all in the F-bridged interior — identical to C5L). Live + C6A: main 5020 mm², **identical trapped set (17), no new trapped or unfilled GND item**. Rejected alternatives: SPARE_GPIO2 via the top corridor (y 6.1) + CS_N-parallel diagonal islanded a 391 mm² region (J1.S1 shield, U4.J1.1/J2.1, 5 GND vias); SPARE_GPIO1 pocket exit islanded (36.55,47.5) and (43.2,48.4).

## 17. Isolation audit
DI_ISO_GAP and RS485_ISO_GAP rule areas: none of the four routes intersects them (checker flags keepouts; 0 hits). DI4_SENSE (field side) was analysed only inside the field region; no barrier crossing proposed. No route joins DI_FIELD_GND/RS485_GND to LOGIC_GND; no new copper inside the isolated regions.

## 18. Antenna audit
No antenna rule area exists on the board; an analytic keepout x ≥ 93.45 mm, y 30.5–49.5 (module x-end 99.95 − 6.5 mm) was enforced in all checks: 0 hits. Max x of new copper: UART0_RX via (91.3,50.75) → 91.6 mm (outside the module's y span anyway).

## 19. USB VBUS safety audit
Net inventory proves USB_VBUS is sense-only (J1 VBUS pins, D1.5 clamp, R45 39k → Q2 gate divider). The proposed tie only bonds the receptacle's second VBUS pad pair to the same node; no connection to 5V_MAIN/3V3/VIN, no diode/regulator path created; powered-off back-feed impossible (no on-board source on that net). J8.2 carries USB_VBUS_DET_N (Q2 drain, logic level), not VBUS.

## 20. C5L protected-copper audit
Live SHA unchanged. Planned C6A copper touches no existing object; closest approaches to C5L copper: UART0_RX loop ↔ DO1_CTRL F hook/via 0.264–0.296 mm (req 0.20) and DO2 via 0.313; SPARE_GPIO2 B ↔ nothing from C5L; USB_VBUS/TP12 far from C5L copper. K5, BOOT-A, U3 bridges, CTRL/DI rows untouched.

## 21. Files / SHA mutation audit
`diff sha_start sha_end`: 17/17 files identical. `.kicad_pcb` = `2226d0c333d513f57f18effbb9a015a3ea746f1a00f7d89d933ec515458890dc` at start and end. Only file created: the persistent backup (§2). `.lck` 0, GUI 0 at end.

## 22. Recommended PCB-02C6A implementation manifest (MCP only; batch = one net; refill + kicad-cli DRC gate after each; abort on any electrical DRC or new GND unconnected)
Batches: B1 USB_VBUS → B2 TP12 → B3 UART0_RX → B4 SPARE_GPIO2 → B5 final refill. Pre-write: procs 0, lck 0, SHA == `2226d0c3…`, new timestamped PRE-C6A backup in `kicad_mcp_test-backups/`, disposable reproduction on a /tmp copy first, then live batch-by-batch.

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


Widths: USB_VBUS 0.4 (matches existing), TP12 0.5 (test-point tie), others 0.25 (Default). Vias: MCP `add_via` (0.6/0.3). Expected new: 35 tracks, 5 vias.

## 23. Expected unconnected count after C6A
6 − 4 = **2** (DI4_SENSE, SPARE_GPIO1). Electrical DRC expected 0; GND unconnected 0; trapped GND vias 0 (plane model).

## 24. Final decision
All six airwires are real required connections (no NC/spare). Four are safely additive and fully planned. Two required nets (DI4_SENSE, SPARE_GPIO1) cannot be routed without moving protected copper (Phase-2A DI field copper / Phase-2 DO-block copper, both inside the frozen set) — a follow-up local-rework study (PCB-02C6B) is needed for them; candidate minimal reworks for human decision: DI4_SENSE — jog the DI_FIELD_GND B row under U7 (mirror of the U6 jog) and re-lay the DI3_SENSE B row so a DI4_SENSE via fits north of it; SPARE_GPIO1 — re-lay the DO_BUF_OE_N B stub (37.6,47.4)…(37.9,48.2) or give Q1.2 a second ground path so the pocket exit can be used.

PCB-02C6: ROUTING CONFLICT
NET: DI4_SENSE, SPARE_GPIO1
OBSTRUCTION: DI4_SENSE — R28.2 (DI4_IN) / C27.2 (DI_FIELD_GND) 0.85 mm gap, F DI4_IN (61.413,59.6→61.9), F+via DI_FIELD_GND (63.953,60.45)→(63.2,61.2)→(63.2,62.225), B DI_FIELD_GND (51.9,60.3)→(63.2,60.3), B DI3_SENSE (60.5,62.2)→(68.0,62.2)+(60.5,62.2→64.0); SPARE_GPIO1 — B DO_BUF_OE_N (42.2,42.75→47.9)→(41.9,48.2)→(37.9,48.2)→(37.6,47.9)→via (37.6,47.4), B DO4_CTRL (33.26,45.35)→(40.5,45.35)→via (40.975,47.45), LOGIC_GND via (36.55,47.5) (sole Q1.2 ground; would be islanded), DO4_DRIVE via (34.02,46.0)
LIVE PCB MODIFIED: NO
WAITING FOR HUMAN REVIEW

(Information: AIRWIRES CLASSIFIED 6/6; ELECTRICAL DRC 0; C5L PROTECTED COPPER UNCHANGED; ISOLATION PRESERVED; ANTENNA KEEPOUT PRESERVED; PERSISTENT BACKUP VERIFIED; C6A PLAN for the 4 additive nets READY, expected post-C6A unconnected 2.)
