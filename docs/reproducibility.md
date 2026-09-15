# Reproducibility Instructions

## Scope

These instructions reproduce the research platform through the current Phase 1A / 1B boundary. They do **not** claim that later IDS/ML experiments have been reproduced yet.

## 1. Clone the repository

```bash
git clone https://github.com/QuantumNkryption/quantum-resilient-ids.git
cd quantum-resilient-ids
git checkout research-platform-v1
```

After this branch is merged, use the corresponding tagged release or commit SHA instead of the development branch.

Record the commit used:

```bash
git rev-parse HEAD
```

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

## 5. Phase 1B acceptance run

Phase 1B must be treated as an evidence-generating validation exercise. For every TLS regime tested, create a run record containing:

- repository commit SHA;
- UTC timestamp;
- exact client/server command or script version;
- TLS regime and algorithms requested;
- TLS version and cipher negotiated;
- negotiated group / key-establishment evidence when available;
- certificate/signature algorithm evidence;
- packet-capture identifier and SHA-256 in the private evidence store;
- sanitized `tshark`/Wireshark/Zeek-derived summary;
- pass/fail outcome.

The current public repository contains a sanitized ML-DSA-65 certificate summary, but Phase 1B remains incomplete until the required classical/hybrid/PQC handshake validation, capture inspection and clean rebuild are all evidenced.

## 6. Destroy and rebuild

After the first acceptance suite, remove the qualification container/image and rebuild from the repository definition:

```bash
docker rm -f qrp-oqs-phase1a 2>/dev/null || true
docker image rm qrp-oqs:phase1a
docker build --no-cache -t qrp-oqs:phase1a -f environment/Dockerfile .
```

Repeat the Phase 1A checks and Phase 1B acceptance suite. A reproducibility claim requires the relevant acceptance outcomes to survive this clean rebuild.

## 7. Artifact integrity

Raw PCAPs, private keys and credentials are not committed to the public repository. For every private raw artifact used to support a public result, record a SHA-256 hash:

```bash
sha256sum <artifact>
```

Only sanitized summaries belong under `artifacts/sanitized/`.

## 8. Results discipline

`results/results.csv` is the canonical public status/results table. A row may be marked `verified` only when:

1. its evidence exists;
2. its provenance is recorded;
3. the extraction or measurement procedure is documented; and
4. the result can be reproduced from the pinned environment and recorded inputs.

Missing or incomplete results stay explicitly `pending`; they are never back-filled from illustrative thesis figures or simulations.

## 9. Future ML reproducibility

Later ML phases will add pinned Python dependencies, deterministic seeds where feasible, grouped split manifests, feature-schema versions, model configurations and evaluation scripts. Those artifacts should be tied to a release/commit before any CV, SOP or paper wording describes the corresponding numerical performance as reproduced.
