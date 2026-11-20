# asylum-ci

CI of the Asylum IPs:

- `docker/`: image `ghcr.io/deuskane/asylum-env` (GHDL, Yosys and its GHDL plugin, FuseSoC,
  toolchains)
- `.github/workflows/vhdl-ci.yml`: one FuseSoC target (`make <target>`) in the image
- `.github/workflows/badges.yml`: sim / lint pass and fail counts of a CI run, published as
  shields.io endpoint files (`sim.json`, `lint.json`) on the `badges` branch of the IP
