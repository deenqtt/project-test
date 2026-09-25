Phase 2 — Component Selection & Datasheet Verification
Status: selesai secara read-only. Tidak ada schematic atau PCB yang dibuat/diubah.
Label yang digunakan:

- [VERIFIED FROM DATASHEET]
- [VERIFIED FROM KICAD]
- [ENGINEERING ESTIMATE]
- [NEEDS VERIFICATION]
  Kesimpulan utama
  [ENGINEERING ESTIMATE] Arsitektur Rev A yang paling masuk akal:
- Input utama: proteksi fuse/PTC + reverse-polarity + TVS 33 V.
- Buck utama: LMR36510ADDA, 65 V input, 1 A.
- Rail 3.3 V: TLV62569DBV, 2 A dari rail 5 V.
- RS485: ADM2587EBRWZ.
- Digital input: dua buah ISO1212DBQR, masing-masing dua channel.
- Digital output: empat buah ZXMS6005N8Q, low-side protected MOSFET 60 V.
- Ethernet: modul WIZ850io.
- USB-C: Amphenol 124019772112A atau alternatif JAE DX07S016JA1R1500.
- Terminal: Phoenix Contact MKDS 1,5, pitch 5.08 mm.
  [ENGINEERING ESTIMATE] Output Rev A sebaiknya dibatasi pada sekitar 0.5 A kontinu per channel, bukan high-current industrial output. Beban seperti relay coil, lampu indikator, solenoid kecil, dan DC load kecil masih sesuai.
  A. Final Candidate BOM
  Fungsi Komponen yang direkomendasikan Status
  MCU ESP32-S3-WROOM-1 [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  Ethernet WIZ850io [VERIFIED FROM DATASHEET]
  Input buck LMR36510ADDA [VERIFIED FROM DATASHEET]
  3.3 V buck TLV62569DBV [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  RS485 isolated ADM2587EBRWZ [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  RS485 TVS SM712.TCT [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  Digital input 2 × ISO1212DBQR [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  Output MOSFET 4 × ZXMS6005N8Q [VERIFIED FROM DATASHEET] [NEEDS VERIFICATION] symbol/footprint
  Input TVS SMBJ33A [VERIFIED FROM DATASHEET]
  Reverse protection STPS3L60 atau ideal-diode alternative [VERIFIED FROM DATASHEET]
  USB-C ESD USBLC6-2SC6 [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  USB-C connector Amphenol 124019772112A [VERIFIED FROM DATASHEET] [NEEDS VERIFICATION] mechanical footprint
  BME280 External module/header [VERIFIED FROM DATASHEET]
  DS18B20 External 3-pin connector [VERIFIED FROM DATASHEET] [VERIFIED FROM KICAD]
  5 V auxiliary Fused 2-pin connector, target 250 mA [ENGINEERING ESTIMATE]
  Input terminal Phoenix 1715789, 8-position [VERIFIED FROM DATASHEET]
  Output terminal Phoenix 1710726, 6-position [VERIFIED FROM DATASHEET]

ESP32-S3-WROOM-1 membutuhkan supply digital 3.0–3.6 V; Espressif juga merekomendasikan regulator 3.3 V dengan kemampuan minimal 500 mA dan kapasitor input utama minimal 10 µF. VERIFIED FROM DATASHEET
B. Component Comparison Table

1. Proteksi input 12–24 V
   Kandidat Rating/package Kelebihan Kekurangan Keputusan
   Littelfuse SMBJ33A VRWM 33 V, VBR 36.7–40.6 V, clamp 53.3 V, 600 W, SMA/DO-214AA Umum, mudah dicari, cocok untuk transient rail 24 V Clamp 53.3 V berarti buck harus tahan lebih dari 53 V Dipilih dengan buck 65 V
   Littelfuse SMCJ33A VRWM 33 V, 1500 W, SMC/DO-214AB Lebih kuat terhadap surge Lebih besar dan mahal Alternatif
   Littelfuse SM8S33A Sekitar 6600 W, DO-218AB Sangat kuat Overkill untuk portfolio board 2-layer Ditolak untuk Rev A

SMBJ33A memiliki VBR minimum 36.7 V dan clamping voltage 53.3 V. VERIFIED FROM DATASHEET
[ENGINEERING ESTIMATE] Karena clamp TVS dapat mencapai sekitar 53 V, LMR51430 yang maksimum 36 V tidak ideal sebagai regulator pertama jika input transient belum dikendalikan secara ketat.
Reverse-polarity dapat memakai STPS3L60, Schottky 60 V/3 A dalam package SMB. VERIFIED FROM DATASHEET 2. Buck regulator utama
Kandidat Rating/package External components Keputusan
LMR51430 4.5–36 V, 3 A, SOT-23-THN 6 Inductor, diode/internal synchronous stage, capacitors, feedback Baik untuk supply 24 V yang bersih, kurang margin transient
LMR36510ADDA 4.2–65 V, 1 A, HSOIC-8 Inductor, capacitors, feedback Rekomendasi utama
LM2596S-ADJ 4.5–40 V, 3 A, TO-263/TO-220 Inductor, Schottky diode, capacitors, feedback Mudah tetapi besar dan efisiensinya lebih rendah

LMR36510 memiliki input maksimum 65 V dan toleransi transient hingga sekitar 70 V. VERIFIED FROM DATASHEET
LMR51430 memiliki input 4.5–36 V dan output hingga 3 A. VERIFIED FROM DATASHEET
[ENGINEERING ESTIMATE] Untuk board yang menerima input industri 24 V dan memakai TVS 33 V, LMR36510ADDA lebih aman daripada LMR51430, walaupun kapasitas arusnya hanya 1 A. 3. Regulator 3.3 V
Kandidat Rating/package Keputusan
TLV62569DBV 2.5–5.5 V input, 2 A, SOT-23 Rekomendasi utama
TPS62160DGK 3–17 V input, 1 A, VSSOP/WSON Alternatif baik
TPS62177DQC 4.75–28 V input, 500 mA, WSON-10 Terlalu dekat dengan kebutuhan peak

TLV62569 mendukung input 2.5–5.5 V dan output hingga 2 A. VERIFIED FROM DATASHEET
[ENGINEERING ESTIMATE] Rail 3.3 V sebaiknya memiliki kemampuan desain sekitar 0.8–1 A walaupun konsumsi normal kemungkinan lebih rendah, karena ESP32 dan W5500 dapat menghasilkan current burst. 4. Isolated RS485
Kandidat Rating/package Kelebihan Kekurangan Keputusan
ADM2587EBRWZ 3.3/5 V, 2.5 kVrms, 20-pin wide SOIC Isolasi data dan power terintegrasi Mahal, membutuhkan layout EMI yang baik Rekomendasi utama
ISO1410DW + SN6505B + transformer ISO1410 5 kVrms, SOIC-16; SN6505B SOT-23-6 Isolasi kuat, fleksibel Banyak komponen dan layout lebih kompleks Alternatif
MAX3485AEASA+T + ADuM1201AR + isolated DC/DC 3.3 V RS485, SOIC-8 + SOIC-8 Komponen umum dan mudah dipahami Membutuhkan isolator, power isolator, transformer, rectifier Untuk eksperimen/reference

ADM2587E sudah mengintegrasikan transceiver RS485, isolator digital, dan isolated DC/DC dalam satu IC; tersedia sebagai ADM2587EBRWZ dalam 20-pin wide SOIC. VERIFIED FROM DATASHEET
ISO1410 membutuhkan supply terisolasi eksternal. VERIFIED FROM DATASHEET
SN6505B adalah transformer driver 2.25–5.5 V dalam SOT-23-6 dengan current drive tinggi. VERIFIED FROM DATASHEET
MAX3485AE merupakan 3.3 V RS485 transceiver dalam SOIC-8. VERIFIED FROM DATASHEET
Rekomendasi
[ENGINEERING ESTIMATE] Untuk Rev A, pilih ADM2587EBRWZ. Ini mengurangi jumlah komponen, risiko wiring isolation, dan kompleksitas schematic.
[NEEDS VERIFICATION] Layout 2-layer tetap harus menjaga creepage/clearance dan mengikuti panduan EMI Analog Devices. Integrated isoPower bukan berarti otomatis bebas EMI. 5. RS485 TVS
Kandidat Rating/package Keputusan
SM712.TCT 400 W, SOT-23, asymmetric RS485 protection Rekomendasi utama
NUP2105L 27 V dual-line, 350 W/line, SOT-23 Alternatif
TPD2E001 5.5 V TVS array, SOT/WSON Ditolak untuk langsung melindungi A/B RS485

SM712 memang dirancang untuk RS485 dengan perlindungan asymmetric sekitar +12 V sampai −7 V. VERIFIED FROM DATASHEET
NUP2105L adalah dual bidirectional TVS 27 V dalam SOT-23. VERIFIED FROM DATASHEET
[ENGINEERING ESTIMATE] Untuk Rev A gunakan SM712 dekat connector RS485, ditambah resistor termination dan bias yang dapat dipasang melalui jumper. 6. Digital input conditioning
Pendekatan Komponen Kelebihan Kekurangan Keputusan
Divider + clamp Resistor seri/divider, TVS, diode clamp, Schmitt input Murah dan sederhana Threshold, surge, power dissipation harus dihitung teliti Tidak dipilih sebagai utama
Transistor interface MMBT3904 atau transistor NPN setara, SOT-23 Murah dan mudah Threshold bergantung resistor/transistor; noise margin lebih lemah Prototype sederhana
Optocoupler VOS617A per channel Galvanic isolation dan input mudah dipahami 4 optocoupler, resistor input, CTR aging Alternatif kuat
Industrial digital-input IC ISO1212DBQR Threshold IEC, reverse input protection, current limit, isolasi Lebih mahal dan perlu konfigurasi resistor Rekomendasi utama
8-channel industrial IC ISO1228DFBR Banyak fitur, diagnostics, filtering 8 channel padahal hanya butuh 4; package 38-pin Overkill

ISO1212 mendukung input receiver untuk sistem industrial 24 V hingga 60 V dan dapat dikonfigurasi menggunakan resistor eksternal untuk rentang input lebih rendah. Supply logika mendukung 2.25–5.5 V. VERIFIED FROM DATASHEET
ISO1228 memiliki 8 channel, current limiting, glitch filter, diagnostics, dan field supply 8.5–36 V atau 13–36 V tergantung konfigurasi. VERIFIED FROM DATASHEET
VOS617A merupakan optocoupler phototransistor 4-pin dengan VCEO 80 V dan isolation rating 3750 Vrms. VERIFIED FROM DATASHEET
Rekomendasi digital input
[ENGINEERING ESTIMATE] Gunakan dua ISO1212DBQR, sehingga tersedia empat input industrial.
[NEEDS VERIFICATION] Nilai RSENSE, RTHR, resistor surge, dan filter kapasitor belum boleh ditetapkan sebelum dipilih apakah input bekerja sebagai sourcing atau sinking dan apakah threshold 12 V harus dijamin pada kondisi temperatur ekstrem.
[ENGINEERING ESTIMATE] Untuk 12–24 V, ISO1212 lebih tepat daripada divider sederhana karena current limit dan threshold hysteresis lebih terkontrol. 7. Digital output
Kandidat Rating/package Keputusan
ZXMS6005N8Q 60 V, 3.3/5 V logic, protected low-side, SO-8, 250 mΩ max pada 3 V Rekomendasi utama
ZXMS6008N8Q 60 V, 3.3/5 V logic, SO-8, 800 mΩ max pada 3 V Alternatif untuk beban lebih kecil
ZVN3306F 60 V, SOT-23, sekitar 150 mA, RDS(on) tinggi Hanya indikator kecil

ZXMS6005N8Q menyediakan overcurrent, thermal shutdown, active overvoltage clamp, ESD input protection, dan logic input 3.3/5 V. VERIFIED FROM DATASHEET
ZXMS6008N8Q juga merupakan protected low-side MOSFET 60 V dengan input logic 3.3/5 V. VERIFIED FROM DATASHEET
Rekomendasi output
[ENGINEERING ESTIMATE] Target Rev A:

- 4 channel low-side/open-drain.
- 12–24 V external load.
- 0.5 A kontinu per channel.
- Beban maksimum total eksternal: 2 A hanya jika supply, connector, trace, dan return path memang dirancang untuk itu.
- PCB regulator tidak menyuplai beban output.
- Beban output mengambil energi dari supply field eksternal.
  [ENGINEERING ESTIMATE] Setiap gate sebaiknya memiliki:
- resistor seri gate sekitar 47–100 Ω;
- pull-down gate-source sekitar 100 kΩ;
- default OFF saat reset;
- test point pada gate dan drain.
  [NEEDS VERIFICATION] Nilai resistor final harus disesuaikan dengan gate charge, switching speed, EMI, dan cara output dikendalikan dari ESP32.
  [ENGINEERING ESTIMATE] Beban induktif wajib memiliki flyback diode langsung paralel dengan coil. TVS tambahan di sisi connector dapat dipertimbangkan untuk kabel panjang atau beban eksternal yang tidak diketahui.
  [ENGINEERING ESTIMATE] Pull-down gate menjamin output tetap OFF ketika ESP32 masih booting, GPIO masih high-Z, atau firmware belum menginisialisasi pin.

8. USB-C protection
   Kandidat Rating/package Keputusan
   USBLC6-2SC6 USB2, 2 data line, SOT-23-6L, low capacitance Rekomendasi
   TPD2EUSB30 5.5 V, 0.7 pF, SOT-9X3, ±8 kV contact Alternatif
   TPD2E001 USB2/high-speed ESD array, 5.5 V Alternatif, perlu cek routing

USBLC6-2SC6 melindungi dua data line USB dan VBUS dalam SOT-23-6L. VERIFIED FROM DATASHEET
TPD2EUSB30 memiliki rating 5.5 V, capacitance tipikal 0.7 pF, dan surge rating 5 A pada waveform 8/20 µs. VERIFIED FROM DATASHEET
[ENGINEERING ESTIMATE] USB-C device port tetap memerlukan:

- resistor CC 5.1 kΩ ke ground;
- ESD protector dekat connector;
- kapasitor VBUS;
- fuse atau load switch jika VBUS board disediakan ke host;
- routing D+/D− pendek dan simetris.

9. W5500 module interface
   Kandidat Detail Keputusan
   WIZ850io W5500 + PHY + transformer + RJ45 MAG-JACK Rekomendasi utama
   W5500-io W5500 module tanpa RJ45/MAG-JACK Alternatif jika RJ45 dibuat terpisah
   Direct W5500 IC W5500 LQFP-48 7×7 mm Ditolak untuk Rev A

WIZ850io memiliki W5500, PHY, transformer, dan RJ45 MAG-JACK dalam modul. Supply-nya 2.97–3.63 V dan konsumsi tipikal pada kondisi 100 Mb/s sekitar 141 mA. VERIFIED FROM DATASHEET
WIZ850io menggunakan dua header 1×6 pitch 2.54 mm dengan dimensi sekitar 23×25 mm. VERIFIED FROM DATASHEET
[VERIFIED FROM KICAD] KiCad memiliki symbol Interface_Ethernet:W5500, tetapi tidak memiliki footprint native yang mewakili keseluruhan modul WIZ850io.
[ENGINEERING ESTIMATE] Gunakan dua footprint header 1x06_P2.54mm dan buat mechanical courtyard/keepout custom berdasarkan drawing WIZ850io. 10. Connectors
Kandidat Detail Keputusan
Phoenix 1710726 MKDS 1,5/6-5,08, 6 posisi Output connector
Phoenix 1715789 MKDS 1,5/8-5,08, 8 posisi Input connector
Phoenix 1715747 MKDS 1,5/4-5,08, 4 posisi Sensor/service connector
Degson DG301-5.0-02P 5.0 mm, screw terminal, 17.5 A Cost-reduced alternative
Generic 2.54 mm header WIZ850io/BME280/service Internal low-current connection

Phoenix 1710726 memiliki pitch 5.08 mm dan nominal current 17.5 A. VERIFIED FROM DATASHEET
Phoenix 1715789 tersedia sebagai terminal 8 posisi pitch 5.08 mm. VERIFIED FROM DATASHEET
Degson DG301-5.0 tersedia dalam konfigurasi 2–15 posisi, pitch 5.0 mm, dan nominal current 17.5 A. VERIFIED FROM DATASHEET
C. Preliminary Power Budget
BOARD INTERNAL
[ENGINEERING ESTIMATE] Budget desain awal untuk rail 3.3 V:
Beban internal Estimasi desain
ESP32-S3-WROOM-1 300 mA
WIZ850io 141 mA tipikal terverifikasi; gunakan 160 mA allowance
ADM2587E 100 mA allowance
BME280 3 mA
DS18B20 2 mA
Status LEDs 8 mA
Digital-input logic 10 mA
Digital-output gate/control 2 mA
Pull-up, leakage, margin 25 mA
Total estimasi sekitar 610 mA

WIZ850io mencantumkan konsumsi tipikal sekitar 141 mA pada kondisi 100 Mb/s. VERIFIED FROM DATASHEET
[ENGINEERING ESTIMATE] Rail 3.3 V sebaiknya dirancang untuk minimal 0.8 A, bukan hanya 500 mA nominal.
EXTERNAL LOAD
Beban eksternal Target awal
Digital input field current sekitar 2.5 mA/channel sebagai target Type 3, perlu konfigurasi ISO1212
4 output load 0.5 A/channel
Total output load hingga 2 A dari supply field eksternal
Auxiliary 5 V batasi awal sekitar 250 mA
BME280 eksternal, low-current
DS18B20 eksternal, low-current

[ENGINEERING ESTIMATE] Auxiliary 5 V tidak boleh dianggap sebagai rail bebas untuk arbitrary external load. Batasi dengan fuse/PTC atau current-limited load switch.
[ENGINEERING ESTIMATE] Dengan internal board sekitar 2–3 W dan auxiliary 5 V maksimum 1.25 W, LMR36510 1 A masih dapat dipakai sebagai budget awal, tetapi peak current, thermal PCB 2-layer, dan batas AUX_5V harus diverifikasi.
D. Recommended RS485 Architecture
[ENGINEERING ESTIMATE]
ESP32-S3 UART
│
ADM2587E
├── digital isolation
├── isolated power
└── RS485 transceiver
│
SM712
│
termination/bias jumper
│
RS485 connector
Rekomendasi final: ADM2587EBRWZ.
[VERIFIED FROM DATASHEET] ADM2587E memiliki integrated isolated DC/DC, ±15 kV ESD pada RS485 pins, supply 3.3/5 V, dan 20-pin wide SOIC. Datasheet ADI
[NEEDS VERIFICATION] Untuk PCB 2-layer, layout isolation barrier, return current, placement capacitor, dan EMI isoPower harus mengikuti application note ADI sebelum PCB dibuat.
E. Recommended Digital-Input Architecture
[ENGINEERING ESTIMATE]
Gunakan:
12–24 V field input
│
surge resistor / input protection
│
ISO1212 channel
│
isolated 3.3 V logic output
│
ESP32-S3 GPIO
Rekomendasi:

- 2 × ISO1212DBQR
- satu device menangani dua input
- gunakan input current limiting dari datasheet
- tambahkan konektor field dengan return yang jelas
- tambahkan filtering di sisi input
- verifikasi konfigurasi sinking/sourcing
  [VERIFIED FROM DATASHEET] ISO121x menyediakan reverse-polarity tolerance, input tolerance ±60 V, current limiting, dan kompatibilitas logic supply 3.3 V. TI ISO1211
  [NEEDS VERIFICATION] ISO1212 perlu diverifikasi kembali untuk threshold minimum 12 V menggunakan nilai RSENSE dan RTHR final.
  F. Recommended Digital-Output Architecture
  [ENGINEERING ESTIMATE]
  Per channel:
  ESP32 GPIO ── logic input ZXMS6005N8Q
                         │ low-side output
  +12–24 V field supply ─ external load ── DO_FIELD_GND

  Optional system-level input pulldown untuk default OFF saat boot.
  Flyback diode langsung paralel pada beban induktif.
Rekomendasi:

- Protected low-side switch 60 V dengan logic input; bukan discrete bare-gate MOSFET.
- Target normal: 0.5 A/channel.
- Default boot state: OFF.
- Flyback diode wajib untuk relay/solenoid.
- Beban disupply dari field connector, bukan dari regulator board.
- Output connector diberi labeling FIELD_V+, OUTx, dan FIELD_GND.
  G. Components Requiring Custom KiCad Symbols/Footprints
  [VERIFIED FROM KICAD]
  Sudah tersedia di instalasi KiCad:
- RF_Module:ESP32-S3-WROOM-1
- Interface_Ethernet:W5500
- Interface_UART:ADM2587E
- Interface_UART:MAX3485
- Regulator_Switching:LMR51430
- Regulator_Switching:LMR36510ADDA
- Isolator:ISO1211
- Isolator:ISO1212
- Power_Management:SN6505BDBV
- Diode:SM712_SOT23
- Power_Protection:USBLC6-2SC6
- Power_Protection:TPD2EUSB30
- Sensor:BME280
- Sensor_Temperature:DS18B20
- Connector:USB_C_Receptacle_USB2.0_16P
  [VERIFIED FROM KICAD] Footprint yang tersedia:
- ESP32-S3-WROOM-1
- LQFP-48 7×7 mm
- SOIC-20W
- SOIC-16W
- SOIC-8
- SSOP-16
- SSOP-24
- SOT-23/SOT-23-6
- Bosch BME280 LGA-8 2.5×2.5 mm
- USB-C footprints
- Phoenix 5.08 mm terminal footprints
- 1×6 2.54 mm header footprints
  [NEEDS VERIFICATION]
- WIZ850io: perlu custom module courtyard/keepout.
- ZXMS6005N8Q: exact symbol belum ditemukan; generic SO-8 perlu diverifikasi terhadap datasheet.
- ISO1228DFBR: kemungkinan perlu custom/vendor symbol jika dipilih.
- Amphenol 124019772112A: footprint mekanis harus dicocokkan dengan drawing vendor.
- LMR36510ADDA: generic HSOIC/thermal-pad footprint perlu diverifikasi terhadap package drawing TI.
- LMR51430: symbol ada, tetapi exact SOT-23-THN land pattern tetap perlu dicek.
- TVS SMBJ33A: gunakan generic TVS symbol + exact SMB footprint, lalu cocokkan polarity dan pad spacing.
  Catatan: MCP search_symbols sempat timeout saat pencarian langsung. Karena itu status KiCad di atas diverifikasi melalui file library KiCad lokal, bukan dianggap valid berdasarkan hasil search MCP yang timeout.
  H. Remaining Datasheet Uncertainties
  [NEEDS VERIFICATION]

1. Varian tepat ESP32-S3-WROOM-1, terutama flash/PSRAM dan kebutuhan sourcing.
2. Threshold ISO1212 pada input minimum 12 V.
3. Nilai resistor input ISO1212 untuk mode sinking/sourcing.
4. Arus aktual ADM2587E pada baud rate dan bus loading target.
5. EMI akibat integrated isoPower pada PCB 2-layer.
6. Exact transformer/isolation alternative jika ADM2587E tidak tersedia.
7. Thermal performance LMR36510ADDA pada input 24 V dan auxiliary load aktif.
8. Exact land pattern LMR36510ADDA.
9. Thermal performance ZXMS6005N8Q pada 0.5 A/channel.
10. Flyback diode final berdasarkan jenis relay/solenoid yang digunakan.
11. Connector mechanical clearance, enclosure, screw access, dan wire gauge.
12. WIZ850io availability dan dimensi aktual dari vendor/distributor yang digunakan.
13. USB-C connector mechanical footprint dan shield grounding.
14. EMC, surge, EFT, creepage, dan clearance belum tervalidasi melalui laboratory test.
    I. Updated Risk Register
    Risiko Dampak Mitigasi
    Input transient melebihi 36 V Buck rusak Gunakan buck 65 V dan TVS
    ADM2587E EMI RS485/noise issue Ikuti layout ADI dan gunakan short return paths
    12 V tidak terbaca ISO1212 Input false OFF Simulasikan threshold dengan resistor final
    Output load terlalu besar MOSFET/trace panas Batasi 0.5 A/channel
    Relay menghasilkan flyback besar MOSFET rusak/reset MCU Flyback diode langsung di load
    ESP32 boot membuat output aktif Beban menyala tidak sengaja Gate pulldown dan default OFF
    WIZ850io footprint salah Mechanical failure Buat courtyard custom dari drawing vendor
    3.3 V brownout ESP32/W5500 reset Rail 0.8–1 A dan bulk capacitor
    USB-C pin mapping salah USB tidak terdeteksi Cocokkan connector drawing dan symbol
    Sourcing component BOM tidak tersedia Freeze vendor/MPN sebelum Phase 3
    PCB 2-layer isolation kurang Safety/EMI buruk Tetapkan keepout dan creepage sejak schematic
    Design dianggap certified Risiko klaim berlebihan Tetap dokumentasikan sebagai portfolio demonstrator

Keputusan Phase 2
[ENGINEERING ESTIMATE] Kandidat utama yang saya rekomendasikan untuk dilanjutkan ke review Anda:

- LMR36510ADDA
- TLV62569DBV
- ADM2587EBRWZ
- SM712.TCT
- ISO1212DBQR
- ZXMS6005N8Q
- WIZ850io
- USBLC6-2SC6
- Phoenix MKDS 1,5 series
  Belum ada Phase 3. Saya berhenti di sini untuk review Anda.

---

# Final Verification Sebelum Phase 3

Bagian ini adalah keputusan terbaru dan **menggantikan pernyataan yang bertentangan di bagian sebelumnya**. Schematic dan PCB belum dibuat atau diubah.

## A. Final decisions

1. **Power architecture — GO bersyarat**

   ```text
   12–24 V INPUT
       │
       └── LMR36510ADDA ── regulated 5 V main rail
                                  │
                                  ├── current-limited AUX_5V ≈ 250 mA
                                  │
                                  └── TLV62569DBV ── regulated 3.3 V LOGIC
   ```

   - [VERIFIED FROM DATASHEET] LMR36510 menerima 4.2–65 V dan TLV62569 menerima 2.5–5.5 V; urutan 5 V → 3.3 V ini kompatibel.
   - [ENGINEERING ESTIMATE] Budget internal sekitar 610 mA pada 3.3 V membutuhkan sekitar 0.45 A dari 5 V pada efisiensi konversi sekitar 90%. Dengan AUX_5V 250 mA, kebutuhan 5 V sekitar 0.70 A. Ini masih di bawah rating 1 A LMR36510 sebagai budget awal.
   - [NEEDS VERIFICATION] Peak ESP32/WIZ850io, suhu PCB 2-layer, ripple, dan current limit AUX_5V harus diverifikasi sebelum layout final. AUX_5V bukan supply untuk beban output 12–24 V.

2. **ISO1212 digital inputs — konfigurasi Type 3 dipilih**

   [VERIFIED FROM DATASHEET] Topologi TI menggunakan `RTHR` pada jalur input field menuju `SENSE`, `RSENSE` pada jalur return/input-current, dan `CIN` dari `SENSE` ke `FGND`, sesuai reference circuit ISO1212.

   | Konfigurasi TI | RSENSE | RTHR | CIN | Threshold yang relevan |
   |---|---:|---:|---:|---|
   | Type 3, dipilih untuk 12–24 V | 562 Ω | 1 kΩ | 10 nF; 330 nF untuk target surge yang lebih tinggi | VIL typ ≈ 9.2 V; VIH typ ≈ 10.4 V; VIH max ≈ 10.95 V |
   | Type 1, kandidat 24 V | 562 Ω | 2.5 kΩ | 10 nF | VIL typ ≈ 12.7 V; VIH typ ≈ 13.9 V, dihitung dari persamaan TI |

   - [VERIFIED FROM DATASHEET] Satu konfigurasi Type 3 `562 Ω + 1 kΩ` dapat mengenali 12 V nominal dan 24 V nominal sebagai HIGH karena 12 V berada di atas `VIH max` sekitar 10.95 V pada kondisi datasheet.
   - [NEEDS VERIFICATION] Margin 12 V harus diuji dengan toleransi supply, voltage drop kabel, temperatur, surge resistor, dan mode sourcing/sinking yang dipakai. Type 1 tidak dipilih karena threshold tipikalnya terlalu tinggi untuk jaminan 12 V.

3. **ZXMS6005N8Q — protected intelligent low-side switch**

   - [VERIFIED FROM DATASHEET] Perlakukan sebagai protected low-side IntelliFET dengan **logic input**, bukan sebagai discrete MOSFET bare-gate. Datasheet tidak mensyaratkan gate resistor atau gate-source pulldown.
   - [VERIFIED FROM DATASHEET] ESP32 3.3 V kompatibel: `VIH` minimum 3.0 V; input LOW 0–0.7 V. Untuk default OFF yang deterministik saat boot, gunakan optional system-level input pulldown; nilainya belum dikunci karena bukan nilai rekomendasi manufacturer.
   - [VERIFIED FROM DATASHEET] Target 0.5 A kontinu/channel masuk akal. Rating continuous drain minimum pada 3 V adalah 1.4 A pada kondisi datasheet. Dengan `RDS(on)` maksimum 250 mΩ, rugi konduksi pada 0.5 A sekitar 0.063 W; thermal PCB tetap perlu diverifikasi.
   - [VERIFIED FROM DATASHEET] Internal active clamp menangani overvoltage/inductive energy. [ENGINEERING ESTIMATE] Flyback diode eksternal tetap direkomendasikan langsung di coil/solenoid untuk mengurangi energi clamp dan EMI.
   - [FINAL DECISION] Tidak perlu mengganti ZXMS6005N8Q untuk target 0.5 A; jangan memakai terminologi gate network pada schematic final kecuali sebagai catatan sistem-level input pulldown.

4. **Ground domains**

   - `LOGIC_GND`: ESP32-S3, WIZ850io, USB, BME280/DS18B20, regulator return.
   - `PWR_GND`: return input 12–24 V dan return buck. Karena power stage tidak terisolasi, `PWR_GND` tersambung DC ke `LOGIC_GND`.
   - `DO_FIELD_GND`: return beban output; tersambung DC ke `PWR_GND/LOGIC_GND`. Digital outputs **tidak galvanically isolated**.
   - `DI_FIELD_GND`: return field khusus sisi input ISO1212; **tidak** disambungkan ke `LOGIC_GND`.
   - `RS485_GND`: ground sisi bus ADM2587E; **tidak** disambungkan DC ke `LOGIC_GND`.
   - [FINAL DECISION] RS485 dan digital inputs isolated; digital outputs, ESP32, WIZ850io, dan USB non-isolated pada domain logic/power yang sama.

5. **WIZ850io**

   - [VERIFIED FROM DATASHEET] Supply 2.97–3.63 V, nominal 3.3 V; SPI 3.3 V kompatibel dengan ESP32-S3. Arus tipikal yang dicantumkan sekitar 141 mA.
   - [VERIFIED FROM DATASHEET] Header `J1` 1×6: `1 GND, 2 GND, 3 MOSI, 4 SCLK, 5 CSn, 6 INTn`. Header `J2` 1×6: `1 GND, 2 3V3D, 3 3V3D, 4 NC, 5 RSTn, 6 MISO`.
   - [VERIFIED FROM DATASHEET] `RSTn` aktif-low minimal 500 µs; setelah reset dilepas, tunggu minimal 50 ms sebelum SPI.
   - [NEEDS VERIFICATION] Footprint custom harus memakai dua `PinHeader_1x06_P2.54mm_Vertical`, posisi J1/J2 sesuai mechanical drawing vendor, courtyard/keepout modul, tinggi RJ45, dan clearance tepi PCB. Jangan mengandalkan footprint W5500 IC.

## B. Corrected block diagram

```text
12–24V INPUT ── protection ── LMR36510 ── 5V_MAIN ── TLV62569 ── 3V3_LOGIC
     │                              │                    │
     │                              └─ AUX_5V ≤250mA     ├─ ESP32-S3
     │                                                   ├─ WIZ850io
     └─ PWR_GND = LOGIC_GND ────────────────────────────┼─ USB / sensors
                                                        └─ ZXMS6005N8Q ×4
                                                           │
                                             external load +12–24V
                                             return = DO_FIELD_GND

DI_FIELD_GND ── 12–24V DI ── ISO1212 ×2 ── isolated logic ── ESP32

ESP32 UART ── ADM2587E ── isolation barrier ── SM712 ── RS485 connector
LOGIC_GND  ──[isolated from]────────────────── RS485_GND
```

## C. Unresolved issues

- [NEEDS VERIFICATION] Margin ISO1212 pada input 12 V minimum dan nilai resistor surge/filter final.
- [NEEDS VERIFICATION] Thermal/current peak LMR36510 saat ESP32/WIZ850io aktif bersamaan dengan AUX_5V 250 mA.
- [NEEDS VERIFICATION] Nilai final optional input pulldown ZXMS6005N8Q dan perilaku semua GPIO saat reset.
- [NEEDS VERIFICATION] Symbol/footprint exact ZXMS6005N8Q dan mechanical footprint WIZ850io.
- [NEEDS VERIFICATION] Creepage/clearance dan EMI ADM2587E/ISO1212 pada PCB 2-layer.

## D. GO / NO-GO

**CONDITIONAL GO untuk Phase 3 schematic capture.** Keputusan arsitektur sudah cukup untuk mulai schematic, tetapi item pada bagian C harus ditandai sebagai verification checkpoints. Tidak ada schematic atau PCB yang dibuat pada tahap ini.
