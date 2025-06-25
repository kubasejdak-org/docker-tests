FROM ubuntu:22.04
LABEL maintainer="Kuba Sejdak (kuba.sejdak@gmail.com)"

# Settings.
ARG GCC_VERSION=11
ARG LABGRID_VERSION=25.0
ARG DEBIAN_FRONTEND=noninteractive
ARG LIBFAKETIME_URL=https://github.com/wolfcw/libfaketime.git

ENV CRYPTOGRAPHY_DONT_BUILD_RUST=1
ENV TZ=Europe/Warsaw
ENV UV_INSTALL_DIR=/usr/local/bin
ENV UV_TOOL_BIN_DIR=/usr/local/bin
ENV UV_TOOL_DIR=/usr/local/share/uv/tools

# Prepare APT for installing packages.
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo "APT::Install-Recommends \"false\";" >> /etc/apt/apt.conf && \
    echo "APT::Install-Suggests \"false\";" >> /etc/apt/apt.conf && \
    apt update && \
    apt upgrade -y && \
    apt install -y tzdata && \
    \
    # Create common project directories.
    mkdir -p -m 777 /var/log/ks && \
    \
    # Install git and SSH client.
    apt install -y git openssh-client && \
    git config --system --add safe.directory '*' && \
    \
    # Install common tools and libs.
    apt install -y ca-certificates bzip2 wget curl file sudo iproute2 lsof build-essential make libssl-dev libffi-dev \
    libgpiod-dev libiio-dev libsocketcan-dev gpiod spi-tools i2c-tools can-utils && \
    \
    # Install gcc, g++, gdb and lcov.
    apt install -y gcc gcc-${GCC_VERSION} g++ g++-${GCC_VERSION} gdb lcov && \
    \
    # Install python3 and uv.
    apt install -y python3 python3-pip python3-venv python-is-python3 python3-dev && \
    pip config set global.break-system-packages true && \
    curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=${UV_INSTALL_DIR} sh && \
    mkdir -p ${UV_TOOL_DIR} && \
    echo "UV_TOOL_DIR=${UV_TOOL_DIR}" | tee -a /etc/environment && \
    echo "UV_TOOL_BIN_DIR=${UV_TOOL_BIN_DIR}" | tee -a /etc/environment && \
    \
    # Install labgrid.
    uv tool install labgrid==${LABGRID_VERSION} && \
    labgrid-client && \
    \
    # Install pyserial and python-daemon.
    apt install -y python3-serial python3-daemon && \
    \
    # Install valgrind.
    apt install -y valgrind && \
    \
    # Install openocd.
    apt install -y openocd && \
    \
    # Install libfaketime.
    git clone ${LIBFAKETIME_URL} && \
    cd libfaketime && \
    FAKETIME_COMPILE_CFLAGS="-DFAKE_SETTIME" make -j$(nproc) && \
    make install && \
    cd - && \
    rm -rf libfaketime && \
    \
    # Cleanup image.
    apt autoremove -y && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*
