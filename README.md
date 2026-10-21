# APEX-256 Validation & Benchmark Reports (v2.1.6.4)

This repository contains the high-level validation results, formal SMT verification summaries, side-channel leakage evaluations, and hardware SWaP-C (Size, Weight, Power, and Cost) metrics for **APEX-256**, a proprietary embedded stream cipher and cryptographic primitive suite developed by **Apex Research Labs, LLC** (Inventor: Diab Anthony Sheteyh).

> **Note:** APEX-256 is commercial intellectual property protected by 5 pending U.S. Provisional Patent Applications. The reference engine (`apex256_nano.h`), verification harnesses, and authoritative Known Answer Tests (KATs) are distributed exclusively via the [APEX Developer Portal (apex256.io)](https://apex256.io) under an executed Evaluation NDA with **3-Layer Zero-Callback Fingerprinting (3LZCF)**.
>
> Publicly auditable release engineering and security advisories are maintained on the [APEX-256 Public Security & Engineering Ledger](https://apex256.io/security-ledger).

---

## 1. Empirical Statistical Validation Battery

APEX-256 has passed the industry's most demanding empirical statistical randomness batteries with zero anomalies:

| Test Suite | Scope / Workload | Result |
|---|---|---|
| **PractRand (`RNG_test`)** | 32 Terabytes continuous keystream, 235 test statistics, 42 hours | **ALL PASSED (0 Anomalies)** |
| **TestU01 (`BigCrush`)** | 106 tests, 160 test statistics, 1h 54m | **ALL PASSED (160/160)** |
| **NIST SP 800-22 Rev. 1a** | 1 Gigabyte, 15 statistical tests | **ALL PASSED (15/15)** |
| **Strict Avalanche Criterion (SAC)** | 320M trial pairs across 327,680 bit-dependency cells (6-sigma bound) | **PERFECT 50.0000% Mean** |
| **Chi-Squared / Bit Balance / Autocorrelation** | 10 Megabytes | **ALL PASSED** |

---

## 2. Formal SMT Verification, Wave-1 & Wave-2 Cryptanalysis (v2.1.6.4)

Apex Research Labs utilizes formal bitvector SMT solvers (**Microsoft Z3**), CASCADA Rotational-XOR (`RX`) trail search, CLAASP residual neural classifiers, CROCS `BoolTest` polynomial distinguishers, fast Walsh-Hadamard spectral transforms, and `x86_64` symbolic binary taint analysis to verify the 256-bit state manifold:

| Evaluation Vector | Scope / Methodology | Result |
|---|---|---|
| **Single-Step State Bijection** | 64-bit Z3 SMT bitvector injectivity across all four 64-bit registers | **PROVED UNSAT (Strict Bijection)** |
| **Same-Nonce 256-Bit Key Collision Search** | Exhaustive 256-bit symbolic Z3 search for $K_1 \neq K_2$ collapsing to identical post-binding state | **PROVED UNSAT ($0$ Collisions across $2^{256}$ Keys)** |
| **Cryptol / SAW Reference Equivalence** | 5 symbolic equivalence & cross-primitive domain-disjointness theorems (`STREAM != HASH != DRBG != VFS`) | **5 / 5 PROVED UNSAT** |
| **CASCADA Rotational-XOR (`RX`) Trail Search** | 63 symbolic RX differential & invariant queries across rotation offsets $\gamma \in \{1, 3, 4, 5, 7, 8, 13, 16, 17, 29, 31, 32, 47\}$ | **63 / 63 UNSAT ($0$ RX Trails)** |
| **Exact Symbolic Dependency Cone** | Bit-level Z3 dependency cone over all 320 key+nonce bits (`256` key + `64` nonce) | **320 / 320 Full Support at $R = 3$ ($8\times$ Margin)** |
| **CLAASP Avalanche & Gohr Neural Distinguisher** | Exact $320 \times 256$ diffusion matrix + multi-layer residual classifier ($R = 0..24$) | **$50.00\%$ SAC & $0.5000$ Neural Null at $R = 1$ ($24\times$ Margin)** |
| **CROCS `BoolTest` Polynomial Battery** | Degree $d \in \{1, 2, 3\}$ Boolean monomial distinguishers over 128/256/512-bit blocks ($R = 0..24$) | **$\|Z_{\max}\| \le 3.55 < 5.0\sigma$ for All $R \ge 1$** |
| **Dinur-Shamir Cube & Integral Attack** | $2^d$ hypercube superpoly summation ($d \in \{4, 8, 10, 12\}$) + 8-bit exact ANF symbolic degree | **$0 / 2,048$ Biased Superpolys at $R \ge 1$** |
| **FWHT Spectral Coordinate Slice Analysis** | Fast Walsh-Hadamard & autocorrelation spectrum across 8-bit Q32.32 slices (`sboxU`) | **$NL = 110$, $\delta = 8$, $CI = 0$, $AI = 4$ (Matches Random Bijection)** |
| **Binary Symbolic Constant-Time Verification** | `x86_64` ELF instruction inspection across `-O0`, `-O1`, `-O2`, `-O3`, `-Os`, `-march=native` (`36` pairs) | **$0$ `div`/`idiv` & $0$ Secret `Jcc` (`36 / 36 PASS`)** |

---

## 3. Companion Primitive Cryptanalysis Suite (v2.1.6.4)

All companion primitives sharing the 256-bit coupled Rate-Capacity sponge (`64-bit` rate + `192-bit` hidden capacity) are continuously verified against the Wave-1 and Wave-2 adversarial cryptanalysis harnesses:

| Companion Primitive | Security Verification & Architectural Posture | Status |
|---|---|---|
| **APEX-HASH (256-bit Sponge)** | Per-byte non-zero domain marker (`0x80 \| b`), 100% streaming chunk associativity, and 4-register ratcheted output squeeze ($0$ rotational differentials). | **PASSED** |
| **APEX-AEAD (Encrypt-then-MAC)** | Domain-separated (`0xA01` / `0xA02` / `0xA03`) 64-bit LE length-encoded MAC with trailing key envelope and constant-time tag verification prior to decryption (strict RUP immunity). | **PASSED** |
| **APEX-DRBG** | Coupled sponge generation over all 4 state registers with mandatory post-request backtracking ratchet (forward & backward secrecy, $50.01\%$ cross-instance bit distance). | **PASSED** |
| **APEX-VFS (Sector Disk Cipher)** | O(1) random-access 512-byte sector encryption bound to `sector_index` and `write_epoch` anti-replay counter ($50.00\%$ adjacent-sector bit distance). | **PASSED** |
| **APEX-CFB** | Lossless 64-bit barrel-rotated (`rotl64`) ciphertext feedback across all 4 state registers ($50.01\%$ immediate Round-1 avalanche). | **PASSED** |
| **APEX-FPE (Format-Preserving)** | Lemire fast divisionless multiply-high (`__uint128_t`) + 2-round Feistel-10 digit permutation ($\chi^2 = 5.45$, $0$ `div`/`idiv` instructions). | **PASSED** |
| **APEX-STEALTH (EM / Power SCA)** | Branchless complementary Hamming-weight register equalization (`ct` vs. `~ct`, constant $HW = 64$) and cross-compiler `APEX_NOP()` execution barriers. | **PASSED** |

---

## 4. Side-Channel Posture & Bare-Metal SWaP-C Architecture

APEX-256 is engineered for Size, Weight, Power, and Cost (SWaP-C) constrained embedded targets (`ARM Cortex-M4`, `Espressif ESP32-S3`, `Nordic nRF52840`, and `RISC-V`):

| Metric | Specification | Verification Notes |
|---|---|---|
| **Core State Footprint** | **32 Bytes (256 Bits)** | Four 64-bit registers (`state_x`, `state_y`, `state_z`, `sm64_state`). |
| **Dynamic Heap Allocation** | **0 Bytes** | Zero `malloc()` / `free()`. Deterministic stack-only execution for bare-metal RTOS threads. |
| **Payload Expansion (Stream Mode)** | **0 Bytes (1:1 Density)** | Ciphertext length equals plaintext length; prevents sub-GHz / LoRa / BLE MTU packet fragmentation. |
| **Constant-Time Verification (Dudect & Binary Taint)** | **Welch's $\|t\| = 1.12 < 4.5$ & $0$ Secret Branches** | Certified constant-time across 200,000+ cycle measurements and `36 / 36` `x86_64` compiler optimization matrix cells. |
| **Custom Module Stripping** | **Build Forge `#ifdef` AST Pruning** | Unselected companion modules are physically stripped at build time to minimize Flash footprint. |

> **Reproducible Silicon Benchmarks:** Cycle-per-byte, Flash footprint, and microamp power captures (recorded on physical `STM32F405` and `ESP32-S3` boards using a Nordic PPK2 and Saleae Logic Pro 8 in our Henderson, NV lab) depend on compiler flags and target silicon. Rather than publishing uncontextualized marketing numbers, we ship the full reproducible benchmark harness and raw logs inside the **Engineering Evaluation Kit**.

---

## 5. Request Evaluation Access

To inspect the full 3,788-line BigCrush logs, the 32 TB PractRand output, the Z3/CASCADA/CLAASP/BoolTest formal proof scripts, or to provision a 3LZCF-watermarked SDK build:

* **Public & Defense Overview:** [https://apex256.com](https://apex256.com)
* **Public Validation & Security Margin Matrix:** [https://apex256.com/validation.html](https://apex256.com/validation.html)
* **Developer Portal & Build Forge:** [https://apex256.io](https://apex256.io)
* **Public Security Ledger:** [https://apex256.io/security-ledger](https://apex256.io/security-ledger)
* **Evaluation Inquiries:** **info@apex256.com**
