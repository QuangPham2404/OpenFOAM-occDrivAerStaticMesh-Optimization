# results

Extracted structured results and the generated human-readable report.

- `metrics.csv` — structured source of truth; schema (columns/units/semantics)
  is decided with the user before the first sweep and documented here.
- `RESULTS.md` — generated from `metrics.csv` (not written by hand).
- `scripts/` — extraction scripts; raw `.o`/`.e` provenance lives in experiment
  `outputs/`.

This directory holds results, not optimization interpretation.