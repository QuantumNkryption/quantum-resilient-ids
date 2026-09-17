# Experiment Methodology

## 1. Research design

This project uses controlled comparative experimentation. The principal independent variable is the **cryptographic regime**; application workload and other experimental conditions are held as constant as practical. The project then evaluates both observable network-metadata shift and downstream IDS/model generalization.

The methodology is staged:

- **Phase 0:** freeze scientific questions, workloads, labels, features, metrics and provenance rules.
- **Phase 1A:** qualify the OQS/OpenSSL cryptographic instrumentation.
- **Phase 1B:** verify reproducible generation and capture of the target hybrid/PQC-capable TLS path, including clean rebuild reproduction.
- **Phase 1C:** establish the controlled classical / hybrid / PQC-oriented TLS comparison matrix.
- **Phase 2A:** generate matched benign traffic at scale and test whether a detector trained only on classical benign TLS generalizes to unseen hybrid/PQC-oriented regimes.
- **Phase 2B:** introduce controlled attack traffic and supervised IDS evaluation across cryptographic regimes.

The staged structure prevents platform qualification, benign distribution shift and attack-detection performance from being conflated.

## 2. Controlled variables

For comparisons across cryptographic regimes, preserve where applicable:

- workload definition;
- endpoint roles;
- application object/response size;
- connection/session count;
- capture point;
- software image and configuration other than the cryptographic treatment;
- TLS protocol version and cipher suite;
- certificate/authentication scheme;
- feature-extraction version;
- run-record schema; and
- capture-quality controls.

Any unavoidable difference between regimes must be recorded explicitly.

## 3. Cryptographic treatments

The canonical same-stack comparison uses one qualified OQS/OpenSSL runtime:

- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0

The three Phase 1C/2A conditions are:

- **C1-OQS classical control:** `X25519` key establishment, ECDSA P-256 authentication.
- **C2 hybrid:** `X25519MLKEM768` key establishment, ECDSA P-256 authentication.
- **C3 PQC-oriented:** `mlkem768` key establishment, ECDSA P-256 authentication.

All use TLS 1.3 and `TLS_AES_256_GCM_SHA384` with the same certificate/authentication configuration.

C3 is labelled PQC-oriented rather than fully PQC because authentication remains classical ECDSA P-256.

## 4. Phase 2A benign workloads

Phase 2A expands the fixed Phase 1C anchor into six matched benign workload classes:

| Workload | Description | Flows per regime |
|---|---|---:|
| W01 | Phase 1C anchor, 121-byte response | 200 |
| W02 | Small GET, 1 KiB | 200 |
| W03 | API-style response, 8 KiB JSON | 150 |
| W04 | Small download, 64 KiB | 150 |
| W05 | Large download, 1 MiB | 150 |
| W06 | 30 batches × five simultaneous 64 KiB connections | 150 |

This produces 1,000 benign flows per cryptographic regime and 3,000 total accepted observations.

Each logical workload instance is assigned a common `pair_id` across C1-OQS, C2 and C3 so that primary comparisons can be performed on matched workload identities.

Regime order is randomized and position-balanced within the production schedule to reduce systematic collection-order effects.

## 5. Data collection and acceptance

Each accepted comparative flow produces, at minimum:

1. run/production ledger metadata;
2. environment and tool-version provenance;
3. packet capture retained in the private evidence store;
4. capture SHA-256 recorded in the private evidence record;
5. extracted flow metadata and model features;
6. inclusion/exclusion status; and
7. production-quality checks.

Raw PCAPs are not committed to the public repository.

### Capture-quality safeguards

Several runtime artifacts were identified and corrected before final Phase 2A acceptance:

- **Capture buffering:** earlier Phase 1 work showed that successful sessions can still yield header-only PCAPs if capture buffers are not drained. Accepted collection therefore checks packet presence and capture drops.
- **Segmentation offload:** after a VM restart, a Phase 2A diagnostic capture contained a 7,240-byte TCP payload even though pre-reboot captures consistently maxed at 1,448 bytes. GRO/GSO/TSO behavior was standardized across the Docker bridge, host veths and container interfaces. Pre-standardization production observations were archived and excluded from the final analytical dataset.
- **Protocol dissection:** TShark automatically classified one valid TLS stream as another application protocol. Final feature extraction explicitly decodes the experimental TCP ports as TLS rather than relying on automatic protocol heuristics.

## 6. Feature Schema v1

The final Phase 2A model representation contains 25 encrypted-traffic metadata features spanning:

- flow duration;
- total/directional packet counts;
- total/directional TCP payload bytes;
- payload-length statistics;
- byte and packet direction ratios;
- inter-arrival-time statistics;
- TLS record count/length statistics;
- ClientHello and ServerHello lengths; and
- ClientHello-to-ServerHello timing (`hello_rtt_ms`).

Feature extraction does not include plaintext application content.

The following are metadata only and are not model inputs:

- cryptographic regime;
- TLS group;
- workload ID;
- server port;
- retransmission count;
- capture filename/hash; and
- experimental ordering metadata.

The final Phase 2A feature matrix contains 3,000 rows, 25 model features and zero missing values.

## 7. Dataset construction and leakage prevention

The 1,000 C1-OQS observations are partitioned as:

- 600 training flows;
- 200 calibration flows; and
- 200 held-out test flows.

The split is workload-stratified. For W06, entire concurrent batches are assigned to a partition rather than individual connections so that connections from the same batch cannot leak across training, calibration and test.

The 200 held-out C1 `pair_id` values define the primary matched evaluation set. Their corresponding 200 C2 and 200 C3 observations are evaluated without contributing to model fitting or threshold selection.

## 8. Phase 2A anomaly-detection baseline

The frozen baseline uses `sklearn.ensemble.IsolationForest` with:

- `n_estimators=500`;
- `max_samples=256`;
- `contamination="auto"`;
- `max_features=1.0`;
- `bootstrap=False`;
- `random_state=20260917`; and
- all 25 model features.

The Isolation Forest is fitted only on the 600 C1-OQS training flows.

The anomaly threshold is fixed at the 95th percentile of anomaly scores from the 200 separate C1-OQS calibration observations. This produced an observed calibration false-positive rate of 5%.

The primary evaluation then scores the matched held-out set of 200 C1-OQS, 200 C2 and 200 C3 observations.

## 9. Statistical evaluation

Phase 2A reports:

- primary false-positive rate by regime;
- Wilson 95% confidence intervals for regime FPR;
- paired FPR differences / Cryptographic Generalization Gap;
- exact McNemar tests on matched binary anomaly outcomes;
- bootstrap intervals for matched risk differences;
- Wilcoxon signed-rank tests for paired anomaly-score shifts;
- matched rank-biserial effect sizes;
- Holm adjustment for the two prespecified primary comparisons (C2 vs C1 and C3 vs C1);
- workload-specific false-positive rates; and
- feature-shift, counterfactual-sensitivity and mechanism diagnostics.

The C2-vs-C3 comparison is treated as secondary/exploratory rather than as a prespecified primary contrast.

## 10. Cryptographic Generalization Gap

For Phase 2A, the **Cryptographic Generalization Gap (CGG)** is operationalized as the change in benign false-positive behavior when a detector trained exclusively on classical cryptographic traffic is evaluated on matched legitimate traffic generated under an unseen cryptographic regime.

The verified Phase 2A primary gaps are:

- C2 vs C1-OQS: +83.5 percentage points;
- C3 vs C1-OQS: +58.5 percentage points.

This definition is specific to the evaluated task and should not be interpreted as a universal property of every IDS or PQC deployment.

## 11. Mechanism analysis

Phase 2A distinguishes **observable distribution shift** from **direct model use of a feature**.

`clienthello_len` and `serverhello_len` changed dramatically between C1 and C2/C3, but both had zero variance in the C1 training set and therefore zero Isolation Forest tree splits. They are strong evidence of cryptographic distribution shift but were not direct partition variables in the frozen forest.

The forest used timing and flow-morphology variables much more heavily. A matched one-feature counterfactual sensitivity analysis replaces a target-regime feature with its matched C1 value and rescored the observation without retraining. Because network-flow features are correlated, this is interpreted as diagnostic sensitivity rather than causal attribution.

## 12. Reproducibility and provenance

Every canonical result must trace back to immutable or versioned inputs. At minimum, preserve:

- repository commit;
- cryptographic runtime versions;
- workload and production-schedule versions;
- flow/run identifiers;
- cryptographic regime;
- capture identifier/hash in the private evidence store;
- feature-schema and extraction versions;
- model configuration and seed;
- ML environment versions; and
- analysis/output hashes.

Phase 2A additionally freezes dataset, feature-matrix, model, statistical-analysis, feature-driver, mechanism-analysis, figure and closure artifacts using SHA-256 manifests in the private research workspace.

## 13. Result promotion rule

A value enters `results/results.csv` as `verified` only when supporting evidence exists and the computation or acceptance decision is reproducible.

Phase 1A, Phase 1B, Phase 1C and the completed Phase 2A acceptance/result rows are verified. Phase 2B attack-detection claims remain pending until their own design, data and evidence exist.

## 14. Ethics and safety

No real user traffic or personally identifiable information is required for the controlled platform. Future attack-behavior generation is confined to isolated systems owned or controlled by the researcher. Public artifacts are sanitized to remove credentials, private keys, sensitive addressing and unnecessary raw traffic.
