# Phase 1C — Sanitized Cross-Regime TLS Summary

## Supported result IDs

This artifact supports:

- `P1C-TLS-CLASSICAL-001`
- `P1C-TLS-HYBRID-001`
- `P1C-TLS-PQC-001`
- `P1C-PCAP-001`
- `P1C-COMPARABILITY-001`
- `P1C-IMPLEMENTATION-CONTROL-001`

## Purpose

Phase 1C tests whether changing TLS 1.3 key establishment produces repeatable changes in observable network-flow metadata when the runtime, authentication, cipher suite, application payload and capture method are held constant as practical.

## Qualified runtime

Canonical Phase 1C comparison conditions used:

- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0
- OQS OpenSSL binary under `/opt/oqs-provider/.local/bin/openssl`
- provider modules loaded from `/opt/oqs-provider/_build/lib`

## Controlled TLS conditions

| Condition | Key establishment | Authentication | TLS / cipher | Runs |
|---|---|---|---|---:|
| C1-OQS | `X25519` | ECDSA P-256 / SHA-256 | TLS 1.3 / `TLS_AES_256_GCM_SHA384` | 10 |
| C2 | `X25519MLKEM768` | ECDSA P-256 / SHA-256 | TLS 1.3 / `TLS_AES_256_GCM_SHA384` | 30 |
| C3 | `mlkem768` | ECDSA P-256 / SHA-256 | TLS 1.3 / `TLS_AES_256_GCM_SHA384` | 30 |

C3 is **PQC-oriented**, not fully PQC TLS, because key establishment is ML-KEM-768 while authentication remains classical ECDSA P-256.

## Fixed workload

All canonical comparison runs used the same benign HTTP object:

- payload size: 121 bytes
- payload SHA-256: `04b754f4158a69fbc0af6f64d0d4abb5007a24b9e4d2fc7860843ae9a0c4e986`
- session tickets disabled for the controlled workload
- same capture point and flow filter across the canonical comparison

## Aggregate measurements

| Metric | C1-OQS | C2 | C3 |
|---|---:|---:|---:|
| Accepted runs | 10 | 30 | 30 |
| Mean packets | 17.0 | 21.0 | 21.0 |
| Mean C→S TCP payload | 534.0 B | 1710.0 B | 1678.0 B |
| Mean S→C TCP payload | 1009.9 B | 2097.9 B | 2066.0 B |
| Mean total TCP payload | 1543.9 B | 3807.9 B | 3744.0 B |
| Total TCP payload range | 1543–1545 B | 3807–3809 B | 3743–3745 B |
| Mean max TCP payload | 797.9 B | 1448.0 B | 1448.0 B |
| Mean handshake bytes read | 797.9 B | 1885.9 B | 1854.0 B |
| Handshake bytes written | 420 B | 1596 B | 1564 B |
| Mean flow duration | 1.762 ms | 3.725 ms | 2.535 ms |
| Median flow duration | 1.686 ms | 3.520 ms | 2.493 ms |
| Flow-duration SD | 0.352 ms | 0.381 ms | 0.248 ms |
| Flow-duration range | 1.452–2.682 ms | 3.302–4.728 ms | 2.147–3.223 ms |

## Descriptive effect sizes

Relative to C1-OQS:

- C2 packet count: **+23.5%**
- C2 mean total TCP payload: **+146.6%**
- C2 mean handshake bytes read: **+136.4%**
- C2 handshake bytes written: **+280.0%**
- C3 packet count: **+23.5%**
- C3 mean total TCP payload: **+142.5%**
- C3 mean handshake bytes read: **+132.4%**
- C3 handshake bytes written: **+272.4%**

C2 versus C3:

- same mean packet count: 21
- same maximum TCP payload: 1448 B
- C2 mean total TCP payload was approximately **1.7%** larger than C3
- C2 added about 32 B client→server and 31.9 B server→client on average relative to C3

The C2/C3 pattern is consistent with most of the structural traffic increase being associated with ML-KEM material, while the additional X25519 component in the hybrid group contributes a smaller byte increment. This is a descriptive implementation-level observation, not a formal causal decomposition of cryptographic cost.

## Implementation-control decision

A preliminary 30-run classical X25519 baseline was first collected with system OpenSSL 3.0.2. Because C2 required the qualified OQS/OpenSSL 3.4.7 runtime, that created an implementation-version confound.

A 10-run X25519 sanity control was therefore repeated under the exact OQS/OpenSSL 3.4.7 runtime used for C2 and C3. The runtime change modestly increased client-side handshake traffic, but packet count, server-to-client traffic, handshake-read size and maximum TCP payload remained broadly similar to the earlier classical baseline. The canonical Phase 1C comparison therefore uses C1-OQS rather than the preliminary system-OpenSSL baseline.

## Capture-quality corrections

### Buffering failure

An early automated batch produced PCAP files containing only the global header despite packets being received by the capture filter. Those runs were rejected and quarantined. The collector was corrected to use immediate capture delivery, packet-buffered output, a drain interval and explicit zero-packet/header-only rejection.

### Offload artifact

An early diagnostic capture contained an impossible >4 KB TCP payload on a 1500-byte MTU path. The artifact was traced to segmentation/offload behavior. TSO/GSO/GRO were normalized across the relevant virtual interfaces before official collection. Accepted PQC-oriented captures showed normal segmentation with maximum TCP payloads of 1448 B.

## Interpretation

Under a common OQS/OpenSSL runtime, both ML-KEM-based conditions produced substantially different flow-level metadata from classical X25519 TLS. The strongest differences were packet count, directional byte volume, total TCP payload and handshake size.

These results motivate the next experiment: test whether IDS/ML systems trained predominantly on classical TLS exhibit changed calibration, false-positive behavior or generalization performance when legitimate traffic shifts to hybrid or PQC-oriented TLS.

## Timing limitation

Flow-duration measurements are retained as exploratory. Conditions were collected in sequential batches rather than randomized/interleaved order, so timing may be influenced by scheduling, host load, warm-up state or temporal drift. Packet and byte measurements are therefore treated as the stronger Phase 1C evidence.

## Public/private evidence boundary

Raw PCAPs, private keys and sensitive environment details are not committed to this repository. This artifact contains only sanitized aggregate measurements and methodology notes. Private raw evidence is retained separately under the research provenance process.
