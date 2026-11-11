# wolfboot-ci-arm: ARM cross-compilation + clang for wolfBoot CI
ARG BASE_TAG=latest
FROM ghcr.io/wolfssl/wolfboot-ci-base:${BASE_TAG}

ENV DEBIAN_FRONTEND=noninteractive

ENV ARM_GCC_VERSION=15.2.rel1
ENV ARM_GCC_TARBALL=arm-gnu-toolchain-15.2.rel1-x86_64-arm-none-eabi.tar.xz
ENV ARM_GCC_DIR=arm-gnu-toolchain-15.2.rel1-x86_64-arm-none-eabi

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc-aarch64-linux-gnu \
    libc6-dev-arm64-cross \
    gnu-efi \
    clang llvm lld \
    qemu-system-arm \
    check \
    cppcheck \
    gdb-multiarch \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

RUN set -eux; \
    cd /tmp; \
    wget -q "https://armkeil.blob.core.windows.net/developer/files/downloads/gnu/${ARM_GCC_VERSION}/binrel/${ARM_GCC_TARBALL}"; \
    mkdir -p /opt/toolchains; \
    tar -C /opt/toolchains -xf "${ARM_GCC_TARBALL}"; \
    ln -sf "/opt/toolchains/${ARM_GCC_DIR}/bin/arm-none-eabi-"* /usr/local/bin/; \
    rm -f "${ARM_GCC_TARBALL}"

ENV PATH="/opt/toolchains/${ARM_GCC_DIR}/bin:${PATH}"

# ---- qemu-user 11.0.3 (pinned, static) ------------------------------------
# The distro's qemu-user (Devuan testing, 11.1.2+ds-3) has a linux-user TCG
# regression: with a large guest main-thread stack (the glibc 8MB default),
# .text code fetches SIGBUS mid-execution - verified on the MC/DC arm lane,
# where 384KB-8MB guest stacks all trip it (weekly builds #22 and #25).
# 11.0.3 passes the full lane workload with the default 8MB stack. Pinned
# from the official qemu.org tarball and built static, so the runtime needs
# no dependencies. If you bump the version, re-run the MC/DC arm lane
# workload (testwolfcrypt KAT + unit.test rsa/ecc/dh under -fcoverage-mcdc)
# before shipping.
ARG QEMU_VERSION=11.0.3
ARG QEMU_SHA256=da5fcffc32762820568b828ed430a728864d34d50b6d2f30358597760cbb0523

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libglib2.0-dev libpixman-1-dev zlib1g-dev libcap-dev; \
    cd /tmp; \
    wget -q "https://download.qemu.org/qemu-${QEMU_VERSION}.tar.xz"; \
    echo "${QEMU_SHA256}  qemu-${QEMU_VERSION}.tar.xz" | sha256sum -c -; \
    tar xf "qemu-${QEMU_VERSION}.tar.xz"; \
    cd "qemu-${QEMU_VERSION}"; \
    ./configure --target-list=arm-linux-user --static \
        --prefix=/opt/qemu-${QEMU_VERSION} --disable-slirp --disable-docs; \
    make -j"$(nproc)"; \
    make install; \
    cd /tmp; \
    rm -rf "qemu-${QEMU_VERSION}" "qemu-${QEMU_VERSION}.tar.xz"; \
    apt-get remove -y libglib2.0-dev libpixman-1-dev zlib1g-dev libcap-dev; \
    apt-get autoremove -y; \
    rm -rf /var/lib/apt/lists/*; \
    ln -sf "/opt/qemu-${QEMU_VERSION}/bin/qemu-arm" /usr/local/bin/qemu-arm; \
    qemu-arm --version | grep -q "${QEMU_VERSION}"

RUN printf 'int main(void){return 0;}\n' > /tmp/arm-none-eabi-smoke.c \
    && arm-none-eabi-gcc --specs=nano.specs --specs=nosys.specs /tmp/arm-none-eabi-smoke.c -o /tmp/arm-none-eabi-smoke.elf \
    && rm -f /tmp/arm-none-eabi-smoke.c /tmp/arm-none-eabi-smoke.elf

WORKDIR /workspace
