# RTEMS toolchain and the xilinx_zynq_a9_qemu BSP, built from the heads of the
# RTEMS release branch. Takes too long to build in every test run, so it is
# published by `.github/workflows/toolchain.yaml` and pulled by the tests.

FROM ubuntu:24.04 AS build

RUN apt-get update && apt-get install -y --no-install-recommends \
        bison \
        bzip2 \
        ca-certificates \
        curl \
        flex \
        g++ \
        gcc \
        git \
        install-info \
        libncurses-dev \
        make \
        patch \
        pax \
        python3 \
        python3-dev \
        python-is-python3 \
        texinfo \
        unzip \
        xz-utils \
        zlib1g-dev

ARG RTEMS_VERSION=6
ENV PREFIX=/opt/rtems/$RTEMS_VERSION
ENV PATH=$PREFIX/bin:$PATH

RUN git clone --depth 1 --branch "$RTEMS_VERSION" \
        https://gitlab.rtems.org/rtems/tools/rtems-source-builder.git /src/rsb && \
    cd /src/rsb/rtems && \
    ../source-builder/sb-set-builder --prefix="$PREFIX" --jobs="$(nproc)" \
        "$RTEMS_VERSION/rtems-arm" && \
    git -C /src/rsb rev-parse HEAD > "$PREFIX/rsb.commit" && \
    rm -rf /src/rsb

# The POSIX API is off by default; libc exposes signals and timers from it.
# A polled console makes write() synchronous, so no output is still queued
# in termios when a test exits.
RUN git clone --depth 1 --branch "$RTEMS_VERSION" \
        https://gitlab.rtems.org/rtems/rtos/rtems.git /src/rtems && \
    cd /src/rtems && \
    ./waf bspdefaults --rtems-bsps=arm/xilinx_zynq_a9_qemu > config.ini && \
    sed -i -e 's/^RTEMS_POSIX_API = False/RTEMS_POSIX_API = True/' \
        -e 's/^ZYNQ_CONSOLE_USE_INTERRUPTS = True/ZYNQ_CONSOLE_USE_INTERRUPTS = False/' \
        config.ini && \
    grep -q '^RTEMS_POSIX_API = True' config.ini && \
    grep -q '^ZYNQ_CONSOLE_USE_INTERRUPTS = False' config.ini && \
    ./waf configure --prefix="$PREFIX" && \
    ./waf && \
    ./waf install && \
    git rev-parse HEAD > "$PREFIX/rtems.commit" && \
    rm -rf /src/rtems

# `poll`, `select` and the socket API live in libbsd, not in the kernel.
RUN git clone --depth 1 --branch "$RTEMS_VERSION" \
        https://gitlab.rtems.org/rtems/tools/rtems-source-builder.git /src/rsb && \
    cd /src/rsb/rtems && \
    ../source-builder/sb-set-builder --prefix="$PREFIX" --jobs="$(nproc)" \
        --host="arm-rtems$RTEMS_VERSION" --with-rtems-bsp=arm/xilinx_zynq_a9_qemu \
        "$RTEMS_VERSION/rtems-libbsd" && \
    rm -rf /src/rsb

FROM ubuntu:24.04
ARG RTEMS_VERSION=6
COPY --from=build /opt/rtems /opt/rtems
ENV PATH=/opt/rtems/$RTEMS_VERSION/bin:$PATH
