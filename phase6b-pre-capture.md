# Phase 6B — 4-Channel 12–24 V Digital Output Pre-Capture Gate

## A. Existing-project verification

- [VERIFIED FROM KICAD] Project aktif adalah `hardware/kicad_mcp_test.kicad_pro` dengan root `hardware/kicad_mcp_test.kicad_sch`.
- [VERIFIED FROM KICAD] Hierarchy berisi `ESP32_CORE`, `ETHERNET_WIZ850IO`, `RS485_ISOLATED`, dan `DIGITAL_INPUTS_ISOLATED`.
- [VERIFIED FROM KICAD] ERC awal tepat **0 error / 12 warning**.
- [VERIFIED FROM KICAD] GPIO tetap GPIO1/GPIO2/GPIO47/GPIO48 untuk `DO1_CTRL`–`DO4_CTRL`; masing-masing sudah memiliki pull-down 47 kΩ ke `LOGIC_GND`.

## B. ZXMS6005N8Q datasheet findings

- [VERIFIED FROM DIODES INCORPORATED] `ZXMS6005N8Q-13` adalah protected low-side IntelliFET, bukan discrete MOSFET biasa.
- [VERIFIED FROM DIODES INCORPORATED] Package SO-8: pin 1–3 = Source, pin 4 = IN, dan pin 5–8 = Drain.
- [VERIFIED FROM DIODES INCORPORATED] Recommended input range 0–5.5 V; `VIH(min)=3.0 V`, `VIL(max)=0.7 V`. Input current maksimum 100 µA pada 3 V dan 200 µA pada 5 V.
- [VERIFIED FROM DIODES INCORPORATED] `VDS=60 V`, tetapi short-circuit-protection rating `VDS(SC)=24 V`. Clamp voltage 60–70 V dan unclamped inductive energy 120 mJ pada 0.5 A/24 V.
- [VERIFIED FROM DIODES INCORPORATED] `RDS(on)` maksimum 250 mΩ pada `VIN=3 V` dan 200 mΩ pada 5 V. Proteksi internal mencakup overcurrent, overtemperature with auto-restart, active overvoltage clamp, short-circuit auto-restart, dan ESD.
- [VERIFIED FROM DIODES INCORPORATED] Continuous-current capability bergantung pada copper/thermal PCB; datasheet memberi minimal 1.4 A pada 3 V dengan minimum pad dan 1.9 A dengan 1-inch-square copper pada 25 °C.

Referensi utama: [Diodes Incorporated ZXMS6005N8Q datasheet](https://www.diodes.com/datasheet/download/ZXMS6005N8Q.pdf).

## C. 3.3 V logic-drive margin

- [ENGINEERING CALCULATION] Dengan output GPIO ideal 3.3 V, margin terhadap `VIH(min)=3.0 V` hanya **0.30 V**.
- [ENGINEERING CALCULATION] Pull-down 47 kΩ menarik sekitar 70 µA pada 3.3 V. Ditambah input-current maksimum IntelliFET 100 µA, total beban GPIO hanya sekitar 170 µA; secara tipikal pin akan mendekati rail.
- [ENGINEERING CALCULATION] Namun spesifikasi minimum `VOH` ESP32-S3 adalah `0.8 × VDD`; pada 3.3 V nilainya **2.64 V**, yaitu **0.36 V di bawah** `VIH(min)` ZXMS. Pada rail minimum 3.0 V, bahkan drive ideal hanya memberi margin nol.
- [ENGINEERING DECISION] Direct-drive ESP32 → ZXMS6005N8Q tidak memiliki jaminan HIGH worst-case lintas datasheet. Karena prompt mewajibkan berhenti bila margin terlalu kecil, schematic capture **tidak dilakukan**.
- [ENGINEERING DECISION] Koreksi yang direkomendasikan untuk review: buffer 5 V ber-input TTL seperti `SN74AHCT125`, dengan downstream pull-down pada tiap input ZXMS agar tetap OFF ketika buffer unpowered/tri-state. TI menjamin `VIH=2.0 V` untuk buffer dan `VOH≥4.4 V`, sehingga margin formal menjadi ≥0.64 V pada sisi ESP32 nominal dan ≥1.4 V pada sisi ZXMS.
- [NEEDS VERIFICATION] Nilai downstream pull-down, enable/power-sequencing buffer, exact buffer variant/package, dan efek back-power harus dikunci sebelum capture.

Referensi ESP32: [Espressif ESP32-S3 datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf). Kandidat koreksi: [TI SN74AHCT125](https://www.ti.com/product/SN74AHCT125).

## D. 0.5 A/channel electrical and thermal analysis

- [ENGINEERING CALCULATION] Dengan `RDS(on,max)=250 mΩ`: drop maksimum pada 0.5 A adalah **0.125 V**, loss **62.5 mW/channel**, dan empat kanal bersamaan **250 mW total**.
- [ENGINEERING CALCULATION] Datasheet `RθJA=98 °C/W` pada minimum pad memberi kenaikan sekitar **6.1 °C per device** dari conduction loss; 1-inch-square copper dengan `RθJA=76 °C/W` memberi sekitar **4.8 °C**.
- [ENGINEERING DECISION] Target board 0.5 A/channel masuk akal secara steady-state awal dan jauh di bawah device-current rating, tetapi bukan bukti prototype.
- [NEEDS VERIFICATION] Kenaikan `RDS(on)` terhadap temperatur, copper area aktual, empat sumber panas berdekatan, terminal/wire heating, ambient tinggi, inrush, PWM duty cycle, dan short-circuit cycling harus diverifikasi pada PCB/prototype.
- [VERIFIED FROM DIODES INCORPORATED] Rating short-circuit protection hanya 24 V. [ENGINEERING DECISION] Pada sistem 24 V nominal tidak ada margin untuk supply tolerance/transient saat kondisi short; jangan menganggap proteksi internal sebagai pengganti fuse/supply current limit eksternal.

## E. Circuit created

- [VERIFIED FROM KICAD] **Tidak ada circuit atau hierarchical sheet Phase 6B yang dibuat.**
- [VERIFIED FROM KICAD] `hardware/digital_outputs.kicad_sch` tidak dibuat dan semua schematic existing dibiarkan tidak berubah.

## F. Startup-safe-state verification

- [VERIFIED FROM KICAD] Keempat `DOx_CTRL` memiliki pull-down 47 kΩ existing ke `LOGIC_GND`; GPIO1, GPIO2, GPIO47, dan GPIO48 bukan strapping pins ESP32-S3.
- [ENGINEERING CALCULATION] Saat GPIO high-impedance dan tidak ada sumber lain pada node direct-drive, pull-down menetapkan `VIN≈0 V`, jauh di bawah `VIL(max)=0.7 V`, sehingga output OFF.
- [ENGINEERING DECISION] Pull-down existing tidak perlu diduplikasi untuk direct-drive. Jika buffer ditambahkan, pull-down kedua pada sisi output buffer menjadi terjustifikasi karena node input ZXMS dapat high-impedance ketika buffer mati/disabled.

## G. Inductive-load / flyback decision

- [VERIFIED FROM DIODES INCORPORATED] Active clamp dan 120 mJ single-pulse inductive rating tersedia, tetapi nilai ini bukan izin untuk repetitive-clamp tanpa analisis energi dan temperatur.
- [ENGINEERING DECISION] External flyback diode **RECOMMENDED** untuk relay/solenoid agar energi clamp dan EMI berkurang. Topologi yang benar: anode di `DOx_OUT`, cathode di external `DO_FIELD_V+`.
- [ENGINEERING DECISION] Diode dapat dibuat DNP/optional bila load sudah memiliki suppression sendiri. Exact diode MPN belum dipilih karena capture dihentikan.

## H. Output protection decision

- **MANDATORY:** [ENGINEERING DECISION] External load supply harus memiliki current limit/fuse yang sesuai wiring dan load; jangan mengandalkan thermal auto-restart untuk short berkepanjangan.
- **RECOMMENDED:** [ENGINEERING DECISION] Flyback per channel, PCB thermal copper sesuai datasheet, dan connector/wiring rated ≥0.5 A/channel.
- **OPTIONAL:** [ENGINEERING DECISION] Connector-side ESD/TVS tambahan setelah threat model kabel dan repetitive transient ditentukan.
- **INTENTIONALLY OMITTED:** [ENGINEERING DECISION] MOSFET-style gate resistor, gate-source pull-down, current-sense/diagnostic circuitry, dan TVS redundan yang belum memiliki koordinasi clamp jelas.

## I. Connector / field-supply architecture

- [ENGINEERING DECISION] Enam posisi diperlukan bila board menyediakan flyback: `DO_FIELD_V+`, `DO1_OUT`, `DO2_OUT`, `DO3_OUT`, `DO4_OUT`, `DO_FIELD_GND`.
- Halaman resmi Phoenix Contact mengonfirmasi kandidat `MKDS 1,5/ 6-5,08 BD:1-6`, MPN **1706303**, 6 posisi, pitch 5.08 mm, 17.5 A nominal, dan 400 V.
- [VERIFIED FROM KICAD] Footprint tersedia: `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-6-5.08_1x06_P5.08mm_Horizontal`.
- [ENGINEERING DECISION] `DO_FIELD_V+` akan merupakan external 12–24 V only; tidak boleh terhubung ke `VIN_FIELD`, `5V_MAIN`, `3V3_LOGIC`, atau `AUX_5V`.

Referensi: [Phoenix Contact 1706303](https://www.phoenixcontact.com/en-de/products/pcb-terminal-block-mkds-15-6-508-bd1-6-1706303).

## J. Ground-domain audit

- [VERIFIED FROM KICAD] Kondisi proyek tetap: `LOGIC_GND/PWR_GND`, `DI_FIELD_GND`, dan `RS485_GND` adalah net terpisah.
- [ENGINEERING DECISION] Pada capture berikutnya, `DO_FIELD_GND` harus menjadi alias/intentional DC connection ke `LOGIC_GND/PWR_GND`, bukan domain isolated baru.
- [VERIFIED FROM KICAD] Karena capture dibatalkan, tidak ada bridge baru ke `DI_FIELD_GND` atau `RS485_GND`.

## K. ERC before / after

- [VERIFIED FROM KICAD] Sebelum analisis: **0 error / 12 warning**.
- [VERIFIED FROM KICAD] Setelah analisis: **0 error / 12 warning** karena schematic tidak dimodifikasi.
- [VERIFIED FROM KICAD] Tidak ada ERC suppression baru.

## L. Remaining unresolved items

- [NEEDS VERIFICATION] User decision untuk menambahkan 5 V TTL-input buffer atau mengganti output switch dengan device yang memiliki guaranteed `VIH` lebih rendah.
- [NEEDS VERIFICATION] Jika buffer dipilih: exact MPN/package, OE strategy, downstream pull-down, decoupling, dan power sequencing.
- [NEEDS VERIFICATION] Exact flyback diode MPN dan repetitive inductive energy untuk load target.
- [NEEDS VERIFICATION] Thermal copper, fuse/current-limit external, connector arrangement, dan transient strategy pada PCB/prototype.
- [VERIFIED FROM KICAD] Tidak ada symbol stock `ZXMS6005N8Q`; custom symbol dengan pin 1–3 S, 4 IN, 5–8 D akan diperlukan. Footprint stock SOIC-8 3.9×4.9 mm/P1.27 tersedia.

## M. Recommendation

**NO-GO untuk Phase 6B schematic capture dengan direct-drive yang sekarang.**

[ENGINEERING DECISION] Device dan target 0.5 A/channel layak, tetapi interface logic belum memiliki guaranteed HIGH margin. Review dan approve level-shifting/buffer solution terlebih dahulu; setelah itu Phase 6B dapat dilanjutkan tanpa mengubah frozen GPIO map.

Sensors, USB-C, AUX 5 V, PCB placement, dan PCB routing tidak dimulai.
