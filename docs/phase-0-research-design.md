# Phase 0 — Research Design Freeze

## Objective

Freeze the scientific design before infrastructure or model experimentation so later results can be interpreted as tests of predefined questions rather than post-hoc exploration.

## Primary research question

How does migration from classical TLS to hybrid and post-quantum cryptographic regimes change observable network metadata, and how does that distribution shift affect IDS and machine-learning generalization?

## Experimental regimes

- Classical TLS
- Hybrid classical + ML-KEM TLS
- PQC-oriented TLS using ML-KEM / ML-DSA where supported by the qualified stack

## Benign workload catalogue

The approved Phase 0 benign workloads are:

- B01 — HTTPS GETs
- B02 — repeated requests
- B03 — API-style traffic
- B04 — small downloads
- B05 — large downloads
- B06 — concurrent sessions

The intent is to hold application behavior stable while changing the cryptographic regime.

## Initial attack-behavior taxonomy

- A01 — reconnaissance behavior
- A02 — brute-force behavior
- A03 — request flooding

All attack-behavior work is restricted to isolated laboratory systems under the researcher's control.

## Feature Schema v1

The research program prioritizes features that remain observable without payload decryption, including:

- packet counts and byte counts
- packet-size statistics
- flow duration
- inter-arrival timing
- directionality
- handshake-related metadata available to the capture/analysis toolchain
- session-level statistical summaries

The feature schema is versioned. Features added later must not be silently back-propagated into prior experiments.

## Evaluation protocol

Primary evaluation criteria:

- Macro-F1
- per-class precision and recall
- confusion matrices
- calibration / reliability
- grouped train/test splitting to prevent leakage between related flows, captures or runs
- cross-regime evaluation

### Cryptographic Generalization Gap (CGG)

CGG is the performance difference observed when a model trained under one cryptographic regime is tested under another. The exact sign convention and formula must be frozen in the analysis code before Phase 2 model evaluation and then used consistently.

## Leakage controls

The unit of grouping must prevent records derived from the same experiment, connection family, workload run or capture from appearing in both training and test sets. Random row-level splitting alone is not acceptable for the primary result.

## Provenance requirements

Every experimental run should be traceable to:

- run identifier
- UTC timestamp
- phase
- workload ID
- attack/benign label
- cryptographic regime
- source and destination roles
- environment versions
- configuration hash or commit
- capture identifier
- feature-schema version
- random seed when applicable
- model/evaluation configuration when applicable

## Inclusion / exclusion principle

Only runs that satisfy the phase acceptance checks are eligible for the canonical dataset. Failed or malformed runs are preserved as diagnostic evidence but excluded from primary statistical analysis with an explicit reason.

## Open Quantum IDS Benchmark (OQIB)

OQIB is reserved as a future benchmark/data-release concept. Phase 0 defines the provenance and experimental discipline needed to make a later benchmark credible; it does not claim that the benchmark has already been released.

## Exit criterion

Phase 0 is complete when the questions, workloads, labels, feature schema, evaluation logic, leakage controls and provenance rules are frozen sufficiently to build the infrastructure without changing the scientific target.
