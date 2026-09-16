# Phase 1C — Cross-Regime TLS Validation

## Objective

Establish a controlled, directly comparable TLS evidence matrix across three key-establishment regimes using the same qualified OQS/OpenSSL runtime and a fixed benign application workload.

Phase 1C is the bridge between platform qualification and later IDS/ML experimentation. Its purpose is not to measure IDS performance yet; it is to establish whether changing the cryptographic regime changes observable TLS and flow-level metadata under controlled conditions.

## Research question

When application behavior, authentication, cipher suite, capture method and runtime are held constant, what network-flow characteristics change when TLS 1.3 key establishment moves from classical X25519 to hybrid X25519+ML-KEM-768 and then to ML-KEM-768-only key establishment?

## Canonical Phase 1C conditions

All three canonical comparison conditions used the qualified OQS/OpenSSL 3.4.7 runtime with oqs-provider 0.11.0 and the same ECDSA P-256 server certificate.

| Condition | Key establishment | Authentication | TLS / cipher | Accepted runs |
|---|---|---|---|---:|
| **C1-OQS classical control** | `X25519` | ECDSA P-256 / SHA-256 | TLS 1.3 / `TLS_AES_256_GCM_SHA384` | 10 |
| **C2 hybrid** | `X25519MLKEM768` | ECDSA P-256 / SHA-256 | TLS 1.3 / `TLS_AES_256_GCM_SHA384` | 30 |
| **C3 PQC-oriented** | `mlkem768` | ECDSA P-256 / SHA-256 | TLS 1.3 / `TLS_AES_256_GCM_SHA384` | 30 |

C3 is intentionally labelled **PQC-oriented**, not fully PQC TLS, because its key establishment is ML-KEM-768 while authentication remains classical ECDSA P-256.

A preliminary 30-run classical baseline had earlier been collected with the system OpenSSL 3.0.2 runtime. That dataset was retained as exploratory evidence but was not used as the canonical same-stack Phase 1C control because the implementation differed from C2/C3. A 10-run X25519 control was therefore repeated under the exact OQS/OpenSSL 3.4.7 stack used for C2 and C3.

## Fixed workload and controls

The comparison preserved the same:

- endpoint roles and container topology;
- TLS 1.3 protocol version;
- ECDSA P-256 certificate/authentication;
- `TLS_AES_256_GCM_SHA384` cipher suite;
- fixed HTTP request and response object;
- application payload size: **121 bytes**;
- application payload SHA-256: `04b754f4158a69fbc0af6f64d0d4abb5007a24b9e4d2fc7860843ae9a0c4e986`;
- packet-capture point and filter;
- MTU and interface-offload normalization;
- capture validation and run-record schema.

The primary treatment variable in the canonical comparison was therefore the TLS key-establishment group.

## Capture-quality controls and corrective actions

Phase 1C exposed two measurement issues that were corrected before official datasets were accepted.

### Packet-capture buffering

An early automated C1 batch completed successful TLS handshakes but produced 24-byte header-only PCAP files. tcpdump reported packets received by the filter but zero packets captured because the capture process was stopped before buffered packets were flushed.

Those runs were rejected and quarantined. The collector was corrected to use immediate capture delivery, packet-buffered output, a longer drain period, explicit rejection of zero-packet/header-only PCAPs, and kernel-drop checks. A single test run was validated before batch collection resumed.

### Segmentation/offload artifact

Early diagnostic captures showed an impossible TCP payload larger than 4 KB despite a 1500-byte MTU. This was traced to segmentation/offload behavior rather than wire-level packetization. TSO, GSO and GRO were normalized across the relevant bridge, veth and container interfaces before official collection. Subsequent accepted captures showed normal segmentation with maximum observed TCP payloads of 1448 bytes in the larger PQC conditions.

## Aggregated results

| Metric | C1-OQS X25519 | C2 X25519MLKEM768 | C3 mlkem768 |
|---|---:|---:|---:|
| Accepted runs | 10 | 30 | 30 |
| Mean packet count | 17.0 | 21.0 | 21.0 |
| Mean client→server TCP payload | 534.0 B | 1710.0 B | 1678.0 B |
| Mean server→client TCP payload | 1009.9 B | 2097.9 B | 2066.0 B |
| Mean total TCP payload | 1543.9 B | 3807.9 B | 3744.0 B |
| Mean max TCP payload | 797.9 B | 1448.0 B | 1448.0 B |
| Mean handshake bytes read | 797.9 B | 1885.9 B | 1854.0 B |
| Handshake bytes written | 420 B | 1596 B | 1564 B |
| Mean flow duration | 1.762 ms | 3.725 ms | 2.535 ms |
| Flow-duration SD | 0.352 ms | 0.381 ms | 0.248 ms |
| Flow-duration range | 1.452–2.682 ms | 3.302–4.728 ms | 2.147–3.223 ms |

The byte-based measurements were highly repeatable. Total TCP payload ranges were only 1543–1545 B for C1-OQS, 3807–3809 B for C2, and 3743–3745 B for C3.

## Descriptive comparisons

Relative to C1-OQS:

- C2 increased mean packet count by **23.5%** and mean total TCP payload by **146.6%**.
- C3 increased mean packet count by **23.5%** and mean total TCP payload by **142.5%**.
- C2 increased mean handshake bytes read by **136.4%** and handshake bytes written by **280.0%**.
- C3 increased mean handshake bytes read by **132.4%** and handshake bytes written by **272.4%**.

C2 and C3 were much closer to one another than either was to C1-OQS. C2 carried approximately **1.7%** more mean total TCP payload than C3, with the same 21-packet count and the same 1448-byte maximum TCP payload.

This pattern is consistent with most of the network-footprint increase being associated with the ML-KEM portion of key establishment, while the additional X25519 component in the hybrid group contributes a comparatively small byte increase. This is a descriptive implementation-level result, not a cryptographic causality claim.

## Timing interpretation

Flow duration also differed across conditions, but timing is treated more cautiously than packet and byte counts. The conditions were collected in sequential batches rather than randomized or interleaved order, so scheduler load, warm-up state and temporal drift may affect latency.

Accordingly, Phase 1C treats packet count, directional byte volumes and handshake-size differences as the stronger protocol-level evidence. Timing remains exploratory and should be validated under randomized/interleaved collection before stronger performance claims are made.

## IDS/ML implication

The completed Phase 1C matrix shows that legitimate TLS traffic changes substantially at the flow level as key establishment moves from X25519 toward ML-KEM-based configurations. These shifts affect exactly the kinds of metadata often used by encrypted-traffic IDS/ML systems: packet count, directional byte volumes, packet-size structure, duration and handshake characteristics.

Phase 1C therefore motivates the next research question: whether detectors trained primarily on classical TLS traffic retain calibration, false-positive behavior and generalization performance when legitimate traffic transitions to hybrid or PQC-oriented TLS.

## Evidence and acceptance

Each accepted run produced private packet-capture evidence, client/server logs, run metadata and a machine-readable summary. Raw PCAPs and private keys are excluded from the public repository. Public aggregate evidence is preserved in:

- `artifacts/sanitized/phase1c_cross_regime_summary.md`
- `results/phase1c-summary.csv`
- `results/results.csv`

## Relationship to Phase 1B

Phase 1B established that the qualified environment can reproducibly generate and capture the target hybrid/PQC-capable TLS handshake and survive a clean rebuild.

Phase 1C did not reopen that acceptance decision. It extended the platform with a controlled classical/hybrid/PQC-oriented comparison matrix under a common workload and, for the canonical comparison, a common OQS/OpenSSL runtime.

## Current status

**Complete.**

The canonical same-stack Phase 1C matrix comprises 10 accepted C1-OQS runs, 30 accepted C2 runs and 30 accepted C3 runs. The next experimental stage is IDS/ML dataset construction and cross-regime generalization testing.
