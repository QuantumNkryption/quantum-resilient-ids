# Reproducibility Instructions

## Scope

These instructions reproduce the research platform through the verified Phase 1A, Phase 1B and Phase 1C boundary. They do **not** claim that later IDS/ML experiments have been reproduced yet.

## 1. Clone the repository

```bash
git clone https://github.com/QuantumNkryption/quantum-resilient-ids.git
cd quantum-resilient-ids
git checkout main
```

Record the exact commit used:

```bash
git rev-parse HEAD
```

A future archived experiment or publication should cite a tagged release or immutable commit SHA rather than only `main`.

## 2. Confirm the research host/guest baseline

The Phase 1A qualification was performed from an Ubuntu 24.04.4 LTS research guest. Record the reproduction host/guest rather than assuming it is identical:

```bash
cat /etc/os-release
uname -a
```

The cryptographic qualification image itself uses an Ubuntu 22.04 container base. This distinction is intentional and documented in `environment/stack.md`.

## 3. Build the pinned OQS qualification image

From the repository root:

```bash
docker build -t qrp-oqs:phase1a -f environment/Dockerfile .
```

The Dockerfile pins:

- oqs-provider 0.11.0
- OpenSSL 3.4.7
- liboqs 0.15.0
- `OQS_LIBJADE_BUILD=OFF`

Start an interactive qualification container:

```bash
docker run --rm -it --name qrp-oqs-phase1a qrp-oqs:phase1a
```

## 4. Verify the cryptographic instrumentation

The qualified OpenSSL binary is:

```text
/opt/oqs-provider/.local/bin/openssl
```

and the provider module path used by the validated runtime is:

```text
OPENSSL_MODULES=/opt/oqs-provider/_build/lib
```

Verify the stack:

```bash
OPENSSL_MODULES=/opt/oqs-provider/_build/lib \
/opt/oqs-provider/.local/bin/openssl version -a

OPENSSL_MODULES=/opt/oqs-provider/_build/lib \
/opt/oqs-provider/.local/bin/openssl list \
  -providers -provider default -provider oqsprovider

OPENSSL_MODULES=/opt/oqs-provider/_build/lib \
/opt/oqs-provider/.local/bin/openssl list \
  -kem-algorithms -provider default -provider oqsprovider
```

Acceptance requires the `default` and `oqsprovider` providers to be active and the intended `mlkem768` and `X25519MLKEM768` capabilities to be exposed.

## 5. Phase 1B verified acceptance boundary

Phase 1B is complete. Its verified public summary records two independent successful TLS 1.3 runs using:

- hybrid key establishment: `X25519MLKEM768`;
- observed group identifier: `0x11ec`;
- ML-DSA-65 authentication/certificate evidence;
- 23 packets in each accepted capture; and
- the same observed handshake sequence across both captures.

The second accepted run followed clean destruction and rebuilding of the qualified environment and used freshly generated test artifacts.

See:

- `docs/phase-1b-tls-validation.md`
- `artifacts/sanitized/phase1b_tls_reproducibility_summary.md`
- `artifacts/sanitized/phase1b_mldsa65_certificate_summary.txt`

## 6. Phase 1C canonical comparison

Phase 1C is complete and is documented in `docs/phase-1c-cross-regime-validation.md`.

The canonical same-stack comparison uses:

| Condition | Group | Authentication | Accepted runs |
|---|---|---|---:|
| C1-OQS | `X25519` | ECDSA P-256 | 10 |
| C2 | `X25519MLKEM768` | ECDSA P-256 | 30 |
| C3 | `mlkem768` | ECDSA P-256 | 30 |

All three conditions use:

- OpenSSL 3.4.7 / oqs-provider 0.11.0;
- TLS 1.3;
- `TLS_AES_256_GCM_SHA384`;
- the same ECDSA P-256 certificate/authentication configuration;
- the same 121-byte fixed HTTP payload;
- the same capture point, filter and run-record schema.

The fixed payload SHA-256 is:

`04b754f4158a69fbc0af6f64d0d4abb5007a24b9e4d2fc7860843ae9a0c4e986`

C3 is PQC-oriented rather than fully PQC because authentication remains classical ECDSA P-256.

## 7. Phase 1C capture safeguards

### Immediate capture delivery

An early automated trial showed that successful TLS sessions could still produce header-only PCAP files if tcpdump was stopped before buffered packets were flushed. Those runs were rejected.

The accepted collector therefore used immediate capture delivery and packet-buffered writes, followed by a drain period before stopping tcpdump. It also rejected:

- a PCAP at or below the global-header size;
- zero packets reported as captured;
- zero packets found during post-capture analysis;
- nonzero kernel drops; and
- unexpectedly oversized TCP payloads.

### Offload normalization

An early diagnostic capture contained a TCP payload larger than 4 KB despite a 1500-byte MTU. This was identified as a segmentation/offload artifact. TSO, GSO and GRO were normalized across the relevant virtual interfaces before official collection.

Accepted C2/C3 captures showed maximum TCP payloads of 1448 bytes, consistent with normal segmentation on the test path.

Offload state is runtime-sensitive and must be rechecked after container/network recreation or host reboot.

## 8. Phase 1C aggregate acceptance values

The public aggregate measurements are preserved in `results/phase1c-summary.csv` and `artifacts/sanitized/phase1c_cross_regime_summary.md`.

Key values are:

- C1-OQS mean total TCP payload: 1543.9 B; 17 packets.
- C2 mean total TCP payload: 3807.9 B; 21 packets.
- C3 mean total TCP payload: 3744.0 B; 21 packets.
- C1-OQS mean flow duration: 1.762 ms.
- C2 mean flow duration: 3.725 ms.
- C3 mean flow duration: 2.535 ms.

Timing is interpreted as exploratory because the conditions were collected in sequential batches rather than randomized/interleaved order. Packet and byte measurements are the stronger Phase 1C evidence.

## 9. Artifact integrity

Raw PCAPs, private keys and credentials are not committed to the public repository. For every private raw artifact used to support a new public result, record a SHA-256 hash:

```bash
sha256sum <artifact>
```

Only sanitized summaries belong under `artifacts/sanitized/`.

The Phase 1C classical dataset was integrity-frozen in the private experiment workspace by hashing the summary and accepted PCAPs after collection. Equivalent provenance should be preserved for future phases.

## 10. Results discipline

`results/results.csv` is the canonical public status/results table. A row may be marked `verified` only when:

1. its supporting evidence exists;
2. its provenance is recorded;
3. the extraction, measurement or acceptance procedure is documented; and
4. the result can be reproduced from the pinned environment and recorded inputs.

Phase 1A, Phase 1B and Phase 1C acceptance rows are verified. Missing later results are never back-filled from illustrative thesis figures or simulations.

## 11. Future ML reproducibility

Later ML phases will add pinned Python dependencies, deterministic seeds where feasible, grouped split manifests, feature-schema versions, model configurations and evaluation scripts. Those artifacts should be tied to a release/commit before any CV, SOP or paper wording describes the corresponding numerical performance as reproduced.
