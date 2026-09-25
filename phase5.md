# Phase 5A — WIZ850io Ethernet Interface

Only the WIZ850io Ethernet interface was captured. RS485, digital I/O, sensors, USB-C, and PCB placement were not started.

## A. Ethernet circuit created

- [VERIFIED FROM KICAD] Added hierarchical sheet ETHERNET_WIZ850IO in hardware/ethernet_wiz850io.kicad_sch.
- [VERIFIED FROM KICAD] Added U4 = WIZ850io MODULE, carrier-side bypass capacitors C14/C15, and TP12/TP13 for Ethernet 3.3 V and ground.
- [VERIFIED FROM KICAD] The six frozen nets connect without changing the GPIO map: GPIO8 reset, GPIO9 interrupt, GPIO10 chip-select, and GPIO11–13 SPI.

## B. WIZ850io pinout verification

[VERIFIED FROM WIZNET] With the RJ45 at the top and viewed from above, pin 1 is nearest the RJ45 and numbering runs downward:

| Header | Official pinout |
|---|---|
| J1 | 1 GND, 2 GND, 3 MOSI, 4 SCLK, 5 SCSn, 6 INTn |
| J2 | 1 GND, 2 3V3D, 3 3V3D, 4 NC, 5 RSTn, 6 MISO |

Source: [WIZ850io official documentation](https://docs.wiznet.io/Product/ioModule/WIZ850io).

## C. Power and decoupling

- [VERIFIED FROM WIZNET] Supply range is 2.97–3.63 V, nominal 3.3 V; normal current is listed as approximately 141 mA typical.
- [VERIFIED FROM WIZNET] Input thresholds are compatible with ESP32-S3 3.3 V logic.
- [VERIFIED FROM WIZNET] The module revision 1.1 schematic already contains local W5500 bypassing, including 10 uF and 100 nF near the digital supply.
- [ENGINEERING DECISION] C14 = 10 uF 16 V X7R and C15 = 100 nF 16 V X7R provide additional carrier-side bypassing at the module socket. They do not replace the capacitors already installed on the module.

## D. Reset, CS, and INT behavior

- [VERIFIED FROM WIZNET] RSTn is active LOW, must remain LOW for at least 500 us, and SPI must wait at least 50 ms after reset release.
- [VERIFIED FROM WIZNET] Module revision 1.1 includes 4.7 kΩ pull-ups on SCSn, INTn, and RSTn.
- [CORRECTED DECISION] The MCU-side 10 kΩ ETH_CS_N pull-up R13 was removed because the module already guarantees deselection with its 4.7 kΩ pull-up.
- [CORRECTED DECISION] ETH_RST_N pull-down R14 was changed from 10 kΩ to 1 kΩ. A 10 kΩ pull-down against 4.7 kΩ would produce about 2.24 V and would not guarantee reset.
- [ENGINEERING CALCULATION] Using 3.63 V maximum supply, 4.7 kΩ at −5%, and 1 kΩ at +1% gives reset LOW approximately 0.67 V, below the worst-case LOW limit 0.3 × 2.97 V = 0.891 V. Driving reset HIGH consumes approximately 3.3 mA through the 1 kΩ pull-down.
- [VERIFIED FROM WIZNET] INTn is classified as an active-low output, with specified HIGH and LOW output levels; it is not identified as open-drain. No additional carrier pull-up was added because the module already contains 4.7 kΩ.
- [VERIFIED FROM WIZNET] Revision 1.1 contains 3.3 Ω series damping on MOSI, MISO, SCLK, and SCSn. No duplicate carrier SPI series resistors were installed.

## E. Footprint and mechanical status

- [VERIFIED FROM WIZNET] Official mechanical dimensions: body 23.00 × 25.00 mm, header-row spacing 20.32 mm, pin pitch 2.54 mm, pin-1 center 6.4 mm from the RJ45/top edge, and pin-6 center 5.9 mm from the bottom edge.
- [VERIFIED FROM WIZNET] The official revision 1.1 Altium PCB source specifies header pad diameter 1.524 mm and drill diameter 0.9144 mm.
- [VERIFIED FROM KICAD] Created Project_WIZnet:WIZ850io_Module_THT with 12 through-hole pads, official coordinates, official pad/drill dimensions, module body outline, pin-1 marking, and a 0.5 mm engineering courtyard allowance.
- [ENGINEERING REQUIREMENT] Place the RJ45 at the host PCB edge with adequate cable/latch and enclosure clearance. PCB placement remains intentionally deferred.

## F. ERC result and warning-count change

- [VERIFIED FROM KICAD] Fresh ERC: **0 errors, 14 warnings, 0 info**.
- [VERIFIED FROM KICAD] Phase 4B baseline was 18 warnings. The four previously unconnected staged Ethernet globals ETH_INT_N, ETH_MOSI, ETH_SCK, and ETH_MISO are now connected and their warnings disappeared.
- [VERIFIED FROM KICAD] ETH_CS_N and ETH_RST_N now have verified cross-sheet continuity; their earlier local/global and undriven-input errors were corrected.
- [VERIFIED FROM KICAD] Remaining 14 warnings are deferred non-Ethernet global nets. They were not suppressed.
- [VERIFIED FROM KICAD] Parent, ESP32 core, Ethernet sheet, and custom symbol validate through KiCad CLI. Ethernet/core audits report zero orphan wires, zero overlaps, and zero off-grid geometry.

## G. Unresolved issues

- [NEEDS VERIFICATION] Select exact manufacturer part numbers for C14 and C15 and verify effective capacitance under 3.3 V DC bias.
- [NEEDS VERIFICATION] Verify module socket/header mating height and enclosure clearance against the exact production header/socket parts.
- [NEEDS VERIFICATION] Confirm SPI signal integrity on the final PCB; add carrier damping only if layout or measurement demonstrates a need.
- [NEEDS VERIFICATION] Bench-test reset LOW duration, 50 ms firmware delay, CS idle state, and interrupt operation on the assembled prototype.

## H. GO / NO-GO for Phase 5B

**CONDITIONAL GO.** Ethernet electrical capture, frozen GPIO continuity, startup deselect/reset behavior, symbol, and footprint are ready for the next schematic phase. Conditions remaining are production MPN selection for bypass capacitors and later mechanical/bench validation. No PCB placement was performed.
