# Reproducibility Instructions

## Scope

These instructions reproduce the research platform through the verified Phase 1A / Phase 1B boundary and define the reproducibility requirements for the pending Phase 1C cross-regime comparison. They do **not** claim that later IDS/ML experiments have been reproduced yet.

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

Inside the container:

```bash
openssl version
openssl list -providers -provider default -provider oqsprovider
openssl list -kem-algorithms -provider default -provider oqsprovider
openssl list -signature-algorithms -provider default -provider oqsprovider
```

The qualified binary should resolve to the OQS build under `/opt/oqs-provider/.local/bin/openssl`, with provider modules under `/opt/oqs-provider/_build/lib`.

Acceptance requires the OQS provider to load successfully and the intended ML-KEM, hybrid `X25519MLKEM768`, and ML-DSA capabilities to be exposed.

## 5. Phase 1B verified acceptance boundary

Phase 1B is complete. Its acceptance target is the reproducible PQC-capable TLS path rather than a three-regime comparison matrix.

The verified public acceptance summary records two independent successful TLS 1.3 runs using:

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

Raw packet captures and private keys are excluded from the public repository.

## 6. Reproducing the Phase 1B rebuild condition

To reproduce the clean-environment condition, remove the qualification container/image and rebuild from the repository definition:

```bash
docker rm -f qrp-oqs-phase1a 2>/dev/null || true
docker image rm qrp-oqs:phase1a
docker build --no-cache -t qrp-oqs:phase1a -f environment/Dockerfile .
```

Then repeat the Phase 1A capability checks and the Phase 1B target TLS acceptance procedure described in `docs/phase-1b-tls-validation.md`.

A new reproduction attempt should preserve its own run ID, timestamp, exact commands or script version, capture identifier, private capture hash and sanitized evidence rather than overwriting the historical acceptance record.

## 7. Phase 1C cross-regime reproducibility

Phase 1C is pending and is defined in `docs/phase-1c-cross-regime-validation.md`.

For each of the classical, hybrid and PQC-oriented regimes, record:

- repository commit SHA;
- UTC timestamp;
- exact client/server command or script version;
- regime label and exact algorithms requested;
- TLS version and cipher negotiated;
- negotiated group / key-establishment evidence where observable;
- certificate/signature algorithm evidence;
- packet-capture identifier and SHA-256 in the private evidence store;
- sanitized capture-derived summary;
- pass/fail outcome; and
- deviations or exclusion reason if applicable.

The three regimes must use the same standardized workload and capture methodology before any cross-regime numerical comparison is promoted as verified.

Phase 1B does not need to be reopened to execute Phase 1C; the Phase 1C hybrid run is a fresh comparison run performed for direct comparability with its classical and PQC counterparts.

## 8. Artifact integrity

Raw PCAPs, private keys and credentials are not committed to the public repository. For every private raw artifact used to support a new public result, record a SHA-256 hash:

```bash
sha256sum <artifact>
```

Only sanitized summaries belong under `artifacts/sanitized/`.

## 9. Results discipline

`results/results.csv` is the canonical public status/results table. A row may be marked `verified` only when:

1. its supporting evidence exists;
2. its provenance is recorded;
3. the extraction, measurement or acceptance procedure is documented; and
4. the result can be reproduced from the pinned environment and recorded inputs.

Phase 1B rows are verified on the basis of the completed acceptance exercise. Phase 1C rows remain explicitly `pending` until new cross-regime evidence is generated. Missing results are never back-filled from illustrative thesis figures or simulations.

## 10. Future ML reproducibility

Later ML phases will add pinned Python dependencies, deterministic seeds where feasible, grouped split manifests, feature-schema versions, model configurations and evaluation scripts. Those artifacts should be tied to a release/commit before any CV, SOP or paper wording describes the corresponding numerical performance as reproduced.
