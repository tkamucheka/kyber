![A lattice and an FPGA chip implementing masked Kyber512](.github/banner.svg)

# Kyber

FPGA hardware for the Kyber key encapsulation mechanism: the IP cores, the
systems built around them, and the software that drives experiments on them.

## Cores

| Directory | Core |
|-----------|------|
| [`ip/kyber512`](https://github.com/tkamucheka/Kyber512_INDCCA2_AXI_IP) | Kyber512 CCA-secure encapsulation and decapsulation, as an AXI IP |
| [`ip/kyber512-masked`](https://github.com/tkamucheka/Kyber512_INDCCA2_Masked_AXI_IP) | The same core with masking against side-channel analysis |

The cores are Verilog-2001 and are packaged for Vivado.

## Getting the Sources

The cores are git submodules:

```sh
git clone --recurse-submodules https://github.com/tkamucheka/kyber.git
```

In an existing clone, `git submodule update --init` fetches them.

`host/legacy` holds the host software of the original experiments. It is a
private repository and is skipped by the commands above; with access, fetch
it with:

```sh
git submodule update --init --checkout host/legacy
```

## Roadmap

Board design scripts, a parameterized security level, a bulk data interface,
the experiment software, and area and frequency optimization are planned in
[docs/roadmap.md](docs/roadmap.md).
