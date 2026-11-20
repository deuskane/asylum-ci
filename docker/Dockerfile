# GHDL image : same libghdl for the image and the yosys plugin
ARG GHDL_IMAGE=ghdl/ghdl:6.0.0-gcc-ubuntu-24.04
# Yosys of Ubuntu 24.04 (noble release pocket)
ARG YOSYS_VERSION=0.33-5build2
# ghdl-yosys-plugin : commit of the tag ghdl-v6.0.0
ARG GHDL_YOSYS_PLUGIN_COMMIT=07a30ed39fb6a078f1bf7e9e88ce9ed712380ec2

#======================================================
# Build stage : GHDL plugin for Yosys (ghdl.so)
# (only ghdl.so is copied in the image)
#======================================================
FROM ${GHDL_IMAGE} AS ghdl-yosys-plugin
ARG YOSYS_VERSION
ARG GHDL_YOSYS_PLUGIN_COMMIT

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      git \
      g++ \
      make \
      yosys=${YOSYS_VERSION} \
      yosys-dev=${YOSYS_VERSION}

RUN git clone https://github.com/ghdl/ghdl-yosys-plugin.git /ghdl-yosys-plugin && \
    cd /ghdl-yosys-plugin && \
    git checkout ${GHDL_YOSYS_PLUGIN_COMMIT} && \
    make GHDL=ghdl

#======================================================
# Start distrib
#======================================================
FROM ${GHDL_IMAGE}
ARG YOSYS_VERSION

#======================================================
# Run fusesoc and gcc tools for RISC-V
#======================================================
# System Tools
RUN apt update
RUN apt install -y software-properties-common
RUN add-apt-repository ppa:ubuntu-toolchain-r/test -y
RUN apt update
RUN apt install -y --no-install-recommends \
      python3-pip \
      git \
      gcc-riscv64-unknown-elf \
      make \
      procps \
      bsdextrautils

# Python :FuseSoC
RUN pip3 install --no-cache-dir --break-system-packages\
      fusesoc \
      hjson \
      jsonschema \
      prettytable

# Setup Environment
ENV RISCV_PREFIX=riscv64-unknown-elf-

#======================================================
# Debug tools : OpenOCD and GDB
# (OpenOCD + GDB co-simulation of the WardRV debug unit)
#======================================================
RUN apt install -y --no-install-recommends \
      openocd \
      gdb-multiarch

#======================================================
# Synthesis : Yosys and its GHDL plugin
# (resource counts of the IPs from the VHDL)
#======================================================
RUN apt install -y --no-install-recommends \
      yosys=${YOSYS_VERSION}

# yosys -m ghdl loads /usr/lib/yosys/plugins/ghdl.so (Ubuntu yosys)
COPY --from=ghdl-yosys-plugin /ghdl-yosys-plugin/ghdl.so /usr/lib/yosys/plugins/ghdl.so

# libghdl (plugin) looks for the VHDL libraries in its build path :
# give it the prefix of the ghdl command
ENV GHDL_PREFIX=/opt/ghdl/lib/ghdl

#======================================================
# Install toolschain for PicoBlaze
#======================================================
# Get tools for Compilation of toolchain for Picoblaze
RUN apt install -y --no-install-recommends \
      gcc-9 \
      g++ \
      bison \
      flex \
      libboost-dev

# Clone and build the toolchain for Picoblaze
RUN git clone https://github.com/deuskane/asylum-utils-generators.git ~/asylum-utils-generators

# Build the toolchain for Picoblaze
RUN cd ~/asylum-utils-generators && \
    export CC=gcc-9; \
    ./scripts/build_toolchain.sh -s -i /opt/

RUN cd ~/asylum-utils-generators && \
    ./scripts/build_toolchain.sh -p -i /opt/

ENV PBCC_HOME=/opt
ENV PICOASM_HOME=/opt

# Clean
RUN rm -rf ~/asylum-utils-generators && \
    apt-get purge -y --auto-remove \
      g++ \
      gcc-9 \
      bison \
      flex \
      libboost-dev && \
    apt-get clean && rm -rf /var/lib/apt-get/lists/*

WORKDIR /workspace
