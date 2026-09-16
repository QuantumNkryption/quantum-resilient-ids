# Quantum-Resilient Intrusion Detection Research Platform

A reproducible research platform for studying how post-quantum cryptography (PQC) changes observable network traffic and how those changes affect intrusion-detection systems.

This repository extends my MSc research project titled *AI-Powered Intrusion Detection in Quantum-Resistant Cryptographic Systems* into a longitudinal experimental program. The immediate research question is not whether PQC is cryptographically secure; it is whether the transition from classical to hybrid and post-quantum TLS creates **cryptographic distribution shift** in network metadata that can affect IDS performance or model generalization.

## Research status

| Phase | Scope | Status | Public evidence |
|---|---|---:|---|
| Phase 0 | Research design freeze: workloads, attack taxonomy, feature schema, evaluation metrics, grouped splits, provenance rules | Complete | `docs/phase-0-research-design.md` |
| Phase 1A | Reproducible OQS/OpenSSL qualification environment | Complete | `docs/phase-1a-oqs-qualification.md` |
| Phase 1B | PQC/hybrid TLS generation, packet capture and clean rebuild reproducibility | Complete | `docs/phase-1b-tls-validation.md`; `artifacts/sanitized/phase1b_tls_reproducibility_summary.md` |
| Phase 1C | Controlled classical vs hybrid vs PQC-oriented TLS validation matrix | **Complete** | `docs/phase-1c-cross-regime-validation.md`; `artifacts/sanitized/phase1c_cross_regime_summary.md` |

The repository deliberately distinguishes **verified evidence** from planned or thesis-era claims. Results are promoted to `verified` only when supporting evidence and provenance are preserved.

## Current qualified cryptographic stack

Phase 1A qualified a containerized stack comprising:

- Ubuntu 24.04.4 LTS research guest
- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0
- OQS provider separated from the host OpenSSL installation
- ML-KEM, hybrid `X25519MLKEM768`, and ML-DSA capabilities exposed through the provider

Phase 1B verified reproducible TLS 1.3 generation using `X25519MLKEM768` and ML-DSA-65 authentication, including a clean destroy/rebuild reproduction.

Phase 1C then standardized the comparison around one OQS/OpenSSL runtime and a fixed benign workload. The canonical same-stack comparison uses:

- **C1-OQS classical control:** `X25519`, ECDSA P-256 authentication, n=10
- **C2 hybrid:** `X25519MLKEM768`, ECDSA P-256 authentication, n=30
- **C3 PQC-oriented:** `mlkem768`, ECDSA P-256 authentication, n=30
- TLS 1.3 and `TLS_AES_256_GCM_SHA384` in all three conditions
- one fixed 121-byte HTTP object across conditions

C3 is described as **PQC-oriented**, not fully PQC TLS, because key establishment is ML-KEM-768 while authentication remains classical ECDSA P-256.

## Phase 1C preliminary findings

The same-stack comparison produced highly repeatable flow-level differences:

| Metric | C1-OQS X25519 | C2 X25519MLKEM768 | C3 mlkem768 |
|---|---:|---:|---:|
| Accepted runs | 10 | 30 | 30 |
| Mean packet count | 17 | 21 | 21 |
| Mean client→server TCP payload | 534 B | 1710 B | 1678 B |
| Mean server→client TCP payload | 1009.9 B | 2097.9 B | 2066.0 B |
| Mean total TCP payload | 1543.9 B | 3807.9 B | 3744.0 B |
| Mean handshake read | 797.9 B | 1885.9 B | 1854.0 B |
| Handshake written | 420 B | 1596 B | 1564 B |
| Mean flow duration | 1.762 ms | 3.725 ms | 2.535 ms |

Relative to the same-stack X25519 control, mean total TCP payload increased by approximately **146.6%** for the hybrid condition and **142.5%** for the ML-KEM-768 condition. Packet count increased from 17 to 21 in both PQC-oriented conditions.

C2 and C3 were structurally much closer to one another than either was to the classical control: the hybrid condition carried only about **1.7%** more mean TCP payload than the ML-KEM-only condition.

Timing results are treated more cautiously than packet/byte results because the batches were collected sequentially rather than randomized or interleaved. The packet and byte differences are the stronger Phase 1C evidence.

## Scientific framing

The platform tests the hypothesis that changing the cryptographic regime can alter observable metadata even when application behavior is held constant. The Phase 1C result motivates the next question: whether IDS and machine-learning models trained primarily on classical TLS retain performance when legitimate traffic shifts toward hybrid or PQC-oriented TLS.

Candidate downstream features include packet count, directional byte volumes, packet-size statistics, flow duration, timing/inter-arrival statistics and TLS handshake metadata.

## Evaluation plan

Primary IDS/ML evaluation uses grouped train/test splits to prevent leakage across closely related flows or runs. Core metrics include:

- Macro-F1
- Precision and recall by class
- Calibration / reliability
- Confusion matrices
- **Cryptographic Generalization Gap (CGG)** — the change in predictive performance when a detector is trained under one cryptographic regime and evaluated under another

## Repository layout

```text
.
├── README.md
├── .gitignore
├── requirements.txt
├── docs/
│   ├── methodology.md
│   ├── reproducibility.md
│   ├── phase-0-research-design.md
│   ├── phase-1a-oqs-qualification.md
│   ├── phase-1b-tls-validation.md
│   └── phase-1c-cross-regime-validation.md
├── environment/
│   └── stack.md
├── artifacts/
│   └── sanitized/
│       ├── README.md
│       ├── phase1b_mldsa65_certificate_summary.txt
│       ├── phase1b_tls_reproducibility_summary.md
│       └── phase1c_cross_regime_summary.md
├── results/
│   ├── results.csv
│   └── phase1c-summary.csv
└── Quantum-Resilient IDS Simulation.txt   # legacy MSc thesis-era implementation
```

The legacy thesis script is retained for provenance. It should not be interpreted as the final research-platform pipeline; the new platform is being rebuilt around stricter reproducibility, leakage control and evidence preservation.

## Reproducing the current state

See [`docs/reproducibility.md`](docs/reproducibility.md). The current verified reproducibility boundary now includes Phase 1A qualification, Phase 1B clean-rebuild TLS validation, and the completed Phase 1C three-regime comparison.

## Results

The canonical status/results table is [`results/results.csv`](results/results.csv). Aggregated Phase 1C measurements are available in [`results/phase1c-summary.csv`](results/phase1c-summary.csv).

## Data and artifact policy

Raw packet captures, private keys, credentials, host-specific secrets and large third-party datasets are excluded from Git. Only sanitized evidence suitable for public release is stored under `artifacts/sanitized/`.

## Relationship to the MSc thesis

The MSc thesis established the motivating research direction: evaluate IDS performance across classical and post-quantum cryptographic conditions and investigate metadata-based machine-learning detection. This repository turns that direction into a reproducible experimental program with explicit acceptance gates and preserved evidence.

## Author

Charles Emuze  
MSc Enterprise Cybersecurity

## License

MIT. See `LICENSE`.
