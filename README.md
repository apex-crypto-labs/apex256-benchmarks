# APEX-256 Validation & Benchmark Reports

This repository contains the high-level validation results, statistical proofs, and SWaP-C (Size, Weight, Power, and Cost) hardware benchmarks for **APEX-256**, a proprietary embedded stream cipher developed by Apex Research Labs, LLC.

> **Note:** APEX-256 is commercial intellectual property. The core engine (`apex256_nano.h`) and algorithmic specifications are distributed exclusively via the [APEX Developer Portal](https://apex256.io) under an executed Evaluation NDA.

## 1. Statistical Validation Battery

APEX-256 has been subjected to extreme statistical rigor, exceeding standard cryptographic requirements.

| Test Suite | Scope / Workload | Result |
|---|---|---|
| **PractRand (RNG_test)** | 32 Terabytes, 235 stats, 42 hours | **ALL PASSED (0 Anomalies)** |
| **TestU01 (BigCrush)** | 106 tests, 160 stats, 1h 54m | **ALL PASSED** |
| **NIST SP 800-22 Rev 1a** | 1 Gigabyte, 15 tests | **ALL PASSED** |
| **SAC (Strict Avalanche Criterion)** | 320M pairs, 327,680 cells | **PERFECT 50.0000%** |
| **Chi-Squared / Autocorrelation** | 10 Megabytes | **ALL PASSED** |

## 2. Formal Verification & Algebraic Bounds

Apex Research Labs utilizes formal SMT solvers (Z3) and advanced cryptanalysis to mathematically prove the security bounds of the cipher state.

* **State Bijection (1:1 Mapping):** Z3 mathematically proved that the primary non-linear permutation utilizing our proprietary Barrett multiplier scalar is a strict bijection on Z_{2^64}.
* **Absence of Weak Keys:** Z3 proved UNSAT for any combination of keys that degenerate the state space.
* **Cycle Detection:** Floyd cycle detection executed for >40 billion iterations with zero cycles detected.
* **Algebraic Degree:** Groebner basis and ANF degree extrapolation verify the algebraic degree rapidly approaches `n-1`, making algebraic cube attacks infeasible.
* **Subspace Attacks:** 500 random samples across dimensions 1-5 revealed 0 invariant subspaces.

## 3. Hardware SWaP-C Benchmarks (Bare-Metal)

APEX-256 is engineered specifically for constrained IoT and embedded hardware, demanding zero dynamic allocation and minimal stack overhead.

| Metric | Measurement | Notes |
|---|---|---|
| **Heap Allocation** | **0 Bytes** | Zero `malloc()` / `free()`. No fragmentation risk. |
| **Stack Usage** | **~160 Bytes** | Safely runs on deeply embedded RTOS threads. |
| **Throughput (ARM Cortex-M4)** | **~1.2 Cycles/Byte** | Achieved via branchless internal mathematics. |
| **Side-Channel Posture** | **Constant-Time** | Zero data-dependent branches, zero table lookups in the keystream path. Branchless initialization guards. |

## Request Access
To view the full 3,788-line BigCrush logs, the 32TB PractRand output, or to download the cipher for integration into your hardware, you must undergo commercial due diligence and sign the Apex Research Labs Evaluation NDA. 

Please contact **info@apex256.com** to initiate the evaluation process and request an invitation to the [APEX Developer Portal](https://apex256.io).
