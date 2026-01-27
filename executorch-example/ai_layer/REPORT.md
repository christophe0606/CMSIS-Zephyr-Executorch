# ExecuTorch AI Layer Build Report

**Generated:** 2026-01-23T11:00:47Z
**Git Commit:** unknown on unknown
**Repository Status:** ✅ Clean
**Last Commit:** unknown

## 📊 Build Summary

- **Libraries:** 9 static libraries
- **Models:** 1 model assets
- **Operators:** 2 selected operators
- **Build Type:** unknown

## 📚 Library Assets

**Total Size:** 9.3 MiB

| Library | Size | Percentage | Modified | Hash |
|---------|------|------------|----------|------|
| libcortex_m_kernels.a | 17.9 KiB | 0.2% | 2026-01-23 11:00:35 | `832aba894210ba8a` |
| libcortex_m_ops_lib.a | 11.9 KiB | 0.1% | 2026-01-23 11:00:36 | `56b30fe82f4e46c7` |
| libexecutorch.a | 45.3 KiB | 0.5% | 2026-01-23 11:00:35 | `3db8092c5f5b23d1` |
| libexecutorch_core.a | 201.5 KiB | 2.1% | 2026-01-23 11:00:35 | `d56fbb73b1dfd2a0` |
| libexecutorch_delegate_ethos_u.a | 16.1 KiB | 0.2% | 2026-01-23 11:00:36 | `0e5d5644edd3f498` |
| libportable_kernels.a | 8.5 MiB | 92.3% | 2026-01-23 11:00:35 | `baf7190ffddd414e` |
| libportable_ops_lib.a | 189.8 KiB | 2.0% | 2026-01-23 11:00:35 | `b752d73478ad1a58` |
| libquantized_kernels.a | 216.0 KiB | 2.3% | 2026-01-23 11:00:36 | `61a9ef918acee341` |
| libquantized_ops_lib.a | 27.6 KiB | 0.3% | 2026-01-23 11:00:36 | `9797a2435509f588` |

## 🤖 Model Assets

| Asset | Type | Size | Modified | Hash |
|-------|------|------|----------|------|
| ethos_u_minimal_example.pte | Model | 3.8 KiB | 2026-01-23 11:00:46 | `bd7a211160a18572` |

## ⚙️ Build Configuration

### CMake Configuration
- **Build Type:** `unknown`
- **Toolchain File:** `none`
- **ARM Baremetal:** `unknown`
- **Cortex-M Support:** `unknown`
- **Portable Ops:** `unknown`
- **Quantized Kernels:** `unknown`

### Selected Operators

**Source:** Model file: ethos_u_minimal_example.pte (inferred)

**Count:** 2 operators

```
quantized_decomposed::dequantize_per_tensor.out
quantized_decomposed::quantize_per_tensor.out
```

## 🔄 Model Conversion Details


**Vela Compilation Summary:**
  - Accelerator configuration               Ethos_U55_128
  - System configuration             Ethos_U55_High_End_Embedded
  - Memory mode                               Shared_Sram
  - Accelerator clock                                 500 MHz
  - Design peak SRAM bandwidth                       3.73 GB/s
  - Design peak Off-chip Flash bandwidth             0.47 GB/s
  - Total SRAM used                                  0.14 KiB
  - Total Off-chip Flash used                        0.03 KiB
  - CPU operators = 0 (0.0%)
  - NPU operators = 12 (100.0%)
  - Average SRAM bandwidth                           0.27 GB/s
  - Input   SRAM bandwidth                           0.00 MB/batch
  - Weight  SRAM bandwidth                           0.00 MB/batch
  - Output  SRAM bandwidth                           0.00 MB/batch
  - Total   SRAM bandwidth                           0.00 MB/batch
  - Total   SRAM bandwidth            per input      0.00 MB/inference (batch size 1)
  - Average Off-chip Flash bandwidth                 0.04 GB/s
  - Input   Off-chip Flash bandwidth                 0.00 MB/batch
  - Weight  Off-chip Flash bandwidth                 0.00 MB/batch
  - Output  Off-chip Flash bandwidth                 0.00 MB/batch
  - Total   Off-chip Flash bandwidth                 0.00 MB/batch
  - Total   Off-chip Flash bandwidth  per input      0.00 MB/inference (batch size 1)
  - Original Weights Size                            0.00 KiB
  - NPU Encoded Weights Size                         0.00 KiB
  - Neural network macs                                 0 MACs/batch
  - Info: The numbers below are internal compiler estimates.
  - For performance numbers the compiled network should be run on an FVP Model or FPGA.
  - Network Tops/s                                   0.00 Tops/s
  - NPU cycles                                        349 cycles/batch
  - SRAM Access cycles                                 24 cycles/batch
  - DRAM Access cycles                                  0 cycles/batch
  - On-chip Flash Access cycles                         0 cycles/batch
  - Off-chip Flash Access cycles                       32 cycles/batch
  - Total cycles                                      349 cycles/batch
  - Batch Inference time                 0.00 ms, 1432664.76 inferences/s (batch size 1)

## 📦 Source Layer Export

**Total Source Files:** 226 files across 2 layers

| Layer | Description | Groups | Files |
|-------|-------------|--------|-------|
| stage1 | Generated from compile_commands.json (327 files) | 4 | 37 |
| stage2 | Generated from compile_commands.json (240 files) | 1 | 189 |

### Group Details

**STAGE1:**

- `Runtime`: 19 files
- `Schema`: 1 files
- `Kernels/quantized`: 11 files
- `Backends`: 6 files

**STAGE2:**

- `Kernels`: 189 files


## 🛠️ Build Environment

- **Platform:** `Linux b29455e0289b 6.12.54-linuxkit #1 SMP Tue Nov  4 21:21:47 UTC 2025 aarch64 aarch64 aarch64 GNU/Linux`
- **Python:** `Python 3.12.3`
- **CMake:** `cmake version 4.2.1`
- **ARM GCC:** `arm-none-eabi-gcc (Arm GNU Toolchain 13.3.Rel1 (Build arm-13.24)) 13.3.1 20240614`

## 📁 Asset Locations

```
ai_layer/
├── engine/
│   ├── lib/           # Static libraries
│   ├── include/       # Header files
│   └── model/         # Model assets
└── REPORT.md          # This report
```

---
*Report generated by ExecuTorch AI Layer build system at 2026-01-23T11:00:47Z*