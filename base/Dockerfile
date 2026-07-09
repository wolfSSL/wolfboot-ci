# wolfboot-ci-base: Devuan testing base layer for wolfBoot CI
FROM docker.io/devuan/devuan:testing

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl wget xz-utils file \
    git build-essential pkg-config \
    autoconf automake libtool \
    cmake ninja-build \
    python3 python3-pip python3-venv python3-full python-is-python3 \
    bash \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean \
    && ln -sf /bin/bash /bin/sh

# ---- wolfSSL: build from source (removed from Debian/Devuan testing) ----
ENV WOLFSSL_VERSION=v5.9.2-stable

RUN set -eux; \
    wget -q "https://github.com/wolfSSL/wolfssl/archive/refs/tags/${WOLFSSL_VERSION}.tar.gz" -O /tmp/wolfssl.tar.gz; \
    tar -C /tmp -xf /tmp/wolfssl.tar.gz; \
    cd /tmp/wolfssl-*; \
    ./autogen.sh; \
    ./configure --enable-distro; \
    make -j"$(nproc)"; \
    make install; \
    install -m 0644 wolfssl/options.h /usr/local/include/wolfssl/options.h; \
    ldconfig; \
    rm -rf /tmp/wolfssl*

WORKDIR /workspace
