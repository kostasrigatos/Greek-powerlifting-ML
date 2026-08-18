# Greek Powerlifting Performance Prediction

**Overview:** This project builds a performance prediction and uncertainty
quantification system for Greek raw powerlifters using the OpenPowerlifting
dataset. The goal is not simply to minimise prediction error, but to build a
pipeline that is honest about what the available features can and cannot
predict — and to quantify that uncertainty explicitly for each athlete
profile. The project is structured around eight phases, three modelling
approaches, and a strength standards tool that a coach or athlete can run
interactively.

**Status:** Phases 1–5 complete. Phase 6 (Bayesian Ridge — Coverage &
Calibration Curves) next. Phases 7–8 planned.

**Data:** OpenPowerlifting, 2026-08-01 release (CC BY 4.0).
Fetch instructions below — the raw CSV is not committed (> 700 MB).

---

## Project Structure

```
.
├── greek_powerlifting_analysis.ipynb   ← main notebook
├── README.md                           ← this file
├── plots/
│   ├── phase1/                         ← EDA figures
│   ├── phase2/                         ← preprocessing figures
│   ├── phase3/                         ← regression figures
│   ├── phase4/                         ← evaluation figures
│   └── phase5/                         ← boosting & SHAP figures
└── data/
    └── (raw CSV not committed — see below)
```

---

## Phase Overview

### Phase 1 — EDA & Cleaning ✓
**Goal:** define a clean, well-characterised modelling cohort and understand
the data's structure.

**Motivation:** the raw OpenPowerlifting data interleaves event types
(bench-only, push-pull, full-power) in a single TotalKg column. Without
understanding and correcting for this, the whole model would be built on a
confounded quantity.

**Approach:** a seven-step filtering funnel (Greek → raw equipment → SBD
full-power → valid total → no fourth attempts → 2015 onwards →
deduplicated), followed by comparative analysis against Southern European and
Balkan peers on all four targets.

---

### Phase 2 — Feature Engineering & Preprocessing ✓
**Goal:** turn the cohort into leakage-free, well-conditioned feature matrices
ready for modelling.

**Motivation:** the scoring columns (Dots/Wilks/Goodlift) are deterministic
functions of TotalKg and bodyweight — they are the target repackaged and must
be excluded. Federation composition and age missingness require principled
decisions before any model sees the data.

**Approach:** baselines first (predict-mean, per-sex mean, per-sex bodyweight
regression — the floors every model must beat), then feature construction,
train-only scaling, and a diagnostic suite (leakage audit, condition number
check).

---

### Phase 3 — Classical Regression ✓
**Goal:** implement linear regression from first principles and validate it.

**Motivation:** implementing OLS, gradient descent, Ridge, and Lasso from
scratch demonstrates the mathematics is understood, not just called;
validating against sklearn confirms correctness.

**Approach:** normal equation via `np.linalg.lstsq`, then batch gradient
descent verified to converge to the same solution (< 0.001 kg difference),
then Ridge and Lasso from scratch, then sklearn validation confirming all
implementations match to four decimal places. Ridge does not improve over OLS
(condition number 3.3 — well-conditioned matrix); Lasso retains all features
with one exception: age-augmented TotalKg, where Lasso finds a real optimum
at λ = 1.79 and improves RMSE by 0.11 kg. Stratified and unstratified 5-fold
CV confirm the single-split estimates are stable.

---

### Phase 4 — Evaluation & Analysis ✓
**Goal:** move beyond a single held-out RMSE and determine where the models
succeed, where they fail, and why — across demographic subgroups, across
targets, and across the bias-variance spectrum.

**Motivation:** an aggregate RMSE hides subgroup-level failure. A model can
look reasonable overall while explaining almost no variance for a specific
demographic, and a single train/test split cannot distinguish underfitting
from overfitting.

**Approach and findings:**
- **Sex-stratified performance:** RMSE is lower for female lifters across
  every target (smaller absolute totals), but R² is markedly worse and
  negative for several base-feature targets (squat −0.023, bench −0.133) —
  the model explains comparatively little of the variance within the female
  subgroup, even though absolute errors look reasonable.
- **Age-category performance:** the Masters subgroup is poorly explained
  under both base (RMSE 107.1 kg, R² = 0.247) and age-augmented (RMSE
  116.0 kg, R² = 0.038) features — consistent with small sample size
  (33–35 rows) and within-category heterogeneity (Masters spans 40–70+)
  rather than a missing feature.
- **Bias-variance diagnosis:** training RMSE and CV RMSE are close across
  every target and feature variant (gaps of 0.06–0.43 kg) — the model is in
  a high-bias, not high-variance, regime. It is underfitting because six
  demographic features cannot capture what actually drives performance
  (training history, technique, competitive experience), not overfitting to
  the training split.
- **Residual correlation:** squat-deadlift residuals correlate most
  strongly (0.80), squat-bench next (0.75), bench-deadlift weakest (0.69) —
  the model's errors are shared across lifts, not independent per lift,
  consistent with the bias finding.

---

### Phase 5 — Gradient Boosting ✓
**Goal:** explore whether a model with more functional flexibility than a
linear one can improve on Phase 3–4's classical results.

**Motivation:** Phase 4 established the linear models were underfitting
(high bias, not high variance). A specific mechanism was hypothesised: the
relationship between age and performance is plausibly non-monotonic —
young lifters improving rapidly, a plateau in the late 20s to 30s, then
decline — a shape a linear model can only fit as a straight line, but a
tree-based model can capture via threshold splits. This limitation is
structural, independent of how well-conditioned the feature matrix is.

**Approach and findings:**
- **Learning curves (classical vs. boosted):** established the baseline
  shape to compare against before any tuning effort.
- **Baseline (untuned) XGBoost vs. OLS/Ridge:** untuned boosting already
  beat the linear models on every target, with the largest gains
  concentrated specifically in the age-augmented feature set (TotalKg
  −6.67 kg vs. OLS) — the first signal that age's effect might not be
  linear.
- **Optuna hyperparameter tuning** (100 trials per target/variant, TPE
  sampler, search space biased toward shallow trees to counteract
  overfitting on a small dataset) widened these gains further: age-augmented
  TotalKg improved by 8.98 kg over OLS. *Disclosed caveat: the search
  optimises directly against the same CV folds used to report results — a
  mild leakage that likely makes the reported improvement a modest
  overestimate of true generalisation gain. A fully isolated nested-CV
  estimate was judged not worth the added complexity for this project.*
- **Tuned-model evaluation** via stratified 5-fold CV with a per-sex
  breakdown tested whether boosting closes the sex-specific gaps found in
  Phase 4.
- **SHAP analysis** confirmed the age-nonlinearity hypothesis directly: Age
  ranks as the third most important feature across all four targets (well
  above the federation dummies), and SHAP dependence plots show the
  predicted shape explicitly — a steep rise from the mid-teens through the
  mid-20s, a plateau through the early 30s, and a consistent decline
  thereafter, consistently across TotalKg, squat, bench, and deadlift.

---

### Phases 6–8 *(planned)*

| Phase | Title | Notes |
|-------|-------|-------|
| 6 | Bayesian Ridge — Coverage & Calibration Curves | |
| 7 | Uncertainty Quantification | |
| 8 | Strength Standards Tool | Interactive tool for coaches and athletes |

---

## Reproducing the Analysis

**Fetch the data:**

```bash
cd data/
curl -fL -o openpowerlifting-latest.zip \
  "https://openpowerlifting.gitlab.io/opl-csv/files/openpowerlifting-latest.zip"
unzip -o openpowerlifting-latest.zip
mv openpowerlifting-*/openpowerlifting-*.csv .
```

The notebook auto-detects the newest dated CSV in `data/` — no code change
needed after fetching. To reproduce the exact results in this notebook, use
the 2026-08-01 release (hash `55149139`, 778 MB).

**Dependencies:**

```
numpy
pandas
matplotlib
seaborn
scipy
scikit-learn
xgboost
optuna
shap
```

Install via:

```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn xgboost optuna shap
```

**Run:** open `greek_powerlifting_analysis.ipynb` in JupyterLab and run all
cells from top to bottom. Cells have execution dependencies — do not run
individual cells out of order.

---

## Key Results

| Model | Cohort | Features | Test RMSE |
|-------|--------|----------|-----------|
| Predict-mean baseline | Total | — | 145.9 kg |
| Per-sex mean baseline | Total | — | 109.1 kg |
| Per-sex BW regression baseline | Total | — | 94.4 kg |
| OLS (normal equation) | Total | Base | 93.2 kg |
| OLS (normal equation) | Total | + Age | 87.1 kg |
| Ridge (optimal λ) | Total | Base | 93.2 kg |
| Ridge (optimal λ) | Total | + Age | 87.1 kg |
| Lasso (optimal λ) | Total | Base | 93.2 kg |
| Lasso (optimal λ = 1.79) | Total | + Age | 87.0 kg |
| XGBoost, untuned | Total | Base | 91.5 kg |
| XGBoost, untuned | Total | + Age | 82.2 kg |
| XGBoost, Optuna-tuned | Total | Base | 89.1 kg |
| XGBoost, Optuna-tuned | Total | + Age | **81.4 kg** |

All scratch implementations validated against sklearn to four decimal places.
Per-lift (squat / bench / deadlift) results, subgroup breakdowns, and
diagnostic plots — including the SHAP age-dependence analysis — available in
the notebook.

---

## Attribution

All competition data sourced from the OpenPowerlifting project
(https://openpowerlifting.gitlab.io) under CC BY 4.0.
Analysis and code by Kostas Rigatos, 2026.
