# APEX-256 Validation & Benchmark Reports (v2.5.0.4)

This repository contains the public validation summaries, formal verification outcomes, side-channel evaluations, and external memory and SWaP-C (Size, Weight, Power, and Cost) benchmarks for **APEX-256** (`v2.5.0.4`), a proprietary embedded stream cipher and modular cryptographic primitive suite developed by **Apex Research Labs, LLC** (Inventor: Diab Anthony Sheteyh).

> **Note:** APEX-256 is commercial intellectual property protected by pending U.S. and International Patent Applications. The reference engine (`apex256_nano.h`), verification harnesses, and authoritative Known Answer Tests (KATs) are distributed exclusively via the [APEX Developer Portal (apex256.io)](https://apex256.io) under an executed Evaluation NDA with cryptographic supply-chain provenance attribution.
>
> Public release history is maintained on [https://apex256.com/revisions.html](https://apex256.com/revisions.html).

---

## Featured Study: CNSA 2.0 Handshake in Under 20 KiB (`wolfSSL 5.9.4` + `APEX-256 v2.5.0.4`)

* **Engineering Review & Analysis:** [`cnsa20_wolfssl_memory_study/README.md`](cnsa20_wolfssl_memory_study/README.md) (also published at [https://apex256.com/blog/cnsa20-under-20kib.html](https://apex256.com/blog/cnsa20-under-20kib.html))
* **3-Run Valgrind Massif Summary Table:** [`cnsa20_wolfssl_memory_study/RESULTS_3RUNS_SUMMARY.md`](cnsa20_wolfssl_memory_study/RESULTS_3RUNS_SUMMARY.md)
* **Raw 3-Run Massif, `ms_print` & `time -v` Archive (`386 KiB`):** [`cnsa20_wolfssl_memory_study/wolfssl_5.9.4_cnsa20_apex256_v2504_3runs.tar.gz`](cnsa20_wolfssl_memory_study/wolfssl_5.9.4_cnsa20_apex256_v2504_3runs.tar.gz)

Independent reproduction and audit of Anthony Hu's (`wolfSSL`) CNSA 2.0 (`ML-KEM-1024` + `ML-DSA-87`) low-RAM memory study on Yocto Scarthgap (`5.0.20`), paired with raw DER key loading and a zero-heap `APEX-256 v2.5.0.4` inner payload data plane:

| Endpoint Stack (Valgrind Massif `--stacks=yes`, 3-Run Mean) | Peak (Heap + Stack) | Peak Heap | Steady-State Data Heap |
|---|---:|---:|---:|
| `OpenSSL 3.5.8` CNSA 2.0 Client (`MFL 512`) | `939.63 KiB` | `816.09 KiB` | `16+ KiB` / session |
| `wolfSSL 5.9.4` Low-RAM Client (A. Hu flags, `MFL 512` / `no MFL`) | `39.31 / 38.74 KiB` | `35.50 / 34.99 KiB` | `~5 to 16 KiB` / session |
| **`wolfCrypt 5.9.4` `ML-KEM-1024 + ML-DSA-87` (DER) + `APEX-256` Client** | **`19.97 KiB`** | **`13.70 KiB`** | **`0.00 KiB`** |
| `wolfSSL 5.9.4` Low-RAM Server (A. Hu flags, `MFL 512`) | `64.74 KiB` | `62.06 KiB` | `17+ KiB` context |
| **`wolfCrypt 5.9.4` `ML-KEM-1024 + ML-DSA-87` (DER) + `APEX-256` Server** | **`39.03 KiB`** | **`32.07 KiB`** | **`0.00 KiB`** |
| **`APEX-256 v2.5.0.4` Steady-State Data Plane (All 9 Companion Modes)** | **`1.87 KiB` active** | **`0.00 KiB`** | **`0.00 KiB`** |

---

## 1. Empirical Statistical Validation Battery

APEX-256 has passed the industry's most demanding empirical statistical randomness batteries with zero anomalies:

| Test Suite | Scope / Workload | Result |
|---|---|---|
| **PractRand (`RNG_test`)** | 32 Terabytes continuous keystream, 235 test statistics, 42 hours | **ALL PASSED (0 Anomalies)** |
| **TestU01 (`BigCrush`)** | 106 tests, 160 test statistics, 1h 54m | **ALL PASSED (160/160)** |
| **NIST SP 800-22 Rev. 1a & SP 800-90B** | 1 Gigabyte, 15 statistical tests + IID / non-IID min-entropy | **ALL PASSED (`H_min = 7.99985 b/B`)** |
| **Strict Avalanche Criterion (SAC)** | 320M trial pairs across 327,680 bit-dependency cells (6-sigma bound) | **PERFECT 50.0000% Mean** |
| **Chi-Squared / Bit Balance / Autocorrelation** | 10 Megabytes | **ALL PASSED** |

---

## 2. Modular Single-Responsibility Architecture (v2.5.0.4)

The `v2.5.0.4` architecture delivers uniform 256-bit security levels and modular single-responsibility C++11+ companion interfaces while preserving the fixed `40-byte` self-contained zero-allocation core context (`0 bytes` heap, `0 bytes` static `.bss`/`.tbss`/`.data`):

* **3-Layer Defense-in-Depth Integration:** Designed to operate as an inner payload encapsulation layer (Layer 3) beneath standardized **NIST FIPS 203 (`ML-KEM`) / FIPS 204 (`ML-DSA`)** key establishment (Layer 1) and **NIST SP 800-232 (`Ascon`)** authenticated transport framing (Layer 2).
* **Hardware Side-Channel Hardening:** Constant-time secret-independent execution and arithmetic side-channel hardening (`CPA |r| = 0.0024`, Welch `|t| < 1.25`, `100%` wire-compatible with authoritative KATs).
* **Extended 256-Bit Nonce Support:** Expands random-nonce birthday collision safety from `2^32` to `2^128` messages with zero added RAM.
* **Forward-Secrecy Stream Ratcheting:** Executes an irreversible one-way state transition every 256 bytes (`100%` chunk-split invariant) so a post-compromise memory capture cannot invert prior keystream epochs.
* **Standalone Full-256 Companion Interfaces:** Dedicated `40-byte` to `48-byte` and stateless interfaces for hashing, MAC, KDF, AEAD, DRBG, virtual filesystem sector encryption (VFS), cipher feedback (CFB), format-preserving encryption (FPE), and side-channel equalized streaming, with automatic RAII destructor zeroization and fail-closed lifecycle enforcement.

---

## 3. 38-Tool Formal Verification & Reduced-Round Cryptanalysis Battery

Apex Research Labs utilizes 38 independent formal verification, SMT/SAT/MILP solving, interactive theorem proving, side-channel leakage, and hardware RTL synthesis engines to verify the 256-bit architecture:

| Evaluation Vector | Scope / Methodology | Result |
|---|---|---|
| **Single-Step & Full-State Bijection** | Z3, Bitwuzla, cvc5 SMT bitvector + Coq (`coqc`) & F* (`fstar.exe`) proofs | **PROVED UNSAT / QED (Strict Bijection)** |
| **Same-Nonce 256-Bit Key Collision Search** | Exhaustive 256-bit symbolic SMT search for $K_1 \neq K_2$ collapsing to identical post-binding state | **PROVED UNSAT ($0$ Collisions across $2^{256}$ Keys)** |
| **Reduced-Round Security Margin** | Z3/Bitwuzla SMT, CaDiCaL SAT, HiGHS MILP division property, OpenMP SAC, BoolTest, Gohr ResNet (`R = 1..24`) | **CERTIFIED `4.0x` to `24.0x` Safety Margin** |
| **Hardware Side-Channel Hardening (SCA)** | Correlation Power Analysis (CPA) & TVLA across the arithmetic datapath (`SCALib`, `Lascar`, `PROLEAD`) | **`CPA \|r\| = 0.0024`, Welch `\|t\| < 1.25` (`100%` KAT-Exact)** |
| **Rotational-XOR (`RX`) Trail Search** | Symbolic RX differential & invariant queries across all rotation offsets | **ALL UNSAT ($0$ RX Trails)** |
| **Polynomial & Monomial Distinguishers** | Degree $d \in \{1, 2, 3\}$ Boolean monomial distinguishers over 128/256/512-bit blocks | **$\|Z_{\max}\| \le 4.28 < 4.89\sigma$ (No Distinguisher)** |
| **Cube & Integral Cryptanalysis** | Moebius ANF & high-order zero-sum integral cubes + MILP degree bounds | **Full Algebraic Degree & $0$ Integral Bias** |
| **RTL Synthesis & Fault Injection (`VerFI` / `SymbiYosys`)** | Synthesizable Verilog-2005 hardware core & AXI4-Stream wrapper (`429 Core ALMs`, `0 BRAM`, `0 DSP`) | **100% Cycle-Exact Parity & Fault-Lock Verified** |
| **Binary Symbolic Constant-Time Verification** | Cross-ISA ELF disassembly + Valgrind `ctgrind` + symbolic execution | **$0$ `div`/`idiv` & $0$ Secret Branches** |

---

## 4. SWaP-C Footprint Summary

| Metric | Specification | Verification Notes |
|---|---|---|
| **Core Context Footprint** | **40 Bytes Total** | Self-contained zero-allocation context with zero dynamic allocation and zero global state. |
| **Dynamic Heap & Static BSS** | **0 Bytes Heap / 0 Bytes `.bss`** | Zero `malloc()` / `free()`, `0 bytes` static `.bss`/`.tbss`/`.data`. Deterministic stack-only execution. |
| **Payload Expansion (Stream Mode)** | **0 Bytes (1:1 Density)** | Ciphertext length equals plaintext length; prevents constrained wireless or avionics packet fragmentation. |
| **FPGA Hardware Footprint** | **429 Core ALMs (`0 BRAM`, `0 DSP`)** | Synthesizable RTL core verified cycle-exact against the C++ reference engine. |

---

## 5. Request Evaluation Access

* **Public Overview & Architecture:** [https://apex256.com](https://apex256.com)
* **Engineering Blog:** [https://apex256.com/blog/](https://apex256.com/blog/)
* **Public Validation & Security Margin Matrix:** [https://apex256.com/validation.html](https://apex256.com/validation.html)
* **Developer Portal & Build Forge:** [https://apex256.io](https://apex256.io)
* **Public Revision History:** [https://apex256.com/revisions.html](https://apex256.com/revisions.html)
* **Evaluation Inquiries:** **info@apex256.com**
