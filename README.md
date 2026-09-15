# Quantum-Resilient Intrusion Detection Research Platform

A reproducible research platform for studying how post-quantum cryptography (PQC) changes observable network traffic and how those changes affect intrusion-detection systems.

This repository extends the MSc research project **AI-Powered Intrusion Detection in Quantum-Resistant Cryptographic Systems** into a longitudinal experimental program. The immediate research question is not whether PQC is cryptographically secure; it is whether the transition from classical to hybrid and post-quantum TLS creates **cryptographic distribution shift** in network metadata that degrades IDS performance or model generalization.

## Research status

| Phase | Scope | Status | Public evidence |
|---|---|---:|---|
| Phase 0 | Research design freeze: workloads, attack taxonomy, feature schema, evaluation metrics, grouped splits, provenance rules | Complete | `docs/phase-0-research-design.md` |
| Phase 1A | Reproducible OQS/OpenSSL qualification environment | Complete | `docs/phase-1a-oqs-qualification.md` |
| Phase 1B | PQC/hybrid TLS generation, packet capture and clean rebuild reproducibility | Complete | `docs/phase-1b-tls-validation.md`; `artifacts/sanitized/phase1b_tls_reproducibility_summary.md` |
| Phase 1C | Controlled classical vs hybrid vs PQC TLS validation matrix | Planned / pending | `docs/phase-1c-cross-regime-validation.md` |

The repository deliberately distinguishes **verified evidence** from planned or thesis-era claims. Results are never promoted from `pending` to `verified` without preserved evidence and documented provenance.

## Current qualified cryptographic stack

Phase 1A qualified a containerized stack comprising:

- Ubuntu 24.04.4 LTS guest environment
- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0
- OQS provider active and separated from the host OpenSSL installation
- ML-KEM, hybrid `X25519MLKEM768`, and ML-DSA exposed through the provider

Phase 1B then verified that this environment can reproducibly generate and capture the target TLS 1.3 handshake using `X25519MLKEM768` and ML-DSA-65 authentication. Two accepted runs were completed, with the second following a clean destroy/rebuild of the qualified environment.

The next experimental gate is **Phase 1C**, which will place classical, hybrid and PQC-oriented TLS under one standardized comparison protocol before later IDS and machine-learning experiments begin.

## Scientific framing

The platform tests the hypothesis that changing the cryptographic regime can alter observable metadata even when application behavior is held constant. The key experimental regimes are:

1. **Classical TLS** — classical key establishment and authentication.
2. **Hybrid TLS** — classical + ML-KEM key establishment, including the qualified `X25519MLKEM768` path.
3. **PQC-oriented TLS** — ML-KEM / ML-DSA configurations where the qualified stack permits them.

The controlled workload set includes HTTPS GETs, repeated requests, API-style traffic, small and large downloads, and concurrent sessions. Initial attack-behavior classes are reconnaissance, brute-force behavior, and request flooding. These are laboratory-only workloads and are not instructions for attacking external systems.

## Evaluation plan

Primary evaluation uses grouped train/test splits to prevent leakage across closely related flows or runs. Core metrics include:

- Macro-F1
- Precision and recall by class
- Calibration / reliability
- Confusion matrices
- **Cryptographic Generalization Gap (CGG)** — the change in predictive performance when a detector is trained under one cryptographic regime and evaluated under another

Phase 0 also freezes provenance requirements so that each later result can be traced to a workload, cryptographic regime, environment version, capture, feature set, model configuration and run identifier.

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
│       └── phase1b_tls_reproducibility_summary.md
├── results/
│   └── results.csv
└── Quantum-Resilient IDS Simulation.txt   # legacy MSc thesis-era implementation
```

The legacy thesis script is retained for provenance. It should not be interpreted as the final research-platform pipeline; the new platform is being rebuilt around stricter reproducibility, leakage control and evidence preservation.

## Reproducing the current state

See [`docs/reproducibility.md`](docs/reproducibility.md). The current verified reproducibility boundary includes Phase 1A environment qualification and the completed Phase 1B hybrid/PQC-capable TLS acceptance result. Phase 1C remains pending and must be executed under its common cross-regime protocol before comparative TLS claims are made.

## Results

The canonical machine-readable results table is [`results/results.csv`](results/results.csv). Phase 1B acceptance rows are recorded as `verified`; Phase 1C comparison rows remain explicitly `pending` until their corresponding evidence exists.

## Data and artifact policy

Raw packet captures, private keys, credentials, host-specific secrets and large third-party datasets are excluded from Git. Only sanitized evidence suitable for public release is stored under `artifacts/sanitized/`.

## Relationship to the MSc thesis

The MSc thesis established the motivating research direction: evaluate IDS performance across classical and post-quantum cryptographic conditions and investigate metadata-based machine-learning detection. This repository turns that direction into a reproducible experimental program with explicit acceptance gates and preserved evidence.

## Author

Charles Emuze  
MSc Enterprise Cybersecurity

## License

MIT. See `LICENSE`.
