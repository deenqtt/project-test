# Phase 8 USB-C Native USB Design

## Scope

Create only the ESP32-S3 native USB-C service interface in `hardware/usb_c.kicad_sch`, link it as hierarchy `USB_C`, and update GPIO7 from `SPARE_GPIO3` to `USB_VBUS_SENSE`. Do not modify PCB files or unrelated circuitry.

## Frozen architecture

- ESP32-S3 GPIO19 = `USB_D_N`; GPIO20 = `USB_D_P`.
- GPIO6 remains `SPARE_GPIO2`; GPIO7 becomes `USB_VBUS_SENSE`.
- Connector: GCT `USB4105-GF-A`, USB 2.0 Type-C receptacle.
- UFP configuration: independent 5.1 kΩ, 1% Rd from CC1 and CC2 to `LOGIC_GND`.
- ESD: ST `USBLC6-2SC6`, SOT23-6L, placed electrically connector-side of the series resistors.
- Series damping: 22 Ω, 1%, one in D+ and one in D−, retained because Espressif recommends reserving 22/33 Ω close to the ESP32-S3.
- Internal ESP32 USB pull-up is used; no external D+ pull-up.
- `USB_VBUS` is connector/reference/sense only. It may connect only to connector VBUS pins, USBLC6-2SC6 VBUS reference, and the top of the VBUS divider.
- VBUS divider: 91 kΩ top and 130 kΩ bottom, both 1%, midpoint `USB_VBUS_SENSE`; no filter capacitor.
- Connector shield connects directly to `LOGIC_GND`; no fictitious chassis domain exists.
- SBU1/SBU2 are no-connect. D+ duplicates are joined; D− duplicates are joined; all VBUS and GND contacts are joined by the selected symbol.

## VBUS calculations

Divider ratio nominal = 130/(91+130) = 0.58824.

With 1% tolerance, minimum ratio = 128.7/(91.91+128.7) = 0.58338. At VBUS = 4.4 V, sense minimum = 2.567 V. For a preliminary TLV62569 output maximum of 3.366 V, ESP32-S3 VIH requirement 0.75×VDD is 2.525 V, leaving about 42 mV.

Maximum ratio = 131.3/(90.09+131.3) = 0.59307. At VBUS = 5.25 V, sense maximum = 3.114 V. Against preliminary 3V3 minimum 3.234 V plus 0.3 V absolute-input allowance, this leaves about 420 mV.

Divider current at 5 V is 5/221 kΩ = 22.6 µA. With no explicit capacitor, the 130 kΩ discharge path requires less than 17 nF total node capacitance to cross the LOW region within 3 ms; expected GPIO/trace parasitics are orders of magnitude smaller. Prototype unplug timing remains a verification item.

## Connectivity

`USB_C receptacle → USBLC6-2SC6 → 22 Ω series resistors → USB_D_N/P → ESP32-S3`.

`USB_VBUS → USBLC6 VBUS reference + 91 kΩ → USB_VBUS_SENSE → 130 kΩ → LOGIC_GND`.

No connection is permitted from `USB_VBUS` to `5V_MAIN`, `3V3_LOGIC`, `VIN_FIELD`, or `AUX_5V`.

## Validation

Capture through KiCad MCP only. Validate root and child schematics, run fresh ERC, generate and audit the netlist, confirm GPIO pin mapping, connector polarity, CC resistors, ESD pinout, shield ground, VBUS isolation, duplicate references, orphan wires, and off-grid geometry. Do not suppress warnings.
