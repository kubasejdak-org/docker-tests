FROM ubuntu:24.04
LABEL maintainer="Kuba Sejdak (kuba.sejdak@gmail.com)"

# Settings.
ARG CMAKE_VERSION=3.31.0
ARG CONAN_VERSION=2.10.2
ARG GCC_VERSION=13
ARG LABGRID_VERSION=24.0.2
ARG CMAKE_URL=https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-aarch64.sh
ARG CONAN_TARGET_PROFILE=gcc-${GCC_VERSION}
ARG DEBIAN_FRONTEND=noninteractive
ARG LIBFAKETIME_URL=https://github.com/wolfcw/libfaketime.git

ENV CONAN_HOME=/opt/conan/.conan2
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
    apt install -y bzip2 wget file sudo iproute2 lsof build-essential make libssl-dev libffi-dev libgpiod-dev \
    libiio-dev libsocketcan-dev gpiod spi-tools i2c-tools can-utils && \
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
    # Install conan.
    mkdir -p -m 777 ${CONAN_HOME} && \
    pipx install conan==${CONAN_VERSION} && \
    conan profile detect && \
    ln -s $(which conan) /usr/bin/conan && \
    chmod -R 777 ${CONAN_HOME} && \
    \
    # Install labgrid.
    pipx install labgrid==${LABGRID_VERSION} --system-site-packages && \
    labgrid-client && \
    \
    # Install pyserial and python-daemon.
    apt install -y python3-serial python3-daemon && \
    \
    # Install CMake.
    wget --no-check-certificate ${CMAKE_URL} && \
    sudo sh $(basename ${CMAKE_URL}) --skip-license --prefix=/usr && \
    rm $(basename ${CMAKE_URL}); \
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
    apt remove -y software-properties-common gnupg && \
    apt autoremove -y && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# Add conan profile.
COPY ${CONAN_TARGET_PROFILE} ${CONAN_HOME}/profiles
RUN ln -sf ${CONAN_HOME}/profiles/${CONAN_TARGET_PROFILE} ${CONAN_HOME}/profiles/target-default
