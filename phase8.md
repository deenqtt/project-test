# Phase 8 — USB-C Native USB + ESD Protection

## A. Existing-project verification

- [VERIFIED FROM KICAD] Project root adalah `hardware/kicad_mcp_test.kicad_sch`; sebelum Phase 8 terdapat sheet `ESP32_CORE`, `ETHERNET_WIZ850IO`, `RS485_ISOLATED`, `DIGITAL_INPUTS_ISOLATED`, `DIGITAL_OUTPUTS`, dan `SENSORS_SERVICE`.
- [VERIFIED FROM KICAD] Baseline fresh ERC adalah **0 error / 7 warning**.
- [VERIFIED FROM KICAD] Sebelum capture, `USB_D_N` hanya terhubung ke U3/GPIO19 dan `USB_D_P` hanya ke U3/GPIO20.
- [VERIFIED FROM KICAD] Rail existing tetap `VIN_FIELD → 5V_MAIN → 3V3_LOGIC`; tidak ada USB power path existing.

## B. ESP32-S3 native USB findings

- [VERIFIED FROM ESPRESSIF] ESP32-S3 memiliki internal full-speed USB OTG PHY dan USB Serial/JTAG controller; keduanya berbagi satu internal PHY sehingga tidak dapat digunakan bersamaan.
- [VERIFIED FROM ESPRESSIF] GPIO19 adalah D− dan GPIO20 adalah D+. USB Serial/JTAG mendukung flashing, serial console, serta JTAG tanpa USB-UART bridge eksternal.
- [VERIFIED FROM ESPRESSIF] Pull-up USB dikendalikan internal oleh USB peripheral; external D+ pull-up tidak ditambahkan.
- [VERIFIED FROM ESPRESSIF] Espressif merekomendasikan reservasi resistor seri 22/33 Ω dan optional unpopulated shunt capacitors dekat chip.
- [VERIFIED FROM ESPRESSIF] Karena board ini self-powered dari 12–24 V, VBUS monitoring wajib untuk USB device/TinyUSB. GPIO7 telah disetujui sebagai `USB_VBUS_SENSE`.

Referensi: [ESP32-S3 hardware design guidelines](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html), [ESP-IDF self-powered USB guidance](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/peripherals/usb_device.html).

## C. USB-C connector / MPN

- [VERIFIED FROM CONNECTOR MANUFACTURER] Dipilih GCT `USB4105-GF-A`: top-mount horizontal USB 2.0 Type-C receptacle, −40…+85 °C, rated 48 V, dengan 20,000 mating cycles pada current manufacturer drawing/specification.
- [VERIFIED FROM CONNECTOR MANUFACTURER] Pinout: A4/A9/B4/B9 VBUS; A5 CC1; B5 CC2; A6/B6 D+; A7/B7 D−; A8/B8 SBU; A1/A12/B1/B12 GND; shell sebagai shield.
- [VERIFIED FROM KICAD] Symbol `Connector:USB_C_Receptacle_USB2.0_16P` dan footprint `Connector_USB:USB_C_Receptacle_GCT_USB4105-xx-A_16P_TopMnt_Horizontal` tersedia. Footprint memuat semua contact pad, NPTH locating holes, dan shell stake `S1`.

Referensi: [GCT USB4105 drawing](https://gct.co/files/drawings/usb4105.pdf), [GCT USB4105 product specification](https://gct.co/files/specs/usb4105-spec.pdf).

## D. CC1 / CC2 configuration

- [VERIFIED FROM CONNECTOR MANUFACTURER] J10/A5 dan J10/B5 adalah CC1 dan CC2.
- [ENGINEERING DECISION] `R41=5.1 kΩ 1%` dari CC1 ke `LOGIC_GND` dan `R42=5.1 kΩ 1%` dari CC2 ke `LOGIC_GND` menetapkan connector sebagai USB-C device/UFP.
- [VERIFIED FROM KICAD] CC1 dan CC2 memakai resistor terpisah; keduanya tidak disatukan. SBU1/SBU2 diberi explicit no-connect.

Referensi nilai Rd: [USB Type-C Cable and Connector Specification Release 2.0](https://www.usb.org/sites/default/files/USB%20Type-C%20Spec%20R2.0%20-%20August%202019_0.pdf).

## E. USBLC6-2SC6 implementation

- [VERIFIED FROM STMICROELECTRONICS] `USBLC6-2SC6` memakai SOT23-6L: pin 1/6 I/O1, pin 3/4 I/O2, pin 2 GND, dan pin 5 VBUS. Working reverse voltage diuji pada 5.25 V, breakdown minimum 6 V, capacitance I/O-to-GND maksimum 3.5 pF, dan device dinyatakan sesuai USB 2.0 hingga high-speed.
- [VERIFIED FROM KICAD] D12 menggunakan stock symbol `Power_Protection:USBLC6-2SC6` dan exact footprint `Package_TO_SOT_SMD:SOT-23-6`.
- [VERIFIED FROM KICAD] D− melewati J10 A7/B7 → D12 pin 1/6 → R39 → `USB_D_N`; D+ melewati J10 A6/B6 → D12 pin 3/4 → R40 → `USB_D_P`. D12 pin 5 memakai connector-side `USB_VBUS`; pin 2 memakai `LOGIC_GND`.

Referensi: [ST USBLC6-2 datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf).

## F. D+ / D− series-resistor decision

- [ENGINEERING DECISION] Resistor seri diklasifikasikan **RECOMMENDED**, bukan mandatory. `R39=R40=22 Ω, 1%` dipopulasi sebagai initial Espressif-recommended value.
- [VERIFIED FROM KICAD] Resistor berada di sisi MCU setelah ESD protection dan tidak mengubah polaritas D+/D−.
- [ENGINEERING DECISION] Shunt capacitors tidak ditambahkan karena Espressif menyebutnya optional/unpopulated dan belum ada stackup atau SI measurement.

## G. USB VBUS architecture

- [ENGINEERING DECISION] `USB_VBUS` adalah connector-side ESD reference dan VBUS monitor source saja; port USB tidak memberi daya ke board.
- [VERIFIED FROM KICAD] `USB_VBUS` hanya berisi J10 A4/A9/B4/B9, D12/5, dan R43/1.
- [ENGINEERING CALCULATION] Divider memakai `R43=91 kΩ 1%` dan `R44=130 kΩ 1%`. Ratio nominal 0.5882 dan current pada 5 V sekitar 22.6 µA.
- [ENGINEERING CALCULATION] Worst-case ratio minimum 0.5834 memberi 2.567 V saat VBUS 4.4 V. Terhadap `0.75×3.366 V=2.525 V`, preliminary guaranteed HIGH margin sekitar **42 mV**.
- [ENGINEERING CALCULATION] Worst-case ratio maksimum 0.5931 memberi 3.114 V saat VBUS 5.25 V, di bawah preliminary GPIO absolute limit `3V3(min)+0.3 V≈3.534 V`.
- [ENGINEERING CALCULATION] Tanpa filter capacitor, total node capacitance harus melebihi sekitar 17 nF sebelum discharge melalui 130 kΩ melampaui 3 ms. GPIO/trace parasitic yang diharapkan jauh lebih kecil.
- [VERIFIED FROM KICAD] GPIO7/U3, divider midpoint, dan J8 pin 2 tergabung sebagai `USB_VBUS_SENSE`; GPIO6 tetap `SPARE_GPIO2`.

## H. Shield / grounding decision

- [ENGINEERING DECISION] Karena tidak ada chassis/PE domain nyata, J10 shield `S1` dihubungkan langsung ke `LOGIC_GND`. Tidak dibuat fake chassis ground atau isolated USB ground.
- [VERIFIED FROM KICAD] Connector GND, shield, CC return, ESD return, dan divider return semuanya berada pada `LOGIC_GND`.

## I. Circuit created

- [VERIFIED FROM KICAD] Sheet baru `USB_C` dibuat sebagai `hardware/usb_c.kicad_sch` dan ditautkan ke root sebagai page 8.
- [VERIFIED FROM KICAD] Sheet berisi J10, D12, R39–R44, explicit no-connect SBU pins, manufacturer metadata, serta catatan VBUS isolation dan USB layout.
- [VERIFIED FROM KICAD] J8 diperbarui menjadi `SERVICE_GPIO6_VBUSSENSE_1W_3V3_GND`; pin 2 kini input/test `USB_VBUS_SENSE` dan tidak boleh didrive eksternal.
- [VERIFIED FROM KICAD] Dokumentasi GPIO `phase4.md` diperbarui: GPIO6 spare, GPIO7 VBUS sense.

## J. ERC before / after

- [VERIFIED FROM KICAD] Sebelum Phase 8: **0 error / 7 warning**.
- [VERIFIED FROM KICAD] Setelah Phase 8: **0 error / 5 warning**.
- [VERIFIED FROM KICAD] Empat warning adalah off-grid existing pada RS485; satu warning adalah intentional alias `DO_FIELD_GND`/`LOGIC_GND`. Tidak ada warning baru dari USB dan tidak ada ERC suppression.

## K. Netlist / backfeed audit

- [VERIFIED FROM KICAD] `USB_D_N` berisi R39/2 dan U3/GPIO19; `USB_D_P` berisi R40/2 dan U3/GPIO20.
- [VERIFIED FROM KICAD] `USB_VBUS_SENSE` berisi J8/2, R43/2, R44/1, dan U3/GPIO7.
- [VERIFIED FROM KICAD] `USB_VBUS` merupakan hierarchical-local net yang terpisah dari `5V_MAIN`, `3V3_LOGIC`, dan `VIN_FIELD`; `AUX_5V` tidak ada dalam generated netlist.
- [VERIFIED FROM KICAD] Tidak ada bridge USB ke `DI_FIELD_GND` atau `RS485_GND`.

## L. PCB-layout requirements for later

- [VERIFIED FROM ESPRESSIF] Route D+/D− sebagai differential pair 90 Ω ±10%, parallel dan length-matched, dengan minimum via transitions dan continuous ground reference.
- [ENGINEERING DECISION] Tempatkan J10 di edge PCB, D12 sedekat mungkin ke connector, dan R39/R40 dekat sisi ESP32. Jaga path pendek, stub minimal, dan jangan melewati isolation gap.
- [ENGINEERING DECISION] Exact width/spacing belum ditentukan sebelum stackup 2-layer dan impedance capability fabricator tersedia.

## M. Remaining unresolved items

- [NEEDS VERIFICATION] Margin divider sekitar 42 mV bergantung pada preliminary 3V3 tolerance dan assumed 5.25 V USB VBUS maximum; konfirmasi dengan final TLV62569 rail tolerance dan prototype attach/unplug test.
- [NEEDS VERIFICATION] Pastikan J8 pin 2 tidak diload atau didrive oleh accessory eksternal karena kini merupakan sensing node.
- [NEEDS VERIFICATION] Nilai 22 Ω dapat dituning menjadi 33 Ω atau 0 Ω hanya setelah layout/SI testing; jangan menambahkan shunt capacitance tanpa measurement.
- [NEEDS VERIFICATION] Availability `USB4105-GF-A`, exact production resistor MPN, connector edge placement, dan mechanical enclosure clearance.
- [VERIFIED FROM ESPRESSIF] USB OTG dan USB Serial/JTAG berbagi internal PHY; firmware harus memilih controller yang digunakan, bukan mengoperasikan keduanya bersamaan.

## N. Recommendation

**CONDITIONAL GO — Phase 8 schematic capture lengkap.**

[ENGINEERING DECISION] Arsitektur menyediakan native USB-C service dengan UFP CC termination, ESD, recommended series damping, mandatory self-powered VBUS sensing, serta tanpa USB-to-main-power path. Item bagian M tetap menjadi checkpoints sebelum footprint freeze dan PCB routing.

Phase 8 berhenti di schematic. AUX current limiting, cleanup global, footprint freeze, PCB placement/routing, dan manufacturing outputs tidak dimulai.

---

# Phase 8-R1 — USB VBUS Sense Margin Resolution

Bagian ini menggantikan analisis divider dan rekomendasi freeze Phase 8 sebelumnya.

## A. Authoritative voltage limits

- [VERIFIED FROM ESPRESSIF] VBUS valid di atas 4.75 V dan invalid di bawah 4.35 V. Untuk passive sensing, Espressif mensyaratkan node sense mencapai `0.75 × VDD` pada `VBUS = 4.4 V`; node harus menjadi logic LOW maksimal 3 ms setelah unplug.
- [VERIFIED FROM USB-IF] USB Type-C 5 V source dapat mencapai 5.5 V. Analisis maksimum memakai `VBUS_MAX = 5.5 V`.
- [VERIFIED FROM ESPRESSIF] ESP32-S3: `VIH(min)=0.75×VDD`, `VIL(max)=0.25×VDD`, input-high ceiling `VDD+0.3 V`, input leakage maksimum 50 nA, input capacitance tipikal 2 pF.
- [ENGINEERING CALCULATION] Rail Phase 3B, memakai `VFB=0.588…0.612 V`, `R4=453 kΩ ±1%`, dan `R5=100 kΩ ±1%`: `3V3_LOGIC = 3.1989…3.4404 V`. Maka worst-case `VIH=2.5803 V`, `VIL=0.7997 V`, dan powered GPIO ceiling `=3.4989 V`.

Referensi: [ESP32-S3 datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf), [Espressif self-powered USB guidance](https://docs.espressif.com/projects/esp-idf/en/v5.5/esp32s3/api-reference/peripherals/usb_device.html), [USB Type-C specification](https://www.usb.org/sites/default/files/USB%20Type-C%20Spec%20R2.0%20-%20August%202019_0.pdf), [TLV62569 datasheet](https://www.ti.com/lit/ds/symlink/tlv62569.pdf).

## B. Existing 91 kΩ / 130 kΩ

- [ENGINEERING CALCULATION] Dengan toleransi 1%: ratio minimum `0.583382`, ratio maksimum `0.593071`.
- [ENGINEERING CALCULATION] Pada 4.4 V dan worst-case leakage 50 nA yang menarik node turun: `VSENSE_MIN=2.5642 V`; guaranteed HIGH margin `=−16.1 mV`. Existing divider **gagal** freeze criterion.
- [ENGINEERING CALCULATION] Pada 5.5 V dan leakage berlawanan: `VSENSE_MAX=3.2646 V`; powered-GPIO margin `=234.3 mV`.

## C. Candidate divider calculations

Semua kandidat 1%, dihitung pada 4.4/5.5 V, rail 3.1989…3.4404 V, dan ±50 nA GPIO leakage.

| Rtop / Rbottom | VSENSE min | HIGH margin | VSENSE max | Max-voltage margin | Worst divider current |
|---|---:|---:|---:|---:|---:|
| 82 kΩ / 130 kΩ | 2.6747 V | 94.4 mV | 3.4012 V | 97.7 mV | 26.2 µA |
| 47 kΩ / 75 kΩ | 2.6826 V | 102.3 mV | 3.4086 V | 90.3 mV | 45.5 µA |
| **39 kΩ / 62 kΩ** | **2.6789 V** | **98.6 mV** | **3.4035 V** | **95.4 mV** | **55.0 µA** |

## D–G. Selected solution and timing

- [ENGINEERING DECISION] Pilihan pasif: `R43=39 kΩ 1%`, `R44=62 kΩ 1%`. Nilai standar, margin seimbang, dan impedansi lebih rendah terhadap leakage/noise dibanding existing pair.
- [ENGINEERING CALCULATION] Guaranteed HIGH margin: **98.6 mV**. Powered GPIO maximum-voltage margin: **95.4 mV**.
- [ENGINEERING CALCULATION] Total divider dissipation maksimum sekitar 0.303 mW pada 5.5 V.
- [ENGINEERING CALCULATION] Agar turun dari worst-case 3.4035 V ke 0.7997 V dalam 3 ms, total node capacitance harus `<33.1 nF`. Tidak ada filter capacitor; input capacitance ESP32-S3 hanya 2 pF tipikal. Verifikasi parasitic aktual tetap diperlukan.
- [ENGINEERING DECISION] Divider tetap signal-only; tidak ada koneksi schematic ke `5V_MAIN`, `3V3_LOGIC`, `VIN_FIELD`, atau `AUX_5V`. Kondisi USB terpasang saat `3V3_LOGIC=0 V` masih membutuhkan verifikasi injection/back-power karena Espressif tidak memberi guaranteed GPIO injection-current limit pada datasheet yang ditinjau.

## H. KiCad changes

- **Tidak ada perubahan KiCad diterapkan.** MCP membaca keenam resistor sheet `usb_c.kicad_sch` sebagai reference identik `R?`; root netlist juga masih menampilkan simbol USB sebagai `R?`, `J?`, dan `D?`.
- MCP edit membutuhkan reference unik dan tidak menyediakan selector UUID/coordinate. Mengubah `R?` tidak aman karena dapat mengenai resistor CC/data lain.
- Sesuai instruksi “jika MCP tidak dapat safely update existing element, STOP”, direct text edit dan global annotation tidak dilakukan.

## I. ERC / net audit

- [VERIFIED FROM KICAD] Fresh ERC sebelum perubahan: **1 error / 5 warning**. Karena perubahan diblokir, tidak ada ERC-after yang dapat diklaim.
- [VERIFIED FROM KICAD] Netlist saat ini: `USB_VBUS` hanya ke connector, ESD reference, dan divider-top; `USB_VBUS_SENSE` ke GPIO7, J8 pin 2, dan divider; tidak tergabung dengan `5V_MAIN`, `3V3_LOGIC`, `VIN_FIELD`, atau `AUX_5V`.

## J. Recommendation

**NO-GO untuk schematic freeze.** Solusi listrik `39 kΩ / 62 kΩ` memenuhi powered-state margin, tetapi belum dapat diterapkan secara aman melalui MCP karena simbol USB belum memiliki reference unik. Phase 9 tidak dimulai.
