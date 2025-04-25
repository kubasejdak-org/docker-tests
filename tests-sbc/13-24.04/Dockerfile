FROM ubuntu:24.04
LABEL maintainer="Kuba Sejdak (kuba.sejdak@gmail.com)"

# Settings.
ARG GCC_VERSION=13
ARG LABGRID_VERSION=24.0.2
ARG DEBIAN_FRONTEND=noninteractive
ARG LIBFAKETIME_URL=https://github.com/wolfcw/libfaketime.git

ENV CRYPTOGRAPHY_DONT_BUILD_RUST=1
ENV PIPX_HOME=/opt/pipx
ENV PIPX_BIN_DIR=${PIPX_HOME}/bin
ENV PATH=${PIPX_BIN_DIR}:${PATH}:/root/.local/bin
ENV TZ=Europe/Warsaw

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
    apt install -y ca-certificates bzip2 wget file sudo iproute2 lsof build-essential make libssl-dev libffi-dev \
    libgpiod-dev libiio-dev libsocketcan-dev gpiod spi-tools i2c-tools can-utils && \
    \
    # Install gcc, g++, gdb and lcov.
    apt install -y gcc gcc-${GCC_VERSION} g++ g++-${GCC_VERSION} gdb lcov && \
    \
    # Install python3 and pipx.
    apt install -y python3 python3-pip python3-venv python-is-python3 python3-dev && \
    pip config set global.break-system-packages true && \
    mkdir -p -m 777 ${PIPX_HOME} && \
    mkdir -p -m 777 ${PIPX_BIN_DIR} && \
    pip install pipx && \
    \
    # Install labgrid.
    pipx install labgrid==${LABGRID_VERSION} --system-site-packages && \
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
