# Phase 2A Sanitized Evidence Summary

## Scope

This public artifact summarizes the verified Phase 2A benign-distribution-shift experiment. Raw PCAPs, host-specific evidence and private experimental artifacts remain outside the public repository.

## Research question

Does migration from classical TLS to hybrid or post-quantum key establishment cause legitimate benign network flows to appear anomalous to a machine-learning detector trained only on classical benign traffic?

## Treatments

All three conditions used the same qualified OpenSSL/OQS runtime and TLS 1.3 configuration. Only the key-establishment regime changed:

- C1-OQS: `X25519`
- C2: `X25519MLKEM768`
- C3: `mlkem768`

C3 remains PQC-oriented rather than fully post-quantum TLS because authentication remains ECDSA P-256.

## Dataset

- 3,000 accepted benign flow observations
- 1,000 flows per cryptographic regime
- six workload classes (W01-W06)
- 880 production capture units
- matched `pair_id` structure across C1-OQS/C2/C3
- randomized and balanced regime ordering
- 25 frozen model features
- zero missing feature values in the final feature matrix

The 1,000 C1-OQS flows were split into 600 training, 200 calibration and 200 held-out test observations. Concurrent W06 flows were split by whole batch to prevent leakage.

## Baseline model

Isolation Forest configuration:

- 500 estimators
- `max_samples=256`
- 25 model features
- `random_state=20260917`
- threshold fixed at the 95th percentile of the 200-flow C1 calibration anomaly-score distribution
- observed calibration FPR: 5%

No C2 or C3 observation was used for model fitting or threshold calibration.

## Primary matched results

| Regime | n | False positives | FPR | 95% Wilson CI |
|---|---:|---:|---:|---:|
| C1-OQS | 200 | 2 | 1.0% | 0.27–3.57% |
| C2 | 200 | 169 | 84.5% | 78.84–88.86% |
| C3 | 200 | 119 | 59.5% | 52.58–66.06% |

Cryptographic Generalization Gap relative to C1-OQS:

- C2: **+83.5 percentage points**
- C3: **+58.5 percentage points**

## Paired statistical evidence

### C2 vs C1-OQS

- C2-anomalous / C1-normal pairs: 167
- C2-normal / C1-anomalous pairs: 0
- exact McNemar p ≈ `1.07e-50`
- matched FPR difference: +83.5 pp
- bootstrap 95% interval: approximately +78.0 to +88.5 pp
- median matched anomaly-score increase: approximately +0.1198
- matched rank-biserial effect: approximately 0.9997

### C3 vs C1-OQS

- C3-anomalous / C1-normal pairs: 117
- C3-normal / C1-anomalous pairs: 0
- exact McNemar p ≈ `1.20e-35`
- matched FPR difference: +58.5 pp
- bootstrap 95% interval: approximately +51.5 to +65.5 pp
- median matched anomaly-score increase: approximately +0.09225
- matched rank-biserial effect: approximately 0.9934

The C2-vs-C3 comparison is secondary/exploratory; C2 produced a 25-point higher FPR on the matched test set.

## Workload-specific FPR

| Workload | C1-OQS | C2 | C3 |
|---|---:|---:|---:|
| W01 | 2.5% | 100% | 85% |
| W02 | 0% | 100% | 90% |
| W03 | 0% | 100% | 100% |
| W04 | 0% | 20% | 0% |
| W05 | 3.3% | 76.7% | 43.3% |
| W06 | 0% | 100% | 20% |

## Mechanism summary

The cryptographic migration created obvious TLS handshake-size changes:

- C1 `clienthello_len`: 331 bytes with zero training variance
- C1 `serverhello_len`: 118 bytes with zero training variance
- C2 median ClientHello shift: +1,176 bytes
- C3 median ClientHello shift: +1,144 bytes
- C2 median ServerHello shift: +1,088 bytes
- C3 median ServerHello shift: +1,056 bytes

Every C2 and C3 primary test flow fell outside the C1 training range for both handshake-length features. However, the two handshake-size features had zero Isolation Forest tree splits because they had zero variance in the classical training set.

The frozen forest used timing and flow-morphology variables more heavily, particularly `hello_rtt_ms`, `iat_ms_mean`, `iat_ms_p95`, `iat_ms_std` and `flow_duration_ms`.

Matched single-feature counterfactual sensitivity analysis found that replacing `hello_rtt_ms` with the matched C1 value resolved:

- 42 C2 false positives
- 65 C3 false positives

For C3, replacing `iat_ms_p95` resolved 45 false positives.

This analysis is diagnostic rather than causal because network-flow features are correlated.

## Production-quality controls

Final production was accepted only after:

- standardized GRO/GSO/TSO behavior across the capture path;
- rejection and archival of pre-standardization observations;
- explicit TLS decode-as on experimental ports after an automatic TShark misclassification was identified;
- per-flow capture checks and SHA-256 provenance;
- frozen workload, production-schedule and feature-extraction definitions; and
- isolated, pinned Python ML dependencies.

## Supported claim

Within the controlled Phase 2A experiment, legitimate hybrid and PQC-oriented TLS traffic produced a substantial benign distribution shift relative to classical training traffic, causing markedly elevated false-positive rates in a classical-trained Isolation Forest. This result is designated the **Cryptographic Generalization Gap** for the evaluated detector, feature space, workloads and cryptographic regimes.

See `docs/phase-2a-cryptographic-generalization-gap.md` for the full narrative and `results/phase2a-summary.csv` for the public aggregate table.
