# Start distrib
FROM ghdl/ghdl:6.0.0-gcc-ubuntu-24.04

# System Tools
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3-pip git gcc-riscv64-unknown-elf make procps \
    && rm -rf /var/lib/apt-get/lists/*

# Python :FuseSoC, Couverture, Linter (VSG)
RUN pip3 install --no-cache-dir --break-system-packages fusesoc gcovr vsg

WORKDIR /workspace