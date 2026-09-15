# Experiment Methodology

## 1. Research design

This project uses controlled comparative experimentation. The independent variable is the **cryptographic regime**; application workload and other experimental conditions are held as constant as practical. The principal outcome is the change in observable network metadata and downstream IDS/model performance.

The methodology is deliberately staged:

- **Phase 0:** freeze scientific questions, workloads, labels, features, metrics and provenance.
- **Phase 1A:** qualify the cryptographic instrumentation.
- **Phase 1B:** verify reproducible generation and capture of the target PQC-capable TLS handshake, including clean rebuild reproduction.
- **Phase 1C:** establish the controlled classical / hybrid / PQC TLS comparison matrix.
- **Later phases:** generate controlled datasets, evaluate conventional IDS, train/evaluate ML models and perform cross-regime generalization analysis.

The separation between Phase 1B and Phase 1C is deliberate. Phase 1B is a platform-capability and reproducibility gate; Phase 1C is the comparative experimental-control gate.

## 2. Controlled variables

For comparisons across cryptographic regimes, preserve where applicable:

- workload definition;
- endpoint roles;
- payload/application object;
- connection/session count;
- request schedule;
- capture point;
- software image and configuration other than the cryptographic treatment;
- feature-extraction version;
- run duration;
- random seed for stochastic components.

Any unavoidable difference between regimes must be recorded in the run manifest.

## 3. Treatments

The primary treatment variable is cryptographic regime:

- `classical`
- `hybrid`
- `pqc`

Exact algorithm names are recorded as run metadata rather than inferred from the label alone.

Phase 1B verified the reproducible hybrid/PQC-capable path using `X25519MLKEM768` and ML-DSA-65 authentication. Phase 1C requires fresh directly comparable runs for all three regime labels under one common protocol.

## 4. Workloads and labels

### Benign workloads

B01 HTTPS GETs; B02 repeated requests; B03 API-style traffic; B04 small downloads; B05 large downloads; B06 concurrent sessions.

### Initial attack-behavior classes

A01 reconnaissance; A02 brute-force behavior; A03 request flooding.

These are executed only inside isolated laboratory infrastructure owned or controlled by the researcher.

## 5. Data collection

Each accepted comparative run produces, at minimum:

1. run manifest;
2. environment/tool version record;
3. packet capture retained in the private evidence store;
4. capture hash retained in the private evidence record;
5. sanitized handshake/capture summary suitable for public release;
6. extracted flow metadata where applicable;
7. inclusion/exclusion decision.

Raw PCAPs are not committed to the public repository.

For Phase 1C specifically, all three cryptographic regimes must use the same evidence schema and capture methodology before a cross-regime comparison is considered valid.

## 6. Feature extraction

Feature Schema v1 emphasizes encrypted-traffic metadata rather than payload content. Candidate fields include counts, byte volumes, packet-size statistics, duration, timing/inter-arrival statistics, directionality and TLS/handshake metadata exposed by the qualified capture toolchain.

Feature extraction is versioned. The feature-schema version must be written to every processed record or dataset manifest.

## 7. Dataset construction and leakage prevention

Primary model evaluation must use **grouped splitting**. Records derived from the same experimental run, closely related connection family or source capture must remain within a single partition. A naive random row-level split is not accepted as the primary evaluation because it can leak run-specific fingerprints across train and test sets.

## 8. Statistical and ML evaluation

The planned core outputs are:

- Macro-F1;
- per-class precision and recall;
- confusion matrix;
- probability calibration/reliability where probabilistic outputs are available;
- cross-regime train/test matrix;
- Cryptographic Generalization Gap (CGG);
- uncertainty intervals or repeated/grouped resampling where appropriate.

Any hypothesis test must identify the unit of analysis and avoid treating correlated flow rows from a single run as independent experimental replicates.

## 9. Reproducibility and provenance

Every canonical result must trace back to immutable or versioned inputs. At minimum, record repository commit, environment versions, run ID, workload ID, cryptographic regime, capture hash, feature-schema version and model/evaluation configuration.

Phase 1B established clean rebuild reproducibility for the target hybrid/PQC-capable TLS path. Phase 1C extends provenance requirements to the three-regime comparison matrix without reopening the Phase 1B acceptance decision.

## 10. Result promotion rule

A value enters `results/results.csv` as `verified` only when supporting artifacts exist and the computation or acceptance decision is reproducible. Thesis-era illustrative/simulated numbers may be discussed as historical context but are not silently promoted into the new empirical results table.

Phase 1B rows may remain `verified` while Phase 1C rows remain `pending`; completion of the later comparative phase is not a prerequisite for preserving the earlier platform-validation result.

## 11. Ethics and safety

No real user traffic or personally identifiable information is required for the controlled platform. Attack-behavior generation is confined to isolated systems under the researcher's control. Public artifacts are sanitized to remove credentials, private keys, sensitive addressing and unnecessary raw traffic.
