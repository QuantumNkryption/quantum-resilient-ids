# Experiment Methodology

## 1. Research design

This project uses controlled comparative experimentation. The independent variable is the **cryptographic regime**; application workload and other experimental conditions are held as constant as practical. The principal outcome is the change in observable network metadata and, in later phases, downstream IDS/model performance.

The methodology is deliberately staged:

- **Phase 0:** freeze scientific questions, workloads, labels, features, metrics and provenance.
- **Phase 1A:** qualify the cryptographic instrumentation.
- **Phase 1B:** verify reproducible generation and capture of the target PQC-capable TLS handshake, including clean rebuild reproduction.
- **Phase 1C:** establish the controlled classical / hybrid / PQC-oriented TLS comparison matrix.
- **Later phases:** generate controlled datasets, evaluate conventional IDS, train/evaluate ML models and perform cross-regime generalization analysis.

The separation between Phase 1B and Phase 1C is deliberate. Phase 1B is a platform-capability and reproducibility gate; Phase 1C is the comparative experimental-control gate.

## 2. Controlled variables

For comparisons across cryptographic regimes, preserve where applicable:

- workload definition;
- endpoint roles;
- payload/application object;
- connection/session count;
- capture point;
- software image and configuration other than the cryptographic treatment;
- TLS protocol version and cipher suite;
- certificate/authentication scheme;
- feature-extraction version;
- run-record schema.

Any unavoidable difference between regimes must be recorded explicitly.

## 3. Phase 1C treatments

The canonical same-stack Phase 1C comparison uses one qualified OQS/OpenSSL runtime:

- OpenSSL 3.4.7
- oqs-provider 0.11.0
- liboqs 0.15.0

The three conditions are:

- **C1-OQS classical control:** `X25519` key establishment, ECDSA P-256 authentication, n=10.
- **C2 hybrid:** `X25519MLKEM768` key establishment, ECDSA P-256 authentication, n=30.
- **C3 PQC-oriented:** `mlkem768` key establishment, ECDSA P-256 authentication, n=30.

All three use TLS 1.3 and `TLS_AES_256_GCM_SHA384` with the same fixed 121-byte application payload.

C3 is labelled PQC-oriented rather than fully PQC because authentication remains classical ECDSA P-256.

A preliminary 30-run X25519 dataset had been collected under system OpenSSL 3.0.2. Because that introduced an implementation mismatch relative to C2, a 10-run X25519 sanity control was repeated under the exact OQS/OpenSSL 3.4.7 stack used for C2/C3. The same-stack C1-OQS dataset is the canonical Phase 1C classical control.

## 4. Workloads and labels

### Phase 1C fixed workload

Phase 1C uses one fixed benign HTTP object to isolate cryptographic effects. The response payload is 121 bytes with SHA-256:

`04b754f4158a69fbc0af6f64d0d4abb5007a24b9e4d2fc7860843ae9a0c4e986`

### Later benign workloads

B01 HTTPS GETs; B02 repeated requests; B03 API-style traffic; B04 small downloads; B05 large downloads; B06 concurrent sessions.

### Initial attack-behavior classes

A01 reconnaissance; A02 brute-force behavior; A03 request flooding.

These are executed only inside isolated laboratory infrastructure owned or controlled by the researcher.

## 5. Data collection

Each accepted comparative run produces, at minimum:

1. run manifest;
2. environment/tool version record;
3. packet capture retained in the private evidence store;
4. capture hash retained in the private evidence record where recorded;
5. sanitized handshake/capture summary suitable for public release;
6. extracted flow metadata;
7. inclusion/exclusion decision.

Raw PCAPs are not committed to the public repository.

For Phase 1C, all canonical conditions use the same evidence schema and capture methodology.

### Capture-quality safeguards

Two measurement artifacts were identified during pilot collection and corrected before official data acceptance:

- **Capture buffering:** an early automated batch produced header-only PCAPs even though packets reached the capture filter. These runs were rejected. The collector was modified to use immediate delivery, packet-buffered output, a drain period, explicit zero-packet/header-only rejection and kernel-drop checks.
- **Segmentation offload:** an early diagnostic capture showed an impossible TCP payload above 4 KB on a 1500-byte MTU path. TSO/GSO/GRO were normalized across the relevant virtual interfaces. Official PQC-oriented captures then showed normal segmentation with maximum TCP payloads of 1448 bytes.

## 6. Feature extraction

Feature Schema v1 emphasizes encrypted-traffic metadata rather than payload content. Candidate fields include counts, byte volumes, packet-size statistics, duration, timing/inter-arrival statistics, directionality and TLS/handshake metadata exposed by the qualified capture toolchain.

Phase 1C already demonstrates substantial distribution shifts in several of these fields, especially packet count, directional bytes, total TCP payload and handshake size.

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

Phase 1C timing results are treated as exploratory because conditions were collected in sequential batches rather than randomized/interleaved order. Packet and byte measurements are the stronger Phase 1C evidence.

Any hypothesis test must identify the unit of analysis and avoid treating correlated flow rows from a single run as independent experimental replicates.

## 9. Reproducibility and provenance

Every canonical result must trace back to immutable or versioned inputs. At minimum, record repository commit, environment versions, run ID, workload ID, cryptographic regime, capture identifier/hash where retained, feature-schema version and model/evaluation configuration.

Phase 1B established clean rebuild reproducibility for the target hybrid/PQC-capable TLS path. Phase 1C extends provenance to the completed three-regime comparison matrix.

## 10. Result promotion rule

A value enters `results/results.csv` as `verified` only when supporting artifacts exist and the computation or acceptance decision is reproducible. Thesis-era illustrative/simulated numbers may be discussed as historical context but are not silently promoted into the new empirical results table.

Phase 1A, Phase 1B and the completed Phase 1C acceptance rows are now verified. Later IDS/ML claims remain pending until their own evidence exists.

## 11. Ethics and safety

No real user traffic or personally identifiable information is required for the controlled platform. Attack-behavior generation is confined to isolated systems under the researcher's control. Public artifacts are sanitized to remove credentials, private keys, sensitive addressing and unnecessary raw traffic.
