FROM ubuntu:24.04
LABEL maintainer="Kuba Sejdak (kuba.sejdak@gmail.com)"

# Settings.
ARG CMAKE_VERSION=3.29.3
ARG CONAN_VERSION=1.64.0
ARG GCC_VERSION=13
ARG LABGRID_VERSION=23.0.6
ARG PYSERIAL_VERSION=3.5
ARG PYTHON_DAEMON_VERSION=3.0.1
ARG VALGRIND_VERSION=3.23.0
ARG CMAKE_URL=https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-aarch64.sh
ARG CONAN_TARGET_PROFILE=gcc-${GCC_VERSION}
ARG DEBIAN_FRONTEND=noninteractive
ARG LIBFAKETIME_URL=https://github.com/wolfcw/libfaketime.git
ARG VALGRIND_URL=https://sourceware.org/pub/valgrind/valgrind-${VALGRIND_VERSION}.tar.bz2

ENV CONAN_USER_HOME=/opt/conan
ENV PIPX_HOME=/opt/pipx
ENV PIPX_BIN_DIR=${PIPX_HOME}/bin
ENV PATH=${PIPX_BIN_DIR}:${PATH}

# Prepare APT for installing packages.
RUN ln -snf /usr/share/zoneinfo/${CONTAINER_TIMEZONE} /etc/localtime && echo ${CONTAINER_TIMEZONE} > /etc/timezone && \
    echo "APT::Install-Recommends \"false\";" >> /etc/apt/apt.conf && \
    echo "APT::Install-Suggests \"false\";" >> /etc/apt/apt.conf && \
    apt update && \
    apt upgrade -y && \
    \
    # Create common project directories.
    mkdir -p -m 777 /var/log/ks && \
    \
    # Install common tools and libs.
    apt install -y bzip2 wget file sudo iproute2 lsof build-essential make libssl-dev libffi-dev libgpiod-dev \
    libiio-dev libsocketcan-dev gpiod spi-tools i2c-tools can-utils && \
    \
    # Install git and SSH client.
    apt install -y software-properties-common gnupg && \
    add-apt-repository ppa:git-core/ppa && \
    apt install -y git openssh-client && \
    git config --system --add safe.directory '*' && \
    \
    # Install gcc, g++ and lcov.
    add-apt-repository ppa:ubuntu-toolchain-r/test && \
    apt update && \
    apt install -y gcc-${GCC_VERSION} g++-${GCC_VERSION} gdb lcov && \
    update-alternatives --install /usr/bin/gcc-ar   gcc-ar      /usr/bin/aarch64-linux-gnu-gcc-ar-${GCC_VERSION} 10 && \
    update-alternatives --install /usr/bin/gcc      gcc         /usr/bin/aarch64-linux-gnu-gcc-${GCC_VERSION} 10 && \
    update-alternatives --install /usr/bin/g++      g++         /usr/bin/aarch64-linux-gnu-g++-${GCC_VERSION} 10 && \
    update-alternatives --install /usr/bin/ld       ld          /usr/bin/aarch64-linux-gnu-ld 10 && \
    update-alternatives --install /usr/bin/objcopy  objcopy     /usr/bin/aarch64-linux-gnu-objcopy 10 && \
    update-alternatives --install /usr/bin/ranlib   ranlib      /usr/bin/aarch64-linux-gnu-ranlib 10 && \
    update-alternatives --install /usr/bin/size     size        /usr/bin/aarch64-linux-gnu-size 10 && \
    update-alternatives --install /usr/bin/strip    strip       /usr/bin/aarch64-linux-gnu-strip 10 && \
    update-alternatives --install /usr/bin/gcov     gcov        /usr/bin/aarch64-linux-gnu-gcov-${GCC_VERSION} 10 && \
    \
    # Install python3 and pipx.
    apt install -y python3 python3-pip python3-venv python-is-python3 python3-dev && \
    pip config set global.break-system-packages true && \
    mkdir -p -m 777 ${PIPX_HOME} && \
    mkdir -p -m 777 ${PIPX_BIN_DIR} && \
    pip install pipx && \
    \
    # Install conan.
    mkdir -p -m 777 ${CONAN_USER_HOME} && \
    pipx install conan==${CONAN_VERSION} && \
    conan profile new default --detect && \
    conan profile update settings.compiler.libcxx=libstdc++11 default && \
    conan profile update env.CC=gcc default && \
    conan profile update env.CXX=g++ default && \
    ln -s $(which conan) /usr/bin/conan && \
    chmod -R 777 ${CONAN_USER_HOME} && \
    ln -s ${CONAN_USER_HOME}/.conan ${HOME}/.conan && \
    \
    # Install labgrid.
    pipx install labgrid==${LABGRID_VERSION} && \
    \
    # Install pyserial and python-daemon.
    pip install pyserial==${PYSERIAL_VERSION} && \
    pip install python-daemon==${PYTHON_DAEMON_VERSION} && \
    \
    # Install CMake.
    wget --no-check-certificate ${CMAKE_URL} && \
    sudo sh $(basename ${CMAKE_URL}) --skip-license --prefix=/usr && \
    rm $(basename ${CMAKE_URL}); \
    \
    # Compile valgrind and install libc6-dbg.
    wget --no-check-certificate ${VALGRIND_URL} && \
    tar -xf $(basename ${VALGRIND_URL}) && \
    cd valgrind-${VALGRIND_VERSION} && \
    ./configure && \
    make -j$(nproc) && \
    make install && \
    cd - && \
    rm $(basename ${VALGRIND_URL}) && \
    rm -rf valgrind-${VALGRIND_VERSION} && \
    apt install -y libc6-dbg && \
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
COPY ${CONAN_TARGET_PROFILE} ${CONAN_USER_HOME}/.conan/profiles
RUN ln -sf ${CONAN_USER_HOME}/.conan/profiles/${CONAN_TARGET_PROFILE} ${CONAN_USER_HOME}/.conan/profiles/target-default
