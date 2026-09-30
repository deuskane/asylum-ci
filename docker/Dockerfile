# Start distrib
FROM ghdl/ghdl:6.0.0-gcc-ubuntu-24.04

# System Tools
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
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

# Update bash environment
RUN echo export RISCV_PREFIX=riscv64-unknown-elf- >> ~/.bashrc

# Get tools for Compilation of toolchain for Picoblaze
RUN apt-get install -y --no-install-recommends \
      g++ \
      bison \
      flex \
      libboost-dev

# Clone and build the toolchain for Picoblaze
RUN git clone https://github.com/deuskane/asylum-utils-generators.git ~/asylum-utils-generators

# Build the toolchain for Picoblaze
RUN cd ~/asylum-utils-generators && \
    ./scripts/build_toolchain.sh -s -p -i /opt/ && \
    echo export PBCC_HOME=/opt >> ~/.bashrc && \
    echo export PICOASM_HOME=/opt >> ~/.bashrc

# Clean
RUN rm -rf ~/asylum-utils-generators && \
    apt-get purge -y --auto-remove \
      g++ \
      bison \
      flex \
      libboost-dev && \
    apt-get clean && rm -rf /var/lib/apt-get/lists/*

WORKDIR /workspace
