One-liner

Early-stage surrogate that predicts numeric dent resistance and binary oil-canning risk from a few designer-available panel numbers so you get rapid, actionable feedback before expensive Class‑A surfacing and FEA.
Value proposition

Let designers triage and iterate hood-panel concepts in minutes (local estimate + oil‑can flag) instead of waiting weeks for full FEA, reducing cycle time and simulation cost.
What it’s trying to solve

Bridge the gap between rough early geometry and expensive, slow FEA: identify likely dent-prone or oil‑can‑prone designs early so engineering focuses full simulations on the risky subset.
Objective

Produce two outputs from simple inputs:
Dent resistance (regression): predicted force (N) to cause a permanent dent.
Oil‑canning (classification): binary label 1 = oil‑cans (pop in/out) so designers can avoid unstable panels.
Technical details (concise)

Dataset: synthetic, physics-inspired generator (N=1200 samples; cleaned → 1183 rows).
Input features: thickness_mm, panel_span_mm, curvature_1_per_m, yield_strength_mpa, youngs_modulus_gpa.
Synthetic formulas: explicit physics-like relationships for dent_force_N and oilcan_margin with added Gaussian noise to simulate scatter.
Data cleaning: simulated unit-slip and missing-value injection, then row filtering/dropping.
Train/test: single random 80/20 train-test split (random_state=42).
Models: scikit-learn HistGradientBoostingRegressor for dent; HistGradientBoostingClassifier for oil‑canning.
Explainability & checks: permutation feature importance, out‑of‑distribution test (retrain on narrow thickness band and compare inside vs outside error).
Artifacts saved: CSV dataset + three plots (predicted vs actual, feature importance, in/out-of-distribution) to the OUT folder.
Metrics (from the notebook run)

Dataset: initial N=1200 → after cleaning 1183 rows.
Dent regression:
MAE = 23.9 N
Baseline MAE (guess average) = 125.8 N
RMSE = 34.6 N
R2 = 0.954
Oil‑canning classification:
Accuracy = 0.916 (baseline always-majority = 0.540)
Precision = 0.929
Recall = 0.914
ROC‑AUC = 0.979
Confusion matrix [rows=true, cols=pred]: [[100, 9], [11, 117]]
OOD (honest-limit) test:
Dent model trained on thickness 0.75–1.00 mm:
error INSIDE training range = 28.0 N
error OUTSIDE training range = 61.7 N (trust collapses outside training range)
Feature importances (permutation importance mean):
thickness_mm: 0.688
panel_span_mm: 0.684
yield_strength_mpa: 0.324
curvature_1_per_m: 0.237
youngs_modulus_gpa: -0.001
What kind of ML model?

Gradient-boosted decision-tree based ensembles (scikit-learn HistGradientBoostingRegressor and HistGradientBoostingClassifier). Good fit for small-to-medium tabular datasets, low preprocessing required.
Limitations & risks (must-read)

Synthetic training data only: results demonstrate feasibility but are not validated on real FEA or experimental data — do not use for product decisions until retrained/validated on real sim/physical outputs.
Single random split (no CV) → optimistic/fragile error estimates.
Hard-coded, non-portable output path (C:/Users/...). Notebook filename in repo is nonstandard; portability/tooling issues.
No uncertainty quantification or calibrated prediction intervals — regression point estimates lack error bounds for decision-making.
OOD detection is rudimentary (single thickness-band test); production needs stronger OOD/uncertainty checks.
No model persistence, experiment logging, or CI; no reproducible pipeline for retraining or audit.
Potential label/feature leakage not tested across geometry families — real-world deployment should split by geometry family to avoid overly optimistic results.
No hyperparameter tuning shown.
Readout / Ready-to-copy summary (for README or stakeholder slide)

One-liner: Early surrogate to predict dent force and oil‑canning risk from simple panel geometry/material numbers so designers can triage designs before expensive FEA.
Inputs: thickness_mm, panel_span_mm, curvature_1_per_m, yield_strength_mpa, youngs_modulus_gpa.
Outputs: dent_force_N (regression) and oilcans (0/1 classification).
Model: scikit-learn HistGradientBoosting (regressor + classifier).
Key metrics (synthetic data): Dent MAE 23.9 N (baseline 125.8 N), R2 0.954; Oil‑canning accuracy 0.916, ROC‑AUC 0.979, precision 0.929, recall 0.914.
Honest limits: error rises substantially OOD (MAE ≈ 28 N inside band vs ≈ 61.7 N outside band).
Current status: proof‑of‑concept trained on synthetic, physics‑informed data. Useful for method validation and stakeholder demos; NOT yet validated for engineering sign‑off.
Immediate next validation steps: retrain and evaluate on real FEA/experimental data; add uncertainty estimates and OOD detection; add reproducible pipeline, model saving, and configurable I/O.
