# Phase 9 — Current-Limited Auxiliary 5 V Output

## A. Existing power architecture

- [VERIFIED FROM KICAD] `5V_MAIN` tetap berasal dari LMR36510ADDA 1 A dan memasok regulator 3.3 V serta beban 5 V internal.
- [ENGINEERING DECISION] `AUX_5V` adalah output service **OUTPUT ONLY**, non-isolated, dengan return `LOGIC_GND`.
- [VERIFIED FROM KICAD] Phase 9 tidak mengubah rangkaian Phase 3–8 maupun divider USB `R45=39 kΩ 1%` / `R40=62 kΩ 1%`.

## B. Updated preliminary power budget

| Beban pada 5V_MAIN | Typical/normal | Design estimate |
|---|---:|---:|
| Beban 3V3 direfleksikan ke 5 V | ≈0.20 A | ≈0.43 A |
| ADM2587E isolated RS485 | ≈0.072 A | ≈0.120 A |
| Buffer/output-control dan housekeeping 5 V | ≈0.008 A | ≈0.010 A |
| **Board internal** | **≈0.28 A** | **≈0.56 A** |
| AUX output | ≈0.254 A nominal limit | ≈0.292 A maximum limit |
| **Total dengan AUX** | **≈0.53 A** | **≈0.85 A** |

- [ENGINEERING ESTIMATE] Terhadap rating LMR36510 1 A, kombinasi design estimate + AUX maximum meninggalkan sekitar **0.15 A estimate headroom**.
- [NEEDS VERIFICATION] Nilai 0.15 A bukan guaranteed worst-case headroom. Extreme load 3V3 dan AUX secara bersamaan harus ditinjau lagi dan diuji pada prototype.
- [ENGINEERING CALCULATION] Jika rail 3V3 benar-benar memakai 0.8 A sekaligus dengan RS485/direct-5V allowance dan AUX maksimum, kebutuhan 5V_MAIN dapat mendekati atau sedikit melampaui 1 A; kondisi ini tidak boleh dianggap operating point kontinu yang telah tervalidasi.

## C. Candidate comparison

| Candidate | Package | Current-limit suitability | Reverse-current behavior | Decision |
|---|---|---|---|---|
| **TPS2553DBVR** | SOT-23-6 / DBV | Adjustable; 105 kΩ menghasilkan target sekitar 250 mA | Reverse comparator/cutoff memiliki threshold dan delay; bukan zero-backfeed instan | **Selected** |
| TPS25221DBVR | SOT-23-6 / DBV | Minimum programmable window lebih tinggi dari target ini | Reverse blocking berlaku pada kondisi disable sesuai datasheet | Rejected untuk target 250 mA |
| MIC2039AYM6-TR | SOT-23-6 | Adjustable current limit | Tidak memberikan jaminan reverse-blocking yang setara untuk kebutuhan dokumentasi ini | Rejected |

## D. Selected component

- [VERIFIED FROM DATASHEET] Power-distribution switch: **Texas Instruments TPS2553DBVR**, active-high enable, adjustable current limit, DBV SOT-23-6.
- [VERIFIED FROM KICAD] U13 memakai custom project symbol `Project_Power:TPS2553DBV` dan footprint `Package_TO_SOT_SMD:SOT-23-6`.
- [VERIFIED FROM DATASHEET] Operating input range 2.5–6.5 V mencakup regulated `5V_MAIN`.

Referensi utama: [TI TPS2553 datasheet](https://www.ti.com/lit/ds/symlink/tps2553.pdf).

## E. Current-limit calculation

Rumus datasheet, dengan `RILIM` dalam kΩ:

```text
IOS(min) = 25230 / RILIM^1.016
IOS(nom) = 23950 / RILIM^0.977
IOS(max) = 22980 / RILIM^0.94
```

Untuk `R47=105 kΩ ±1%`:

| Condition | Resistance used | Calculated limit |
|---|---:|---:|
| Minimum | 106.05 kΩ | ≈220.8 mA |
| Nominal | 105.00 kΩ | ≈253.9 mA |
| Maximum | 103.95 kΩ | ≈292.1 mA |

- [ENGINEERING CALCULATION] Target terdokumentasi: **≈221–292 mA**, nominal **≈254 mA**.
- [ENGINEERING DECISION] Label produk tetap “approximately 250 mA nominal current limit”; bukan rating beban kontinu yang dijamin tepat 250 mA.

## F. Voltage drop and thermal behavior

- [VERIFIED FROM DATASHEET] `RDS(on)` maximum yang digunakan untuk estimasi adalah 135 mΩ pada specified DBV temperature range.
- [ENGINEERING CALCULATION] Pada 0.250 A: drop maksimum ≈33.8 mV dan rugi konduksi ≈8.4 mW.
- [ENGINEERING CALCULATION] Pada 0.292 A: drop maksimum ≈39.4 mV dan rugi konduksi ≈11.5 mW.
- [ENGINEERING ESTIMATE] Dengan datasheet `θJA≈182.6 °C/W`, kenaikan temperatur steady-state akibat rugi konduksi normal hanya beberapa °C pada test-board assumptions; layout copper dan ambient aktual tetap menentukan.
- [VERIFIED FROM DATASHEET] Saat short circuit, TPS2553 membatasi arus dan thermal shutdown bekerja sekitar 135 °C dengan hysteresis sekitar 10 °C.
- [ENGINEERING CALCULATION] Short ke ground dapat menghasilkan sekitar 1.46 W sesaat pada 5 V dan 292 mA sebelum thermal cycling. Karena itu short-circuit endurance wajib diuji; jangan menganggap output dapat berada pada short terus-menerus tanpa thermal cycling.

## G. Reverse-current / backfeed limitation

- [VERIFIED FROM DATASHEET] TPS2553 menggunakan reverse-voltage comparator dengan threshold `VOUT−VIN` sekitar 95 mV minimum, 135 mV typical, 190 mV maximum dan turn-off delay sekitar 3–7 ms.
- [ENGINEERING DECISION] Dengan demikian `AUX_5V` **tidak** memiliki absolute atau instantaneous zero-backfeed protection. Energi/transient reverse current masih mungkin sampai switch membuka.
- [VERIFIED FROM DATASHEET] Datasheet mencantumkan reverse leakage maksimum 1 µA pada kondisi uji `VOUT=6.5 V`, `VIN=0 V` ketika path telah off.
- [ENGINEERING DECISION] External voltage tidak boleh diterapkan ke J10. Marking/documentation harus menyatakan `AUX_5V — OUTPUT ONLY`.

## H. Enable, FAULT, capacitors, and connector

- [VERIFIED FROM KICAD] U13 pin 3 `EN` terhubung langsung ke `5V_MAIN`; pin tidak floating dan output otomatis enabled ketika main 5 V valid.
- [VERIFIED FROM DATASHEET] EN high threshold maksimum jauh di bawah 5 V; UVLO internal mencegah undefined low-supply operation.
- [VERIFIED FROM KICAD] U13 pin 4 `~FAULT` diberi explicit no-connect. Open-drain diagnostic output sengaja tidak digunakan pada Rev A.
- [VERIFIED FROM KICAD] `C30=1 µF, 10 V, X7R` berada dari input ke `LOGIC_GND`; `C31=1 µF, 10 V, X7R` berada dari `AUX_5V` ke `LOGIC_GND`.
- [VERIFIED FROM DATASHEET] Nilai 1 µF memenuhi bypass/transient usage pada application information; effective capacitance dan penempatan dekat U13 tetap harus diperiksa saat part capacitor dibekukan.
- [VERIFIED FROM KICAD] J10 adalah 2-pin Phoenix Contact **1715721**: pin 1 `AUX_5V`, pin 2 `LOGIC_GND`, dengan footprint 5.08 mm terminal-block family.
- [ENGINEERING DECISION] Tidak ada output TVS untuk Rev A.

Referensi connector: [Phoenix Contact 1715721](https://www.phoenixcontact.com/en-pc/products/pcb-terminal-block-mkds-15-2-508-1715721).

## I. Schematic capture

- [VERIFIED FROM KICAD] Hierarchical sheet `AUX_5V_OUTPUT` dibuat pada root dan menunjuk ke `hardware/aux_5v.kicad_sch`.
- [VERIFIED FROM KICAD] Sheet berisi U13, R47, C30, C31, J10, serta notes untuk output-only, current-limit tolerance, reverse-current limitation, no-TV​​S decision, dan placement.
- [VERIFIED FROM KICAD] Flattened path:

```text
5V_MAIN → U13/TPS2553DBVR → AUX_5V → J10 pin 1
                   │
                   └── R47 105k 1% → LOGIC_GND
J10 pin 2 ─────────────────────────→ LOGIC_GND
```

## J. ERC and netlist validation

- [VERIFIED FROM KICAD] Baseline before Phase 9: **0 errors / 4 known warnings**, 125 flattened components.
- [VERIFIED FROM KICAD] Fresh ERC after capture and after full MCP close/open reload: **0 errors / 4 known warnings**.
- [VERIFIED FROM KICAD] Warning set tidak berubah: tiga RS485 off-grid sheet-pin warnings dan satu intentional `DO_FIELD_GND` / `LOGIC_GND` alias warning.
- [VERIFIED FROM KICAD] Root dan child structural validation lulus; KiCad CLI PDF-load/export checks exit 0.
- [VERIFIED FROM KICAD] Flattened netlist export berhasil dengan **130 components**: tepat 125 baseline + U13, R47, C30, C31, J10.
- [VERIFIED FROM KICAD] Tidak ada duplicate reference. U13, R47, C30, C31, dan J10 masing-masing muncul satu kali setelah reload.
- [VERIFIED FROM KICAD] Net `AUX_5V` hanya berisi `U13/6`, `C31/1`, dan `J10/1`. `ILIM_SET` hanya berisi `U13/5` dan `R47/1`.
- [VERIFIED FROM KICAD] `AUX_5V` tidak terhubung ke `USB_VBUS`, `VIN_FIELD`, `3V3_LOGIC`, `DI_FIELD_GND`, atau `RS485_GND`.
- [VERIFIED FROM KICAD] `5V_MAIN` terhubung ke U13 pin 1/IN dan pin 3/EN; `LOGIC_GND` terhubung ke U13 pin 2, kedua capacitor returns, R47 return, dan J10 pin 2.

## K. Remaining checkpoints

- [NEEDS VERIFICATION] Freeze exact manufacturer MPN untuk C30/C31 dan pastikan effective capacitance tetap memadai pada 5 V, toleransi, dan temperatur.
- [NEEDS VERIFICATION] Cocokkan courtyard/drill/outline footprint J10 terhadap drawing/revision produksi Phoenix Contact 1715721 sebelum PCB release.
- [NEEDS VERIFICATION] Uji prototype pada AUX nominal load, maximum current-limit corner, hard short, hot-plug, main-rail droop, dan thermal cycling.
- [NEEDS VERIFICATION] Uji simultaneous worst-case 3V3 load + RS485 + AUX; bila 5V_MAIN mendekati current limit, firmware/load policy atau power-stage budget harus direvisi.
- [NEEDS VERIFICATION] Pastikan label fisik/enclosure mencegah penggunaan J10 sebagai external power input.

## L. Recommendation

**CONDITIONAL GO — Phase 9 schematic capture complete.**

Rangkaian memenuhi arsitektur yang disetujui dan seluruh validasi schematic kembali ke baseline. Status tetap conditional karena thermal short-circuit, capacitor derating, connector mechanical fit, dan simultaneous full-system loading belum divalidasi pada prototype.

Phase 9 berhenti di schematic. Full engineering review dan PCB work belum dimulai.
