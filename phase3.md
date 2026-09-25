# Phase 3A — Power Input Protection and 5V Main Rail

Status: schematic capture selesai untuk blok `12–24V INPUT → protection → LMR36510ADDA → 5V_MAIN`. Tidak ada pekerjaan pada regulator 3.3 V, ESP32, Ethernet, RS485, digital I/O, sensor, atau PCB layout.

## A. Schematic section created

File schematic: `hardware/kicad_mcp_test.kicad_sch`

```text
J2 12–24V INPUT
  → F1 resettable PPTC
  → D2 series reverse-polarity Schottky
  → D3 SMBJ33A shunt TVS
  → C1 bulk damping + C2/C3 local ceramic input network
  → U1 LMR36510ADDA
  → L1 + C6/C7 output filter
  → 5V_MAIN
```

Nets utama telah diberi label `VIN_FIELD`, `PWR_GND`, dan `5V_MAIN`. `PWR_GND` dinyatakan electrically common dengan future `LOGIC_GND`.

Test point:

- `TP1`: `VIN_FIELD`, setelah fuse, reverse-polarity diode, dan TVS node.
- `TP2`: `5V_MAIN`.
- `TP3`: `PWR_GND`.

## B. Selected component values and source/reason

| Ref | Komponen / nilai | Source dan alasan |
|---|---|---|
| J2 | Phoenix Contact 1715721, 2-pin, 5.08 mm | Exact connector dan footprint `MKDS-1,5-2-5.08`; rated jauh di atas kebutuhan board. [Phoenix Contact](https://www.phoenixcontact.com/en-pc/products/pcb-terminal-block-mkds-15-2-508-1715721) |
| F1 | Littelfuse `2920L110/60MR`, 1.1 A hold, 2.2 A trip, 60 V | Dipilih berdasarkan input current calculation dan temperature derating; hold sekitar 0.50 A pada 85 °C. [Littelfuse 2920L datasheet](https://www.littelfuse.com/assetdocs/2920l_datasheet_update.pdf?assetguid=f237e8c2-1ed9-4c13-a738-dbe0738b3d2c) |
| D2 | ST `STPS3L60U`, 60 V, 3 A, SMB | Series Schottky reverse-polarity protection; adequate untuk estimated input current. [ST datasheet](https://www.st.com/resource/en/datasheet/stps3l60.pdf) |
| D3 | Littelfuse `SMBJ33A`, unidirectional, 33 V standoff, 53.3 V clamp, 600 W, SMB | Clamp maksimum berada di bawah 65 V operating maximum LMR36510. Cathode ke `VIN_FIELD`, anode ke `PWR_GND`. [Littelfuse SMBJ33A](https://www.littelfuse.com/de/products/overvoltage-protection/tvs-diodes/surface-mount/smbj/smbj33a) |
| C1 | 47 µF, 63 V, Panasonic `EEU-FR1J470` | Nilai dipilih di dalam rekomendasi TI 20–100 µF untuk damping kabel/input; exact part 6.3 mm × 11.2 mm, pitch 2.5 mm. [Panasonic](https://industrial.panasonic.com/ww/products/pt/aluminum-cap-lead/models/EEUFR1J470) |
| C2 | 2.2 µF, 100 V, X7R | Minimum effective ceramic input capacitance dari TI Design 1. Footprint 1210 dipakai untuk membantu DC-bias performance. |
| C3 | 220 nF, 100 V, X7R | Mandatory high-frequency VIN bypass dari TI; harus ditempatkan sangat dekat VIN/PGND. |
| U1 | `LMR36510ADDA`, 4.2–65 V, 1 A, 400 kHz | Exact KiCad symbol dan TI DDA/HTSOP-8 exposed-pad footprint. [TI LMR36510 datasheet](https://www.ti.com/lit/ds/symlink/lmr36510.pdf) |
| C4 | 100 nF, minimum 16 V, X7R | Bootstrap capacitor wajib antara BOOT dan SW menurut TI. |
| C5 | 1 µF, 16 V, X7R | VCC bypass wajib; VCC tidak digunakan untuk external load. |
| L1 | 22 µH, Coilcraft `XAL5050-223MEC` | TI Design 1 memilih 22 µH. Exact part mempunyai Isat 3.6 A, lebih tinggi dari LMR36510 high-side current-limit maximum 2.4 A. [Coilcraft](https://www.coilcraft.com/en-us/products/power/shielded-inductors/molded-inductor/xal/xal50xx/xal5050-223/) |
| R2 | 100 kΩ, 1% | `RFBT` dari TI 5 V reference design. |
| R3 | 24.9 kΩ, 1% | `RFBB` dari TI 5 V reference design. Dengan VREF nominal 1 V: `VOUT = 1 × (1 + 100k/24.9k) ≈ 5.016 V`. |
| C6, C7 | 2 × 22 µF, 25 V, X7R | TI nominal output network untuk superior 0–100% load transient response. Effective capacitance under 5 V bias harus dipertahankan. |

LMR36510 menggunakan fixed internal 400 kHz switching frequency dan internal compensation. Tidak dipasang frequency-setting resistor, external compensation network, atau feed-forward capacitor. `EN` diikat ke `VIN_FIELD` karena TI mengizinkan direct-to-VIN untuk always-on operation dan melarang EN floating. `PG` diberi no-connect karena belum digunakan.

Input filtering sengaja berupa damped bulk capacitor dan local ceramics, tanpa series LC filter tambahan. TI memperingatkan bahwa input LC yang tidak dirancang dengan benar dapat menyebabkan instability.

Layout checkpoints untuk Phase PCB nanti:

- C2/C3 sedekat mungkin ke pin VIN dan PGND U1; hot-loop harus minimum.
- C5 dekat VCC/PGND dan C4 dekat BOOT/SW dengan jalur pendek/lebar.
- R2/R3 dekat FB; feedback trace dijauhkan dari SW.
- SW copper area dibuat kecil; VIN, `5V_MAIN`, dan `PWR_GND` memakai jalur lebar.
- Exposed pad U1 terhubung ke `PWR_GND` dengan copper area dan thermal vias yang memadai.

## C. Estimated regulator loading

- Preliminary output load: `5 V × 0.7 A = 3.5 W`.
- Regulator utilization: sekitar 70% dari rating 1 A; current headroom sekitar 0.3 A.
- Dengan asumsi efisiensi 90% berdasarkan karakteristik/reference TI, estimated input power sekitar 3.89 W.
- Estimated input current: sekitar 0.324 A pada 12 V dan 0.162 A pada 24 V.
- Pada full 1 A output, estimated 12 V input current sekitar 0.463 A pada asumsi efisiensi yang sama.

Nilai di atas adalah engineering calculations; bench validation tetap diperlukan.

## D. ERC result

Fresh KiCad MCP verification:

- ERC: **0 errors, 0 warnings, 0 info**.
- Schematic structure validation: valid; KiCad CLI export test exit code 0.
- Off-grid check: none pada grid 1.27 mm.
- Orphaned-wire check: none.
- Nine expected nets ditemukan dan koneksi `VIN_FIELD`, `PWR_GND`, `5V_MAIN`, `SW_LMR`, `BOOT_LMR`, `VCC_LMR`, dan `FB_5V` sesuai design.

Tidak ada ERC exclusion yang digunakan. Dua `PWR_FLAG` dipakai secara justified untuk externally powered `VIN_FIELD` dan return `PWR_GND`. `PG` memakai no-connect karena intentionally unused.

Tiga warning ERC awal berasal dari endpoint off-grid rangkaian LED test lama. Empat endpoint wire legacy di-snap 0.49 mm ke grid tanpa mengubah topology; ERC kemudian menjadi bersih.

## E. Unresolved issues

- Exact manufacturer MPN untuk ceramic C2–C7 dan resistor R2/R3 belum dibekukan; DC-bias/effective capacitance harus diperiksa sebelum procurement.
- Footprint C1 memakai body/pitch yang benar (`D6.3 mm`, `P2.5 mm`), tetapi model KiCad menyebut tinggi 11.0 mm sementara datasheet part menyebut 11.2 mm; enclosure/3D clearance perlu dikoreksi nanti.
- PPTC trip time, hot-ambient derating, startup inrush, diode loss, dan regulator thermal performance perlu bench/thermal validation.
- SMBJ33A memberikan demonstrator-level transient protection, bukan bukti compliance surge/EFT industri.
- LMR36510 layout guidance menyukai ground plane/multilayer; implementasi 2-layer memerlukan perhatian khusus pada hot-loop, thermal copper, dan vias.
- Cosmetic crossing checker masih mendeteksi wire yang berakhir pada pin LED `D1` dari rangkaian test lama; ini bukan ERC violation dan bukan bagian Phase 3A.

## F. GO / NO-GO recommendation for Phase 3B

**GO untuk Phase 3B schematic capture**, dengan unresolved items di bagian E dipertahankan sebagai verification checkpoints sebelum PCB layout dan procurement.

Phase 3B belum dimulai.

---

# Phase 3B — Final Verification: 3.3V Logic Power Supply

Status: blok `5V_MAIN → TLV62569DBV → 3V3_LOGIC` telah selesai dan diverifikasi. Bagian ESP32 belum dimulai.

## Calculated VOUT

- R4 = 453 kΩ dan R5 = 100 kΩ.
- Persamaan datasheet: `VOUT = VFB × (1 + R4/R5)`.
- Dengan `VFB = 0.6 V`: `VOUT = 0.6 × (1 + 453/100) = 3.318 V` nominal.
- Dengan toleransi resistor ±1% dan rentang datasheet `VFB = 0.588–0.612 V`, worst-case calculation sekitar 3.20–3.44 V.

## Verified component values

| Ref | Nilai / part | Hasil verifikasi |
|---|---|---|
| U2 | TI `TLV62569DBV`, SOT-23-5 | Input 2.5–5.5 V, output hingga 2 A; sesuai untuk input `5V_MAIN`. |
| R4 | 453 kΩ, 1% | Upper feedback resistor; nilai yang digunakan TI untuk implementasi 3.3 V. |
| R5 | 100 kΩ, 1% | Lower feedback resistor; di bawah batas maksimum 200 kΩ dari TI. |
| L2 | 2.2 µH, Coilcraft `XAL4020-222MEC` | Kombinasi standar/recommended TI untuk output ≥1.8 V dengan COUT 10 µF. Isat 5.6 A; Irms 4.0 A untuk kenaikan 20°C. |
| C8 | 4.7 µF, 10 V, X7R, Murata `GRM21BR71A475KA73L` | Nilai input capacitance yang dinyatakan cukup untuk sebagian besar aplikasi TLV62569. |
| C9 | 10 µF, 10 V, X7R, Murata `GRM21BR71A106KE51L` | Berada dalam rentang TI 10–47 µF. Tabel TI mengantisipasi derating capacitance hingga −50%, sehingga conservative effective value adalah sekitar 5 µF. |
| C10 | 6.8 pF, C0G | Feed-forward capacitor tidak wajib untuk regulasi dasar, tetapi direkomendasikan TI ketika lower feedback resistor bernilai 100 kΩ untuk memperbaiki transient response. |

`EN` dihubungkan langsung ke `5V_MAIN`, sehingga tidak floating dan regulator selalu aktif ketika rail 5 V tersedia. TLV62569DBV menggunakan internal soft-start sekitar 800 µs, mendukung startup pada pre-biased output, beroperasi sekitar 1.5 MHz pada moderate/heavy load, dan otomatis masuk Power Save Mode pada light load.

Datasheet/reference:

- [TI TLV62569 datasheet](https://www.ti.com/lit/ds/symlink/tlv62569.pdf)
- [TI 3.3 V reference implementation](https://www.ti.com/lit/ug/tidued0/tidued0.pdf)
- [Coilcraft XAL4020-222MEC specifications](https://www.coilcraft.com/en-us/products/power/shielded-inductors/molded-inductor/xal/xal40xx/xal4020-222/)

## Current and thermal margin

- Calculated inductor ripple pada 5 V → 3.3 V, 2.2 µH, dan 1.5 MHz: sekitar 0.34 A.
- Peak inductor current pada beban normal 0.6 A: sekitar 0.77 A.
- Peak inductor current pada design target 0.8 A: sekitar 0.97 A.
- Rekomendasi Isat dengan margin 30% pada target 0.8 A: sekitar 1.26 A.
- L2 Isat 5.6 A dan Irms 4.0 A memberikan margin yang besar.
- TLV62569 mempunyai rating output 2 A, sehingga headroom terhadap 0.6 A adalah 1.4 A dan terhadap target 0.8 A adalah 1.2 A.
- Thermal performance package SOT-23 tetap bergantung pada copper area, airflow, ambient temperature, switching loss, dan layout aktual.

## Ground-domain verification

`LOGIC_GND` adalah canonical schematic net untuk domain yang sama dengan nama fungsional `PWR_GND`. Semua return Phase 3A dan Phase 3B berada pada satu net; tidak ada net-tie atau accidental isolated ground domain.

## ERC result

Fresh KiCad MCP verification:

- ERC: **0 errors, 0 warnings, 0 info**.
- Schematic structural validation: valid; KiCad CLI validation exit code 0.
- Net connectivity terkonfirmasi:
  - `5V_MAIN`: U2 VIN dan EN serta C8.
  - `3V3_LOGIC`: L2, C9, R4, C10, dan TP4.
  - `FB_3V3`: U2 FB, R4, R5, dan C10.
  - `LOGIC_GND`: U2 GND, C8, C9, R5, TP5, serta seluruh return Phase 3A.

## Unresolved issues

- Exact effective capacitance C9 pada bias 3.3 V harus dikonfirmasi menggunakan data Murata SimSurfing atau pengukuran. Conservative design check memakai 5 µF berdasarkan allowance −50% dari tabel TI.
- Output ripple dan transient response perlu diuji dengan beban aktual ESP32 dan Ethernet.
- Temperatur TLV62569DBV perlu diverifikasi setelah PCB layout menggunakan kondisi ambient dan copper area aktual.
- Rentang worst-case output 3.20–3.44 V perlu dipertahankan sebagai tolerance checkpoint untuk semua beban 3.3 V.

## GO / NO-GO

**GO untuk Phase ESP32**, dengan DC-bias capacitor, transient response, dan thermal measurement dipertahankan sebagai PCB/bench verification checkpoints.
