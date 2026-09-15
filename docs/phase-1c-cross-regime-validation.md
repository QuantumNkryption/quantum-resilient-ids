# Phase 1C — Cross-Regime TLS Validation Protocol

## Objective

Establish a controlled, directly comparable TLS evidence matrix across three cryptographic regimes:

1. **Classical** — classical key establishment and authentication.
2. **Hybrid** — classical + post-quantum key establishment using the qualified hybrid group, including `X25519MLKEM768`.
3. **PQC-oriented** — ML-KEM / ML-DSA configuration supported by the qualified OQS/OpenSSL stack.

Phase 1C is the bridge between platform qualification and later IDS/ML experimentation. Its purpose is not to measure IDS performance yet; it is to prove that all three regimes can be generated, captured and compared under controlled conditions without changing unrelated experimental variables.

## Research question

When application behavior and capture conditions are held constant, what TLS and flow-level metadata differ across classical, hybrid and PQC-oriented cryptographic regimes?

## Experimental principle

The **cryptographic regime is the treatment variable**. Everything else should remain as constant as practical.

For the three-regime acceptance matrix, preserve the same:

- endpoint roles;
- application request/workload;
- payload/object where applicable;
- connection/session pattern;
- capture point;
- network path;
- software image and toolchain;
- TLS protocol version where technically compatible;
- evidence-extraction procedure; and
- run-record format.

Any unavoidable difference must be recorded explicitly in the run manifest rather than silently treated as equivalent.

## Regime definitions

### C1 — Classical control

A TLS 1.3 connection using classical key establishment and classical authentication supported by the same qualified OpenSSL environment.

The exact algorithms used must be recorded in the run manifest.

### C2 — Hybrid control

A TLS 1.3 connection using the qualified hybrid key-establishment group `X25519MLKEM768` and the selected authentication configuration.

This regime reuses the capability established in Phase 1B, but must be rerun under the Phase 1C comparison protocol so that its capture is directly comparable with C1 and C3.

### C3 — PQC-oriented control

A TLS 1.3 connection using the supported ML-KEM key-establishment configuration and ML-DSA authentication available in the qualified stack.

The exact group and signature/certificate algorithm must be recorded from the negotiated or independently observable evidence. The label `pqc` must never substitute for the exact algorithm names.

## Acceptance sequence

For each regime:

1. Start from the same qualified environment and record repository commit and tool versions.
2. Generate or select the regime-appropriate test certificate material.
3. Execute the standardized TLS test workload.
4. Capture the complete handshake at the same capture point.
5. Independently inspect the capture and client/server evidence.
6. Record negotiated TLS version, cipher suite, key-establishment/group and certificate/signature metadata where observable.
7. Record packet count and other non-sensitive handshake metadata.
8. Retain the raw capture privately and preserve a sanitized public summary.
9. Mark the run accepted or excluded, with a reason for any exclusion.

After all three regimes pass individually, build a single comparison table from the sanitized summaries.

## Evidence record

Every accepted Phase 1C run should record at minimum:

- Phase 1C test ID;
- run ID;
- UTC timestamp;
- repository commit SHA;
- environment/tool versions;
- cryptographic regime label;
- exact key-establishment/group algorithm;
- exact certificate/signature algorithm;
- TLS protocol version;
- negotiated cipher suite;
- packet-capture identifier;
- raw-capture SHA-256 in the private evidence store;
- packet count;
- sanitized capture-derived handshake summary;
- acceptance result; and
- deviation/exclusion notes where applicable.

Private keys, credentials and raw captures are not committed to the public repository.

## Comparability gate

Phase 1C is complete only when all of the following are true:

- Classical, hybrid and PQC-oriented TLS sessions each pass their individual handshake acceptance test.
- Each regime has independently inspected packet-capture evidence.
- The three accepted captures were produced using the same standardized workload and capture methodology.
- Exact algorithms are recorded rather than inferred from regime labels.
- A sanitized cross-regime comparison summary exists.
- No numerical performance or IDS claim is promoted beyond what the captured evidence supports.

## Planned comparison outputs

The Phase 1C comparison may report directly observable metadata such as:

- negotiated TLS version;
- cipher suite;
- key-establishment/group;
- certificate/signature algorithm;
- handshake packet count;
- capture-observed byte counts where extraction is reproducible;
- handshake message sequence; and
- other fields admitted by the versioned feature schema.

Latency should only be reported if the timing method and clock/capture conditions are controlled and documented. IDS or machine-learning performance does not belong to Phase 1C.

## Relationship to Phase 1B

Phase 1B established that the PQC-capable environment can reproducibly generate and capture the target hybrid/PQC-capable TLS handshake and survive a clean rebuild.

Phase 1C does not reopen that acceptance decision. It extends the platform with a controlled **classical vs hybrid vs PQC** comparison matrix required for later distribution-shift and IDS generalization experiments.

## Current status

**Planned / pending execution.**

No Phase 1C row should be promoted to `verified` until its corresponding run evidence and sanitized artifact exist.
