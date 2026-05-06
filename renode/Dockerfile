# wolfboot-ci-renode: Renode emulation environment for wolfBoot CI
ARG BASE_TAG=latest
FROM antmicro/renode:latest AS renode-src

ARG BASE_TAG=latest
FROM ghcr.io/wolfssl/wolfboot-ci-base:${BASE_TAG}

ENV DEBIAN_FRONTEND=noninteractive

ENV ARM_GCC_VERSION=15.2.rel1
ENV ARM_GCC_TARBALL=arm-gnu-toolchain-15.2.rel1-x86_64-arm-none-eabi.tar.xz
ENV ARM_GCC_DIR=arm-gnu-toolchain-15.2.rel1-x86_64-arm-none-eabi

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc-multilib \
    libc6-dev-i386 \
    libicu-dev \
    make \
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

RUN printf 'int main(void){return 0;}\n' > /tmp/arm-none-eabi-smoke.c \
    && arm-none-eabi-gcc --specs=nano.specs --specs=nosys.specs /tmp/arm-none-eabi-smoke.c -o /tmp/arm-none-eabi-smoke.elf \
    && rm -f /tmp/arm-none-eabi-smoke.c /tmp/arm-none-eabi-smoke.elf

# Install .NET 8 runtime (required by Renode >= 1.16)
RUN curl -sSL https://dot.net/v1/dotnet-install.sh -o /tmp/dotnet-install.sh \
    && chmod +x /tmp/dotnet-install.sh \
    && /tmp/dotnet-install.sh --channel 8.0 --runtime dotnet --install-dir /usr/share/dotnet \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
    && rm /tmp/dotnet-install.sh

COPY --from=renode-src /opt/renode /opt/renode

ENV PATH="/opt/renode:${PATH}"
RUN ln -s /opt/renode/renode /usr/bin/renode \
    && chmod +x /opt/renode/renode \
    && mkdir -p /tmp/test_results

WORKDIR /workspace
