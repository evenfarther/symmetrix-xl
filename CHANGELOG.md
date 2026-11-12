# Changelog

All notable changes to Symmetrix-XL are documented in this file.

## Unreleased

- Fixed the Kokkos pair style on type-filtered neighbor lists, where
  `pair_style hybrid` or `hybrid/overlay` gives the pairs of some atom types to
  another sub-style and the list skips those atoms. `no_domain_decomposition`
  indexed each neighbor by its local atom instead of its receiver row, which
  read the wrong features or crashed whenever a skipped atom came before a
  MACE atom; it now maps atoms to receiver rows. `mpi_message_passing`, which
  needs every owned atom in its list, stops with an error naming the cause
  instead of writing past its receiver rows.
- Fixed the prepared `no_domain_decomposition` route of the Kokkos pair style
  (`streamed_edges direct`) after an atom type changes without a neighbor list
  build, as in `fix sgcmc`: it kept the graph built for the old types and
  returned the energy and forces of the old species. The pair style now
  compares the receivers' types with the graph's on each step and rebuilds the
  graph when one differs.
- Fixed the Kokkos pair style after a re-issued `pair_style` line changes its
  options following `pair_coeff`. LAMMPS passes such a line to the existing
  pair style, which kept the model loaded with the earlier options: a new
  `streamed_edges`, `profile` or `head` was ignored silently, and a change of
  the domain mode crashed. The next run now stops with an error; issue
  `pair_coeff` again after the change. An identical line, or one that changes
  only the electric field components, still needs none.
- Fixed `pair_coeff` of the Kokkos pair style terminating the process when
  the model cannot be loaded (an unreadable file, a malformed model or an
  unknown `head`). It now reports a LAMMPS error, which a Python `lammps` host
  can catch.
- Fixed both pair styles reading positions and returning energies in the
  wrong units, without a message, under unit styles other than `metal`. They
  now require `units metal` and stop with an error otherwise.
- Fixed the Kokkos pair style under `pair_style hybrid` or `hybrid/overlay`
  with `mpi_message_passing`, the default on more than one MPI rank: the
  wrapper kept the message sizes it had before `pair_coeff` set this style's,
  LAMMPS sized the host communication buffers from them, and the per-atom
  message passing wrote past those buffers (a crash in CPU Kokkos builds).
  `init_style()` now raises the wrapper's sizes to this style's.

## 0.1.2 - 2026-11-04

- Added an opt-in Apple Metal backend for FP32 standard-MACE execution on
  Apple silicon, with installation guidance, examples, and runtime
  qualification tools.
- Added native response and capacity execution for MACEField models, including
  electric-field observables, polarization, Born effective charges,
  polarizability, trusted execution-JIT cache handling, and streamed-edge
  coverage.
- Added source provenance to PEP 517 builds and expanded backend diagnostics,
  build qualification, and runtime checks.
- Added extractor support for forwarding `--num-spline-points` to
  MACE_Nonlinear exports.
- Expanded CPU and CUDA wheel builds and hardened release automation for
  reruns, ABI-tagged extensions, compound manylinux tags, and AVX-512 codegen
  checks.
- Refreshed user and developer documentation, paper benchmarks, citations, and
  benchmark guidance; removed superseded benchmark drivers.
- Improved CUDA execution and its qualification workflow.
- Hardened host OpenMP execution by blocking unsafe worker GEMMs, selecting the
  compatible OpenBLAS runtime, and fixing BLAS/LAPACK wiring.
- Model JSON format version 4 for every exported variant: the first readout and
  the H1->H2 skip are stored without the inverse of `interactions[1].linear_up`
  (`h1_affine_mode` `m0_composite` or `pre_linear_up`). Format-1/2 files still
  load with the old arithmetic and print a warning with an fp32 cancellation
  estimate computed from the file; `symmetrix_recompose_mace_json` converts
  them. Loading an old file does not opt it into the improved arithmetic:
  re-exporting the original MACE checkpoint to v4 is strongly recommended.
  Use `symmetrix_recompose_mace_json` only when the original checkpoint is not
  available.
- In the 192-atom MACE-MPA-0 water validation against the original fp64 e3nn
  model, a v0.1.2 v4 export measured maximum fp32 force deviations of
  `6.24e-5 eV/A` with 256 radial spline points and `1.98e-6 eV/A` with 512
  points. The v0.1.1 LAMMPS comparison in
  [Liu et al., arXiv:2610.23371v1](https://arxiv.org/abs/2610.23371v1)
  reported `1.6e-4 eV/A` for its legacy 256-point export; its scripts and input
  structure are archived in the
  [`mace-million` paper snapshot](https://github.com/ShiQiaoL/mace-million/tree/paper-d572103).
  The v4 fp32 result with 256 points is already small, and the corresponding
  fp64 256-point results agree (`6.15e-5 eV/A` here and `6.1e-5 eV/A` in the
  paper of Liu et al.). Because the fp32 error decreased while retaining the
  same 256-point grid, the larger legacy fp32 deviation was primarily caused by
  the inverse-based H1 affine arithmetic removed by format v4, rather than by
  the radial-grid density. Increasing the grid to 512 points further reduces
  interpolation error but is not required to address that legacy format issue.
  The benefit of a denser radial grid is workload-dependent and is generally
  more pronounced when structures sample short bonds, compressed environments,
  or close contacts, where radial derivatives vary rapidly. For those
  workloads, export with `--num-spline-points 512` and validate representative
  configurations.
- The serial `symmetrix/mace` pair style and the field-free `compute_H1` entry
  refuse MACEField models (the serial pair style crashed on them); MACEField
  runs through `symmetrix/mace/kk electric_field Ex Ey Ez` or the field entries.
- Fixed the LAMMPS pair styles' `mpi_message_passing` and
  `no_mpi_message_passing` modes, which omitted the first-interaction residual
  of residual-first models.

## 0.1.1 - 2026-09-22

- Added automatic installation of architecture-qualified accelerator backends.
- Improved LAMMPS MPI execution, including fixed-workspace and FP64 support.
- Hardened periodic neighbor-graph indexing for large flattened edge arrays.
- Updated public documentation and qualified performance summaries.

## 0.1.0 - 2026-09-16

Initial public Symmetrix-XL release.

- Added streamed-edge MACE and MACEField execution for reduced graph workspace.
- Added model-specialized host, CUDA, and HIP execution with runtime-compiled
  artifacts.
- Added separately installable CPU, CUDA, and HIP backend packages with runtime
  diagnostics.
- Added ASE calculators, ensemble evaluation, prediction heads, and LAMMPS pair
  styles.
- Added reproducible correctness, capacity, performance, and backend
  qualification harnesses.
