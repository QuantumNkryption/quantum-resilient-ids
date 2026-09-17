# Sanitized Public Artifacts

This directory contains evidence that is useful for independent review but safe for public release.

## Current artifacts

- `phase1b_mldsa65_certificate_summary.txt` — supports `P1B-CERT-001`.
- `phase1b_tls_reproducibility_summary.md` — supports `P1B-TLS-HYBRID-001`, `P1B-PCAP-001` and `P1B-REBUILD-001`.
- `phase1c_cross_regime_summary.md` — supports the verified Phase 1C classical / hybrid / PQC-oriented comparison, packet-capture acceptance and implementation-control rows.
- `phase2a_cryptographic_generalization_summary.md` — supports the verified Phase 2A benign-distribution-shift, Isolation Forest, Cryptographic Generalization Gap, workload and mechanism-analysis rows.

## Included

- concise certificate/public-key metadata;
- version and provider summaries;
- sanitized capture-derived summaries;
- aggregate packet/byte/timing measurements;
- aggregate ML evaluation and uncertainty statistics;
- hashes or identifiers that allow a public result to be mapped to privately retained raw evidence where recorded;
- non-sensitive experiment manifests and methodology notes.

## Excluded

The following must **not** be committed here:

- private keys;
- credentials, tokens or passwords;
- raw secrets or environment files;
- personally identifiable information;
- production or third-party traffic;
- raw PCAPs containing unnecessary addressing/payload information;
- full host inventories or sensitive internal network details;
- large private feature matrices or model binaries unless a later release explicitly approves them for public distribution.

## Evidence model

The public artifact is a reviewable derivative, not a substitute for raw research evidence. Raw artifacts are retained separately and integrity-linked with hashes in the experiment provenance record where applicable.

Each sanitized artifact should identify the phase/test ID it supports and should avoid claiming more than the preserved evidence establishes.
