# APEX-256 Validation & Benchmark Reports (v2.1.2.1)

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
| **Strict Avalanche Criterion (SAC)** | 320M trial pairs across 327,680 bit-dependency cells (6σ bound) | **PERFECT 50.0000% Mean** |
| **Chi-Squared / Bit Balance / Autocorrelation** | 10 Megabytes | **ALL PASSED** |

---

## 2. Formal SMT Verification & Algebraic Cryptanalysis

Apex Research Labs utilizes formal bitvector SMT solvers (**Microsoft Z3**) and algebraic cryptanalysis to prove the structural bounds of the 256-bit state manifold:

* **Single-Step State Bijection (`PROVED UNSAT`):** Exhaustive 64-bit Z3 SMT bitvector verification proved that the primary non-linear shear permutation combined with its coprime odd modular multiplication scalar (`gcd(C, 2^64) = 1`) is a strict mathematical bijection on $\mathbb{Z}_{2^{64}}$ with zero entropy loss per step.
* **Absence of Weak Keys (`PROVED UNSAT`):** Z3 proved `UNSAT` for key-schedule state collapse, enforced by branchless bitwise mask guards during initialization.
* **Fixed-Point Isolation & Constant-Time 24-Round Mitigation:** Z3 isolated the single non-trivial fixed point of the unwhitened primary step (random reachability $< 2^{-192}$), which is unconditionally neutralized via a branchless probe and constant-time 24-round (`12 + 12`) coupled initialization sequence.
* **Multi-Nonce & Key-Nonce Equivalence Immunity:** Asymmetric non-linear key-nonce binding across all four 64-bit registers (`state_x`, `state_y`, `state_z`, `sm64_state`) and 12-round bidirectional burn-in coupling eliminate linear key-nonce equivalence classes (`0 / 1,000` collisions).
* **Cycle & Orbit Analysis:** Floyd cycle detection executed for $> 40 \times 10^9$ iterations with zero cycles detected; lattice orbit analysis across all 7 $v_2(k)$ valuation classes confirmed 0 degenerate trajectories.
* **Algebraic Degree & Subspace Bounds:** Cube attack degree testing (degree $\ge 16$ at 4 rounds), Groebner basis / ANF degree extrapolation (approaching $n-1$), and 500-sample linear subspace probing across dimensions 1–5 confirmed 0 invariant subspaces.

---

## 3. Companion Primitive Cryptanalysis Suite (v2.1.0.0+)

All companion primitives sharing the 256-bit coupled Rate-Capacity sponge (`64-bit` rate + `192-bit` hidden capacity) are continuously verified against the 13-check adversarial cryptanalysis harness:

| Companion Primitive | Security Verification & Architectural Posture | Status |
|---|---|---|
| **APEX-HASH (256-bit Sponge)** | Per-byte non-zero domain marker (`0x80 \| b`), 100% streaming chunk associativity, and 4-register ratcheted output squeeze. | **PASSED** |
| **APEX-AEAD (Encrypt-then-MAC)** | Domain-separated (`0xA01` / `0xA02` / `0xA03`) 64-bit LE length-encoded MAC with trailing key envelope and constant-time tag verification prior to decryption (strict RUP immunity). | **PASSED** |
| **APEX-DRBG** | Coupled sponge generation over all 4 state registers with mandatory post-request backtracking ratchet (forward & backward secrecy). | **PASSED** |
| **APEX-VFS (Sector Disk Cipher)** | O(1) random-access 512-byte sector encryption bound to `sector_index` and `write_epoch` anti-replay counter. | **PASSED** |
| **APEX-CFB** | Lossless 64-bit barrel-rotated (`rotl64`) ciphertext feedback across dual state registers (`0 / 512` silent bit truncations). | **PASSED** |
| **APEX-FPE (Format-Preserving)** | Unconditional keystream stepping with 64-bit folded modulo-10 reduction ($\chi^2 = 8.68$) and branchless ASCII digit masking. | **PASSED** |
| **APEX-STEALTH (EM / Power SCA)** | Branchless complementary Hamming-weight register equalization (`ct` vs. `~ct`) and cross-compiler `APEX_NOP()` execution barriers. | **PASSED** |

---

## 4. Side-Channel Posture & Bare-Metal SWaP-C Architecture

APEX-256 is engineered for Size, Weight, Power, and Cost (SWaP-C) constrained embedded targets (`ARM Cortex-M4`, `Espressif ESP32-S3`, `Nordic nRF52840`, and `RISC-V`):

| Metric | Specification | Verification Notes |
|---|---|---|
| **Core State Footprint** | **32 Bytes (256 Bits)** | Four 64-bit registers (`state_x`, `state_y`, `state_z`, `sm64_state`). |
| **Dynamic Heap Allocation** | **0 Bytes** | Zero `malloc()` / `free()`. Deterministic stack-only execution for bare-metal RTOS threads. |
| **Payload Expansion (Stream Mode)** | **0 Bytes (1:1 Density)** | Ciphertext length equals plaintext length; prevents sub-GHz / LoRa / BLE MTU packet fragmentation. |
| **Constant-Time Verification (Dudect)** | **Welch's $\|t\| = 1.12 < 4.5$** | Certified constant-time across 200,000+ cycle measurements; zero data-dependent branches or S-box lookup tables. |
| **Custom Module Stripping** | **Build Forge `#ifdef` AST Pruning** | Unselected companion modules are physically stripped at build time to minimize Flash footprint. |

> **Reproducible Silicon Benchmarks:** Cycle-per-byte, Flash footprint, and microamp power captures (recorded on physical `STM32F405` and `ESP32-S3` boards using a Nordic PPK2 and Saleae Logic Pro 8 in our Henderson, NV lab) depend on compiler flags and target silicon. Rather than publishing uncontextualized marketing numbers, we ship the full reproducible benchmark harness and raw logs inside the **Engineering Evaluation Kit**.

---

## 5. Request Evaluation Access

To inspect the full 3,788-line BigCrush logs, the 32 TB PractRand output, the Z3 SMT proof scripts, or to provision a 3LZCF-watermarked SDK build:

* **Public & Defense Overview:** [https://apex256.com](https://apex256.com)
* **Developer Portal & Build Forge:** [https://apex256.io](https://apex256.io)
* **Public Security Ledger:** [https://apex256.io/security-ledger](https://apex256.io/security-ledger)
* **Evaluation Inquiries:** **info@apex256.com**
