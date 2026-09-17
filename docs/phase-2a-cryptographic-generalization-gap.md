# Phase 2A — Benign Cryptographic Distribution Shift and the Cryptographic Generalization Gap

## Objective

Phase 2A tested a focused question: **does migration from classical TLS to hybrid or post-quantum key establishment cause legitimate benign network flows to appear anomalous to a machine-learning detector trained only on classical benign traffic?**

The experiment intentionally excluded attack traffic. The purpose was to isolate cryptographic migration itself as a possible source of machine-learning distribution shift before introducing malicious behavior in Phase 2B.

## Experimental treatments

All three conditions used the same qualified OpenSSL/OQS runtime, TLS 1.3, the same certificate/authentication configuration, the same cipher suite and the same capture path. The independent variable was the key-establishment regime:

- **C1-OQS classical control:** `X25519`
- **C2 hybrid:** `X25519MLKEM768`
- **C3 PQC-oriented:** `mlkem768`

C3 uses post-quantum ML-KEM-768 for key establishment but retains ECDSA P-256 authentication; it is therefore described as PQC-oriented rather than fully post-quantum TLS.

## Benign workload design

The final dataset contains **3,000 accepted benign flow observations**, with 1,000 flows per regime. Six workload classes were used:

| Workload | Description | Flows per regime |
|---|---|---:|
| W01 | Phase 1C anchor, 121-byte response | 200 |
| W02 | Small GET, 1 KiB | 200 |
| W03 | API-style response, 8 KiB JSON | 150 |
| W04 | Small download, 64 KiB | 150 |
| W05 | Large download, 1 MiB | 150 |
| W06 | Concurrent workload: 30 batches × five simultaneous 64 KiB connections | 150 |
| **Total** |  | **1,000** |

Each logical workload instance was assigned a common `pair_id` across C1-OQS, C2 and C3. Regime order was randomized and balanced within the production schedule to reduce systematic collection-order effects.

Production comprised **880 capture units** and yielded the required 3,000 official flows.

## Feature Schema v1

Twenty-five frozen model features were extracted from every accepted flow:

1. `flow_duration_ms`
2. `packets_total`
3. `packets_c2s`
4. `packets_s2c`
5. `tcp_payload_bytes_total`
6. `tcp_payload_bytes_c2s`
7. `tcp_payload_bytes_s2c`
8. `payload_packet_count`
9. `payload_len_mean`
10. `payload_len_std`
11. `payload_len_min`
12. `payload_len_max`
13. `byte_ratio_c2s_s2c`
14. `packet_ratio_c2s_s2c`
15. `iat_ms_mean`
16. `iat_ms_std`
17. `iat_ms_p50`
18. `iat_ms_p95`
19. `tls_record_count`
20. `tls_record_len_mean`
21. `tls_record_len_std`
22. `tls_record_len_max`
23. `clienthello_len`
24. `serverhello_len`
25. `hello_rtt_ms`

Cryptographic regime, TLS group, workload ID, server port, retransmission count and other experiment metadata were retained for analysis but were **not supplied to the model**.

The final production feature matrix contains **3,000 rows × 25 model features with zero missing values**.

## Engineering challenges and corrective controls

### VM storage capacity

Large-scale capture collection approached the capacity of the original research VM filesystem. The virtual disk was expanded, the final partition was extended and the ext4 filesystem was resized online. Frozen design, workload, schedule and runner hashes were revalidated after the storage change before production resumed.

### Docker containers after restart

The VM restart left the OQS client and server containers stopped. They were restarted and the qualified runtime, Docker image identity, network identities and cryptographic algorithms were revalidated before collection continued.

### Segmentation-offload artifact

After the reboot, an early Unit 6 capture contained a `tcp.len` value of 7,240 bytes. Pre-reboot pilot captures consistently showed a maximum TCP payload of 1,448 bytes, and 7,240 equals five 1,448-byte segments aggregated by the host stack. The issue was traced to runtime GRO/GSO/TSO behavior on the Docker capture path.

Because packet counts, payload statistics and inter-arrival timing were model features, mixing capture semantics would have contaminated the experiment. The solution was to standardize GRO/GSO/TSO state across the Docker bridge, host veth interfaces and container interfaces, archive the pre-standardization official observations, and restart final production under the standardized capture condition. After correction, Unit 6 again produced `max_tcp_len=1448` across all three regimes.

### TShark application-protocol misclassification

At Unit 471, production validation initially reported missing TLS handshake metadata even though the full TCP connection was present and the client reported a successful TLS 1.3 session and HTTP response. TShark had automatically classified the application stream as another protocol rather than TLS.

Explicit decode-as recovered the expected handshake immediately. Final feature extraction therefore forced experimental TCP ports 4433–4437 to be decoded as TLS. This changed packet interpretation only; it did not alter captured bytes.

### Python/SciPy dependency mismatch

The host Python environment produced a SciPy/NumPy compatibility warning. Rather than modify the system Python stack, Phase 2A modelling used a dedicated virtual environment with pinned packages:

- Python 3.10.12
- NumPy 1.26.4
- SciPy 1.13.1
- pandas 2.3.3
- scikit-learn 1.7.2
- joblib 1.5.3

The environment was functionally tested with a small Isolation Forest fit before the final experiment and then integrity-frozen.

## Classical reference split and leakage control

The 1,000 C1-OQS flows were partitioned before modelling as:

- **600 training flows**
- **200 calibration flows**
- **200 held-out test flows**

The split was workload-stratified. For W06, entire concurrent batches were assigned to a partition so that connections from the same batch could not leak across train, calibration and test.

The 200 held-out C1 pair IDs defined the primary matched evaluation set: the corresponding 200 C2 and 200 C3 flows were evaluated alongside them.

## Isolation Forest baseline

The frozen baseline used:

- 500 estimators
- `max_samples=256`
- all 25 model features
- no bootstrapping
- deterministic random state `20260917`

The model was fitted only on the 600 classical C1-OQS training flows. The anomaly threshold was then fixed at the 95th percentile of anomaly scores from the 200 C1 calibration flows, yielding an observed calibration false-positive rate of 5%.

No C2 or C3 observation influenced fitting or threshold selection.

## Primary held-out results

| Regime | False positives | FPR | 95% Wilson CI | Gap vs C1 |
|---|---:|---:|---:|---:|
| C1-OQS | 2 / 200 | **1.0%** | 0.27–3.57% | baseline |
| C2 hybrid | 169 / 200 | **84.5%** | 78.84–88.86% | **+83.5 pp** |
| C3 PQC-oriented | 119 / 200 | **59.5%** | 52.58–66.06% | **+58.5 pp** |

The same detector that maintained a 1.0% false-positive rate on unseen classical benign traffic classified most matched C2 and C3 flows as anomalous.

## Matched statistical evidence

For **C2 vs C1-OQS**, 167 matched pairs were C2-anomalous/C1-normal and zero showed the reverse pattern. The exact McNemar p-value was approximately `1.07e-50`. The matched FPR difference was +83.5 percentage points with a bootstrap 95% interval of approximately +78.0 to +88.5 points.

For **C3 vs C1-OQS**, 117 pairs were C3-anomalous/C1-normal and zero showed the reverse pattern. The exact McNemar p-value was approximately `1.20e-35`. The matched FPR difference was +58.5 points with a bootstrap 95% interval of approximately +51.5 to +65.5 points.

Continuous anomaly-score analysis supported the same conclusion. Median matched anomaly-score increases were approximately +0.1198 for C2 and +0.09225 for C3. Holm-adjusted Wilcoxon tests remained extremely small, with matched rank-biserial effects of approximately 0.9997 for C2 and 0.9934 for C3.

The C2-vs-C3 comparison was treated as secondary/exploratory. C2 produced a 25-point higher FPR than C3 on the matched test set.

## Workload dependence

The effect was strongly workload-dependent:

| Workload | C1-OQS | C2 | C3 |
|---|---:|---:|---:|
| W01 | 2.5% | 100% | 85% |
| W02 | 0% | 100% | 90% |
| W03 | 0% | 100% | 100% |
| W04 | 0% | 20% | 0% |
| W05 | 3.3% | 76.7% | 43.3% |
| W06 | 0% | 100% | 20% |

This heterogeneity is important: the experiment does not imply that PQC-oriented traffic is intrinsically anomalous. It shows that the interaction between cryptographic regime and application workload can move benign traffic by very different amounts relative to a classical-trained model.

## Feature-level mechanism

Cryptographic migration produced large deterministic handshake-size shifts. In C1 training, `clienthello_len` was constant at 331 bytes and `serverhello_len` was constant at 118 bytes. Their median increases were:

- C2 ClientHello: +1,176 bytes
- C3 ClientHello: +1,144 bytes
- C2 ServerHello: +1,088 bytes
- C3 ServerHello: +1,056 bytes

Every C2 and C3 primary test flow fell outside the C1 training range for both handshake-length features.

However, both handshake-length variables had **zero variance in C1 training and zero Isolation Forest tree splits**. They therefore document the cryptographic distribution shift but were not direct decision variables in the frozen model.

The forest instead partitioned timing and flow-morphology variables heavily. The most frequently used features included `hello_rtt_ms`, `iat_ms_mean`, `iat_ms_p95`, `iat_ms_std` and `flow_duration_ms`.

A matched single-feature counterfactual sensitivity analysis further showed that replacing only `hello_rtt_ms` with the matched classical value resolved 42 of 169 C2 false positives and 65 of 119 C3 false positives. For C3, replacing `iat_ms_p95` resolved 45 false positives.

Because network-flow features are correlated, this counterfactual analysis is diagnostic rather than causal attribution. Nevertheless, the combined evidence indicates that the detector primarily responded to timing and traffic-shape consequences of cryptographic migration rather than directly to the most obvious zero-variance handshake-size features.

## Cryptographic Generalization Gap

Phase 2A operationalizes the **Cryptographic Generalization Gap (CGG)** as the increase in benign false-positive behavior when a detector trained exclusively on classical cryptographic traffic is evaluated on matched legitimate traffic generated under an unseen cryptographic regime.

Under the frozen Phase 2A baseline:

- **C2 hybrid CGG:** +83.5 percentage points
- **C3 PQC-oriented CGG:** +58.5 percentage points

The result demonstrates that cryptographic migration can itself become a material machine-learning distribution shift in network anomaly detection.

## Reproducibility discipline

The final result was generated only after:

- deterministic workload and production schedule generation;
- randomized/balanced regime ordering;
- accepted-flow ledgering and per-PCAP hashes;
- capture-drop and segmentation checks;
- standardized GRO/GSO/TSO behavior;
- explicit TLS decode-as policy;
- frozen Feature Schema v1;
- an isolated pinned ML environment;
- frozen model definition and deterministic random seed;
- matched statistical analysis;
- feature-driver and mechanism analyses; and
- SHA-256 freezing of the final tables, figures and closure artifacts.

Raw PCAPs, secrets and private experimental evidence remain outside the public repository. Public evidence is limited to sanitized summaries and aggregate results.

## Limitations

Phase 2A evaluates one anomaly-detection algorithm, one frozen 25-feature representation, one controlled OpenSSL/OQS implementation, three key-establishment regimes and six benign workload classes. It does not establish that every IDS or every PQC deployment will exhibit the same behavior.

The traffic was generated in a controlled laboratory rather than an unconstrained production network. Timing features may be environment-sensitive, although matched workloads, randomized regime ordering and capture-path controls reduce several important confounders.

No malicious traffic was included. Phase 2A therefore measures benign generalization and false-positive behavior, not attack-detection effectiveness.

## Conclusion

Phase 2A provides controlled evidence that a detector trained only on classical benign TLS can generalize poorly when legitimate traffic migrates to hybrid or post-quantum key establishment. The held-out classical false-positive rate was 1.0%, while matched C2 and C3 traffic produced false-positive rates of 84.5% and 59.5%, respectively.

The effect remained clear in matched binary outcomes and continuous anomaly scores, varied strongly by workload, and was associated primarily with timing and flow-morphology consequences of the cryptographic change.

Phase 2B will build on this baseline by introducing controlled malicious traffic and supervised IDS evaluation across classical, hybrid and PQC-oriented regimes, allowing the study to test whether benign cryptographic distribution shift also affects attack-detection performance.
