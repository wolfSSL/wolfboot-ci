# m33mu-ci: Devuan testing CI image for Zephyr + m33mu
FROM docker.io/devuan/devuan:testing

ENV DEBIAN_FRONTEND=noninteractive

ENV ARM_GCC_VERSION=15.2.rel1
ENV ARM_GCC_TARBALL=arm-gnu-toolchain-15.2.rel1-x86_64-arm-none-eabi.tar.xz
ENV ARM_GCC_DIR=arm-gnu-toolchain-15.2.rel1-x86_64-arm-none-eabi

# ---- Base build tooling + deps (as provided) ----
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl wget xz-utils file \
    git build-essential pkg-config \
    autoconf automake libtool \
    cmake ninja-build \
    rustc cargo \
    python3 python3-pip python3-venv python3-full \
    gdb-multiarch \
    libtpms-dev vde2 \
    libcapstone-dev \
    libvdeplug-dev \
    vde-switch \
    libdw-dev libelf-dev \
    libncurses-dev \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

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

# ---- Zephyr SDK ----
#ARG ZEPHYR_SDK_VERSION=0.17.4
#ARG ZEPHYR_SDK_ARCH=linux-x86_64
#
#RUN set -eux; \
#    cd /tmp; \
#    wget -q "https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${ZEPHYR_SDK_VERSION}/zephyr-sdk-${ZEPHYR_SDK_VERSION}_${ZEPHYR_SDK_ARCH}.tar.xz"; \
#    wget -q "https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${ZEPHYR_SDK_VERSION}/sha256.sum"; \
#    grep "zephyr-sdk-${ZEPHYR_SDK_VERSION}_${ZEPHYR_SDK_ARCH}.tar.xz" sha256.sum | sha256sum -c -; \
#    mkdir -p /opt; \
#    tar -C /opt -xf "zephyr-sdk-${ZEPHYR_SDK_VERSION}_${ZEPHYR_SDK_ARCH}.tar.xz"; \
#    rm -f "zephyr-sdk-${ZEPHYR_SDK_VERSION}_${ZEPHYR_SDK_ARCH}.tar.xz" sha256.sum
#
#ENV ZEPHYR_TOOLCHAIN_VARIANT=zephyr
#ENV ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk-${ZEPHYR_SDK_VERSION}
#
## ---- Python venv for west (avoids PEP 668 externally-managed-environment) ----
#ENV VIRTUAL_ENV=/opt/venv
#ENV PATH="${VIRTUAL_ENV}/bin:${PATH}"
#
#RUN python3 -m venv "${VIRTUAL_ENV}" \
# && "${VIRTUAL_ENV}/bin/pip" install --no-cache-dir --upgrade pip setuptools wheel \
# && "${VIRTUAL_ENV}/bin/pip" install --no-cache-dir west


# ---- m33mu: clone, build, install ----
RUN set -eux; \
    git clone --depth 1 https://github.com/danielinux/m33mu.git /tmp/m33mu; \
    cmake -S /tmp/m33mu -B /tmp/m33mu/build; \
    cmake --build /tmp/m33mu/build -j"$(nproc)"; \
    if [ -f /tmp/m33mu/build/m33mu ]; then \
      install -m 0755 /tmp/m33mu/build/m33mu /usr/local/bin/m33mu; \
    fi; \
    rm -rf /tmp/m33mu

WORKDIR /workspace
