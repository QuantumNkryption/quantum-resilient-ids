# Phase 1B — PQC/Hybrid TLS Generation, Capture & Reproducibility Validation

## Objective

Validate that the qualified Phase 1A cryptographic stack can generate the target PQC-capable TLS traffic, that the negotiated cryptographic properties are observable in packet-capture evidence, and that the result survives a clean destroy/rebuild/reproduce cycle.

Phase 1B is intentionally narrower than the later cross-regime experiment. Its purpose is to establish that the PQC-capable experimental platform itself works reproducibly before classical, hybrid and PQC regimes are compared under a common protocol.

## Acceptance sequence

1. Generate suitable test credentials inside the qualified environment.
2. Establish the target TLS 1.3 session using the qualified hybrid key-establishment group `X25519MLKEM768` and ML-DSA-65 authentication.
3. Capture the handshake using packet-capture tooling.
4. Inspect the capture independently and confirm the negotiated handshake metadata.
5. Preserve sanitized evidence suitable for public release while excluding private keys and raw captures from Git.
6. Destroy the qualified environment.
7. Rebuild from the documented versions/configuration.
8. Generate fresh test artifacts and reproduce the same acceptance result.

## Verified evidence

Phase 1B is **complete**.

The acceptance exercise produced two independent successful runs. Run 2 followed a clean destroy/rebuild of the qualified environment rather than reuse of the original runtime state.

Across both accepted runs:

- TLS 1.3 negotiation succeeded;
- the negotiated hybrid key-establishment group was `X25519MLKEM768`;
- the corresponding observed group identifier was `0x11ec`;
- ML-DSA-65 was used for certificate/authentication evidence;
- each accepted packet capture contained 23 packets;
- the same handshake sequence was observed across both captures; and
- the rebuilt run used freshly generated test artifacts.

The previously verified ML-DSA-65 certificate evidence includes:

- Subject: `CN=oqs-phase1b-server`
- Issuer: `CN=oqs-phase1b-server`
- Signature algorithm: `mldsa65`
- Public-key algorithm: `mldsa65`
- Serial: `47FC538165C7C0E9596443BE2027FFCE96A226DA`
- Validity: 2026-09-15 01:36:49 UTC through 2026-10-15 01:36:49 UTC

Public sanitized evidence is stored under `artifacts/sanitized/`. Raw captures and private keys are excluded from the public repository.

## Acceptance decision

**PASS — Phase 1B complete.**

The required platform capability, packet-capture observability and clean rebuild/reproduction gate were satisfied. Phase 1B therefore establishes a reproducible PQC-capable TLS traffic-generation foundation.

## Scope boundary

Phase 1B does **not** claim that classical, hybrid and PQC traffic have already been compared under a common controlled matrix. That broader comparison is now defined separately as **Phase 1C — Cross-Regime TLS Validation**.

The following items are therefore not prerequisites for Phase 1B completion:

- a classical TLS control run;
- a pure/PQC-oriented ML-KEM control run;
- direct classical-vs-hybrid-vs-PQC packet-size comparison;
- comparative latency or handshake-size statistics; or
- alternative signature combinations such as `p384_mldsa65`.

Those questions belong to Phase 1C or later experimental phases.

## Publication rule

Phase 1B supports the claim that the qualified environment reproducibly generated and captured the target TLS 1.3 handshake using `X25519MLKEM768` and ML-DSA-65 authentication.

It does not support numerical claims about relative handshake size, latency, IDS performance or cross-regime distribution shift. Those claims require the controlled Phase 1C matrix and later empirical analysis.
