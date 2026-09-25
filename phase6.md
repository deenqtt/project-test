# Phase 6A — 4-Channel Isolated 12–24 V Digital Input

## A. Existing-project verification

- [VERIFIED FROM KICAD] Project aktif: `hardware/kicad_mcp_test.kicad_pro`; root schematic: `hardware/kicad_mcp_test.kicad_sch`.
- [VERIFIED FROM KICAD] Baseline sebelum perubahan cocok dengan prompt: **0 error / 16 warning**.
- [VERIFIED FROM KICAD] Ethernet dan RS485 tidak diubah. GPIO tetap: GPIO35–GPIO38 = `DI1_LOGIC`–`DI4_LOGIC`.

## B. ISO1212 datasheet findings

- [VERIFIED FROM TEXAS INSTRUMENTS] `ISO1212DBQR` adalah receiver digital-input dual-channel dalam SSOP-16/DBQ. `VCC1` menerima 2.25–5.5 V sehingga rail 3.3 V kompatibel; output logic juga dispesifikasikan untuk 3.3 V.
- [VERIFIED FROM TEXAS INSTRUMENTS] Field side tidak memerlukan supply terpisah. Pin field adalah `SENSE1/2`, `IN1/2`, dan `FGND1/2`; logic side adalah `VCC1`, `GND1`, `EN`, dan `OUT1/2`.
- [VERIFIED FROM TEXAS INSTRUMENTS] Absolute maximum `SENSE/IN` adalah ±60 V. Reverse input didukung dengan arus negatif yang sangat kecil; `EN` HIGH/open mengaktifkan output, tetapi TI menyarankan mengikat EN ke VCC pada lingkungan bising.
- [VERIFIED FROM TEXAS INSTRUMENTS] `RSENSE=562 Ω` menghasilkan current limit tipikal sekitar 2.25 mA. `RTHR` harus dimaksimalkan selama threshold masih sesuai dan TI mensyaratkan resistor MELF 0.25 W untuk ketahanan surge.
- [VERIFIED FROM TEXAS INSTRUMENTS] `CIN >= 1 nF`; 10 nF digunakan pada tabel/reference design TI. Tiap IC diberi 100 nF dari `VCC1` ke `GND1`.
- [VERIFIED FROM KICAD] Symbol `Isolator:ISO1212` cocok dengan pinout TI dan footprint yang dipakai adalah `Package_SO:SSOP-16_3.9x4.9mm_P0.635mm`.

Referensi utama: [TI ISO1212 datasheet Rev. G](https://www.ti.com/lit/ds/symlink/iso1212.pdf).

## C. 12 V input margin calculation

- [VERIFIED FROM TEXAS INSTRUMENTS] Jaringan awal 562 Ω / 1 kΩ memberikan `VIH typ ≈ 10.4 V` dan `VIH max ≈ 10.95 V`.
- [ENGINEERING CALCULATION] Margin pada tepat 12.0 V hanya sekitar **1.05 V**; pada supply 12 V yang turun 10% menjadi 10.8 V, level HIGH tidak lagi dijamin. Karena itu jaringan 1 kΩ tidak dipilih.
- [ENGINEERING DECISION] Nilai final per kanal: `RSENSE=562 Ω 1%`, `RTHR=330 Ω 1% 0.25 W pulse-proof MELF`, `CIN=10 nF 50 V X7R`.
- [ENGINEERING CALCULATION] Persamaan TI memberi:

  `VIH(typ) = 8.25 V + 2.25 mA × 330 Ω = 8.993 V`

  `VIL(typ) = 7.10 V + 2.25 mA × 330 Ω = 7.843 V`

- [ENGINEERING CALCULATION] Dengan dasar `VIH max=8.55 V` pada `RTHR=0`, kontribusi arus worst-case yang diturunkan dari tabel TI, dan toleransi resistor 1%, estimasi konservatif `VIH max ≈ 9.36 V`. Margin menjadi sekitar **2.64 V pada 12.0 V** dan **1.44 V pada 10.8 V**.
- [ENGINEERING DECISION] Konfigurasi final layak untuk nominal 12 V dan 24 V sebagai demonstrator, tetapi bukan klaim sertifikasi IEC 61131-2 tanpa pengujian threshold/EMC pada hardware.

## D. 24 V input/current calculation

- [VERIFIED FROM TEXAS INSTRUMENTS] Arus HIGH dengan 562 Ω dibatasi sekitar **2.25 mA tipikal**; rentang karakteristik datasheet yang relevan sekitar **2.05–2.75 mA**.
- [ENGINEERING CALCULATION] Pada 24 V, drop tipikal di `RTHR` adalah 0.743 V dan `VSENSE ≈ 23.26 V`.
- [ENGINEERING CALCULATION] Pada 2.25 mA: `PRTHR ≈ 1.67 mW`, `PRSENSE ≈ 2.85 mW`, dan disipasi internal field-side sekitar **52 mW/kanal**. Empat kanal aktif sekitar **0.21 W internal total**, sebelum toleransi/temperatur.
- [ENGINEERING CALCULATION] Pada 2.75 mA: `PRTHR ≈ 2.50 mW` dan `PRSENSE ≈ 4.25 mW`; rating resistor dipilih berdasarkan pulse/surge, bukan daya DC.

## E. Circuit created

- [VERIFIED FROM KICAD] Dibuat `hardware/digital_inputs_isolated.kicad_sch` dan ditautkan sebagai hierarchical sheet `DIGITAL_INPUTS_ISOLATED`.
- [VERIFIED FROM KICAD] Dua `ISO1212DBQR` (`U6/U7`) menangani empat kanal. Setiap jalur adalah:

  `DIx_FIELD → RTHR 330 Ω → SENSE node + CIN + TVS → RSENSE 562 Ω/INx → ISO1212 → DIx_LOGIC`

- [VERIFIED FROM KICAD] `EN` kedua IC terikat ke `3V3_LOGIC`; masing-masing memiliki 100 nF ke `LOGIC_GND`. Pin NC dan SUB ditandai intentionally unconnected.
- [VERIFIED FROM TEXAS INSTRUMENTS] SUB1/SUB2 tidak boleh dihubungkan secara elektrik. Floating copper thermal island per SUB hanya boleh ditambahkan saat PCB dengan mengikuti panduan TI.

## F. Input protection decision

- **MANDATORY:** [ENGINEERING DECISION] `RTHR 330 Ω` pulse-proof MELF, `RSENSE 562 Ω 1%`, dan `CIN 10 nF/50 V X7R`.
- **RECOMMENDED, IMPLEMENTED:** [VERIFIED FROM KICAD] Dua TVS dual bidirectional `VCAN33A2-03S-E3-08`, masing-masing melindungi dua SENSE node ke `DI_FIELD_GND`.
- Datasheet resmi Vishay mengonfirmasi stand-off ±33 V, breakdown 36–40 V, clamp maksimum 56 V pada pulse 2.7 A/8–20 µs, dan package SOT-23. [ENGINEERING DECISION] Ini memberi margin supply lebih baik daripada TVS stand-off 26.5 V untuk sistem 24 V.
- **OPTIONAL, OMITTED:** [ENGINEERING DECISION] `CIN=330 nF` untuk target surge line-to-FGND yang lebih tinggi; tidak dipasang karena 10 nF memadai untuk demonstrator dan memberi respons lebih cepat.
- **INTENTIONALLY OMITTED:** [ENGINEERING DECISION] Bridge rectifier/series reverse diode, proteksi PE, dan jaringan surge certification-grade. ISO1212 sudah memiliki toleransi input negatif; produk ini tidak mengklaim sertifikasi industri.

Referensi TVS: [Vishay VCAN33A2-03S datasheet](https://www.vishay.com/docs/86323/vcan33a2-03s.pdf).

## G. Isolation architecture

- [VERIFIED FROM KICAD] `DI_FIELD_GND` hanya menghubungkan J5 pin 5, C25–C28, D6/D7 common, U6/U7 FGND pins, dan TP14.
- [VERIFIED FROM KICAD] `LOGIC_GND` hanya berada di sisi logic ISO1212 untuk U6/U7 GND1 dan C23/C24.
- [VERIFIED FROM KICAD] Tidak ada wire, label global, power symbol, capacitor, TVS, connector, hidden pin, atau net-tie yang menghubungkan kedua domain.
- [VERIFIED FROM TEXAS INSTRUMENTS] Barrier ISO1212 menyediakan basic isolation; rating datasheet mencakup 2500 Vrms isolation withstand. Creepage/clearance PCB belum dinilai karena PCB belum dikerjakan.

## H. Connector / MPN

- Halaman resmi Phoenix Contact mengonfirmasi J5 sebagai **MKDS 1,5/ 5-5,08**, MPN **1715750**, 5 posisi, pitch 5.08 mm.
- [VERIFIED FROM KICAD] Footprint: `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-5-5.08_1x05_P5.08mm_Horizontal`.

Referensi: [Phoenix Contact 1715750](https://www.phoenixcontact.com/en-us/products/printed-circuit-board-terminal-mkds-15-5-508-1715750).

## I. ERC before / after

- [VERIFIED FROM KICAD] Sebelum Phase 6A: **0 error / 16 warning**.
- [VERIFIED FROM KICAD] Setelah Phase 6A: **0 error / 12 warning**.
- [VERIFIED FROM KICAD] Empat warning `DI1_LOGIC`–`DI4_LOGIC` hilang setelah tersambung ke U6/U7.
- [VERIFIED FROM KICAD] Sisa 12 warning adalah existing/staged: 4 off-grid pada RS485 yang tidak diubah, serta 8 global labels untuk `SPARE_GPIO1–3`, `ONEWIRE_DATA`, `I2C_SDA/SCL`, dan `USB_D_N/P` yang belum memiliki peripheral. Tidak ada suppression baru.

## J. Isolation and connectivity audit

- [VERIFIED FROM KICAD] Root dan sub-sheet lolos structural validation/KiCad CLI.
- [VERIFIED FROM KICAD] Empat jalur field lengkap pada generated netlist; `DI1_LOGIC`–`DI4_LOGIC` masing-masing terhubung dari U6/U7 ke U3 GPIO35–GPIO38.
- [VERIFIED FROM KICAD] Tidak ditemukan orphan wire, floating label, off-grid geometry pada sheet baru, overlap terdeteksi, atau wire yang menyeberangi symbol.
- [VERIFIED FROM KICAD] Generated netlist menunjukkan `/DIGITAL_INPUTS_ISOLATED/DI_FIELD_GND` dan `LOGIC_GND` sebagai dua net berbeda tanpa komponen bersama.
- [VERIFIED FROM KICAD] Semua reference pada generated netlist unik; footprint U6/U7, D6/D7, J5, resistor, capacitor, dan TP14 terisi.

## K. Remaining unresolved items

- [NEEDS VERIFICATION] Exact production MPN untuk RTHR pulse-proof MELF, RSENSE, dan capacitor.
- [NEEDS VERIFICATION] Threshold 10.8/12/24 V, temperatur, cable noise, ESD/EFT/surge, dan thermal harus diuji pada prototype.
- [NEEDS VERIFICATION] TVS clamp 56 V berada di bawah absolute maximum 60 V ISO1212, tetapi margin transient harus diverifikasi dengan parasitic/trace inductance pada PCB nyata.
- [NEEDS VERIFICATION] Creepage, clearance, isolated copper keepout, serta floating SUB thermal islands pada layout 2-layer.

## L. Recommendation

**CONDITIONAL GO untuk fase berikutnya.**

[ENGINEERING DECISION] Phase 6A secara elektrik lengkap, netlist dan isolation audit lulus, dan ERC tidak memiliki error. Status conditional tetap diperlukan sampai MPN pasif, threshold 12 V minimum, transient, thermal, serta isolation-layout checks diverifikasi pada BOM/layout/prototype.

Phase digital outputs, sensors, USB, AUX power, dan PCB belum dimulai.
