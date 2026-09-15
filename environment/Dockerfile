FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    cmake \
    git \
    ninja-build \
    perl \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

RUN git clone --branch 0.11.0 --depth 1 https://github.com/open-quantum-safe/oqs-provider.git

WORKDIR /opt/oqs-provider

RUN OPENSSL_BRANCH=openssl-3.4.7 \
    LIBOQS_BRANCH=0.15.0 \
    OQS_LIBJADE_BUILD=OFF \
    MAKE_PARAMS="-j2" \
    bash scripts/fullbuild.sh

ENV OPENSSL_MODULES=/opt/oqs-provider/_build/lib
ENV PATH=/opt/oqs-provider/.local/bin:${PATH}

CMD ["/bin/bash"]
