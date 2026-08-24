# OpenFOAM (SPUMA) occDrivAerStaticMesh — HPC Optimization

Optimization of the **ISC26 SCC OpenFOAM task** (CPU-only backend) on two HPC
clusters: **Aspire2a** and **Gadi**. The project follows the reusable Agent HPC
Optimization Workflow Pack in `workflow/`.

Workload: steady-state incompressible CFD of the **DrivAer** vehicle body on a
65M-cell static mesh, solved with SPUMA's `simpleFoam` solver (fork of
OpenFOAM-v2412). The optimization goal is to minimize the `simpleFoam`
wall-clock time and time-per-step while keeping the mean drag coefficient
within the specified validation tolerance.

## Repository layout

```
workflow/     reusable workflow rules, SSH/execution policy, and templates
builds/       source, build scripts, and build evidence (per build variant)
experiments/  run configurations, PBS scripts, and raw outputs (per run)
planning/     optimization hypotheses, priorities, and authorized analysis
results/      structured metrics (metrics.csv) and generated reports
progress/     dated session handoff notes
scripts/      reusable/probing scripts and their outputs
```

## Where to start

1. `AGENTS.md` — active application, cluster, and project permissions.
2. `APPLICATION.md` — application overview, build/run commands, and validation
   criteria.
3. `workflow/` — the operational rules; read the numbered files in order before
   acting.

See the workflow pack for the full build, run, result-logging, and analysis
lifecycle.