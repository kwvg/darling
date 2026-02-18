# TODO: switch to trixie when LLVM APT stops using deprecated SHA1
FROM debian:bullseye-slim

ENV APT_ARGS="-y --no-install-recommends --no-upgrade"\
    DEBIAN_FRONTEND="noninteractive" \
    TZ="UTC"

# Install packages needed for scripts to work
RUN set -ex; \
    apt-get update && apt-get install ${APT_ARGS} \
    bash \
    ca-certificates \
    curl \
    git \
    gnupg \
    gpg \
    lsb-release \
    software-properties-common \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Install toolchain
ARG LLVM_VERSION=21
RUN set -ex; \
    echo "Installing LLVM and Clang ${LLVM_VERSION}..."; \
    curl -sL https://apt.llvm.org/llvm.sh | bash -s -- "${LLVM_VERSION}"; \
    echo "Installing additional packages..."; \
    apt-get update && apt-get install ${APT_ARGS} \
    "libc++-${LLVM_VERSION}-dev" \
    "libc++abi-${LLVM_VERSION}-dev" \
    "libclang-${LLVM_VERSION}-dev" \
    "libclang-rt-${LLVM_VERSION}-dev" \
    "lld-${LLVM_VERSION}" \
    "llvm-${LLVM_VERSION}-dev"; \
    rm -rf /var/lib/apt/lists/*; \
    echo "Setting defaults..."; \
    llvmUpdAltArgs="update-alternatives --install /usr/bin/llvm-config llvm-config /usr/bin/llvm-config-${LLVM_VERSION} 100"; \
    for binName in clang clang++ clangd dsymutil lld lldb lldb-server llvm-ar llvm-cov llvm-nm llvm-objdump llvm-ranlib llvm-strip run-clang-tidy; do \
        llvmUpdAltArgs="${llvmUpdAltArgs} --slave /usr/bin/${binName} ${binName} /usr/bin/${binName}-${LLVM_VERSION}"; \
    done; \
    for binName in ld64.lld ld.lld lld-link wasm-ld; do \
        llvmUpdAltArgs="${llvmUpdAltArgs} --slave /usr/bin/${binName} ${binName} /usr/bin/lld-${LLVM_VERSION}"; \
    done; \
    sh -c "${llvmUpdAltArgs}";
# LD_LIBRARY_PATH is empty by default, this is the first entry
ENV LD_LIBRARY_PATH="/usr/lib/llvm-${LLVM_VERSION}/lib"

# Install dependencies needed to build Darling
RUN set -ex; \
    apt-get update && apt-get install ${APT_ARGS} \
    bison \
    ccache \
    cmake \
    flex \
    git-lfs \
    libavcodec-dev \
    libavformat-dev \
    libbsd-dev \
    libc6-dev \
    libcairo2-dev \
    libcap2-bin \
    libcurl4-openssl-dev \
    libdbus-1-dev \
    libedit-dev \
    libegl1-mesa-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libfuse-dev \
    libgif-dev \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    libpulse-dev \
    libssl-dev \
    libswresample-dev \
    libtiff5-dev \
    libudev-dev \
    libvulkan-dev \
    libxcursor-dev \
    libxkbfile-dev \
    libxml2-dev \
    libxrandr-dev \
    pkg-config \
    xz-utils \
    && rm -rf /var/lib/apt/lists/*

ARG USER_ID=1000 \
    GROUP_ID=1000
ENV CCACHE_DIR="/home/darling/.cache/ccache"
RUN set -ex; \
    getent group ${GROUP_ID} || groupadd -g ${GROUP_ID} darling; \
    useradd -u ${USER_ID} -g ${GROUP_ID} -s /bin/bash -m -d /home/darling darling; \
    mkdir -p /home/darling/.cache/ccache /home/darling/.config/gdb /src/darling; \
    echo "add-auto-load-safe-path /usr/lib/llvm-${LLVM_VERSION}/lib" | tee /home/darling/.config/gdb/gdbinit; \
    chown ${USER_ID}:${GROUP_ID} -R /home/darling /src;

WORKDIR /src/darling

USER darling
