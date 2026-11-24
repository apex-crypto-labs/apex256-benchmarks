# A CNSA 2.0 handshake in under 20 KiB, and what happens to the heap afterward

By Diab Anthony Sheteyh, Apex Research Labs, LLC

Earlier this month Anthony Hu at wolfSSL published a memory study that I think more embedded people should read. He built a CNSA 2.0 TLS 1.3 connection (ML-KEM-1024 key exchange, an ML-DSA-87 certificate chain, AES-256-GCM) on a Yocto Scarthgap image and on OpenWrt, then measured both OpenSSL and a stripped-down wolfSSL with Valgrind Massif. Every flag, every pinned commit and every helper script is in the report, which is the only reason I could do what follows. His report is here: https://github.com/anhu/demos_and_experiments/blob/master/yocto_openwrt_memory_usage/REPORT_CNSA_2.0.md

I rebuilt his setup from scratch and audited his numbers against my own before changing anything, because I don't trust a number I haven't reproduced. My bench is a 32-thread box, so the cold Yocto build took about half an hour instead of the few hours his notes warn about.

My numbers landed on top of his. He reports about 940 KiB for the OpenSSL client, 38.9 KiB for the wolfSSL client and 63.8 KiB for the wolfSSL server. I measured 939.6, 39.3 and 64.7. All three runs were byte-identical to each other. That is what a good methodology looks like, and it is worth saying out loud.

## Where the memory actually goes

The most useful part of his report is the allocation breakdown. He resolved the peak snapshot down to individual call sites. On the server, the ML-DSA-87 signing scratch is about 27.8 KB and the PEM-to-DER buffers for the certificate and key hold another 17 KB for the life of the context. ML-KEM-1024 itself is only around 3 KB on the server and 4.7 KB on the client. In other words, the post-quantum key exchange everyone worries about is the cheap part. The signature scheme and the certificate handling are where the bytes go.

He also noted that loading DER directly would probably save several KB, but he didn't test it. I did, sort of, with a caveat I want to be upfront about.

## What I changed

I took the same wolfCrypt build, with his SMALL_MEM and DYNAMIC_KEYS options exactly as published, and ran the ML-KEM-1024 and ML-DSA-87 exchange directly through wolfCrypt with the keys loaded as raw DER. I did not run a full TLS 1.3 state machine and I did not parse an X.509 chain. That matters, because his 38.9 and 63.8 KiB figures include all of that, and mine do not. They are different measurements and nobody should line them up as if they were the same test.

With that said, here is what came out:

| Endpoint | Peak (heap + stack) | Peak heap |
|---|---|---|
| OpenSSL 3.5.8, CNSA 2.0 client | 939.6 KiB | 816.1 KiB |
| wolfSSL 5.9.4 low-RAM client (Anthony's flags) | 39.3 KiB | 35.5 KiB |
| wolfCrypt ML-KEM-1024 + ML-DSA-87, DER keys, client | 19.97 KiB | 13.70 KiB |
| wolfSSL 5.9.4 low-RAM server (Anthony's flags) | 64.7 KiB | 62.1 KiB |
| wolfCrypt ML-KEM-1024 + ML-DSA-87, DER keys, server | 39.03 KiB | 32.07 KiB |

The client heap is almost entirely the verification scratch from his table plus the public key:

2,592 B + 11,433 B = 14,025 B = 13.70 KiB

That is exactly what Massif reports as the client's maximum heap. I take that as a sign that everything wrapped around the wolfCrypt calls, the transcript hashing and the key derivation, added nothing to the heap on top of it. The floor for this operation is set by wolfCrypt's own ML-DSA work, and Anthony's SMALL_MEM options are what make that floor so low in the first place.

## What happens after the handshake

Once the session keys exist, the post-quantum material is freed. In my setup the steady-state data path is APEX-256, a stream and AEAD layer I design, which uses a 40-byte context and allocates nothing. Across all three runs it sat at 0.00 KiB of heap with 1.87 KiB of active stack. The security model is layered: ML-KEM and ML-DSA handle key establishment and authentication under the NIST standards, Ascon handles lightweight authenticated framing, and APEX-256 sits underneath as an inner payload layer. It does not replace any of the standardized pieces.

I'm not publishing the internals of APEX-256 here. The memory behavior is easy to verify from the outside, and I've attached the raw Massif, ms_print and time output from all three runs so you can check it yourself.

## One build detail worth sharing

If you write C++ with destructors that scrub secrets, check what your toolchain links. My first run showed a 72 KiB allocation happening inside the dynamic loader before main() started. The cause was libstdc++ being pulled in for exception handling, which allocates an emergency pool at startup. Building with -fno-exceptions -fno-rtti removed it, and the binary went back to linking only libwolfssl and libc. It cost me an afternoon of staring at call trees, so maybe it saves someone else one.

## Credit

All of this sits on work wolfSSL and Anthony Hu already did and gave away. The small-memory ML-KEM and ML-DSA paths, the build flags, the reproducible kit and the honest accounting of where the bytes go are the reason a number like 13.70 KiB is even possible. If you're doing post-quantum work on constrained hardware, start with his report, and take a serious look at wolfSSL while you're there.

Raw logs from my runs: wolfssl_5.9.4_cnsa20_apex256_v2504_3runs.tar.gz
</Content>

---

*Published on the Apex Research Labs Engineering Blog: https://apex256.com/blog/cnsa20-under-20kib.html*
*Detailed 3-run summary table: [RESULTS_3RUNS_SUMMARY.md](RESULTS_3RUNS_SUMMARY.md)*
*Browse raw Massif, `ms_print` ASCII charts, and `time -v` logs in-browser: [`run_1/`](run_1/) | [`run_2/`](run_2/) | [`run_3/`](run_3/)*
*Single-file tarball download: [wolfssl_5.9.4_cnsa20_apex256_v2504_3runs.tar.gz](wolfssl_5.9.4_cnsa20_apex256_v2504_3runs.tar.gz)*
