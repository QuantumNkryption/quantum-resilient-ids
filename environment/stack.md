# Qualified Environment Stack

This file records the environment qualified during Phase 1A and the separation between the research host/guest and the containerized cryptographic toolchain.

| Layer | Component | Version / value |
|---|---|---|
| Research VM / guest | Ubuntu | 24.04.4 LTS |
| Qualification container | Ubuntu base image | 22.04 |
| Cryptographic stack | OpenSSL | 3.4.7 |
| OQS integration | oqs-provider | 0.11.0 |
| PQC library | liboqs | 0.15.0 |
| OQS build option | libjade | disabled (`OQS_LIBJADE_BUILD=OFF`) |
| Build parallelism | make parameters | `-j2` |
| Qualified OpenSSL binary | path | `/opt/oqs-provider/.local/bin/openssl` |
| OQS provider module | path | `/opt/oqs-provider/_build/lib/oqsprovider.so` |
| Provider module directory | `OPENSSL_MODULES` | `/opt/oqs-provider/_build/lib` |

## Isolation principle

The qualified OQS/OpenSSL installation is deliberately separate from the host/system OpenSSL. Experiments must use the qualified binary and module path rather than assuming that `/usr/bin/openssl` has PQC support.

## Verification

Inside the qualification container:

```bash
openssl version
openssl list -providers -provider default -provider oqsprovider
openssl list -kem-algorithms -provider default -provider oqsprovider
openssl list -signature-algorithms -provider default -provider oqsprovider
openssl list -tls1_3 -provider default -provider oqsprovider 2>/dev/null || true
```

For an explicit path-based provider check:

```bash
OPENSSL_MODULES=/opt/oqs-provider/_build/lib \
/opt/oqs-provider/.local/bin/openssl list -providers \
  -provider default -provider oqsprovider
```

The acceptance record should confirm that the default and OQS providers are active and that the required ML-KEM / hybrid / ML-DSA algorithms are exposed.

## Build definition

The pinned build recipe is stored in [`Dockerfile`](Dockerfile). The image should be rebuilt rather than relying on an unversioned pre-existing container.
