# Kyber Project Instructions

## Layout
* `ip/kyber512`: Kyber512 CCA-secure KEM core, packaged as an AXI IP (git submodule)
* `ip/kyber512-masked`: masked variant of the core (git submodule)
* `docs/roadmap.md`: project roadmap

## Code Style and Architecture
* Language Standard: Verilog-2001 for hardware; Tcl for Vivado scripts
* Naming Convention: camelCase for signals, variables, and functions; PascalCase
  for module names
* Changes to an IP core are committed in its own repository first; this
  repository then records the new submodule commit.
* Vivado scripts do not pin IP versions; they resolve them at run time
  (`get_ipdefs`), so they run on every supported Vivado release.

## Implementation Guidelines
* All documentation must be technical/academic.
* Never use emojis in documentation, comments, or commit messages.

## Repository Hygiene
* Do not create temporary files in the project tree. Use the system temp
  directory (or an agent scratch directory) and clean up afterwards.
* Do not create markdown files other than README files unless explicitly
  asked. No summaries, notes, reports, or changelogs written as standalone
  files; report results in the conversation instead.
