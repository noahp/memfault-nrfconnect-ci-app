FROM ubuntu:22.04

# set noninteractive installation
ENV DEBIAN_FRONTEND=noninteractive

# install host packages for Zephyr builds. package list is from here:
# https://docs.zephyrproject.org/3.5.0/develop/getting_started/index.html#install-dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
  ccache \
  cmake \
  file \
  g++-multilib \
  gcc \
  gcc-multilib \
  git \
  gperf \
  make \
  ninja-build \
  python3-venv \
  wget \
  xz-utils

# install the Zephyr toolchain only for arm-zephyr-eabi
ARG ZEPHYR_SDK_VERSION=0.16.4
RUN wget -q https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${ZEPHYR_SDK_VERSION}/zephyr-sdk-${ZEPHYR_SDK_VERSION}_linux-x86_64_minimal.tar.xz \
  && mkdir -p /opt/toolchains \
  && tar xf zephyr-sdk-${ZEPHYR_SDK_VERSION}_linux-x86_64_minimal.tar.xz -C /opt/toolchains \
  && /opt/toolchains/zephyr-sdk-${ZEPHYR_SDK_VERSION}/setup.sh -c -t arm-zephyr-eabi \
  && rm zephyr-sdk-${ZEPHYR_SDK_VERSION}_linux-x86_64_minimal.tar.xz

# tell cmake where to find the Zephyr SDK cmake packages
ENV CMAKE_PREFIX_PATH /opt/toolchains

# create a Python virtual environment and install Python packages
RUN python3 -m venv /venv \
  && . /venv/bin/activate \
  && pip install --no-cache-dir \
    west==1.2.0

# fetch our target NCS version to make west update faster when we build
COPY west.yml /zephyr-workspace-cache/.manifest/west.yml

# retrieve a lighter version of the Zephyr workspace, which is enough to
# populate a cache for use when building. we'll need to re-generate this image
# if the manifest changes, to keep the benefit.
RUN . /venv/bin/activate \
  && cd /zephyr-workspace-cache \
  && west init -l .manifest \
  && west update --narrow --fetch-opt=--filter=tree:0

# install Python dependencies for the Zephyr workspace
RUN . /venv/bin/activate \
  && cd /zephyr-workspace-cache \
  && pip install --no-cache-dir -r zephyr/scripts/requirements.txt
