# APPLICATION.md — OpenFOAM (SPUMA) — occDrivAerStaticMesh

This document is a strict application overview. Optimization plans, hypotheses,
and conclusions belong in `planning/`, not here.

Task source: ISC26 SCC (In-Person Part) — "Getting Started with OpenFOAM task"
https://hpcadvisorycouncil.atlassian.net/wiki/spaces/HPCWORKS/pages/4314791937/Getting+Started+with+OpenFOAM+task+for+ISC26+SCC+In-Person+Part

## What the application is

The task uses **SPUMA**, a fork of OpenFOAM (Open-source Field Operation And
Manipulation), that builds and runs on CPUs, NVIDIA GPUs, and AMD GPUs from a
single code base. This project targets the **CPU-only** backend. Everyone uses
the *same* case, mesh, `fvSchemes`, `fvSolution`, and `simpleFoam` solver; only
the build backend and launch options differ.

The workload is the **DrivAer** external-aerodynamics case: a steady-state,
incompressible turbulent CFD simulation of a passenger-vehicle body on a
**65M-cell** static mesh, solved with the `simpleFoam` (SIMPLE) solver.

## Source repository and exact revision

- Repository: `https://gitlab-hpc.cineca.it/exafoam/spuma.git`
- Branch / tag: `0.2-v2412` (SPUMA `0.2-v2412`, an OpenFOAM-v2412 fork)
- Clone command (self-contained recursive):
  ```bash
  git clone --branch 0.2-v2412 --recursive \
      https://gitlab-hpc.cineca.it/exafoam/spuma.git SPUMA-0.2-v2412
  ```
- Case location in source: `tutorials/incompressible/simpleFoam/occDrivAerStaticMesh`
- The 65M mesh is **not** in git; download it from the Zenodo link in the case
  README and unpack into `constant/polyMesh`.

## Dependencies

CPU-only build requires:

- A C++14 compiler (**GCC**; `WM_COMPILER=Gcc`).
- An MPI library (`SYSTEMOPENMPI`; `mpicc` must be in the environment).
- Common build tools: `cmake`, `flex`, `bison`, `make`, `git`, `zlib`.
- Nothing GPU-specific (no CUDA/NVHPC/ROCm).
- No separate vanilla-OpenFOAM or ThirdParty build is required; SPUMA is
  self-contained and the build simply skips a missing `ThirdParty`.

## Build configuration (CPU)

`etc/prefs.sh` for the CPU backend:

```bash
export WM_COMPILER=Gcc
export WM_MPLIB=SYSTEMOPENMPI
export WM_PRECISION_OPTION=DP
export WM_LABEL_SIZE=32
export WM_COMPILE_OPTION=Opt
```

Build sequence:

```bash
module load gcc openmpi            # CPU compiler + MPI modules (adapt to cluster)
source etc/bashrc
./Allwmake -j $(nproc) -k -s -l
```

Verify the solver built:

```bash
ls $FOAM_APPBIN/simpleFoam
```

The stock solver/models, `simpleFoam`, the mesh, and `fvSchemes`/`fvSolution`
must remain unchanged. The only editable fields in
`system/include/caseDefinition` are `nCores`, `decompositionMethod`, and
`nHierarchical`. Compile-flag changes under `wmake/rules/` are allowed and must
be submitted as patch files.

## Key run configuration and inputs

- Run length: `endTime 1000;` in `system/controlDict` (tutorial default 4000).
- Decomposition: edit `system/include/caseDefinition`; `nCores` = number of MPI
  ranks (one rank per CPU core), and `nHierarchical` must multiply to it.
- `Allrun` pipeline: `decomposePar` → `restore0Dir` → `renumberMesh` →
  `potentialFoam` → `applyBoundaryLayer` → `simpleFoam`.
- Launcher/pool for CPU:
  ```bash
  poolType="dummyMemoryPool"   # dozens of ranks share host RAM
  poolSize="0"
  parEx="mpirun -np ${nProcs}" # or srun -n ${nProcs}
  ```
  SPUMA executables accept `-pool <type> -poolSize <GiB-per-rank>`; pool size is
  per rank. For CPU use `dummyMemoryPool` (or a small per-rank `poolSize`) to
  avoid exhausting host RAM.

## Correctness / validation metric

Mean drag coefficient `Cd` over iterations `[800, 1000]` read from
`postProcessing/forceCoeffsAll/*/coefficient.dat` (column 2 = `Cd`) must be
within **±1.5% of the reference value `0.2699`**. This reference was established
on the same setup and applies to both CPU and GPU SPUMA builds.

## Performance / headline metric

The **headline** runtime is the last `ClockTime = X s` printed after the
`Time = 1000` block in `logFiles/50_simpleFoam.*.log`. Setup phases
(`decomposePar`, `renumberMesh`, `potentialFoam`, `applyBoundaryLayer`) are **not**
counted. Report the `ClockTime` value plus the *Average wall-clock time per time
step* printed by the `FOwallClockTimeStatistics` function object.

## Expected output markers

- `logFiles/50_simpleFoam.*.log` containing a final `Time = 1000` block with a
  trailing `ClockTime = X s` line.
- `postProcessing/forceCoeffsAll/*/coefficient.dat` with `Cd` values; the mean
  over the last window must land within ±1.5% of `0.2699`.

## Baseline build and run command

Building:

```bash
module load gcc openmpi
source etc/bashrc
./Allwmake -j $(nproc) -k -s -l
ls $FOAM_APPBIN/simpleFoam   # expected marker
```

Running (CPU example), set at the top of `Allrun`:

```bash
poolType="dummyMemoryPool"
poolSize="0"
parEx="mpirun -np ${nProcs}"
```

then:

```bash
./Allrun
```

## Submission deliverables

Build script, run script, `50_simpleFoam.*.log`, the edited `caseDefinition`,
and a copy of `coefficient.dat`. A single result; no binaries.