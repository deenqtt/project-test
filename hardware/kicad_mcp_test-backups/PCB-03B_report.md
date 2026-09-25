# PCB-03B — MANUFACTURABILITY & ASSEMBLY AUDIT (READ-ONLY)

Board: Rev-A Industrial IoT Edge Node — `hardware/kicad_mcp_test.kicad_pcb` (KiCad 9.0.9, 2-layer, 100 × 80 × 1.6 mm)
Date: 2026-09-22 · Stage type: read-only audit · PCB WRITES PERFORMED: **NONE**
Live SHA-256 at start and end: `35a86ad07cf60335912c68746bbf8a2061c7b5df18de864f93c5d0d416d57d35` (== PCB-03A2 end state)
Evidence dir: `/tmp/pcb03b/` (drc/, e/, final/). Renders: `/tmp/pcb03b/final/01…13_*.png` (analytic overlays on kicad-cli SVG exports, watermark "PCB-03B READ-ONLY AUDIT").

Method: kicad-cli DRC (severity-all, all-track-errors), pcbnew SWIG read-only inspection (LoadBoard only — no Save), shapely geometry on the 2127-item live geometry snapshot (`e/geo.pkl`, identical to the 03A2 post-write snapshot), schematic kicadxml netlist. No MCP writer, no KiCad GUI, no disposable copy was needed for any measurement in this stage.

Classification vocabulary (as mandated): **BLOCKER** · **CORRECT BEFORE RELEASE** · **REVIEW** · **DESIGN INPUT** (REQUIRES DESIGN / MECHANICAL / FAB INPUT) · **ACCEPT AS-IS**.

---

## 1. Baseline integrity

| Check | Result |
|---|---|
| `.kicad_pcb` SHA-256 | `35a86ad07cf60335…d57d35` — identical to PCB-03A2 end state, unchanged at end of stage |
| `.kicad_pro` / `.kicad_dru` | `21fd0939…` / `c511468f…` — unchanged since PCB-02C6 baseline |
| Hardware tree file inventory | 17 files, same set and hashes as PCB-03A2 end |
| Board content | 136 footprints / 1054 tracks / 192 vias / 105 nets / 4 zones (LOGIC_GND B ×2 fills, RS485_GND F, RS485_GND B) / 3 rule areas + U3 footprint keepout |
| Unconnected (pcbnew connectivity) | **0** |
| DRC (electrical: clearance, shorts, tracks, vias, courtyards, mask, silk-over-copper, hole-hole, edge) | **0** |
| DRC total | 12 — all `silk_edge_clearance` (see §10) |
| Netlist consistency | 425/425 schematic nets present on board + 5 board-only nets (Q2/R48 unannotated in schematic, known from 03A) |
| Writers / locks during stage | 0 KiCad GUI, 0 MCP server processes, 0 `.lck` — verified at start and end |

Baseline: **PASS**.

---

## 2. Fabrication capability audit

### 2A. Package-internal minima (J1 USB-C receptacle only; not routed by us)

| Feature | Value | Governing rule | Class |
|---|---|---|---|
| J1 signal-pad copper gap (A5↔A6, B5↔B6 etc.) | 0.198 mm | `.dru` J1 pad-pad exception 0.20 (0.1983 measured, tolerance-covered) | TIGHT BUT COMMON — inherent to any 16/24-pin USB-C receptacle |
| J1 mask web between signal pads | 0.198 mm (mask expansion 0) | — | TIGHT BUT COMMON; most fabs accept ≥0.10 mm web on green LPI, confirm for other colours |
| J1 shield PTH slots/holes | 4 × Ø0.60 PTH (through pads) + pad annulus 0.15 | board min annular 0.10 | COMMON FAB |

### 2B. Custom-rule minima (`.kicad_dru` rules)

| Rule | Value | Where exercised | Class |
|---|---|---|---|
| ISO1212 courtyard exception 0.25 mm | U7/U8 (DI) | pin-row clearance to nearby fills | COMMON FAB |
| J1 pad exception 0.20 mm | J1 only | as above | TIGHT BUT COMMON |
| Rule areas RS485_ISO_GAP / DI_ISO_GAP / DI_ISO_GAP_EXT | copper-free bands | isolation barriers (§8) | no fab impact (absence of copper) |

### 2C. Normal routing minima (our routing)

| Feature | Minimum | Count at minimum | Location | Class |
|---|---|---|---|---|
| Track width | **0.15 mm** | 4 segments | DI4_IN / DI4_SENSE stubs at U7.12 corner (62.05,59.6–60.95), authorised in C6C | TIGHT BUT COMMON (6 mil) |
| Track width | 0.20 mm | 12 segments | DO_CTRL4 lane, USB D+/D− breakout at J1 | COMMON FAB (8 mil) |
| Track width (bulk) | 0.25–1.0 mm | rest | — | COMMON FAB |
| Copper-copper clearance (pad-track) | **0.200 mm** | 1 | R45.2 ↔ ETH_RST_N | COMMON FAB (8 mil) |
| Clearance (track-track) | 0.200 mm | 1 | ETH_MOSI ↔ ETH_SCK | COMMON FAB |
| Clearance (track-via) | 0.205 mm | 2 | BOOT_N ↔ TXD via; OE_N via ↔ DO4_CTRL row | COMMON FAB |
| Clearance (pad-via) | 0.208 mm | 1 | — | COMMON FAB |
| Clearance (via-via) | 0.228 mm | 1 | — | COMMON FAB |
| Zone-to-copper clearance | 0.30 mm | zones | zone setting | COMMON FAB |
| Via | 0.60 / 0.30 mm, annular 0.15 mm | 192 (all identical) | — | COMMON FAB (0.3 mm drill is standard; 0.15 annular ≥ typical 0.125) |
| Smallest PTH drill | **0.20 mm** | U3 thermal pad vias (Ø0.60 pad) | ESP32 module EP array (library footprint) | ADVANCED-CONFIRM WITH FAB — 0.20 mm mechanical drill / 0.2 mm annular ring; many pooled 2-layer services quote min 0.30 mm |
| Next smallest PTH drill | 0.33 mm (Ø0.63 pad) ×6 | U1 exposed pad | COMMON FAB (annular 0.15) |
| Other PTH drills | 0.6 / 0.8 / 0.914 / 1.0 / 1.1 / 1.3 mm | connectors, TPs, Q2 | COMMON FAB |
| NPTH | 3.2 mm ×4 | H1–H4 | COMMON FAB |
| Hole-to-hole (wall-to-wall) | 0.276 mm | GND vias near U5 | COMMON FAB (≥0.25 typical) |
| Copper-to-board-edge | 0.50 mm | rule minimum, honoured | COMMON FAB |
| Hole-to-edge (PTH) | 2.30 mm (J1 shield) | — | COMMON FAB |
| Solder-mask expansion | 0 (board setup) | global | see §4 — mask web = copper gap; SSOP web 0.235, 0402 pads ≥0.5 | COMMON FAB; **confirm fab's mask-sliver minimum (typ. 0.10 mm) with 0 expansion** |
| Silk line widths | 0.12 / 0.15 / 0.20 mm | 393 graphics | 0.12 mm is at the common minimum (0.10–0.15) | TIGHT BUT COMMON |
| Silk text | min height 0.8 mm / stroke 0.12 mm | 150 texts | — | TIGHT BUT COMMON |

Fabrication classification summary: no PROBLEMATIC feature. One ADVANCED-CONFIRM item (0.20 mm drills in the ESP32 module thermal array). Two TIGHT BUT COMMON items (0.15 mm stubs ×4, 0.12 mm silk).

**Findings**
- F-1 **DESIGN INPUT (FAB)** — 0.20 mm thermal-via drills under U3 (library footprint). If the target fab's minimum drill is 0.30 mm, the footprint (not routing) must change; if 0.20 mm accepted, cost/"advanced" class may apply. Fab quote required before release.
- F-2 **REVIEW** — 0.15 mm ×4 stubs (DI4). Acceptable at 6 mil fabs; if fab min is 0.20 mm (8 mil) this becomes a re-route item. Recommend stating 6/6 mil (0.15/0.15) capability in fab spec.
- F-3 **ACCEPT AS-IS** — clearances ≥0.200 mm everywhere in our routing; 0.200 cluster count = 2, 0.205 = 2.

---

## 3. Copper weight & current-path inputs

No current ratings are assumed. Widths measured on the live board (min width along each path, F unless noted); fill-in column for the designer.

| Net / path | Route (min width) | Layer | Vias in path | Cross-section @1 oz (35 µm) | Design current (INPUT) | ΔT @ input (IPC-2221 est.) |
|---|---|---|---|---|---|---|
| VIN_24V: J2 → F1 → D2 → C1 → U1.VIN | 1.0 mm | F | 0 | 0.035 mm² | ____ A | — |
| 5V_MAIN: U1.SW/L1 → C5/C6 → rows | 1.0 mm (0.5 mm SW hook 12.7 mm, see 03A) | F | 2 × 0.6/0.3 to U13/U2 | 0.035 mm² / via ~0.012 mm² ea | ____ A | — |
| 3V3: U2.VOUT → C8 → distribution | 0.8 mm | F | plane-return | 0.028 mm² | ____ A (ESP32 peak Wi-Fi ≥0.5 A typ.) | — |
| AUX_5V: → J10 | 0.6 mm ⇢ 1.0 mm | F | 0 | 0.021 mm² | ____ A | — |
| DO_FIELD_V+ (external supply sense/pull) | 0.8 mm | F | 0 | 0.028 mm² | ____ A (sinking topology: field current does NOT flow in this track unless the external V+ is routed through the board — confirm) | — |
| DO1–DO4_OUT: J6 ↔ U9–U12 drain | 1.0 mm | F | 0 | 0.035 mm² | ____ A per channel | — |
| DO source return: U9–U12 pins 1–3 → GND | 0.5 mm stub → **1 via 0.6/0.3** each → B plane | F→B | 1 | via wall ≈ 0.6·π·0.3·0.025 ≈ 0.024 mm² (25 µm plating assumed) | ____ A per channel (= DOx_OUT current) | — |
| LOGIC_GND return | B plane (4585 mm²) with necks A 0.6 mm / B 0.45 mm (03A) | B | 60 vias | — | — | — |

Copper weight: **not specified in the project** (no board stackup entry). 1 oz assumed for the table. **DESIGN INPUT** required: copper weight (1 oz vs 2 oz), per-net currents, ambient/ΔT target. With inputs supplied, the ΔT column can be completed in 03C.

**Findings**
- P-1 **DESIGN INPUT** — copper weight + currents table (above). Cannot be closed by the auditor.
- P-2 **REVIEW** — DO source-return via: one 0.6/0.3 via per switch. Rule of thumb ~1 A per 0.3 mm via at modest ΔT; if DO channel current > ~1 A, add a second via per switch (R3 from 03A; needs current input). Location: (15.235/22.535/29.835/37.135, 64.925). Render 04.
- P-3 **REVIEW** — 5V_MAIN feed to U13/U2 passes through 2 vias; adequate up to ~1–1.5 A, confirm against input.

---

## 4. Via & thermal review

| Item | Value | Finding |
|---|---|---|
| Via geometry | 192 × 0.6/0.3, annular 0.15 | ACCEPT AS-IS |
| Via tenting | board setup `(tenting front back)` → all vias covered both sides | ACCEPT AS-IS for fab; see A-1 for the 72 near-pad vias |
| U1 exposed pad | 6 × 0.33/0.63 PTH thermal vias in pad (footprint) | ACCEPT AS-IS — in-pad vias are on the EP (intended); tenting on B side avoids solder wicking; note fab may need "plugged/filled" if wicking observed. **REVIEW** (process note for assembler: EP paste reduction ~50–60 %). |
| U3 (ESP32) thermal pad | 0.2 mm drill array (§2C) | F-1 |
| Zone thermal reliefs | gap 0.5 / spoke 0.5, applied to all pads in zones | ACCEPT AS-IS — solderable; large-pad GND pins (U4, J1 shield) receive 4 spokes |
| Direct-connect pads | none (thermal relief global) | ACCEPT AS-IS |
| GND vias (LOGIC_GND) | 60; RS485_GND 3; J1 shield 4 PTH | ACCEPT AS-IS (no stitching campaigns per standing rules) |
| Via-in-pad / mask-dam overlap (via ring intersecting SMD pad on same net) | 7 cases, listed in §5 | see A-2 |

---

## 5. Solderability / assembly access

### 5.1 Via-to-pad proximity (same net, ring overlaps or nearly touches SMD pad)

Hole-edge-to-pad-edge distance (mask expansion 0; tented vias):

| Pad | Via | Hole-to-pad (mm) | Comment | Class |
|---|---|---|---|---|
| Q2.3 (SOT-23) | (82.3, 14.3) | **0.000** | hole tangent to pad edge — true via-in-pad edge; tenting covers, but solder wicking possible if tent is incomplete | **CORRECT BEFORE RELEASE** |
| R11.2 (0603) | (43.2, 48.4) | 0.025 | mask dam impossible (<0.10) → via effectively open into pad unless tented cleanly | **CORRECT BEFORE RELEASE** |
| R10.2 (0603) | (43.2, 50.0) | 0.025 | as above | **CORRECT BEFORE RELEASE** |
| R30.1 (0603) | (38.3, 46.85) | 0.075 | dam < 0.10 mm; G1 rework via (C6C) | REVIEW (move 0.1–0.15 mm east if fab mask sliver min is 0.10) |
| U5.11 (SSOP) | (79.4, 66.0) | 0.075 | dam < 0.10 | REVIEW |
| R32.1 | (11.0, 45.0) | 0.125 | dam ≥0.10 — OK for most fabs | ACCEPT AS-IS |
| TP12 (1.0 mm test pad) | (71.75, 9.95) | 0.000 | test pad, no component — harmless | ACCEPT AS-IS |

72 additional same-net vias within 0.35 mm of an SMD pad edge (R17.1 0.16; R33.2/R34.2/R35.2/C25.2/U5.14/U5.15 0.175; …). With `tenting front back` these are covered; wicking risk is low. **A-1 REVIEW**: state "tented vias both sides" explicitly in fab spec; alternatively specify via mask expansion so that dams ≥0.10 mm exist where possible.

Rationale for CORRECT BEFORE RELEASE on the three 0.000/0.025 cases: mask expansion 0 + tent means the mask film must hold a sliver <0.03 mm between the via annulus and the pad opening; fabs typically strip slivers <0.10 mm, opening the via into the pad → paste loss/tombstone/void on R10, R11 (0603) and Q2. Fix = move via ≥0.2 mm from pad edge (local, 3 vias, no plane impact expected; to be verified in a correction study).

### 5.2 Courtyard / placement
- Courtyard overlaps: **0** (DRC + analytic).
- Intentional courtyard overhangs beyond the board edge: J1 5.3 mm², J2/J10 5.5 mm², J4 17.8, J5 28.9, J6 34.5 mm² (terminal-block wire entry faces outward — correct), U3 717 mm² (antenna region of module courtyard, module body is on-board; see §7).
- Component-to-component courtyard spacing minimum: 0.0 (touching, no overlap) at C-clusters near U2; pick-and-place OK, rework access tight but acceptable for 0402/0603.
- Smallest passive: 0402 (C17, C22 and others); everything else 0603+/SOIC/SSOP/SOT. **COMMON assembly.**

### 5.3 Tombstone heuristic (asymmetric pad thermal attachment, 0402/0603 two-terminal)
Flagged: C16, R28, C25, C27, R26, R38, C22 (0402), C17 (0402) — one pad tied to a zone (thermal relief 0.5/0.5) or a 1.0 mm track, the other to a 0.2–0.25 mm track. Thermal reliefs already mitigate; 0603 parts are low-risk. **REVIEW** for the two 0402s (C17, C22): assembler paste/reflow profile note; no layout change required unless assembler yield is an issue.

### 5.4 Component orientation / polarity
- Polarised parts: D1, D2, D4, D5, D8–D11, C1 (electrolytic?), U-devices — pin-1/cathode silk marks present in library footprints for all checked (D2, D4, U1–U13, J-headers pin-1 chamfer). No orientation DRC exists; **REVIEW** at assembly drawing stage (03C).

Assembly blockers: **0**.

---

## 6. Component-to-board-edge

| Item | Distance | Class |
|---|---|---|
| Copper-to-edge min | 0.50 mm (rule) | ACCEPT |
| Pad-to-edge min (non-overhang parts) | 1.70 mm (J1) | ACCEPT |
| U4 (Ethernet PHY/transformer?) body to edge | 0.60 mm | REVIEW — if V-score/routing tolerance ±0.2 mm, 0.6 mm clearance to a body is fine; ensure no tab-route breakaway at that side |
| Terminal blocks J2/J10/J4/J5/J6 | bodies overhang edge intentionally (wire entry) | ACCEPT AS-IS — **note for fab: overhanging parts require panel gaps / no adjacent boards on those edges; REQUIRES MECHANICAL INPUT for enclosure cut-outs** |
| J1 USB-C | flush with north edge (receptacle face at edge) | ACCEPT AS-IS; enclosure opening → MECHANICAL INPUT |
| SW1 (north edge) | tact switch actuator inboard | ACCEPT |
| Mounting-hole head zone vs parts | J7 2.04 / SW1 2.41 / J6 2.45 / JP1 2.35 mm from hole centre-6 mm-head boundary | see §11 |

---

## 7. Antenna / physical (ESP32 module U3)

- Module antenna end sits at x = 99.95 mm = board east edge (antenna overhangs? no: courtyard overhangs 15 mm but the module body ends at the edge; antenna section is ON the module and lies within the final ~6 mm of board).
- Footprint rule area x ≥ 93.95 mm: no copper, no tracks, no vias, no zone fill (verified: 0 items). Render 06.
- Nearest components to antenna region: SW1/SW2, C11–C14 (all > 6 mm west of the keep-out edge; outside module vendor's recommended no-metal zone).
- Silk: U3 outline reaches the board edge (part of the 12 silk_edge items).
- Physical: 2-layer, module on top, plane below is cleared under the antenna → ACCEPT AS-IS electrically. Enclosure material/metal proximity/standoff head at H2 (95,5) is 11 mm from antenna edge along the edge; if metal standoff/screw head used at H2/H4 near the antenna, vendor guidance applies → **REQUIRES MECHANICAL INPUT**.
- No RF certification claim is made (module-level pre-certification depends on module SKU and antenna keep-out compliance; no working voltage/standard stated).

ANTENNA / BOARD-EDGE: **REVIEW** (mechanical inputs outstanding; electrical/copper side PASS).

---

## 8. Isolation manufacturability

| Barrier | Device pin-row gap | Board copper-free gap (min) | Rule areas | Mask/silk in gap | Class |
|---|---|---|---|---|---|
| RS485 (ADM2587E, SOIC-20W) | 7.25 mm (package) | 6.0 mm (isolation channel y 57.5–63.5 / x 70–74 = 4.0 mm to logic pocket) | RS485_ISO_GAP F+B | mask present (no slot), silk lines cross gap only as outline | REVIEW |
| DI (ISO1212 ×2, SOIC-16W) | 4.0 mm | **4.250 mm** (after 03A2) | DI_ISO_GAP + DI_ISO_GAP_EXT F+B | mask present, no slot | REVIEW |

- No mechanical slot in either barrier (creepage = surface distance over mask; clearance = same). Fab impact: none (slots would be a fab feature; absent).
- No certification/working-voltage claim: the board-level gaps (6.0 / 4.25 mm) are stated as measured; whether they satisfy a standard depends on **working voltage, pollution degree and CTI** — none specified. **DESIGN INPUT**.
- DI barrier 4.25 mm is smaller than the RS485 barrier 6.0 mm; ISO1212 package itself limits at 4.0 mm creepage, so the board is not the bottleneck.
- Rule areas are `no-pour` only (tracks/vias not disallowed); currently 0 tracks/vias inside either area (verified). **REVIEW**: consider (in a later stage) enabling track/via disallow in the rule areas so future edits cannot violate the barrier — not a fab issue.

ISOLATION MANUFACTURABILITY: **REVIEW** (copper side PASS; working voltage/standard unspecified).

---

## 9. (reserved — merged into §5/§6)

---

## 10. Silkscreen

- DRC `silk_edge_clearance`: 12 items — J2/J10 (west edge), J6/J5/J4 (south edge), U3 (east edge): footprint outline segments drawn to or beyond the board edge. All belong to intentionally edge-mounted parts; fab will clip silk at the outline. Render 12. **ACCEPT AS-IS** (optionally cleaned in a silk stage, cosmetic only).
- `silk_over_copper`, `silk_overlap`, `silk_clearance` violations: 0.
- Silk minimums: 0.12 mm line / 0.8 mm text height — at common minimum; some fabs (0.15 mm min) will thicken. TIGHT BUT COMMON.
- Reference designators: visible on 89 footprints; **hidden on 47**: all 18 TPs, J1–J10, H1–H4 (acceptable: connectors labelled by function silk; TPs labelled by net), plus 15 components where hidden refdes hinders assembly/inspection: **R10, R11, R17, R19, R33, C10, C11, C14, C18, C19, C25, C27, D5, SW1, SW2** → **CORRECT BEFORE RELEASE** (assembly drawing/inspection needs refdes, or at least an assembly drawing without silk dependency — designer's call; if a full fab-drawing / pos-file based assembly is used, downgrade to REVIEW).
- J8.1 silk label "IO2" while net is SPARE_GPIO2 = ESP32 IO6 (known from C6): **CORRECT BEFORE RELEASE** (do NOT fix in this stage; recorded for the silk stage).
- Polarity/pin-1 marks: present (§5.4).
- Board identification: check for revision/part-number text on silk: no "Rev-A"/board-name text found among 150 texts → **REVIEW** (add board ID + rev + date-code area in silk stage).

---

## 11. Mounting holes H1–H4

| Hole | Position | Type | Ø | Edge distance | Copper nearest to hole edge | Nearest courtyard to 6 mm head | Zone |
|---|---|---|---|---|---|---|---|
| H1 | (5,5) | NPTH | 3.2 | 3.4 mm | 0.25 mm (LOGIC_GND B fill) | — | LOGIC_GND |
| H2 | (95,5) | NPTH | 3.2 | 3.4 mm | 0.25 mm (LOGIC_GND B fill) | SW1 2.41 mm | LOGIC_GND |
| H3 | (5,75) | NPTH | 3.2 | 3.4 mm | 0.25 mm (LOGIC_GND B fill) | J6 2.45 mm; J7 2.04 mm | LOGIC_GND |
| H4 | (95,75) | NPTH | 3.2 | 3.4 mm | 0.25 mm (**RS485_GND** B fill) | JP1 2.35 mm | RS485_GND |

- Hole-copper 0.25 mm = board hole-clearance rule minimum: fab-legal. But with a metal screw head/standoff (typ. Ø5.5–6 mm on M3) the head lands on **mask over live copper fill**; if the mask is scratched, H1–H3 tie chassis to LOGIC_GND and **H4 ties chassis to RS485_GND (isolated side)** — this would bridge the isolation barrier through the chassis if H1–H3 are also metal-grounded. → **DESIGN INPUT / REQUIRES MECHANICAL INPUT**: choose (a) nylon standoffs at H4 (or all), or (b) add copper keep-out ≥ head radius + 0.5 mm around holes (a later corrective stage; changes fills only). Render 10.
- Courtyard-to-head clearances 2.04–2.45 mm from the 6 mm head boundary: tools fit; ACCEPT.
- H-holes are NPTH with no pad copper: no plating, no ring — COMMON FAB.

---

## 12. Test access

18/18 test points present (from 03A), all on F, all reachable from the top with the module/connectors populated:

| TP | Net | Pad | Location / access |
|---|---|---|---|
| TP1–TP5 | VIN / 5V / 3V3 / GND / AUX (power) | 2.5 mm | west/centre-west, ≥2 mm from neighbours — probe/clip OK |
| TP6–TP11 | JTAG (TMS/TDI/TDO/TCK) + UART0 TX/RX | 1.0 mm, 2.2 mm pitch | row south of U3 body (1.26 mm from body) — pogo-pin fixture OK; hand-probe OK; a 2.54 mm header cannot be retrofitted (pitch 2.2) |
| TP12 | ETH_INT/… (C6A net) | 1.0 mm | 1.27 mm from J1 shell — probe OK, fixture pin needs J1 clearance |
| TP13 | ETH | 1.0 mm | OK |
| TP14 | DI_FIELD_GND (COM) | 1.0 mm | isolated side, OK |
| TP15/16 | DO ctrl / LOGIC_GND ("GND" silk) | 1.0 mm | OK |
| TP17/18 | I2C SDA/SCL | 1.0 mm | OK |

- TP3 and TP16 are both LOGIC_GND with silk "GND" — fine (two GND probe returns).
- No B-side TPs; single-sided fixture possible. No TP within 1 mm of the board edge.
- 1.0 mm pads are small for hand probing but standard for 1.0 mm pogo pins; **REVIEW**: if hand-probing during bring-up is the primary use, 1.5 mm would be preferable (cosmetic/UX, not blocking).

TEST ACCESS: **PASS**.

---

## 13. Connector / field wiring

| Conn | Function | Orientation | Notes |
|---|---|---|---|
| J2 | VIN 24 V in | west edge, rot −90 | wire entry outward; polarity silk present? (silk text "+"/"−" not found in text list → REVIEW add) |
| J10 | AUX 5 V out | west edge | as above |
| J6 | DO1–DO4 outputs, **low-side sinking** (ZXMS6004 drains) | south edge | field load between external DO_FIELD_V+ and DOx_OUT; J6 GND/COM pin = LOGIC_GND? (03A: DO source return to LOGIC_GND plane) — **DESIGN INPUT**: confirm whether field return shares logic ground intentionally (non-isolated DO section) |
| J5 | DI1–DI4 + COM | south edge | COM = **DI_FIELD_GND** (isolated), not LOGIC_GND — verified by netlist; correct |
| J4 | RS485 A/B/GND(iso) | south edge | GND = RS485_GND (isolated) — verified |
| J1 | USB-C (power/debug) | north edge, rot 180 | VBUS sense-only via R-divider (C6A) |
| J7/J8/J9 | headers (rot 90, north) | inboard | J8.1 silk mismatch (§10) |
| JP1 | jumper near H4 | — | 2.35 mm from H4 head boundary |

Mechanical: terminal-block bodies overhanging edges need enclosure openings — REQUIRES MECHANICAL INPUT.

---

## 14. Assembly process

- All SMD on F; through-hole: J1 (shield PTH), J2/J4/J5/J6/J10 terminal blocks, J7/J8/J9 headers, SW?; → single-sided reflow + selective/hand solder for PTH. No B-side components (verified: 0 footprints on B) → no second reflow, no wave-side constraints.
- Fiducials: **none found** (no fiducial footprints among 136) → **REVIEW**: add ≥2 global fiducials (1 mm Cu, 3 mm mask clear) if machine assembly is intended (silk/fab stage; no routing impact — placement space exists at (10,10)/(90,70) quadrants outside fills? fills are on B, fiducials on F: OK).
- Panelisation: overhanging connectors on W/S edges → panel must use tabs on N/E edges or single-board delivery. MECHANICAL/FAB INPUT.
- Paste: EPs U1 (window-pane), U3 (module thermal pad, 0.2 mm vias) — assembler stencil note.
- Cleaning: isolation barriers rely on mask surface — specify no-clean or thorough cleaning (residue across barrier lowers CTI). REVIEW note for 03C.

---

## 15. BOM / footprint consistency

- 73/73 passives carry value only (no MPN) — normal for generic parts; assembler substitutes.
- Parts lacking an MPN-like field: D1, D2, D8–D11, Q2, L2, J8, JP1, SW1, SW2, D4 → **REVIEW**: MPN required for polarised/semiconductor/mechanical parts before BOM release (schematic property, not a PCB write).
- Q2/R48 exist on board but are unannotated in schematic (5 board-only nets) → **CORRECT BEFORE RELEASE** (schematic annotation/sync — schematic stage, not PCB; without it the BOM exported from the schematic omits Q2/R48).
- Footprint vs value sanity: 0402/0603 passives, SOT-23 (Q2), SOIC/SSOP/QFN — no mismatch detected by name heuristics.
- Netlist: 425/425 nets match; 0 extra/missing pads.

BOM / FOOTPRINT CONSISTENCY: **REVIEW** (Q2/R48 annotation + MPNs).

---

## 16. Fab release inputs

**REQUIRED before Gerber release**
1. Copper weight (1 oz / 2 oz) and per-net currents (§3 table) — designer.
2. Fab capability confirmation: 0.20 mm drill (U3 EP array), 0.15 mm track, 0.10 mm mask sliver with 0 expansion, 0.12 mm silk — fab.
3. Via-in-pad corrections (Q2.3, R10.2, R11.2) — layout (correction study).
4. Q2/R48 schematic annotation + sync (BOM completeness) — schematic.
5. Mounting-hole standoff material / copper relief decision (H4 isolation) — mechanical + designer.
6. J8.1 silk "IO2"→"IO6" + refdes visibility on 15 parts + board ID/rev text — silk stage.
7. Isolation working voltage / standard (or explicit statement that none is claimed) — designer.

**NICE TO HAVE**
- Fiducials ×2–3; TP pad 1.5 mm for hand probing; via mask-expansion policy note; rule areas with track/via disallow; polarity silk on J2; second DO source via per switch if current > ~1 A; R30.1/U5.11 via nudges.

---

## 17. Release-risk matrix & next stage

| # | Item | Class | Owner | Stage |
|---|---|---|---|---|
| 1 | Q2.3 via hole tangent to pad (0.000) | CORRECT BEFORE RELEASE | layout | 03B1 |
| 2 | R11.2 via 0.025 mm from pad | CORRECT BEFORE RELEASE | layout | 03B1 |
| 3 | R10.2 via 0.025 mm from pad | CORRECT BEFORE RELEASE | layout | 03B1 |
| 4 | Hidden refdes on 15 non-connector/TP parts | CORRECT BEFORE RELEASE | silk | 03D |
| 5 | J8.1 silk IO2 → IO6 | CORRECT BEFORE RELEASE | silk | 03D |
| 6 | Q2/R48 not in schematic (BOM) | CORRECT BEFORE RELEASE | schematic | separate |
| 7 | 0.20 mm drills U3 EP | DESIGN INPUT (FAB) | fab | 03C |
| 8 | Copper weight / currents / DO via count | DESIGN INPUT | designer | 03C |
| 9 | H4 copper under standoff (RS485_GND) / H1–H3 | DESIGN + MECHANICAL INPUT | mech | 03C/03B1 |
| 10 | Isolation working voltage / standard | DESIGN INPUT | designer | 03C |
| 11 | Enclosure openings, antenna proximity, panelisation | MECHANICAL INPUT | mech | 03C |
| 12 | R30.1 / U5.11 via dams 0.075 | REVIEW | layout | 03B1 (optional) |
| 13 | 0.15 mm stubs ×4, 0.12 silk | REVIEW (fab capability) | fab | 03C |
| 14 | Tented near-pad vias ×72 | REVIEW (spec note) | fab spec | 03C |
| 15 | Tombstone heuristic C17/C22 (0402) | REVIEW | assembler | 03C |
| 16 | No fiducials / no board ID text / J2 polarity silk | REVIEW | silk | 03D |
| 17 | MPNs missing on 13 parts | REVIEW | schematic/BOM | separate |
| 18 | 12 silk_edge DRC items | ACCEPT AS-IS | — | 03D optional |
| 19 | 1.0 mm TP pads | REVIEW (UX) | — | optional |
| 20 | Rule areas no-pour only | REVIEW | layout | optional |

Counts: FABRICATION BLOCKERS 0 · ASSEMBLY BLOCKERS 0 · CORRECT BEFORE RELEASE 6 (3 layout, 2 silk, 1 schematic) · REVIEW / DESIGN INPUT 14.

**Next stage recommendation — option A: PCB-03B1 targeted correction study.** Three layout items (via-in-pad Q2.3/R10.2/R11.2) are the only CORRECT-BEFORE-RELEASE items that require PCB writes; they must be studied on a disposable copy (plane-model equality, DRC 0, unconnected 0) before the fab spec (03C) and silk cleanup (03D) can be finalised. Optional inclusions for the same study: R30.1/U5.11 via nudges (item 12) and mounting-hole copper relief (item 9, if mechanical input is available by then). Silk items (4, 5, 16) and schematic item (6) are deferred to their own stages.

---

PCB-03B MANUFACTURABILITY / ASSEMBLY AUDIT: REVIEW
LIVE PCB MODIFIED: NO
UNCONNECTED: 0
ELECTRICAL DRC: 0
FABRICATION BLOCKERS: 0
ASSEMBLY BLOCKERS: 0
CORRECT-BEFORE-RELEASE ITEMS: 6
REVIEW / DESIGN-INPUT ITEMS: 14
ISOLATION MANUFACTURABILITY: REVIEW
ANTENNA / BOARD-EDGE: REVIEW
TEST ACCESS: PASS
BOM / FOOTPRINT CONSISTENCY: REVIEW
NEXT PROPOSED STAGE: PCB-03B1 — targeted correction study (via-in-pad Q2.3 / R10.2 / R11.2; optional R30.1, U5.11, mounting-hole copper relief)
WAITING FOR HUMAN REVIEW
