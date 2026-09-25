# Phase 6B-R1 — Digital Output Logic-Interface Resolution

Laporan ini menggantikan keputusan **NO-GO direct-drive**. Laporan pre-capture lama disimpan sebagai `phase6b-pre-capture.md`.

## A. AHCT125 verification

- [VERIFIED FROM TEXAS INSTRUMENTS] Dipilih `SN74AHCT125PWR`: quad non-inverting 3-state buffer, package PW/TSSOP-14, supply 4.5–5.5 V, dengan OE aktif-LOW.
- [VERIFIED FROM TEXAS INSTRUMENTS] Pada `VCC=4.5 V`: `VIH(min)=2.0 V`, `VIL(max)=0.8 V`, `VOH(min)=3.8 V` pada −8 mA, dan `VOL(max)=0.44 V` pada 8 mA untuk rentang temperatur penuh.
- [VERIFIED FROM TEXAS INSTRUMENTS] Absolute maximum: VCC −0.5…7 V, input −0.5…7 V, output −0.5…VCC+0.5 V, output-clamp ±20 mA, continuous output ±25 mA, dan VCC/GND ±50 mA.
- [VERIFIED FROM TEXAS INSTRUMENTS] Propagation delay maksimum 6.5 ns (`CL=15 pF`) atau 8.5 ns (`CL=50 pF`).
- [VERIFIED FROM TEXAS INSTRUMENTS] TI merekomendasikan bypass 0.1 µF dekat VCC dan pull-up OE ke VCC untuk high-impedance saat power-up/down. `C29=100 nF, 16 V, X7R` dipasang pada `5V_MAIN`–`LOGIC_GND`.
- [VERIFIED FROM TEXAS INSTRUMENTS] Datasheet tidak memberikan parameter `Ioff`; power-off back-current tidak diklaim terjamin.

Referensi: [TI SN74AHCT125 datasheet](https://www.ti.com/lit/ds/symlink/sn74ahct125.pdf).

## B. ESP32 → AHCT input margin

- [ENGINEERING CALCULATION] ESP32-S3 menjamin `VOH(min)=0.8×VDD`. Pada VDD minimum 3.0 V, `VOH(min)=2.40 V`; terhadap `VIH(min)=2.0 V`, HIGH margin = **0.40 V**. Pada 3.3 V nominal, margin = **0.64 V**.
- [ENGINEERING CALCULATION] `VOL(max)=0.1×VDD=0.33 V` pada 3.3 V; terhadap `VIL(max)=0.8 V`, LOW margin = **0.47 V**.
- [VERIFIED FROM KICAD] Pull-down existing `R8–R11=47 kΩ` dipertahankan untuk memastikan `DO1_CTRL–DO4_CTRL` LOW saat GPIO high-impedance.

## C. AHCT → ZXMS output margin

- [VERIFIED FROM DIODES INCORPORATED] ZXMS mensyaratkan `VIH(min)=3.0 V`, `VIL(max)=0.7 V`; input current maksimum 100 µA pada 3 V dan 200 µA pada 5 V.
- [ENGINEERING CALCULATION] Dengan `VOH(min)=3.8 V`, HIGH margin = **0.80 V**. Dengan `VOL(max)=0.44 V`, LOW margin = **0.26 V**.
- [ENGINEERING CALCULATION] Beban worst-case output sekitar 0.5 mA dari pull-down 10 kΩ pada 5 V + input ZXMS 0.2 mA = **0.7 mA**, jauh di bawah kondisi pengujian AHCT 8 mA.
- [ENGINEERING CALCULATION] Saat AHCT high-Z, `IOZ(max)=2.5 µA` menghasilkan paling banyak 25 mV pada 10 kΩ. `R32–R35=10 kΩ` menjaga input ZXMS jauh di bawah 0.7 V.

Referensi: [Diodes Incorporated ZXMS6005N8Q datasheet](https://www.diodes.com/datasheet/download/ZXMS6005N8Q.pdf).

## D. Startup/power-sequencing safe state

- [VERIFIED FROM KICAD] Saat ESP32 reset/boot atau tidak bertenaga, `R31=47 kΩ` menahan basis Q1 LOW, Q1 OFF, dan `R29=10 kΩ` menahan seluruh OE AHCT HIGH. Buffer high-Z dan R32–R35 menahan ZXMS OFF.
- [VERIFIED FROM KICAD] Ketika 5 V startup atau MCU mati sementara 5 V ada, hardware tetap default disabled. Firmware harus menaikkan `SPARE_GPIO1` secara sengaja untuk mengaktifkan output.
- [ENGINEERING CALCULATION] Jika 5 V mati tetapi 3.3 V masih ada sesaat, downstream pull-down tetap memerintahkan ZXMS OFF. Namun AHCT125 tidak memiliki jaminan `Ioff`, sehingga kondisi ini harus dihindari atau diuji.

## E. OE strategy

- [VERIFIED FROM KICAD] Empat OE aktif-LOW digabung sebagai `DO_BUF_OE_N`; R29 menariknya ke 5 V untuk default disabled.
- [VERIFIED FROM KICAD] Frozen spare GPIO5/`SPARE_GPIO1` menggerakkan `Q1=MMBT3904-7-F` melalui `R30=10 kΩ`; R31 memastikan Q1 OFF saat GPIO floating. Mapping `DO1_CTRL–DO4_CTRL` tidak berubah.
- [ENGINEERING CALCULATION] R29 membutuhkan sink sekitar 0.5 mA. Arus basis setelah beban R31 sekitar 0.22 mA, sehingga forced beta sekitar 2.3 dan Q1 memiliki margin saturasi besar.
- [ENGINEERING CALCULATION] Firmware contract: `SPARE_GPIO1=HIGH` mengaktifkan buffer; LOW/reset menonaktifkannya.

## F. Circuit created

- [VERIFIED FROM KICAD] Hierarchical sheet `DIGITAL_OUTPUTS` dibuat pada root dan mengarah ke `hardware/digital_outputs.kicad_sch`.
- [VERIFIED FROM KICAD] Empat channel: `DOx_CTRL → U8 SN74AHCT125 → DOx_DRIVE → U9–U12 ZXMS6005N8Q → DOx_OUT`.
- [VERIFIED FROM KICAD] GPIO1/2/47/48 tetap menuju DO1/2/3/4; GPIO5 tetap bernama `SPARE_GPIO1` dan hanya dipakai sebagai master OE.
- [VERIFIED FROM KICAD] U8 hanya memakai `5V_MAIN`; `DO_FIELD_V+` tidak dipakai sebagai logic supply.

## G. ZXMS custom symbol/footprint

- [VERIFIED FROM DIODES INCORPORATED] `ZXMS6005N8Q-13` adalah protected intelligent low-side switch SO-8: pin 1–3 Source, pin 4 IN, pin 5–8 Drain.
- [VERIFIED FROM KICAD] Symbol project-local `Project_Outputs:ZXMS6005N8Q` dibuat di `hardware/project_symbols.kicad_sym` dan didaftarkan melalui `hardware/sym-lib-table`; validation lulus.
- [VERIFIED FROM KICAD] Footprint `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` memiliki delapan pad 1–8.

## H. Flyback decision

- [VERIFIED FROM DIODES INCORPORATED] D8–D11 memakai kandidat `B360Q-13-F`, Schottky 3 A/60 V package SMC.
- [VERIFIED FROM KICAD] Setiap diode terhubung anode ke `DOx_OUT`, cathode ke `DO_FIELD_V+`, dan ditandai **DNP/optional**.
- [ENGINEERING CALCULATION] Flyback lokal direkomendasikan untuk coil/solenoid tanpa suppression agar energi repetitive clamp dan EMI berkurang.

Referensi: [Diodes Incorporated B360Q datasheet](https://www.diodes.com/datasheet/download/B360Q.pdf).

## I. Connector/field architecture

- [VERIFIED FROM KICAD] `J6` memakai footprint 6-posisi 5.08 mm `TerminalBlock_Phoenix:TerminalBlock_Phoenix_MKDS-1,5-6-5.08_1x06_P5.08mm_Horizontal`, candidate MPN Phoenix Contact `1706303`.
- [VERIFIED FROM KICAD] Mapping: pin 1 `DO_FIELD_V+`, pin 2–5 `DO1_OUT–DO4_OUT`, pin 6 `DO_FIELD_GND`.
- [VERIFIED FROM KICAD] Net `DO_FIELD_V+` hanya berisi J6/1, TP15, dan cathode D8–D11; tidak terhubung ke `VIN_FIELD`, `5V_MAIN`, `3V3_LOGIC`, atau auxiliary rail.

## J. Ground-domain audit

- [VERIFIED FROM KICAD] J6/6, TP16, Source U9–U12, R32–R35, U8 GND, dan control return tergabung pada net final `LOGIC_GND`.
- [VERIFIED FROM KICAD] Label `DO_FIELD_GND` sengaja di-alias secara DC ke `LOGIC_GND`; `PWR_GND` adalah domain DC yang sama.
- [VERIFIED FROM KICAD] Generated netlist mempertahankan `DI_FIELD_GND` dan `RS485_GND` sebagai net terpisah; tidak ada bridge.

## K. ERC before/after

- [VERIFIED FROM KICAD] Baseline: **0 error / 12 warning**.
- [VERIFIED FROM KICAD] Fresh ERC setelah capture: **0 error / 12 warning**, tanpa exclusion/suppression baru.
- [VERIFIED FROM KICAD] Empat warning off-grid berasal dari sheet RS485 existing; tujuh warning adalah staged global nets. Warning baru tunggal adalah alias intentional `DO_FIELD_GND`/`LOGIC_GND`; warning staged `SPARE_GPIO1` hilang karena sekarang terhubung.
- [VERIFIED FROM KICAD] Root dan child lulus structural validation dan KiCad CLI load/export. Netlist berisi 109 komponen dan 93 net.

## L. Remaining unresolved items

- [ENGINEERING CALCULATION] Kondisi abnormal 3.3 V aktif saat 5 V mati perlu dicegah atau diuji karena AHCT125 tidak menjamin `Ioff`.
- [ENGINEERING CALCULATION] Pada 0.5 A, `RDS(on,max)=250 mΩ` memberi sekitar 62.5 mW per ZXMS. Copper area, ambient, empat channel simultan, inrush, PWM, dan short-circuit cycling tetap harus diuji.
- [VERIFIED FROM DIODES INCORPORATED] `VDS(SC)` ZXMS hanya 24 V; pada supply 24 V nominal tidak ada margin untuk toleransi/transient saat short. External supply fuse/current limit tetap wajib.
- [ENGINEERING CALCULATION] D8–D11 tetap DNP sampai load, energi coil, repetitive duty, dan suppression eksternal diketahui.

## M. Recommendation

**CONDITIONAL GO — Phase 6B schematic capture selesai dan logic-interface dibekukan.**

[ENGINEERING CALCULATION] Margin logika worst-case terjamin untuk operasi normal 5 V/3.3 V dan semua kondisi reset normal default OFF. Power-sequencing abnormal, thermal PCB, short-circuit 24 V, dan populasi flyback tetap menjadi verification checkpoints.

Sensor, USB-C, AUX 5 V, PCB placement, dan PCB routing tidak dimulai.
