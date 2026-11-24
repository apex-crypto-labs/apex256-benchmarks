# 3-Run Valgrind Massif (`--stacks=yes`) Results: `wolfSSL 5.9.4` CNSA 2.0 + `APEX-256 v2.5.0.4` Modular Single-Responsibility Suite

**Environment:**
- **OS / Distro:** Yocto Scarthgap (`poky 5.0.20`), `Linux 6.6.21-yocto-standard x86_64` (KVM)
- **Libraries & Binaries:**
  - `wolfSSL 5.9.4` (`libwolfssl.so.46.0.0`: `489,600 bytes` stripped; `.text = 477,426 B`, `.data = 4,848 B`, `.bss = 164 B`)
  - `OpenSSL 3.5.8` (`libssl.so.3` + `libcrypto.so.3`: `7,147,728 bytes`; `.data + .bss = 571.7 KiB`)
  - `APEX-256 v2.5.0.4` (`40-byte` zero-allocation core context + modular single-responsibility `40B-48B` / stateless companion interfaces, compiled with `-std=c++17 -fno-exceptions -fno-rtti -O2` so `DT_NEEDED` links strictly `libwolfssl.so.46` and `libc.so.6` with `0 B` `libstdc++` runtime heap)
- **`wolfSSL 5.9.4` Configure Flags (matching A. Hu `REPORT_CNSA_2.0.md`):**
  ```
  --enable-tls13 --disable-oldtls
  --disable-rsa --disable-dh --disable-dsa --disable-psk
  --enable-sha384 --disable-sha512
  --disable-chacha --disable-poly1305 --disable-aescbc
  --disable-ecc --disable-curve25519 --disable-ed25519
  --enable-mlkem=1024,make,enc,dec,small --enable-tls-mlkem-standalone --disable-pqc-hybrids
  --enable-mldsa=87,sign,verify,small
  --disable-sp-math-all --enable-sp-math
  --enable-smallstack --enable-lowresource --enable-maxfragment
  --enable-singlethreaded
  CFLAGS += -DHAVE_SUPPORTED_CURVES
            -DWOLFSSL_MLKEM_MAKEKEY_SMALL_MEM -DWOLFSSL_MLKEM_ENCAPSULATE_SMALL_MEM -DWOLFSSL_MLKEM_DYNAMIC_KEYS
            -DWOLFSSL_MLDSA_SIGN_SMALL_MEM -DWOLFSSL_MLDSA_VERIFY_SMALL_MEM -DWOLFSSL_MLDSA_DYNAMIC_KEYS
  ```

---

## 3-Run Summary Table (`Run 1`, `Run 2`, `Run 3` & Mean)

Instruction-level Valgrind Massif (`--stacks=yes --peak-inaccuracy=0.1`) peaks were deterministic (`+/- 0.00 KiB`) across all three independent passes (`run_1`, `run_2`, `run_3`).

| Scenario | Role | Cryptographic Stack | Run 1 (KiB) | Run 2 (KiB) | Run 3 (KiB) | 3-Run Mean Peak (KiB) | Max Heap (KiB) | Active Stack (KiB) | Mean RSS (KiB) | Reduction vs. OpenSSL 3.5.8 |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| **`cnsa20_wolfserver_wolfclient`** | **Server** | **`wolfSSL 5.9.4` TLS 1.3 CNSA 2.0 (Tuned, MFL 512)** | `64.74` | `64.74` | `64.74` | **`64.74 KiB`** | `62.06` | `2.73` | `2,261` | **`14.6x` smaller** |
| **`cnsa20_wolfserver_wolfclient`** | **Client** | **`wolfSSL 5.9.4` TLS 1.3 CNSA 2.0 (Tuned, MFL 512)** | `39.31` | `39.31` | `39.31` | **`39.31 KiB`** | `35.50` | `3.34` | `2,304` | **`23.9x` smaller** |
| **`cnsa20_wolfserver_wolfclient_nomfl`** | **Server** | **`wolfSSL 5.9.4` TLS 1.3 CNSA 2.0 (Tuned, no MFL)** | `64.79` | `64.79` | `64.79` | **`64.79 KiB`** | `62.14` | `2.64` | `2,261` | **`14.6x` smaller** |
| **`cnsa20_wolfserver_wolfclient_nomfl`** | **Client** | **`wolfSSL 5.9.4` TLS 1.3 CNSA 2.0 (Tuned, no MFL)** | `38.74` | `38.74` | `38.74` | **`38.74 KiB`** | `34.99` | `3.42` | `2,261` | **`24.3x` smaller** |
| **`cnsa20_wolfserver_osslclient`** | **Server** | **`wolfSSL 5.9.4` Server vs. `OpenSSL 3.5.8` Client** | `64.74` | `64.74` | `64.74` | **`64.74 KiB`** | `62.06` | `2.73` | `2,176` | **`14.6x` smaller** |
| **`cnsa20_osslserver_wolfclient`** | **Client** | **`wolfSSL 5.9.4` Client vs. `OpenSSL 3.5.8` Server** | `39.33` | `39.33` | `39.33` | **`39.33 KiB`** | `35.50` | `3.39` | `2,261` | **`23.9x` smaller** |
| **`cnsa20_apex_endpoint`** | **Server** | **`wolfCrypt` CNSA 2.0 (`ML-KEM-1024` + `ML-DSA-87` DER) + `APEX-256 v2.5.0.4`** | `39.03` | `39.03` | `39.03` | **`39.03 KiB`** | `32.07` | `8.36` | `1,877` | **`24.3x` smaller** (`0 B` data-plane heap) |
| **`cnsa20_apex_endpoint`** | **Client** | **`wolfCrypt` CNSA 2.0 (`ML-KEM-1024` + `ML-DSA-87` DER) + `APEX-256 v2.5.0.4`** | `19.97` | `19.97` | `19.97` | **`19.97 KiB`** | `13.70` | `6.25` | `1,963` | **`47.0x` smaller** (`0 B` data-plane heap) |
| **`apex_pure_endpoint`** | **Server/Client** | **Pure `APEX-256 v2.5.0.4` Steady-State (`0-Heap`)** | `1.87` *(7.45 ld.so)* | `1.87` | `1.87` | **`1.87 KiB` active** | **`0.00`** | `1.96` | `1,578` | **`0 B` heap across all snapshots** |
| **`apex_v250_primitives`** | **Server/Client** | **Full `APEX-256 v2.5.0.4` All-9 Companion Modes (`0-Heap`)** | `1.87` *(7.45 ld.so)* | `1.87` | `1.87` | **`1.87 KiB` active** | **`0.00`** | `1.87` | `1,621` | **`0 B` heap across all snapshots** |
| **`cnsa20_osslserver_wolfclient`** | **Server** | **`OpenSSL 3.5.8` TLS 1.3 CNSA 2.0 (MFL 512)** | `946.47` | `946.47` | `946.47` | **`946.47 KiB`** | `823.70` | `8.52` | `9,045` | `1.0x` baseline |
| **`cnsa20_wolfserver_osslclient`** | **Client** | **`OpenSSL 3.5.8` TLS 1.3 CNSA 2.0 (MFL 512)** | `939.63` | `939.63` | `939.63` | **`939.63 KiB`** | `816.09` | `11.39` | `9,216` | `1.0x` baseline |
| **`baseline_cnsa20_wolfserver`** | **Server** | **`wolfSSL 5.9.4` `WOLFSSL_CTX` Init-Only Baseline** | `31.27` | `31.27` | `31.27` | **`31.27 KiB`** | `29.40` | `2.72` | - | **`22.7x` smaller** than OpenSSL (`710.46 KiB`) |
| **`baseline_cnsa20_apex`** | **Server** | **`wolfCrypt` DER `ML-DSA-87` + `APEX-256 v2.5.0.4` Init-Only** | `10.98` | `10.98` | `10.98` | **`10.98 KiB`** | `4.78` | `6.43` | - | **`64.7x` smaller** than OpenSSL (`710.46 KiB`) |

---

## Key Engineering Notes vs. Prior `v2.3.2 / v2.4.x` Run
1. **Modular Single-Responsibility Refactor (`v2.5.0.4`):** Splitting the monolithic wrapper into standalone `40 B` / `48 B` and stateless zero-allocation interfaces shaved peak stack on both `cnsa20_apex_endpoint` endpoints, pushing the `wolfCrypt` CNSA 2.0 + `APEX-256` client **below `20 KiB` total (`19.97 KiB` = `13.70 KiB` heap + `6.25 KiB` stack)** and server to **`39.03 KiB`** (`1,877 KiB` RSS), while exercising all 9 companion modes (`apex_v250_primitives`) in **`0.00 KiB` heap and `1.87 KiB` active stack**.
2. **Direct DER Key Loading (`4.78 KiB` Idle Heap vs. `29.40 KiB` PEM):** Loading pre-decoded DER keys directly into `wolfCrypt` `ML-DSA-87` avoids `PemToDer` base64 buffer retention (`17,140 B`), dropping server idle heap to `4,896 B` (`4.78 KiB`).
3. **Freestanding C++ Linkage (`-fno-exceptions -fno-rtti`):** Adding RAII zeroizing destructors in C++ pulls in `libstdc++.so.6` (`73,728 B` / `72.00 KiB` emergency exception pool in `_dl_init`) if compiled with default `-fexceptions`. Building with `${CC} -x c++ -std=c++17 -fno-exceptions -fno-rtti` preserves automatic destructor zeroization while linking strictly `[libwolfssl.so.46, libc.so.6]` with `0 B` C++ runtime heap.

*Note: Internal `APEX-256` specifications, reference headers, and KAT suites are proprietary to Apex Research Labs, LLC.*
