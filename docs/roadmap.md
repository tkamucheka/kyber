# Roadmap

This repository brings the Kyber hardware together: the IP cores, the scripts
that build systems around them, and the software that drives experiments on
them. The work proceeds in the phases below, in order; each phase leaves the
repository buildable and its results reproducible.

## Starting Point

* **Cores.** `ip/kyber512` and `ip/kyber512-masked` implement Kyber512
  encapsulation and decapsulation; key generation is not implemented in
  hardware. Both are Verilog-2001 with Block Memory Generator instances,
  packaged as Vivado 2019.1 IP.
* **Interface.** Each core has two AXI4-Lite slaves (32-bit data, 5-bit
  addresses). Keys, ciphertexts, and shared secrets move one 32-bit register
  write or read at a time.
* **Systems.** The cores were evaluated in MicroBlaze systems on the VC707
  (XC7VX485T-2) at 100 MHz, built as Vivado 2019.1 projects with Xilinx SDK.
  A MicroBlaze application exposes the core over UART with a SimpleSerial
  command protocol.
* **Host software.** The scripts that drove the experiments from a host
  computer are not yet in any repository.

## Supported Tool Versions

Scripts support Vivado 2019.1 and later and pin no IP versions: each IP is
resolved at run time to the newest version that the running Vivado provides.
The software flow changes between releases (Xilinx SDK with `.hdf` hardware
descriptions up to 2019.1; Vitis with `.xsa` from 2019.2; the Vitis Python
interface replacing XSCT in recent releases), so one script selects the flow
by version and the rest of the build does not depend on it. Versions are
listed in the README as each flow is verified on them.

## Phase 1: Board Design Generators

Replace the Vivado projects of the original system repositories with Tcl
scripts that generate them.

* One script per board builds the block design (MicroBlaze, local memory,
  UART, clocking, reset, and the core), the constraints, and the bitstream,
  and exports the hardware description for the software build.
* Boards: VC707, reproducing the original systems, and the Arty A7-100T
  (XC7A100T-1).
* An out-of-context synthesis and implementation script for a core alone,
  with the clock period as an argument, reports utilization and timing. The
  original systems are constrained to 100 MHz, which reports whether that
  frequency is met but not the maximum frequency; the script finds the
  maximum by tightening the constraint.
* Baseline: utilization, maximum frequency, and encapsulation and
  decapsulation cycle counts of both cores, recorded before any change to the
  cores.

## Phase 2: Parameterized Security Level

* Parameterize the cores by `KYBER_K` (2, 3, 4) for the three security
  levels, deriving the noise parameters, compression widths, buffer sizes,
  and memory depths from it.
* Known-answer tests for each level, run in simulation and on hardware.
* The parameter set the cores implement is to be confirmed against the
  reference implementation: the noise sampler implements eta = 2 only, while
  the 768-byte ciphertext matches the round-3 Kyber512 parameters, which use
  eta1 = 3. The target specification, round-3 Kyber or FIPS 203 (ML-KEM),
  is decided in this phase.

## Phase 3: Interface and Runtime

* Replace register-at-a-time transfers with a bulk interface: an AXI4
  memory-mapped slave with burst transfers, or AXI4-Stream with an AXI DMA.
  The choice follows from measured transfer time against computation time.
* A completion interrupt in place of status polling.
* A driver library with encapsulation and decapsulation calls, used by both
  the experiment application and the tests.
* End-to-end runtime, including transfers, measured from software.

## Phase 4: Software Tooling

* The MicroBlaze application, moved from the original SDK workspaces, built
  by script against the generated hardware description.
* The host scripts that drive experiments over SimpleSerial, added to the
  repository.
* One command per experiment: build, program the board, run, and collect
  results.

## Phase 5: Area and Maximum Frequency

Ordered by expected effect. Each change is measured against the Phase 1
baseline and kept only if the known-answer tests pass.

* **Narrow memory ports.** Four memories are read 1024 to 4096 bits at a
  time. Their read width, not their contents (about 37 kbit in total), sets
  their block RAM count, at least 170 BRAM36 by port width alone. Reading one
  or a few coefficients at a time removes most of these block RAMs and the
  multiplexers that select coefficients from the wide words.
* **Keys and ciphertexts in block RAM.** About 25,600 bits of public key,
  secret key, and ciphertext are copied from block RAM into registers; they
  are read from block RAM where they are needed instead.
* **One Keccak core.** Four instances of a Keccak-f[1600] core computing two
  rounds per cycle become one shared instance computing one round per cycle,
  which also shortens its critical path.
* **One butterfly unit.** The NTT, the inverse NTT, polynomial
  multiplication, and the reductions each have their own Montgomery or
  Barrett multipliers. A single pipelined butterfly unit serves all of them,
  and multiplication by q = 3329 = 2^11 + 2^10 + 2^8 + 1 uses adders instead
  of DSP slices.
* **Pipelined NTT.** The NTT computes one butterfly every nine to ten cycles;
  a pipelined butterfly computes one per cycle or one every two.
* **Timing.** Synchronous resets in place of asynchronous ones, so that
  datapath registers can be placed in DSP and block RAM primitives; block RAM
  output registers enabled; DSP pipeline registers used.
* **Inferred memories.** Block Memory Generator instances become inferred
  memories, which removes version-specific IP from the cores.
* **Unused sources.** Superseded copies of modules (`_old`, `_backup`) and
  unused files are removed.

Target: both cores fit the Arty A7-100T with their MicroBlaze system and meet
100 MHz on it.

## Masked Core

Each phase applies to the masked core as well. Optimizations that share
hardware between computations, such as a single Keccak core or butterfly
unit, can combine shares in one circuit; they are applied to the masked core
only where they preserve the separation of shares.
