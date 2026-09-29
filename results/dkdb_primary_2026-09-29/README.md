# dKDB primary diagnostic results - 2026-09-29

This directory is the Git-friendly publication package for configuration
`2e219c54a929d16f0cc7` from `dkdb_overfitting_generalisation_diagnostic.ipynb`.

## Coverage

- 1,900 of 1,920 configured trajectories completed (98.96%).
- Seven datasets completed all 240 trajectories.
- PokerHand completed 220 of 240 trajectories.
- The missing PokerHand block is `repeat=1, fold=2`; the recorded failure is
  `UnboundLocalError: local variable 'resume' referenced before assignment`.
- All implementation and numerical preflight checks passed.

The PokerHand figures and aggregate rows therefore describe the available
220 trajectories and must not be presented as complete until the missing block
has been rerun.

## Included

- `configuration.json`: exact experiment configuration.
- `tables/`: compact audit, coverage, baseline, trajectory-summary, aggregate,
  sensitivity, and diagnostic files.
- `figures_pdf/`: 137 vector plots, one PDF per figure.

## Deliberately excluded from Git

- Processed dataset arrays and downloaded/cache data.
- Optimizer checkpoints and per-trajectory JSON cache files.
- `trajectory_candidates_long.csv` and `trajectory_results_long.csv` (large,
  mechanically reconstructible long-form exports).
- Duplicate PNG copies of the vector figures.
- Notebook cell outputs and Jupyter checkpoint files.

The complete local cache remains under `dkdb overfitting diagnostic cache/`,
which is ignored by Git. Run the notebook against that cache to regenerate the
long-form tables or image formats.
