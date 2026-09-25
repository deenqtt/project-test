# Phase 4A — ESP32-S3 Core Planning and GPIO Allocation

Status: planning selesai. Schematic dan PCB **belum diubah** pada fase ini.

## A. Selected ESP32-S3-WROOM-1 variant

**Final selection: `ESP32-S3-WROOM-1-N8R2`**, versi PCB antenna, 8 MB Quad-SPI flash, 2 MB Quad-SPI PSRAM, ambient rating −40 °C sampai +85 °C.

- [VERIFIED FROM DATASHEET] `N8R2` adalah part number resmi dengan 8 MB Quad-SPI flash dan 2 MB Quad-SPI PSRAM. [ESP32-S3-WROOM-1 datasheet, Table 1-1](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf)
- [ENGINEERING DECISION] 8 MB flash dipilih sebagai kapasitas yang wajar untuk firmware Industrial IoT, NVS, recovery, dan dua OTA application slot. Ukuran partition final tetap harus dikunci di firmware.
- [ENGINEERING DECISION] 2 MB PSRAM memberi ruang untuk network/TLS/application buffers tanpa kebutuhan 8 MB Octal PSRAM pada Rev A.
- [VERIFIED FROM DATASHEET] Pemilihan Quad-SPI PSRAM mempertahankan GPIO35–GPIO37. Pada varian R8/R16V dengan Octal-SPI PSRAM, ketiga pin tersebut terhubung ke PSRAM dan tidak tersedia untuk penggunaan lain.
- [FINAL CONSTRAINT] BOM tidak boleh disubstitusi ke `N8R8`, `N16R8`, atau varian Octal-PSRAM lain tanpa melakukan remap DI1–DI3.

## B. Restricted/reserved pin analysis

| Pin | Status Rev A | Alasan |
|---|---|---|
| GPIO0 | `BOOT` only | [VERIFIED FROM DATASHEET] Strapping pin dengan weak pull-up; LOW saat reset memilih Joint Download Boot bila GPIO46 juga LOW. Tidak dipakai sebagai peripheral. |
| GPIO3 | Leave unused/test pad only | [VERIFIED FROM DATASHEET] Strapping pin pemilih sumber JTAG; tidak memiliki internal pull resistor. Default eFuse memilih USB Serial/JTAG sehingga strap ini diabaikan, tetapi beban eksternal tetap dihindari. |
| GPIO45 | Leave unused | [VERIFIED FROM DATASHEET] Strapping `VDD_SPI`; weak pull-down memilih default 3.3 V. Jangan diberi pull-up atau beban yang mengubah level saat reset. |
| GPIO46 | Leave unused/test pad only | [VERIFIED FROM DATASHEET] Strapping boot/ROM-print dengan weak pull-down. GPIO0=LOW dan GPIO46=LOW masuk download boot. |
| GPIO19 / GPIO20 | Reserved native USB | [VERIFIED FROM DATASHEET] Fixed native USB D−/D+ dan USB Serial/JTAG pins. Keduanya memiliki documented startup glitches, sehingga tidak boleh digunakan untuk fungsi lain. |
| GPIO26–GPIO34 | Unavailable/not exposed | [VERIFIED FROM DATASHEET] Dipakai atau direkomendasikan untuk in-package flash/PSRAM; tidak diekspos sebagai GPIO umum pada WROOM-1. |
| GPIO35–GPIO37 | Available only because N8R2 is locked | [VERIFIED FROM DATASHEET] Tersedia pada Quad-PSRAM N8R2, tetapi hilang pada Octal-PSRAM R8/R16V. Dialokasikan sebagai input saja pada Rev A. |
| GPIO39–GPIO42 | Reserved external JTAG | [VERIFIED FROM DATASHEET] Default pad-JTAG functions MTCK/MTDO/MTDI/MTMS. Disimpan untuk debug dan bring-up. |
| GPIO43 / GPIO44 | Reserved UART0 console | [VERIFIED FROM DATASHEET] U0TXD/U0RXD dan ROM boot messages default dapat keluar melalui UART0. |
| GPIO47 / GPIO48 | General 3.3 V I/O on N8R2 | [VERIFIED FROM DATASHEET] Hanya menjadi 1.8 V pada R16V; pada selected N8R2 merupakan 3.3 V I/O. Dipakai untuk DO3/DO4. |
| EN | Reset/enable, bukan GPIO | [VERIFIED FROM DATASHEET] Tidak boleh floating. EN LOW mematikan/reset chip; minimum low reset time 50 µs. Harus memakai Espressif-recommended RC/pull-up dan pushbutton ke ground pada schematic. |

[VERIFIED FROM DATASHEET] Semua GPIO yang diekspos oleh selected WROOM-1 ditandai `I/O/T`; tidak ada pin input-only dalam map ini. Walaupun demikian, peripheral fixed pins, straps, flash/PSRAM, USB, JTAG, dan UART0 membatasi pemakaian praktis.

[VERIFIED FROM DATASHEET] Saat reset, banyak pin hanya memiliki input-enable atau tidak memiliki predefined pull. Espressif juga mendokumentasikan power-up glitches sekitar 60 µs pada GPIO1–GPIO20 tertentu. Karena itu firmware saja tidak cukup untuk menjamin keadaan aman pada output kritis. [ESP32-S3 Series datasheet, pin overview and power-up glitches](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf)

## C. Proposed GPIO map

| GPIO | Function | Peripheral | Direction | Boot/reset concern | Selection reason |
|---:|---|---|---|---|---|
| 0 | `BOOT_N` pushbutton | Boot service | Input | Strap; default weak pull-up. Button grounds pin only when download mode is requested. | Dedicated boot function; no shared load. |
| 1 | `DO1_CTRL` | ZXMS6005N8Q DO1 | Output | Documented low-level glitch ≈60 µs is safe because LOW=OFF; mandatory external input pull-down keeps switch OFF while pin is high-Z/unconfigured. | Non-strap, low startup behavior, direct GPIO. |
| 2 | `DO2_CTRL` | ZXMS6005N8Q DO2 | Output | Same as GPIO1; mandatory external pull-down and early firmware LOW. | Non-strap and startup glitch is toward safe state. |
| 4 | `RS485_DIR` | ADM2587E `DE` + `/RE` | Output | Documented low-level glitch ≈60 µs is safe: LOW disables driver and enables receiver. Add external pull-down. | Deterministic receive/default-safe bus state with one GPIO. |
| 5 | `SPARE_GPIO1` / digital-output master enable | Phase 6B AHCT125 OE control | Output | External transistor and pull network keep all outputs disabled during reset. | Consumed by the approved Phase 6B safe-output architecture; legacy net name retained. |
| 6 | `SPARE_GPIO2` | Spare/header or test pad | Bidirectional | Same caution as GPIO5. | Accessible non-strap spare. |
| 7 | `USB_VBUS_SENSE` | Self-powered USB VBUS monitor | Input | Never connect 5 V directly; Phase 8 uses a verified resistor divider and no filter capacitor. | Required by Espressif for VBUS attach/detach monitoring on a self-powered USB device. |
| 8 | `ETH_RST_N` | WIZ850io reset | Output | WIZ reset is active LOW. External pull-down should hold module reset until firmware drives HIGH; wait at least 50 ms before SPI. | Starts Ethernet deterministically after 3V3 and MCU initialization. |
| 9 | `ETH_INT_N` | WIZ850io interrupt | Input | Active LOW; input bias/pull-up must be defined in schematic. | Allocated for event-driven firmware; polling remains possible. |
| 10 | `ETH_CS_N` | WIZ850io / SPI2 | Output | Must remain HIGH while MCU is unconfigured; add external pull-up. | Native SPI2 `FSPICS0`, clean SPI grouping. |
| 11 | `ETH_MOSI` | WIZ850io / SPI2 | Output | Low glitch is not hazardous while CS is pulled HIGH and WIZ is held reset. | Native SPI2 `FSPID`. |
| 12 | `ETH_SCK` | WIZ850io / SPI2 | Output | Same containment by CS HIGH/reset LOW. | Native SPI2 `FSPICLK`. |
| 13 | `ETH_MISO` | WIZ850io / SPI2 | Input | No critical output state; avoid bus contention if future SPI devices are added. | Native SPI2 `FSPIQ`. |
| 14 | `ONEWIRE_DATA` | DS18B20 | Bidirectional open-drain | Documented low glitch is benign; bus needs an external pull-up selected for cable/load conditions. | Keeps service bus separate from I2C and SPI. |
| 15 | `I2C_SDA` | I2C0 service/sensors | Bidirectional open-drain | External pull-up required; no critical boot output. | Adjacent to SCL; GPIO matrix supports I2C. |
| 16 | `I2C_SCL` | I2C0 service/sensors | Bidirectional open-drain | External pull-up required; no critical boot output. | Adjacent to SDA; simple firmware map. |
| 17 | `RS485_TXD` | UART1 → ADM2587E TxD | Output | Documented low glitch cannot drive the bus because `RS485_DIR` defaults LOW. | Native U1TXD IO-MUX pin. |
| 18 | `RS485_RXD` | UART1 ← ADM2587E RxD | Input | GPIO18 has documented low/high glitches, but it is used as an input after reset; optional small series resistor may limit transient contention. | Native U1RXD IO-MUX pin. |
| 19 | `USB_D_N` | Native USB | Bidirectional | Fixed USB pin; documented startup behavior belongs to USB block. | Preserves native USB download/debug/device operation. |
| 20 | `USB_D_P` | Native USB | Bidirectional | Fixed USB pin; do not add GPIO loading. | Preserves native USB download/debug/device operation. |
| 21 | `STATUS_LED` | Heartbeat/status LED | Output | Configure LED active-HIGH with external pull-down so it stays OFF during reset. | No documented power-up glitch and no strap/debug conflict. |
| 35 | `DI1_LOGIC` | ISO1212 DI1 | Input | Available only on selected Quad-PSRAM variant; no power-up glitch listed. | Keeps isolated-input outputs away from glitching GPIO4–GPIO7. |
| 36 | `DI2_LOGIC` | ISO1212 DI2 | Input | Same N8R2 lock as GPIO35. | Clean contiguous input group. |
| 37 | `DI3_LOGIC` | ISO1212 DI3 | Input | Same N8R2 lock as GPIO35. | Clean contiguous input group. |
| 38 | `DI4_LOGIC` | ISO1212 DI4 | Input | Non-strap; no listed power-up glitch. | Completes contiguous DI group. |
| 47 | `DO3_CTRL` | ZXMS6005N8Q DO3 | Output | No listed power-up glitch, but external pull-down is still mandatory. 3.3 V I/O only because N8R2 is locked. | Non-strap and free of USB/JTAG/UART0 conflicts. |
| 48 | `DO4_CTRL` | ZXMS6005N8Q DO4 | Output | Same as GPIO47. | Completes paired high-number output group. |

### Interface-specific decisions

- **WIZ850io INT:** [ENGINEERING DECISION] `INTn` is optional if firmware polls W5500 socket/common interrupt registers. It is nevertheless allocated on GPIO9 because it enables event-driven receive/link handling and costs only one available GPIO. Firmware Rev A may begin with polling, but the hardware pin should be connected.
- **WIZ850io reset:** [VERIFIED FROM WIZNET] `RSTn` is active LOW for at least 500 µs and SPI must wait at least 50 ms after release. [WIZ850io documentation](https://docs.wiznet.io/Product/ioModule/WIZ850io)
- **ADM2587E control:** [VERIFIED FROM DATASHEET] MCU TX connects to ADM `TxD`, ADM `RxD` connects to MCU RX, `DE` is active HIGH, and `/RE` is active LOW. [ENGINEERING DECISION] For Rev A half-duplex, tie `DE` and `/RE` together as `RS485_DIR`: LOW = transmitter disabled/receiver enabled; HIGH = transmitter enabled/receiver disabled. This needs one GPIO, not two. [ADM2587E datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/adm2582e-2587e.pdf)

## D. Peripheral budget

| Resource | Allocation | Remaining |
|---|---|---|
| Exposed module GPIO | 36 total | 24 assigned including BOOT and USB; 9 deliberately reserved; 3 spare |
| GPIO used for runtime peripheral/service signals | 25 | Includes USB data, USB VBUS sense, and Phase 6B output-enable; excludes BOOT button |
| Strapping pins | GPIO0 used only for BOOT; GPIO3/45/46 unused | No strap drives a critical output |
| Spare GPIO | GPIO6 | One remaining spare; GPIO5 was consumed by Phase 6B and GPIO7 by Phase 8 |
| SPI0/SPI1 | Internal flash/PSRAM | Not available to application |
| SPI2 | WIZ850io on GPIO10–13 | Used; Mode 0 or Mode 3 supported by WIZ850io |
| SPI3 | Unassigned | Available in silicon; would require remapping spare/reserved GPIO |
| UART0 | Debug/programming console on GPIO43/44 | Reserved |
| UART1 | ADM2587E on GPIO17/18 | Used, plus GPIO4 direction |
| UART2 | Unassigned | Available in silicon; pins would use GPIO matrix |
| I2C0 | Service/sensor bus on GPIO15/16 | Used |
| I2C1 | Unassigned | Available in silicon; pins would require spare/remapped GPIO |
| USB OTG / USB Serial-JTAG | GPIO19/20 | Preserved; shared integrated transceiver behavior must be selected in firmware |

[VERIFIED FROM DATASHEET] ESP32-S3 provides three UARTs, two I2C controllers, and two general-purpose SPI controllers (`SPI2`, `SPI3`); SPI0/SPI1 serve flash/PSRAM. Native USB uses GPIO19/20.

## E. Boot/reset risk analysis

1. **Digital outputs:** [FINAL REQUIREMENT] Each ZXMS6005N8Q logic input gets a physical pull-down so DO1–DO4 remain OFF before firmware, during reset, watchdog resets, and brownout. Firmware must configure GPIO1, GPIO2, GPIO47, and GPIO48 LOW before enabling application tasks. Exact pull-down value remains a Phase output-schematic verification item.
2. **RS485 bus:** [FINAL REQUIREMENT] A physical pull-down on `RS485_DIR` makes reset state receive-only and prevents bus drive. Tying `DE` and `/RE` is valid for half-duplex but intentionally disables local receive while transmitting.
3. **Ethernet startup:** [FINAL REQUIREMENT] Hardware bias must keep `ETH_CS_N` HIGH and `ETH_RST_N` LOW until firmware deliberately releases reset. This prevents the GPIO8–GPIO13 startup glitches from becoming SPI commands.
4. **Boot straps:** [FINAL REQUIREMENT] GPIO0/3/45/46 must not be loaded by peripherals. `BOOT_N` uses GPIO0 only; GPIO45 must remain LOW at strap sampling for 3.3 V VDD_SPI.
5. **EN/reset:** [VERIFIED FROM DATASHEET] EN cannot float and must remain LOW at least 50 µs for reset. Use Espressif’s recommended EN RC network and reset switch; component values are to be copied from the official hardware guideline during schematic capture.
6. **USB:** [FINAL REQUIREMENT] Keep GPIO19/20 exclusive to USB, route as a differential pair in PCB phase, and do not rely on them for safe output state.
7. **Variant substitution:** [HIGH RISK] DI1–DI3 depend on GPIO35–GPIO37 availability. Procurement and assembly documentation must specify full part number `ESP32-S3-WROOM-1-N8R2`.

## F. Unresolved questions

- [NEEDS VERIFICATION] Lock the flash partition table and confirm that 8 MB supports the final dual-OTA/recovery/logging plan before firmware release.
- [NEEDS VERIFICATION] Select exact external pull-down values for ZXMS inputs and `RS485_DIR`, and pull-up/pull-down values for WIZ850io CS/reset/INT from the connected-device leakage/current limits during schematic capture.
- [NEEDS VERIFICATION] Decide whether native USB is used as USB Serial/JTAG only, USB OTG device, or time-multiplexed; GPIO19/20 remain reserved in every case.
- [NEEDS VERIFICATION] Decide whether external pad-JTAG GPIO39–GPIO42 and UART0 GPIO43/44 become populated headers or test pads.
- [NEEDS VERIFICATION] Confirm ISO1212 output topology and required logic-side pull-up/series resistance before connecting DI1–DI4.
- [NEEDS VERIFICATION] Confirm WIZ850io `INTn` board-level pull-up behavior; add an external pull-up if the module does not provide a guaranteed one.

## G. GO / NO-GO recommendation

**CONDITIONAL GO untuk ESP32 schematic capture.** GPIO count, native USB, safe RS485 direction, clean SPI2/UART1 grouping, four isolated inputs, four protected outputs, service buses, status LED, and three spare GPIO all fit the selected `ESP32-S3-WROOM-1-N8R2`.

Before completing the ESP32 schematic, the hardware bias networks listed in Section E/F must be selected from the relevant datasheets. No ESP32 placement, wiring, or schematic modification was performed in Phase 4A.

---

# Phase 4B — ESP32-S3 Core Schematic Capture

Bagian ini mencatat hasil capture dan verifikasi final Phase 4B. Capture peripheral, USB-C, dan PCB belum dimulai.

## A. ESP32 core created

- [VERIFIED FROM KICAD] Core dibuat sebagai hierarchical sheet `ESP32_CORE` di `hardware/esp32_core.kicad_sch` dan ditautkan dari `hardware/kicad_mcp_test.kicad_sch`.
- [VERIFIED FROM KICAD] `U3` memakai symbol `RF_Module:ESP32-S3-WROOM-1`, footprint `RF_Module:ESP32-S3-WROOM-1`, dan nilai/MPN dikunci ke `ESP32-S3-WROOM-1-N8R2` (8 MB Quad-SPI flash + 2 MB Quad-SPI PSRAM).
- [VERIFIED FROM KICAD] Section yang dibuat hanya mencakup MCU power/decoupling, EN/reset, BOOT, GPIO net staging, safe-state bias, status LED, serta UART0/JTAG test pads. Tidak ada rangkaian WIZ850io, RS485, DI, DO, sensor, USB-C, atau perubahan PCB.
- [VERIFIED FROM KICAD] `3V3_LOGIC` dan `LOGIC_GND` terhubung sebagai global nets. `PWR_GND` dan `LOGIC_GND` tetap satu domain non-isolated melalui power section yang sudah ada; tidak dibuat ground island baru.

## B. GPIO-map verification

| GPIO | Frozen function | KiCad connection | Result |
|---:|---|---|---|
| 0 | `BOOT_N` | U3 pin 27 | PASS |
| 1 / 2 | `DO1_CTRL` / `DO2_CTRL` | U3 pins 39 / 38 | PASS |
| 4 | `RS485_DIR` | U3 pin 4 | PASS |
| 5 / 6 / 7 | `SPARE_GPIO1/2/3` | U3 pins 5 / 6 / 7 | PASS |
| 8 / 9 / 10 | `ETH_RST_N` / `ETH_INT_N` / `ETH_CS_N` | U3 pins 12 / 17 / 18 | PASS |
| 11 / 12 / 13 | `ETH_MOSI` / `ETH_SCK` / `ETH_MISO` | U3 pins 19 / 20 / 21 | PASS |
| 14 | `ONEWIRE_DATA` | U3 pin 22 | PASS |
| 15 / 16 | `I2C_SDA` / `I2C_SCL` | U3 pins 8 / 9 | PASS |
| 17 / 18 | `RS485_TXD` / `RS485_RXD` | U3 pins 10 / 11 | PASS |
| 19 / 20 | `USB_D_N` / `USB_D_P` | U3 pins 13 / 14 | PASS |
| 21 | `STATUS_LED` | U3 pin 23 | PASS |
| 35 / 36 / 37 / 38 | `DI1_LOGIC`–`DI4_LOGIC` | U3 pins 28 / 29 / 30 / 31 | PASS |
| 39 / 40 / 41 / 42 | JTAG TCK/TDO/TDI/TMS test pads | U3 pins 32 / 33 / 34 / 35 | PASS |
| 43 / 44 | UART0 TX/RX test pads | U3 pins 37 / 36 | PASS |
| 47 / 48 | `DO3_CTRL` / `DO4_CTRL` | U3 pins 24 / 25 | PASS |

- [VERIFIED FROM KICAD] Strap pins GPIO3, GPIO45, dan GPIO46 diberi explicit no-connect; tidak ada beban tidak sengaja.
- [VERIFIED FROM DATASHEET] GPIO35–GPIO37 tersedia pada N8R2 karena pembatasan pin tersebut berlaku pada varian Octal-PSRAM, bukan N8R2 Quad-PSRAM.

## C. Power, reset, and boot implementation

- [VERIFIED FROM DATASHEET] Local supply network menggunakan `C11 = 22 uF` dan `C12 = 100 nF` dari `3V3_LOGIC` ke `LOGIC_GND`, mengikuti peripheral schematic Espressif.
- [VERIFIED FROM DATASHEET] EN tidak floating: `R6 = 10 kΩ` pull-up ke 3.3 V, `C13 = 1 uF` ke ground, dan `SW1 RESET` menarik EN ke ground. Nilai RC mengikuti rekomendasi hardware-design Espressif.
- [VERIFIED FROM DATASHEET] GPIO0 memakai `R7 = 10 kΩ` pull-up dan `SW2 BOOT` ke ground; tidak ada kapasitor besar yang membebani strap.
- [VERIFIED FROM KICAD] Semua pin ground module, termasuk exposed/ground pins, terhubung ke `LOGIC_GND`.
- [VERIFIED FROM KICAD] Catatan keepout antena dibuat prominent pada sheet: module harus berada di tepi PCB dan area antena bebas copper, plane, trace, serta komponen sesuai guideline Espressif.

## D. Hardware-bias values and justification

| Signal | Network | Verification / reason |
|---|---|---|
| `DO1_CTRL`–`DO4_CTRL` | `R8`–`R11 = 47 kΩ` pull-down | [VERIFIED FROM DATASHEET + ENGINEERING CALCULATION] ZXMS6005N8Q menerima logic 3.3 V dan input current maksimum 100 uA pada 3 V. Pull-down menahan input LOW saat GPIO high-Z; saat HIGH, beban resistor hanya sekitar 70 uA pada 3.3 V. Output default OFF. |
| `RS485_DIR` | `R12 = 10 kΩ` pull-down | [VERIFIED FROM DATASHEET + ENGINEERING CALCULATION] Dengan worst-case gabungan leakage `DE` dan `/RE` sekitar 20 uA, kenaikan maksimum sekitar 0.2 V, di bawah `VIL` maksimum yang diizinkan pada 3.3 V. Reset state menjadi receive-only. |
| `ETH_CS_N` | `R13 = 10 kΩ` pull-up | [VERIFIED FROM DATASHEET] Memperkuat internal pull-up W5500 dan memastikan SPI deselected selama MCU reset. |
| `ETH_RST_N` | `R14 = 10 kΩ` pull-down | [VERIFIED FROM DATASHEET + ENGINEERING CALCULATION] Melawan worst-case internal pull-up 50 kΩ menghasilkan sekitar 0.55 V, masih LOW terhadap batas input W5500; firmware harus meng-drive HIGH lalu menunggu minimum 50 ms sebelum SPI. |
| `ETH_INT_N` | Tanpa pull-up eksternal | [VERIFIED FROM DATASHEET] `INTn` W5500 diklasifikasikan sebagai output aktif-low, bukan open-drain; pull-up board-level tidak diperlukan. |
| `STATUS_LED` | `R15 = 1 kΩ`, green LED `D4`, `R16 = 100 kΩ` pull-down | [ENGINEERING CALCULATION] LED aktif-HIGH dan tetap OFF saat reset. Arus indikator diperkirakan sekitar 1 mA, cukup rendah untuk GPIO; brightness perlu dikonfirmasi di bench. |
| I2C / 1-Wire | Belum dipasang pull-up | [FINAL DECISION] Nilai ditunda sampai bus voltage, kapasitansi, panjang kabel, dan jumlah perangkat dikunci pada fase sensor/service. |

## E. ERC result

- [VERIFIED FROM KICAD] Fresh ERC: **0 errors, 18 warnings, 0 info**.
- [VERIFIED FROM KICAD] Semua 18 warning adalah `Global label not connected anywhere else in the schematic` untuk net peripheral staged: `SPARE_GPIO1–3`, `I2C_SDA/SCL`, `RS485_TXD/RXD`, `USB_D_N/P`, `ETH_INT_N/MOSI/SCK/MISO`, `ONEWIRE_DATA`, dan `DI1_LOGIC–DI4_LOGIC`.
- [ENGINEERING DECISION] Warning tidak disuppress karena net tersebut memang belum memiliki endpoint kedua; warning akan diselesaikan ketika peripheral terkait dicapture.
- [VERIFIED FROM KICAD] Parent dan core schematic valid melalui KiCad CLI; core memiliki 0 orphan wire, 0 overlap, dan 0 off-grid geometry pada grid 1.27 mm.
- [VERIFIED FROM KICAD] Phase 3A/3B component set dan electrical topology tidak diubah; parent hanya mendapat hierarchical-sheet link dan global power-net continuity yang diperlukan oleh core.

## F. Unresolved issues

- [NEEDS VERIFICATION] Pilih exact manufacturer part number C11/C12/C13 dan pastikan effective capacitance terhadap DC bias/temperature sebelum BOM release.
- [NEEDS VERIFICATION] Verifikasi brightness `LTST-C190KGKT` pada arus sekitar 1 mA di prototype.
- [NEEDS VERIFICATION] Finalisasi pull-up I2C dan 1-Wire pada fase interface masing-masing.
- [NEEDS VERIFICATION] Warning ERC staged-net diselesaikan, bukan dikecualikan, saat endpoint peripheral dibuat.
- [NEEDS VERIFICATION] Reset timing Ethernet dan semua safe-state output diuji pada power-up, manual reset, watchdog reset, dan brownout prototype.

## G. GO / NO-GO

**CONDITIONAL GO untuk peripheral schematic phases.** ESP32 core, frozen GPIO mapping, power/reset/boot, native USB net reservation, debug pads, dan safe-state bias sudah tercapture serta lolos ERC tanpa error. Kondisi GO mensyaratkan setiap fase peripheral berikutnya menyelesaikan staged-net warning yang relevan dan memverifikasi interface-specific components. Phase peripheral belum dimulai.

> **Phase 5A correction:** WIZ850io revision 1.1 contains onboard 4.7 kΩ pull-ups on SCSn, INTn, and RSTn. Therefore the earlier MCU-side ETH_CS_N 10 kΩ pull-up was removed as redundant, and the ETH_RST_N pull-down was changed from 10 kΩ to 1 kΩ so reset remains guaranteed LOW against the module pull-up. This correction supersedes the corresponding Phase 4B bias entries; the GPIO map is unchanged.
