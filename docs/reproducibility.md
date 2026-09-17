# Reproducibility Instructions

## Scope

These instructions describe the verified research platform through **Phase 2A**. They cover OQS/OpenSSL qualification, cross-regime TLS validation, benign dataset construction, feature extraction and the frozen Isolation Forest generalization experiment. Phase 2B attack-detection experiments are not yet part of the verified boundary.

## 1. Clone and record the repository state

```bash
git clone https://github.com/QuantumNkryption/quantum-resilient-ids.git
cd quantum-resilient-ids
git checkout main
git rev-parse HEAD
```

A publication or archived experiment should cite an immutable commit or tagged release rather than only `main`.

## 2. Qualified cryptographic runtime

The empirical cryptographic experiments use the qualified OQS/OpenSSL container runtime rather than the host OpenSSL installation.

Qualified versions:

- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0

The validated OpenSSL executable is:

```text
/opt/oqs-provider/.local/bin/openssl
```

with provider modules loaded from:

```text
OPENSSL_MODULES=/opt/oqs-provider/_build/lib
```

Verify that `default` and `oqsprovider` are active and that `mlkem768` and `X25519MLKEM768` are exposed before reproducing any cryptographic comparison.

## 3. Canonical cryptographic regimes

Phase 1C and Phase 2A use the same treatment definitions:

| Regime | Key establishment | Authentication |
|---|---|---|
| C1-OQS | `X25519` | ECDSA P-256 |
| C2 | `X25519MLKEM768` | ECDSA P-256 |
| C3 | `mlkem768` | ECDSA P-256 |

All use TLS 1.3 and `TLS_AES_256_GCM_SHA384`.

C3 is PQC-oriented rather than fully PQC because authentication remains classical ECDSA P-256.

## 4. Phase 1B verified boundary

Phase 1B verified reproducible TLS 1.3 generation and capture using `X25519MLKEM768` with ML-DSA-65 authentication, including a clean destroy/rebuild reproduction.

See:

- `docs/phase-1b-tls-validation.md`
- `artifacts/sanitized/phase1b_tls_reproducibility_summary.md`
- `artifacts/sanitized/phase1b_mldsa65_certificate_summary.txt`

## 5. Phase 1C verified boundary

Phase 1C established the same-stack classical/hybrid/PQC-oriented comparison under one fixed 121-byte benign workload.

See:

- `docs/phase-1c-cross-regime-validation.md`
- `artifacts/sanitized/phase1c_cross_regime_summary.md`
- `results/phase1c-summary.csv`

## 6. Phase 2A benign dataset

The final Phase 2A dataset contains **3,000 accepted benign flows**:

- 1,000 C1-OQS flows;
- 1,000 C2 flows; and
- 1,000 C3 flows.

Six workload classes are represented:

- W01: 121-byte anchor response, 200 flows per regime;
- W02: 1 KiB small GET, 200;
- W03: 8 KiB JSON/API-style response, 150;
- W04: 64 KiB download, 150;
- W05: 1 MiB download, 150;
- W06: 30 batches × five simultaneous 64 KiB connections, 150.

The production schedule contains 880 capture units. Each logical workload instance uses a matched `pair_id` across the three regimes, and regime order is randomized/balanced.

Raw PCAPs are retained outside the public repository. Public aggregate evidence is in `artifacts/sanitized/phase2a_cryptographic_generalization_summary.md`.

## 7. Capture-path controls required for reproduction

### GRO/GSO/TSO standardization

A post-reboot diagnostic capture showed a 7,240-byte aggregated TCP payload while the accepted path normally produced a 1,448-byte maximum. Because packetization and timing are model features, final Phase 2A production standardized GRO/GSO/TSO behavior across the Docker bridge, relevant veth interfaces and container interfaces.

Pre-standardization production observations were archived and excluded from the final analytical dataset.

### Explicit TLS decode-as

Automatic TShark protocol detection misclassified one valid TLS stream during production. Final feature extraction therefore explicitly decodes the experimental TCP ports as TLS rather than depending on automatic protocol heuristics.

These controls are part of the accepted Phase 2A measurement definition and should be reproduced before collecting a comparable dataset.

## 8. Feature Schema v1

The frozen Phase 2A representation contains 25 model features covering flow duration, packet counts, directional TCP payload bytes, payload-length statistics, direction ratios, inter-arrival-time statistics, TLS-record statistics, ClientHello/ServerHello lengths and handshake timing.

The final feature matrix contains:

- 3,000 rows;
- 25 model features; and
- zero missing values.

Cryptographic regime, TLS group, workload ID, server port, retransmission count and provenance fields are metadata only and are not model inputs.

## 9. Leakage-safe C1 split

The 1,000 C1-OQS observations are split into:

- 600 training flows;
- 200 calibration flows; and
- 200 held-out test flows.

The split is workload-stratified. W06 is split by whole concurrent batch so that connections from the same batch do not cross train/calibration/test boundaries.

The 200 held-out C1 pair IDs identify the corresponding 200 C2 and 200 C3 observations used for the primary matched evaluation.

## 10. ML runtime

The final Phase 2A modelling environment is isolated from the system Python environment and uses:

- Python 3.10.12
- NumPy 1.26.4
- SciPy 1.13.1
- pandas 2.3.3
- scikit-learn 1.7.2
- joblib 1.5.3

A reproduction should pin compatible versions rather than rely on whatever packages happen to be installed globally.

## 11. Frozen Isolation Forest baseline

The verified baseline configuration is:

```text
n_estimators = 500
max_samples = 256
contamination = auto
max_features = 1.0
bootstrap = false
random_state = 20260917
```

The model is fitted **only** on the 600 C1-OQS training observations.

The anomaly threshold is the 95th percentile of anomaly scores from the separate 200-flow C1-OQS calibration set. The observed calibration FPR is 5%.

No C2/C3 observation is used for fitting or threshold selection.

## 12. Verified primary Phase 2A result

The primary matched held-out set contains 200 observations per regime.

| Regime | False positives | FPR | 95% Wilson CI |
|---|---:|---:|---:|
| C1-OQS | 2/200 | 1.0% | 0.27–3.57% |
| C2 | 169/200 | 84.5% | 78.84–88.86% |
| C3 | 119/200 | 59.5% | 52.58–66.06% |

Cryptographic Generalization Gap relative to C1-OQS:

- C2: +83.5 percentage points;
- C3: +58.5 percentage points.

Aggregate public results are in `results/phase2a-summary.csv`.

## 13. Statistical reproduction

The primary inferential analysis uses:

- Wilson intervals for regime-level FPR;
- exact McNemar tests on matched anomaly flags;
- bootstrap intervals for matched FPR differences;
- Wilcoxon signed-rank tests on matched anomaly scores;
- matched rank-biserial effect sizes; and
- Holm adjustment for the two primary C2-vs-C1 and C3-vs-C1 comparisons.

The C2-vs-C3 comparison is secondary/exploratory.

## 14. Mechanism analysis

The mechanism analysis intentionally separates distribution shift from direct feature use by the model.

In C1 training, `clienthello_len` and `serverhello_len` are constant and therefore receive zero Isolation Forest tree splits, even though their values shift substantially under C2/C3.

The frozen forest uses timing and flow-morphology variables much more frequently. Counterfactual one-feature replacement is used only as diagnostic sensitivity analysis because the features are correlated.

## 15. Artifact integrity

Raw captures, private keys, credentials and host-specific evidence are excluded from Git. The private research workspace retains SHA-256 manifests for the final production dataset, feature matrix, model, statistical analysis, feature-driver analysis, mechanism analysis, figures and Phase 2A closure artifacts.

Public results are sanitized derivatives that map to that private provenance trail.

## 16. Result discipline

`results/results.csv` is the canonical public status/results table. A row is marked `verified` only when supporting evidence exists and the procedure can be traced to recorded inputs, methods and provenance.

Phase 1A, Phase 1B, Phase 1C and Phase 2A rows are now verified. Phase 2B remains outside the verified boundary until its experimental design and evidence are completed.
