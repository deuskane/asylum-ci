# asylum-ci

CI of the Asylum IPs:

- `docker/`: image `ghcr.io/deuskane/asylum-env` (GHDL, Yosys and its GHDL plugin, sky130
  liberty `$ASIC_LIBERTY`, FuseSoC, toolchains)
- `.github/workflows/vhdl-ci.yml`: one FuseSoC target (`make <target>`) in the image
- `.github/workflows/badges.yml`: sim / lint / syn pass and fail counts of a CI run, published
  as shields.io endpoint files (`sim.json`, `lint.json`, `syn.json`) on the `badges` branch of
  the IP; an IP can add groups of jobs in `.github/badges.json`
  (`[{"name": "act3_rv32i", "label": "ACT3 RV32I", "prefix": "sim_compliance_act3_"}]`), one
  `<name>.json` each
