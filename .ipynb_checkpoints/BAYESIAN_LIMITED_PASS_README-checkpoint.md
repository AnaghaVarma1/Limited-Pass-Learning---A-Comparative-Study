# Limited-pass NB, KDB and SKDB implementation

## Definition of a Data Pass

One data pass is one complete sequential traversal of the training split,
during which each training observation is read exactly once for a specified
learning stage. Validation/test evaluation is not counted as a training pass.
Computation performed from already accumulated in-memory statistics does not
add another pass.

Under the current preprocessing convention:

- Naive Bayes uses 1 learner pass to estimate class and feature-conditional
  sufficient statistics. With MDL preprocessing, its total is **2 passes**.
- KDB uses 2 learner passes: structure MI/CMI statistics, then sparse CPT
  parameters. With preprocessing, its total is **3 passes**.
- SKDB uses 3 learner passes: KDB-family structure statistics, complete sparse
  counts through `K_MAX`, then incremental leave-one-out selection of `k` and
  the MI-ranked feature-prefix length. With preprocessing, its total is
  **4 passes**.
- Supervised MDL discretisation uses 1 additional training pass. Cut points are
  fitted from `X_train/y_train` only and are then applied on the fly. Missing
  and non-finite values use a dedicated state.

## Scientific choices

The code preserves the notebooks' stratified 60/20/20 split and
`random_state=5`. Skin's positive class remains `"2"`; MiniBooNE's remains
`"True"`. Every classifier uses the same Fayyad-Irani-style supervised MDL
states, string class encoding, sparse categorical count tables, additive
Laplace smoothing (`alpha=1.0`), unseen-context rule, log-space probability
accumulation, and exact scikit-learn ROC-AUC/Brier implementations.

`KDB_K=1` is a configurable **provisional predeclared setting**, not a result of
validation or test tuning. It must be confirmed before final experiments.
`SKDB K_MAX=5` is configurable. SKDB performs its selection internally on the
training split using incremental LOOCV probability RMSE; it does not use
validation or test labels.

Pass 3 uses virtual count discounting. For the held-out row's actual class, its
contribution is subtracted from every relevant probability lookup. This is
exactly equivalent to mutating and restoring the sparse counters, while being
safer and faster. A count fingerprint before/after the pass proves that the
full fitted model remains unchanged.

## Files

- `src/bnc/discretization.py`: supervised MDL cut points and missing state.
- `src/bnc/pass_tracking.py`: verified row iterator and audit records.
- `src/bnc/core.py`: MI/CMI, sparse CPTs, smoothing and stable probabilities.
- `src/bnc/naive_bayes.py`: shared `k=0` one-pass classifier.
- `src/bnc/kdb.py`: fixed-`k` two-pass KDB.
- `src/bnc/skdb.py`: faithful three-pass incremental-LOOCV SKDB.
- `src/bnc/metrics.py`: common external metrics.
- `src/bnc/experiment.py`: results, CSV, pass audit and point-only figures.
- `scripts/run_bayesian_experiment.py`: Skin/MiniBooNE opt-in runner.
- `tests/test_bnc.py`: pass, probability, equivalence, structure and restoration
  checks.

## Running

The full experiment is deliberately gated:

```powershell
python scripts/run_bayesian_experiment.py --dataset skin --execute
```

This evaluates validation and writes `bayesian_limited_pass_results.csv`,
`bayesian_pass_audit.csv`, `pass_vs_roc_auc.png`, and
`pass_vs_probability_rmse.png` to the existing dataset output folder. Test data
is evaluated only when `--run-final-test` is added after procedures are frozen.

Run small correctness tests first:

```powershell
python -m unittest discover -s tests -v
```

## Performance and memory note

Counts are sparse and no Cartesian CPT tensor is allocated. Structure learning
still requires pairwise state/class counts (`O(d^2)` feature pairs), and SKDB
selection requires `O(N * K_MAX * d * classes)` arithmetic. MiniBooNE should
therefore be run only after the Skin experiment is reviewed. The runner prints
progress and a best-effort final-statistics-memory estimate. Raw pair counters
are released after the MI/CMI structure has been constructed. No new dependency
was added beyond NumPy, pandas, matplotlib and scikit-learn already used by the
notebooks.

The old RF/XGBoost tree/round values remain algorithm-native nominal units.
They are not mixed into the new genuine full-data-pass plots.
