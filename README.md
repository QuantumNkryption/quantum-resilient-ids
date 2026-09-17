# Quantum-Resilient Intrusion Detection Research Platform

A reproducible research platform for studying how post-quantum cryptography (PQC) changes observable network traffic and how those changes affect intrusion-detection systems.

This repository extends my MSc research project titled *AI-Powered Intrusion Detection in Quantum-Resistant Cryptographic Systems* into a longitudinal experimental program. The central research problem is not whether PQC is cryptographically secure; it is whether the transition from classical to hybrid and post-quantum TLS creates **cryptographic distribution shift** in network metadata that can affect IDS performance or model generalization.

## Research status

| Phase | Scope | Status | Public evidence |
|---|---|---:|---|
| Phase 0 | Research design freeze: workloads, attack taxonomy, feature schema, evaluation metrics, grouped splits, provenance rules | Complete | `docs/phase-0-research-design.md` |
| Phase 1A | Reproducible OQS/OpenSSL qualification environment | Complete | `docs/phase-1a-oqs-qualification.md` |
| Phase 1B | PQC/hybrid TLS generation, packet capture and clean rebuild reproducibility | Complete | `docs/phase-1b-tls-validation.md`; `artifacts/sanitized/phase1b_tls_reproducibility_summary.md` |
| Phase 1C | Controlled classical vs hybrid vs PQC-oriented TLS validation matrix | Complete | `docs/phase-1c-cross-regime-validation.md`; `artifacts/sanitized/phase1c_cross_regime_summary.md` |
| Phase 2A | Benign cryptographic distribution shift and Isolation Forest generalization | **Complete** | `docs/phase-2a-cryptographic-generalization-gap.md`; `artifacts/sanitized/phase2a_cryptographic_generalization_summary.md`; `results/phase2a-summary.csv` |
| Phase 2B | Controlled attack traffic and supervised IDS evaluation across cryptographic regimes | Next | Design pending |

The repository deliberately distinguishes **verified evidence** from planned or thesis-era claims. Results are promoted to `verified` only when supporting evidence and provenance are preserved.

## Current qualified cryptographic stack

The qualified containerized stack comprises:

- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0
- OQS provider separated from the host OpenSSL installation
- ML-KEM, hybrid `X25519MLKEM768`, and ML-DSA capabilities exposed through the provider

Phase 1B verified reproducible TLS 1.3 generation using `X25519MLKEM768` and ML-DSA-65 authentication, including a clean destroy/rebuild reproduction.

Phase 1C standardized the comparison around one OQS/OpenSSL runtime and a fixed benign workload. The canonical same-stack comparison uses:

- **C1-OQS classical control:** `X25519`, ECDSA P-256 authentication
- **C2 hybrid:** `X25519MLKEM768`, ECDSA P-256 authentication
- **C3 PQC-oriented:** `mlkem768`, ECDSA P-256 authentication
- TLS 1.3 and `TLS_AES_256_GCM_SHA384` in all three conditions

C3 is described as **PQC-oriented**, not fully PQC TLS, because key establishment is ML-KEM-768 while authentication remains classical ECDSA P-256.

## Phase 1C structural finding

The same-stack Phase 1C comparison established that cryptographic regime changes observable network structure even when the application workload is fixed:

| Metric | C1-OQS X25519 | C2 X25519MLKEM768 | C3 mlkem768 |
|---|---:|---:|---:|
| Accepted runs | 10 | 30 | 30 |
| Mean packet count | 17 | 21 | 21 |
| Mean client→server TCP payload | 534 B | 1710 B | 1678 B |
| Mean server→client TCP payload | 1009.9 B | 2097.9 B | 2066.0 B |
| Mean total TCP payload | 1543.9 B | 3807.9 B | 3744.0 B |
| Mean flow duration | 1.762 ms | 3.725 ms | 2.535 ms |

Relative to the same-stack X25519 control, mean total TCP payload increased by approximately **146.6%** for C2 and **142.5%** for C3.

## Phase 2A: Cryptographic Generalization Gap

Phase 2A tested whether those benign structural changes are large enough to destabilize an ML anomaly detector trained only on classical TLS.

The final dataset contains:

- **3,000 accepted benign flows**
- 1,000 flows per cryptographic regime
- six workload classes (W01-W06)
- 880 production capture units
- 25 frozen model features
- zero missing feature values

The 1,000 C1-OQS flows were partitioned into 600 training, 200 calibration and 200 held-out test observations. The Isolation Forest was fitted only on the 600 classical training flows. The anomaly threshold was set from the 200 classical calibration flows at the 95th percentile of the calibration anomaly-score distribution.

The primary matched held-out test used 200 C1-OQS flows and the corresponding 200 C2 and 200 C3 flows.

### Primary result

| Regime | False positives | FPR | 95% Wilson CI | CGG vs C1 |
|---|---:|---:|---:|---:|
| C1-OQS | 2 / 200 | **1.0%** | 0.27–3.57% | baseline |
| C2 hybrid | 169 / 200 | **84.5%** | 78.84–88.86% | **+83.5 pp** |
| C3 PQC-oriented | 119 / 200 | **59.5%** | 52.58–66.06% | **+58.5 pp** |

Matched exact McNemar tests strongly supported the binary shifts: C2 vs C1 produced 167 discordant C2-anomalous/C1-normal pairs and zero reverse pairs; C3 vs C1 produced 117 and zero, respectively. Continuous anomaly-score tests reached the same conclusion.

The result is called the **Cryptographic Generalization Gap (CGG)**: the increase in benign false-positive behavior when a detector trained exclusively on one cryptographic regime encounters legitimate traffic produced under an unseen cryptographic regime.

### Workload dependence

The effect was not uniform. C2 workload-specific false-positive rates ranged from 20% to 100%; C3 ranged from 0% to 100%. This means the finding should not be interpreted as PQC-oriented traffic being intrinsically anomalous. The observed shift depends on the interaction between cryptographic regime and application workload.

### Mechanism finding

The most obvious cryptographic features changed dramatically: C1 `clienthello_len` was constant at 331 bytes and `serverhello_len` at 118 bytes, while C2/C3 handshake lengths were much larger. Yet both handshake-length variables had zero variance in classical training and therefore appeared in zero Isolation Forest tree splits.

The model instead used timing and flow-morphology dimensions heavily, including `hello_rtt_ms`, inter-arrival statistics and flow duration. A matched counterfactual sensitivity analysis found that replacing only `hello_rtt_ms` with its classical matched value resolved 42 C2 and 65 C3 false positives.

This supports a more nuanced interpretation: the detector was responding primarily to **network-level consequences of cryptographic migration** rather than directly to the zero-variance handshake-size fields.

## Engineering controls discovered during Phase 2A

Several measurement and infrastructure issues were identified and corrected before final acceptance:

- the research VM storage was expanded without changing frozen experiment artifacts;
- stopped Docker containers were restarted and the qualified runtime revalidated after reboot;
- a 7,240-byte aggregated TCP payload exposed runtime GRO/GSO/TSO behavior, leading to explicit offload standardization and a clean production restart;
- TShark automatically misclassified one valid TLS stream, so final feature extraction explicitly decoded experimental ports as TLS; and
- a host SciPy/NumPy mismatch led to a dedicated pinned Python virtual environment for ML analysis.

These failures were retained as methodological evidence rather than silently discarded.

## Scientific framing

The platform now supports two empirical findings:

1. **Phase 1C:** changing the key-establishment regime materially changes observable encrypted-flow metadata under a fixed workload.
2. **Phase 2A:** those benign changes can be large enough to produce severe false-positive generalization failure in a detector trained only on classical traffic.

The next research question is whether this benign distribution shift also affects attack-detection performance when controlled malicious traffic is introduced across classical, hybrid and PQC-oriented regimes.

## Evaluation plan

Phase 2B and later supervised IDS evaluation will retain grouped/leakage-safe splitting and will report metrics such as:

- Macro-F1
- precision and recall by class
- calibration / reliability where applicable
- confusion matrices
- cross-regime train/test comparisons
- Cryptographic Generalization Gap
- uncertainty intervals and matched/grouped statistical tests where appropriate

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
│   ├── phase-1c-cross-regime-validation.md
│   └── phase-2a-cryptographic-generalization-gap.md
├── environment/
│   └── stack.md
├── artifacts/
│   └── sanitized/
│       ├── README.md
│       ├── phase1b_mldsa65_certificate_summary.txt
│       ├── phase1b_tls_reproducibility_summary.md
│       ├── phase1c_cross_regime_summary.md
│       └── phase2a_cryptographic_generalization_summary.md
├── results/
│   ├── results.csv
│   ├── phase1c-summary.csv
│   └── phase2a-summary.csv
└── Quantum-Resilient IDS Simulation.txt   # legacy MSc thesis-era implementation
```

The legacy thesis script is retained for provenance. It should not be interpreted as the final research-platform pipeline; the empirical platform is being rebuilt around stricter reproducibility, leakage control and evidence preservation.

## Reproducing the current state

See [`docs/reproducibility.md`](docs/reproducibility.md). The verified reproducibility boundary now includes Phase 1A qualification, Phase 1B clean-rebuild TLS validation, the completed Phase 1C three-regime comparison and the completed Phase 2A benign generalization experiment.

## Results

The canonical status/results table is [`results/results.csv`](results/results.csv). Aggregate Phase 1C measurements are in [`results/phase1c-summary.csv`](results/phase1c-summary.csv), and Phase 2A aggregate ML/statistical results are in [`results/phase2a-summary.csv`](results/phase2a-summary.csv).

## Data and artifact policy

Raw packet captures, private keys, credentials, host-specific secrets and large third-party datasets are excluded from Git. Only sanitized evidence suitable for public release is stored under `artifacts/sanitized/`.

## Relationship to the MSc thesis

The MSc thesis established the motivating direction: evaluate IDS performance across classical and post-quantum cryptographic conditions and investigate metadata-based machine-learning detection. This repository turns that direction into a reproducible experimental program with explicit acceptance gates, leakage controls, preserved evidence and cross-regime statistical evaluation.

## Author

Charles Emuze  
MSc Enterprise Cybersecurity

## License

MIT. See `LICENSE`.
