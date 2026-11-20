# GHDL image : same libghdl for the image and the yosys plugin
ARG GHDL_IMAGE=ghdl/ghdl:6.0.0-gcc-ubuntu-24.04
# Yosys of Ubuntu 24.04 (noble release pocket)
ARG YOSYS_VERSION=0.33-5build2
# ghdl-yosys-plugin : commit of the tag ghdl-v6.0.0
ARG GHDL_YOSYS_PLUGIN_COMMIT=07a30ed39fb6a078f1bf7e9e88ce9ed712380ec2
# asylum-utils-generators : sources of the PicoBlaze toolchain
ARG ASYLUM_UTILS_GENERATORS_COMMIT=bc32be8eaf50163bc04bfc48cd1604c9f30e7fc8
# sky130_fd_sc_hd standard cells (FOSSi Foundation, Apache-2.0)
ARG SKY130_FD_SC_HD_COMMIT=9cb2d7cb8ed4619094263614039a61b6b2d22a88

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
# Build stage : toolchain for PicoBlaze (SDCC and picoasm)
# (only the install directory /opt/bin, /opt/share is copied in the image)
#======================================================
FROM ${GHDL_IMAGE} AS picoblaze-toolchain
ARG ASYLUM_UTILS_GENERATORS_COMMIT

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      git \
      make \
      gcc-9 \
      g++ \
      bison \
      flex \
      libboost-dev

RUN git clone https://github.com/deuskane/asylum-utils-generators.git /asylum-utils-generators && \
    cd /asylum-utils-generators && \
    git checkout ${ASYLUM_UTILS_GENERATORS_COMMIT}

# SDCC is built with gcc-9 (old sources)
RUN cd /asylum-utils-generators && \
    CC=gcc-9 ./scripts/build_toolchain.sh -s -i /opt && \
    ./scripts/build_toolchain.sh -p -i /opt

#======================================================
# Start distrib
#======================================================
FROM ${GHDL_IMAGE}
ARG YOSYS_VERSION
ARG SKY130_FD_SC_HD_COMMIT

#======================================================
# Run fusesoc and gcc tools for RISC-V
#======================================================
# System Tools
# (ca-certificates : https for git and fusesoc library add)
# (apt lists removed in the same RUN : not kept in the layer)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      python3-pip \
      git \
      gcc-riscv64-unknown-elf \
      make \
      procps \
      bsdextrautils && \
    rm -rf /var/lib/apt/lists/*

# Python :FuseSoC
RUN pip3 install --no-cache-dir --break-system-packages \
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
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      openocd \
      gdb-multiarch && \
    rm -rf /var/lib/apt/lists/*

#======================================================
# Synthesis : Yosys and its GHDL plugin
# (resource counts of the IPs from the VHDL)
#======================================================
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      yosys=${YOSYS_VERSION} && \
    rm -rf /var/lib/apt/lists/*

# yosys -m ghdl loads /usr/lib/yosys/plugins/ghdl.so (Ubuntu yosys)
COPY --from=ghdl-yosys-plugin /ghdl-yosys-plugin/ghdl.so /usr/lib/yosys/plugins/ghdl.so

# libghdl (plugin) looks for the VHDL libraries in its build path :
# give it the prefix of the ghdl command
ENV GHDL_PREFIX=/opt/ghdl/lib/ghdl

#======================================================
# ASIC liberty : SkyWater sky130_fd_sc_hd, typical corner
# (yosys synthesis on open-source standard cells)
#======================================================
ARG SKY130_FD_SC_HD_URL=https://raw.githubusercontent.com/fossi-foundation/skywater-pdk-libs-sky130_fd_sc_hd/${SKY130_FD_SC_HD_COMMIT}

ADD --chmod=644 --checksum=sha256:8e78e14442062dba34d414fca6490b2f6b96038d4510d1438ca44fee31487135 \
    ${SKY130_FD_SC_HD_URL}/timing/sky130_fd_sc_hd__tt_025C_1v80.lib /opt/asic/sky130_fd_sc_hd__tt_025C_1v80.lib
ADD --chmod=644 --checksum=sha256:cfc7749b96f63bd31c3c42b5c471bf756814053e847c10f3eb003417bc523d30 \
    ${SKY130_FD_SC_HD_URL}/LICENSE /opt/asic/sky130_fd_sc_hd.LICENSE

ENV ASIC_LIBERTY=/opt/asic/sky130_fd_sc_hd__tt_025C_1v80.lib

#======================================================
# Toolchain for PicoBlaze (build stage picoblaze-toolchain)
#======================================================
COPY --from=picoblaze-toolchain /opt/bin   /opt/bin
COPY --from=picoblaze-toolchain /opt/share /opt/share

ENV PBCC_HOME=/opt
ENV PICOASM_HOME=/opt

WORKDIR /workspace
