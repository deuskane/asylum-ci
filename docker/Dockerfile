# Start distrib
FROM ghdl/ghdl:6.0.0-gcc-ubuntu-24.04

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
