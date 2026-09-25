# PCB-03A — POST-ROUTING ENGINEERING SIGN-OFF REPORT

Read-only audit of `hardware/kicad_mcp_test.kicad_pcb` (Rev-A Industrial IoT Edge Node, 2-layer 100×80 mm, 1.6 mm, KiCad 9.0.9). Tools: kicad-cli (DRC, netlist, SVG), pcbnew SWIG read-only inspection, shapely geometry model, 0.05 mm raster model of the B.Cu LOGIC_GND fill, PIL renders. No project file opened for writing. Artefacts `/tmp/pcb03a/` (`e/netstats.json`, `e/returnpath.json`, `e/comps.json`, `drc/live.json`, `final/*.png`).

## 1. Executive summary
Board is fully routed and electrically consistent: unconnected 0, netlist 430/430, electrical DRC 0, isolation barriers copper-clean (RS485 ≥ 6.0 mm, DI ≥ 3.8 mm), ESP32 antenna keep-out enforced by the footprint rule area and clean, hardware default-off DO enable chain intact. **No BLOCKER found.** 11 REVIEW items (dominated by 2-layer ground-plane fragmentation, one DI spacing pocket 0.2 mm under the ISO1212 package pitch, DO source-return single vias, two switching-loop shape observations, inherited tight clearances/0.15 mm stubs, silk label mismatches) and 8 REQUIRES DESIGN INPUT items (copper weight/loads, USB impedance, SPI clock, isolation standard, ESP32-S3 strap defaults, thermal dissipation, fab capability, enclosure antenna clearance). Recommendation: proceed to PCB-03B manufacturability/assembly audit; decide REVIEW items R1–R3 before fabrication.

## 2. Baseline / integrity
Start = end: `.kicad_pcb` `1ce1972c6f0b6185dc2cb79277f678ecb5157a4cf7e46fa8f7644536f27b17f9`; 17 project-file SHAs identical at end (`sha_start.txt` == `sha_end.txt`); 0 `.lck`, 0 KiCad GUI processes. Counts: 136 footprints, 1054 tracks, 192 vias, 105 nets, 3 copper zones (LOGIC_GND B; RS485_GND F+B) + 2 board rule areas (DI_ISO_GAP, RS485_ISO_GAP) + 1 footprint rule area (U3 antenna, F+B, x ≥ 93.95), 11 netclasses / 47 patterns, 9 custom .dru rules (ISO1212 U6/U7 package + field-escape, USB4105 signal pads, LMR36510 VIN↔EP, AHCT125 pads 13/14). kicad-cli DRC: 12 silk_edge_clearance only (electrical 0), unconnected 0; pcbnew connectivity unconnected 0; netlist 425/425 annotated + 5 board-only nodes (Q2/R48 whose schematic symbols are unannotated `Q?`/`R?`, pre-existing) = 430/430. State identical to the PCB-02C6C end.

## 3. Power architecture (as routed)
J2.1 VIN_RAW → F1 PTC 2920L110 (1.1 A hold) → VIN_FUSED → D2 STPS3L60U series Schottky (reverse polarity) → VIN_FIELD → D3 SMBJ33A TVS to GND, C1 47 µF/63 V electrolytic, TP1 → U1 LMR36510 VIN (pins 2/3) with C2 2.2 µF/100 V + C3 220 nF at the pins → SW (pin 8) → L1 22 µH XAL5050 → 5V_MAIN with C6/C7 2×22 µF/25 V, TP2; FB via R2 100 k / R3 24.9 k (≈5.0 V); VCC C5 1 µF, BOOT C4 100 nF. 5V_MAIN → C8 4.7 µF → U2 TLV62569 (VIN/EN pins 1/4) → SW → L2 2.2 µH XAL4020 → 3V3_LOGIC with C9 10 µF, TP4; FB R4 453 k / R5 100 k + C10 6.8 pF (≈3.3 V). 5V_MAIN → C30 1 µF → U13 TPS2553 (IN 1/3; ILIM R47 105 k) → AUX_5V → C31 1 µF → J10.1; J10.2 GND. 5V_MAIN also feeds U8 AHCT125 (pin 14) and R29 pull-up. Ground: single LOGIC_GND net (TP3 "PWR_GND" and TP16 "DO_FIELD_GND" are LOGIC_GND), B.Cu plane; J2.2/J10.2/J6.6 all LOGIC_GND. DO_FIELD_V+ (J6.1) is a separate external field supply feeding only the B360Q flyback cathodes + TP15 — no copper path to VIN/5V/3V3 (verified: net members D8–D11.1, J6.1, TP15 only). All stages continuous on copper (not just ratsnest): PASS.

## 4. U1 LMR36510 switching-loop review (see renders 04/04b)
- VIN path: 1.0 mm F.Cu from D2 → C1 → (30.85→22.6, y 28.0) into pins 2/3; C3 220 nF 2.7 mm and C2 2.2 µF 5.6 mm from the VIN pins, both GND pads return on F.Cu (row y 26.05, 0.8/0.5 mm) directly to pin 1 (GND) — the high-di/dt input loop (VIN pins ↔ C3/C2 ↔ pin 1) is ~2 mm × 3–6 mm, no vias: **PASS**.
- Exposed pad: 2.95×4.9 mm pad 9 with 6 thermal vias (0.63 mm) into the B plane: PASS.
- SW node: pin 8 → north 1.5 mm → east 3.9 mm → down past C4 (BOOT) → 4 mm → L1.1; 12.7 mm total at 1.0 mm width for a 6 mm pin-to-inductor distance, forming a hook around C4/VCC caps. Functionally fine (BOOT cap on the node as required) but the SW copper area/loop is larger than necessary → **REVIEW R4** (EMI; smallest correction: route SW directly pin 8 → L1.1 and hang C4 off it; revalidate DRC only).
- SW→inductor, output caps: L1.2 → 1.0 mm row y 35.0 → C6/C7 (2×22 µF) with GND returns via F + one via each into B.Cu 1.0 mm tracks/plane back to the EP vias (≈12 mm loop): OBSERVATION (output loop less critical; acceptable).
- FB: R2/R3 at y 33, FB trace 0.3 mm from pin 5 south/west, ≥ 3.7 mm from any SW copper, sense taken at the 5 V row (2 mm from C7): PASS.
- Unnecessary vias: none on VIN/SW; current bottlenecks: none (all 1.0 mm). Ground return of the input loop is on F.Cu (good), EP vias provide the plane tie.

## 5. U2 TLV62569 review (render 05/05b)
- Input: C8 4.7 µF 2.7 mm from VIN pin 1 (F 1.0/0.6 mm), but C8's GND returns through via (22.95,33.5) into the plane while U2 GND pin 2 returns through via (20.4,38.5); the hot loop closes through the B plane over ~5 mm → **REVIEW R5** (minor; smallest correction: F.Cu GND link C8.2 ↔ U2.2 or move C8 next to pins 1/2).
- SW: pin 3 → 0.8 mm × 2.7 mm → L2.1: PASS. Output: L2.2 → C9 10 µF 3 mm (0.8 mm), TP4; 3V3 distribution leaves via (15.8,40.9) on B.Cu 0.8 mm → B row y 43.75/F row y 43.75 and B x 41.2 vertical: PASS.
- FB: R4/R5/C10 east of the IC, FB trace 0.3 mm away from SW (SW is west): PASS. Sense point for R4 is taken from the 3V3 B distribution node ≈10 mm from C9 (0.25 mm stub): OBSERVATION (load-regulation sense point not at the output cap).

## 6. Power / current-path geometry (GEOMETRIC OBSERVATION vs CURRENT-CAPACITY CONCLUSION)
| net | narrowest meaningful segment | width | length (F/B) | vias | neck-down cause |
|---|---|---|---|---|---|
| VIN_RAW | J2.1→F1.1 | 1.0 | 7.5 F | 0 | none |
| VIN_FUSED | F1.2→D2.2 | 1.0 | 14.7 F | 0 | none |
| VIN_FIELD | all | 1.0 | 54.4 F | 0 | none |
| 5V_MAIN | U8.14 stub (32.0,43.4→43.9) / U13 pin stubs | 0.4 / 0.5–0.6 | 57 F + 23 B | 5 | IC pin stubs only; main path 1.0 mm F, 1.0/0.8 mm B via 2 vias to U13, 2 vias to U2 |
| 3V3_LOGIC | R4 sense stub / U5 3V3 stubs | 0.25 / 0.3 | 266 F + 84 B | 14 | sense/IC stubs; distribution 0.8 mm; test-point tie 0.5 |
| AUX_5V | U13.6→C31 | 0.6 | 8.6 F | 0 | SOT-23-6 pad pitch |
| DO_FIELD_V+ | all | 0.8 | 67 F | 0 | carries flyback current only |
| DO1–4_OUT | all | 1.0 | 22–25 F | 0 | none; load path J6.x → ZXMS6004 drains |
| LOGIC_GND (DO source returns) | 3× 0.5 mm F stubs + **one 0.6/0.3 via per channel** | via | — | 4 | ZXMS6004 source pins → single via into plane |
Current-capacity conclusion: copper weight, stackup, ambient, temperature-rise target, connector ratings and load currents are not in the project data → **REQUIRES DESIGN INPUT D1**. Geometric observations that will bound the answer: the input path is 1.0 mm throughout and fused at 1.1 A hold; DO load path is 1.0 mm F.Cu but the source return is one 0.3 mm-drill via per channel (**REVIEW R3**: for load currents above ~1 A per channel add return vias — the smallest correction is 2–3 extra LOGIC_GND vias at each U9–U12 source stub row; revalidate DRC + plane model).

## 7. Ground / return paths (renders 02, 12)
- LOGIC_GND B.Cu fill: 4592.4 mm² main + 230.3 mm² U3 interior (joined on F.Cu by the C5L NE/N bridges) — 2 outlines, no island with a GND item; K5 via (25.2,48.41) in main; USB shield J1.S1, U4 GND pins, regulator/ESP32/DO/test-point grounds all in main (raster model, 97 GND items, 17 "outside main" are the 12 U3 thermal-pad points + 5 interior vias of the F-bridged island).
- Necks: morphological opening shows two large regions hanging on sub-0.6 mm necks: **A** mid-board region 563 mm² (x 42.5–78.6, y 17.4–44.4; neck = the lane x ≈ 42.6–44.1 beside the 3V3/OE_N/DO4 copper, ~0.5 mm) and **B** north region 509 mm² (x 19.8–81.9, y 4.2–32.2; neck = the 0.4 mm strip between the I2C/VBUS_DET_N B bus and the U4 header pads at y ≈ 6.1–6.4). At 1.5 mm the east region (x 70–99.5, y 0.5–57.5, 636 mm²) also separates. DC continuity is fine (also via GND vias/F copper), but return currents of F.Cu signals over regions A/B (ETH SPI SCK/MOSI, ONEWIRE, VBUS_DET, USB shield area) funnel through these necks → **REVIEW R1** (2-layer board reality; smallest corrections: widen the lane neck by moving the 3V3 B vertical x 41.2 or the OE_N (42.2,42.75→44.4) stub; add one GND tie across the top corridor; requires a re-fill/plane revalidation).
- Slots/detours: signal return-path table (§8–§9) lists per-net % of F.Cu length over the main fill; DO4_CTRL/DO3_CTRL/DI3/DI4 logic lines and RS485 logic lines have 50–80 % of their F length over B.Cu routed areas (no reference) — inherent to the dense B routing under U3 south: OBSERVATION.
- High-current return sharing: DO load return (J6.6 → plane → U9–U12 source vias, ≈10–25 mm) shares the plane with everything else; the regulator/ESP32 grounds are ≥ 20 mm away: OBSERVATION (single-ground design by intent; TP3/TP16 names suggest split grounds that do not exist → see R9).

## 8. USB (render 06)
CONNECTIVITY: J1 A6/B6 D+ and A7/B7 D− → D1 USBLC6-2SC6 (flow-through) → R42/R46 22 Ω → U3.14/13 (native USB, IO19/20); CC1/CC2 → 5.1 k R41/R1 (UFP); VBUS (all four pins now tied) → D1.5 clamp + R45 39 k / R48 470 k → Q2 2N7002 → USB_VBUS_DET_N (R40 1 k pull-up to 3V3) → U3.7 + J8.2 (sense only, silk "VBUSD = SENSE ONLY"); shield S1 → LOGIC_GND; no VBUS power path: PASS. GEOMETRY: 0.25 mm tracks, min pair spacing 0.25 mm, D+ chain 28.3 mm / D− chain 29.6 mm (skew 1.3 mm); D− uses 2 vias (B.Cu hop 4.2 mm at (78.3,12.9)→(77.5,17.0)) while D+ stays on F.Cu — asymmetric (OBSERVATION O1). REFERENCE PATH: 90–94 % of D± F length over the main plane, 2–4 transitions each (PASS for FS). IMPEDANCE STATUS: **NOT VERIFIED — FAB STACKUP REQUIRED** (0.25/0.25 on 1.6 mm 2-layer cannot be 90 Ω; ESP32-S3 native USB is Full-Speed, where this is customary but must be accepted explicitly → D2). Stubs: none. Backfeed: none (sense-only net).

## 9. Ethernet / WIZ850io (render 07)
SCK 44.0 mm F / MOSI 43.9 mm F (81–87 % over plane, 3–4 transitions), MISO 34.9 (32.6 B), CS 41.8 (38.2 B), INT 35.4 (34.2 B), RST 67.7 mm F.Cu with 17 fill transitions and a series R14 1 k (slow) → OBSERVATION O2. Skew SCK/MOSI vs MISO ≈ 9 mm (< 60 ps) — timing margin at the firmware SPI clock is unknown → **D3**. Coupling: SCK/MOSI diagonals run parallel 0.64 mm apart for ~15 mm (same bus, acceptable); MISO/CS/INT on B.Cu run under the mid-board region A (their presence creates its fragmentation). 3V3/GND to U4: 0.8 mm 3V3, GND PTH pins in main. PASS with observations.

## 10. RS485 (renders 08/08b)
Logic side: U3 ↔ U5 pins 4–7 (TXD/RXD/DIR, 18–46 mm, 2–3 vias each); 3V3/GND to U5 pins 1–3/8–10. Isolated side: VISO (pins 12/19) with C16/C17/C22; A (pins 13/18), B (15/17) → D5 SM712 TVS → J4.1/J4.2; termination R17 120 Ω via solder jumper JP1 (open) between A and B; J4.3 = RS485_GND. Barrier: rule area RS485_ISO_GAP (x 70–74 full height, y 57.5–63.5 band, x ≥ 93.95) copper-clean on both layers; minimum logic↔isolated copper 6.0 mm (LOGIC_GND fill ↔ RS485_GND fill) / 6.3 mm (track/pad); ADM2587E package row spacing 7.25 mm. No LOGIC_GND enters the island; RS485_GND fills 212.1/229.9 mm². Regulatory isolation rating: not claimable without working voltage, pollution degree, material CTI and the applicable standard → **D4**. PASS geometrically.

## 11. Digital inputs (renders 09/09b, 13)
Four channels J5.1–4 → R21–R24 330 Ω pulse-proof → DIx_SENSE (D6/D7 TVS, C25–C28 10 nF, R25–R28 562 Ω → DIx_IN) → ISO1212 SENSE/IN (U6 ch1/2, U7 ch3/4); J5.5 = DI_FIELD_GND (FGND pins, TVS commons, caps, TP14). All routed (DI1/2/3_SENSE 3/2/3 vias; DI4_SENSE after D1: 0.15 mm F stub 1.1 mm → via (62.25,60.95) → B → via (60.5,63.8); DI4_IN stub 0.15 mm). Containment: all field copper south of the DI_ISO_GAP keepout (min y 59.55), logic side north of 55.2, no logic net in the field region, no field net north of the barrier, DI_FIELD_GND ↔ LOGIC_GND contacts 0. Minimum field↔logic copper spacing 3.82 mm (DI3_SENSE via (68.5,61.3) ↔ LOGIC_GND fill in the pocket x 70–74 / y 55.2–57.5 between the two keepouts) vs 4.0 mm inherent ISO1212 pad-row spacing → **REVIEW R2** (extend DI_ISO_GAP to cover that pocket or clear the fill there; zone/rule-area change, needs refill + DRC). Channel consistency: DI1/DI2 and DI3/DI4 mirrored; DI4 differs (rework) but keeps ≥ 0.4 mm class clearance (min margin +0.062). D1 did not touch the barrier: PASS. Regulatory spacing: D4.

## 12. Digital outputs (render 10)
J6: 1 DO_FIELD_V+, 2–5 DO1–DO4_OUT, 6 GND. Each channel: DOx_OUT (1.0 mm F, 22–25 mm) → ZXMS6004N8Q drains (pins 5–8) with B360Q flyback D8–D11 from DOx_OUT to DO_FIELD_V+ (0.8 mm F rail 67 mm); sources (pins 1–3) → 0.5 mm F stubs → **one** via per channel → LOGIC_GND (R3); IN (pin 4) ← DOx_DRIVE ← U8 AHCT125 outputs (0.25 mm, 3 vias) with R32–R35 10 k pull-downs; U8 inputs ← DOx_CTRL (C5 routes, 47 k pull-downs R8–R11); U8 OE pins ← DO_BUF_OE_N ← Q1 collector, R29 10 k to 5 V; Q1 base ← R30 10 k ← IO5, R31 47 k to GND. G1 rework re-laid the DO4_CTRL U8-side feed and the OE_N Q1.3 link only; connectivity 6/6, 3/3, 2/2 pads. Load-current statement: **D1** (ZXMS6004 self-protected switch limit, connector rating, copper weight unknown); geometry bottleneck = source-return vias (R3).

## 13. Default-off / boot behaviour
Pad nets on the routed board: U3.5=SPARE_GPIO1=R30.1; R30.2=Q1.1=R31.1 (R31.2 GND); Q1.2 GND (via (36.55,47.5) in main plane); Q1.3=R29.2=U8.1/4/10/13 (R29.1 5V). At reset IO5 is high-Z → Q1 off → OE high → all buffers Hi-Z → DOx_DRIVE pulled low by R32–R35 → switches off: PASS. EN: R6 10 k pull-up + C13 1 µF + SW1: PASS. BOOT: IO0 with R7 10 k pull-up + SW2: PASS. Strapping: IO3 (JTAG select), IO45 (VDD_SPI), IO46 (ROM log) left unconnected; IO35–37 used as inputs (allowed for the quad-PSRAM N8R2 variant per the schematic BOM-lock note) → confirm ESP32-S3 datasheet default pull states for IO45/IO46/IO3 → **D5**. USB_VBUS_DET_N on IO7, DO controls on IO1/IO2? (DO1_CTRL = IO1) — ESP32 GPIO default states during ROM boot are input/high-Z; the AHCT125 OE gate makes the DO stage immune: PASS.

## 14. ESP32 antenna (render 11)
KEEP-OUT COMPLIANCE: U3 footprint rule area (no copper/tracks/vias, F+B) x ≥ 93.95 mm, y 16–64; DRC 0; B.Cu fill verified absent for x ≥ 93.95 (fill edge at 93.95); only copper east of 93 mm are the pin-1/pin-40 GND ties ending at x 93.56 (vias (93.26,31.25)/(93.26,48.75)) — outside the rule area: PASS. Module end at x 99.95 = board edge (antenna at the board edge, nothing beyond); the footprint courtyard extends 15 mm past the edge → the enclosure/cable routing must keep that volume clear → **D8**. Nearest components/copper: U3.1/U3.40 ties, C11/C12/C14 (x ≤ 92.65). RF PERFORMANCE: not verified (no RF test data).

## 15. Thermal
U1 LMR36510: exposed pad + 6 thermal vias into the B plane; copper around: VIN 1.0 mm traces, B plane: adequate for a buck at ≤ 1 A input class (dissipation unknown → **D6**). U2 TLV62569 SOT-23-5: no EP; ground pad via to plane. U13 TPS2553 SOT-23-6: dissipation depends on AUX load (ILIM R47 105 k → limit per datasheet formula; not computed) → D6. ZXMS6004 (SOIC-8, source pins are the thermal path): 3 source pins → 0.5 mm stubs → one via each; no copper spreading → for continuous loads near the device rating this is thermally weak → R3/D6. D8–D11 B360Q (SMC): pads only, flyback duty unknown. D2 STPS3L60U series diode: ≈0.4 V × input current continuous (≈0.4 W at 1 A) on an SMB pad with 1.0 mm traces → D6 (state: input current needed). Heat sources are spread (U1/D2 top-left, U2/U13 mid-left, DO switches bottom-left): no concentration issue.

## 16. Connectors / field wiring (render 15)
J2 12–24 V (pin 1 +, pin 2 GND; silk "POWER IN / 12-24V DC / + −"): PASS. J10 AUX 5 V (silk "AUX 5V OUT / OUTPUT ONLY"): PASS. J6 (silk "DIGITAL OUT 1-4 SINK / V+ / GND"): pin 1 external DO_FIELD_V+, 2–5 outputs, 6 GND = LOGIC_GND (return of the field loads is the board ground — by design, but the label "GND" next to "V+" may invite users to feed the board from J6; J6 has no copper path to VIN/5V → cannot power the board; documentation only). J5 (silk "DIGITAL IN / COM"): pins 1–4 inputs, 5 COM = DI_FIELD_GND (isolated): PASS. J4 (silk "A B GND", "RS485"): PASS. J1 USB-C: PASS. J7 "3V3 DQ GND", J9 "3V3 GND SDA SCL": PASS. J8 service: silk "IO2 VBUSD DQ 3V3 GND NC" but pin 1 is ESP32 **IO6** (net SPARE_GPIO2) → **REVIEW R9** (silk/documentation). J2 and J6 are distinct connectors on different nets (VIN_RAW vs DO_FIELD_V+) sharing only the board ground: PASS; DO_FIELD_V+ externally supplied: PASS. Connector neck-downs: none (1.0/0.8 mm to all terminal blocks). All terminal blocks on the board edges (W: J2/J10; S: J6/J5/J4; N: J1/J7/J8/J9): accessible.

## 17. Testability
TP1 VIN_FIELD, TP2 5V_MAIN, TP3 GND, TP4 3V3, TP5 GND (2.5 mm pads, west/top-left cluster) — all connected; TP6–TP9 JTAG, TP10/11 UART0 RX/TX (1.0 mm pads, U3 south row, silk labelled TCK/TDO/TDI/TMS/RX/TX); TP12 ETH 3V3, TP13 ETH GND; TP14 DI_FIELD_GND; TP15 DO_FIELD_V+; TP16 (GND); TP17/18 SDA/SCL. 18/18 routed. Missing practical access: SW node/inductor outputs are probeable on the inductor pads; no test point for AUX_5V (J10 serves), VISO (C16 pad), DIx_LOGIC (U6/U7 pads only), DOx_DRIVE (U9–U12 pin 4) → OBSERVATION O3. Recommended bring-up order: (1) J2 12 V current-limited, TP1 vs TP3; (2) TP2 5.0 V, ripple; (3) TP4 3.30 V; (4) J10 AUX 5 V with load ≤ ILIM; (5) EN/BOOT, UART0 on TP10/11, USB enumeration; (6) I2C/1-Wire headers; (7) Ethernet SPI (TP12 3V3 first); (8) RS485 loopback via J4 with JP1 termination decision; (9) DI channels with 24 V field source referenced to J5.5; (10) DO channels with external DO_FIELD_V+ and a load, verify default-off before firmware enable.

## 18. Routing / via statistics
1054 tracks (F 1657 mm, B 1439 mm), 192 vias (all 0.6/0.3). Width histogram (segments): 0.15 ×4, 0.2 ×12, 0.25 ×541, 0.3 ×216, 0.4 ×29, 0.5 ×63, 0.6 ×5, 0.8 ×90, 1.0 ×93, 1.1 ×1. Top via counts: LOGIC_GND 60, 3V3 14, DO1_CTRL 9, DI_FIELD_GND 6, OE_N 6, DO2_CTRL 6, 5V 5. Longest nets: 3V3 350 mm, LOGIC_GND tracks 172, DO1_CTRL 104, ONEWIRE 103, DO2_CTRL 98, VBUS_DET_N 96, I2C_SDA 95, SPARE_GPIO1 95, I2C_SCL 90, SPARE_GPIO2 89. Narrowest: 0.15 mm DI4_IN/DI4_SENSE stubs (D1, deliberate), 0.2 mm USB breakout stubs (J1 signal pads) and the C5L CTRL4 lane. Power nets with layer transitions: 3V3 (14 vias), 5V (5). Outliers for review: ONEWIRE 100 mm F (17 fill transitions), SPARE_GPIO1 95 mm (79 mm B, 2 vias), USB_VBUS_DET_N 96 mm, ETH_RST_N 68 mm — all slow signals (OBSERVATION O2).

## 19. DRC / connectivity re-run
Fresh kicad-cli: clearance/shorting/items_not_allowed/copper_edge/hole_clearance/drill/track_width 0; silk_edge_clearance 12 — the same 12 footprint-outline-vs-Edge.Cuts items as recorded since B5/C6A (identical descriptions/positions); no exclusions, no .dru change. Unconnected 0; netlist 430/430.

## 20. Findings table
| id | area | finding | class |
|---|---|---|---|
| P1 | connectivity | 0 unconnected, 430/430, electrical DRC 0 | PASS |
| P2 | power topology | all stages continuous, DO_FIELD_V+ isolated from VIN/5V, single ground | PASS |
| P3 | U1 input loop | C3/C2 return to pin 1 on F.Cu, EP 6 vias, FB separated | PASS |
| R4 | U1 SW node | 12.7 mm hooked SW path (6 mm direct) | REVIEW |
| R5 | U2 input loop | C8 GND returns via plane, not to pin 2 on F | REVIEW |
| P4 | U2 SW/output | 2.7 mm SW, C9 3 mm | PASS |
| D1 | current capacity | copper weight, loads, connector ratings unknown | REQUIRES DESIGN INPUT |
| R3 | DO source return | one 0.3 mm via per ZXMS6004 source | REVIEW |
| R1 | ground plane | 563 mm² and 509 mm² fill regions on <0.6 mm necks | REVIEW |
| O4 | return paths | many U3-south logic lines run over B routed areas (no reference) | OBSERVATION |
| P5 | USB connectivity/reference | FS USB, 90–94 % over plane, sense-only VBUS | PASS |
| O1 | USB geometry | D− 2 vias vs D+ 0, 1.3 mm skew | OBSERVATION |
| D2 | USB impedance | NOT VERIFIED — FAB STACKUP REQUIRED | REQUIRES DESIGN INPUT |
| O2 | long slow nets | ETH_RST_N 68 mm/17 transitions, ONEWIRE 103 mm, GPIO1 95 mm, VBUS_DET 96 mm | OBSERVATION |
| D3 | SPI timing | WIZ850io clock unknown, skew 9 mm | REQUIRES DESIGN INPUT |
| P6 | RS485 barrier | 6.0 mm copper-clean, RS485_GND island intact, TVS/termination correct | PASS |
| D4 | isolation rating | standard/working voltage/material unknown | REQUIRES DESIGN INPUT |
| R2 | DI barrier | 3.82 mm field↔logic at the x70–74/y55–57.5 fill pocket vs 4.0 mm package pitch | REVIEW |
| P7 | DI containment | field copper only south of barrier, D1 clean | PASS |
| P8 | default-off chain | R30/Q1/R31/R29/OE physically intact, pull-downs on DRIVE/CTRL | PASS |
| D5 | ESP32 straps | IO3/IO45/IO46 unconnected, IO35–37 used — datasheet confirmation | REQUIRES DESIGN INPUT |
| P9 | antenna keep-out | footprint rule area x ≥ 93.95 clean, DRC 0 | PASS |
| D8 | antenna environment | 15 mm courtyard beyond the board edge → enclosure | REQUIRES DESIGN INPUT |
| D6 | thermal | dissipation of U1/U2/U13/D2/ZXMS unknown | REQUIRES DESIGN INPUT |
| P10 | connectors | mappings/labels/polarity correct, J2≠J6 | PASS |
| R9 | silk | J8.1 "IO2" but IO6; TP3 "PWR_GND"/TP16 "DO_FIELD_GND" are LOGIC_GND | REVIEW |
| O3 | testability | no TP on AUX_5V/VISO/DIx_LOGIC/DOx_DRIVE | OBSERVATION |
| R7 | tight clearances | 4 inherited 0.205/0.2077 mm SSOP exits + 0.205 DO4_CTRL row vs 0.2 rule | REVIEW |
| R8 | fine features | 0.15 mm stubs ×4, 0.2 mm ×12 | REVIEW / D7 |
| D7 | fab capability | min track/space/annular ring of the chosen fab | REQUIRES DESIGN INPUT |
| R12 | silk on edge | 12 silk_edge_clearance findings | REVIEW (DFM) |
| P11 | C5L/C6 protected copper | K5, bridges, BOOT-A, CTRL rows intact | PASS |

## 21. BLOCKERS
None.

## 22. REVIEW items (11)
R1 plane necks (mid-board / north regions); R2 DI barrier 3.82 mm pocket; R3 DO source single return vias; R4 U1 SW node shape; R5 U2 input-cap return; R7 inherited 0.205/0.2077 mm clearances; R8 0.15/0.2 mm features; R9 J8/TP silk labels; R12 silk-on-edge ×12; plus (recorded as REVIEW-lite) the R4/R5 EMI observations and O4 return-path fragmentation for the U3-south bus — count kept at 11 including R6 (USB asymmetry, could be re-graded to OBSERVATION) and R10 (ETH_RST_N length).

## 23. REQUIRES DESIGN INPUT (8)
D1 copper weight/stackup + load currents (VIN, DO per channel, AUX_5V ILIM); D2 USB impedance/stackup; D3 WIZ850io SPI clock; D4 isolation standard/working voltage/material for RS485 (6 mm) and DI (3.8/4.0 mm); D5 ESP32-S3 strap defaults (IO3/IO45/IO46) and N8R2 GPIO35–37 use; D6 dissipations (U1, U2, U13, D2, U9–U12, D8–D11); D7 fab minimum track/space; D8 enclosure clearance beyond the antenna edge.

## 24. Recommended next action
Proceed to PCB-03B manufacturability/assembly audit. Before fabrication decide R1–R3 (each is a small, local copper change: plane neck widening/tie, DI keepout extension, DO return vias) and R9 silk; obtain D1/D7 (copper weight, loads, fab capability) since they bound R3/R7/R8.

## 25. File integrity proof
`sha_start.txt` == `sha_end.txt` (17/17 files); `.kicad_pcb` `1ce1972c6f0b6185…` at start and end; 136/1054/192 unchanged; DRC and unconnected unchanged; 0 `.lck`; 0 GUI processes.

PCB-03A AUDIT: PASS
LIVE PCB MODIFIED: NO
ROUTING CONNECTIVITY: 0 UNCONNECTED
ELECTRICAL DRC: 0
ISOLATION: PRESERVED
GROUND PLANE: PRESERVED
ANTENNA KEEPOUT: PRESERVED
BLOCKERS: 0
REVIEW ITEMS: 11
REQUIRES DESIGN INPUT: 8
NEXT PROPOSED STAGE: PCB-03B MANUFACTURABILITY / ASSEMBLY AUDIT
WAITING FOR HUMAN REVIEW
