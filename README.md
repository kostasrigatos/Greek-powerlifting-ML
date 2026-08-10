# Greek Powerlifting Performance Prediction

**Overview:** This project builds a performance prediction and uncertainty
quantification system for Greek raw powerlifters using the OpenPowerlifting
dataset. The goal is not simply to minimise prediction error, but to build a
pipeline that is honest about what the available features can and cannot
predict — and to quantify that uncertainty explicitly for each athlete
profile. The project is structured around eight phases, three modelling
approaches, and a strength standards tool that a coach or athlete can run
interactively.

**Status:** Phases 1–2 complete. Phase 3 in progress (OLS and gradient
descent done; Ridge, Lasso, and sklearn validation pending). Phases 4–8
planned.

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
│   └── phase3/                         ← regression figures
└── data/
    └── (raw CSV not committed — see below)
```

---

## Phase Overview

### Phase 1 — EDA & Cleaning
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

### Phase 2 — Feature Engineering & Preprocessing
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

### Phase 3 — Classical Regression *(in progress)*
**Goal:** implement linear regression from first principles and validate it.

**Motivation:** implementing OLS, gradient descent, Ridge, and Lasso from
scratch demonstrates the mathematics is understood, not just called;
validating against sklearn confirms correctness.

**Approach:** normal equation via `np.linalg.lstsq`, then batch gradient
descent verified to converge to the same solution (< 0.001 kg difference),
then Ridge and Lasso from scratch, then sklearn validation. *(Ridge/Lasso
and sklearn validation pending.)*

---

### Phases 4–8 *(planned)*

| Phase | Title |
|-------|-------|
| 4 | Evaluation & Analysis |
| 5 | Gradient Boosting |
| 6 | Bayesian Ridge — Coverage & Calibration Curves |
| 7 | Uncertainty Quantification |
| 8 | Strength Standards Tool |

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
pathlib (standard library)
```

Install via:

```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn
```

**Run:** open `greek_powerlifting_analysis.ipynb` in JupyterLab and run all
cells from top to bottom. Cells have execution dependencies — do not run
individual cells out of order.

---

## Key Results (Phases 1–3, partial)

| Model | Cohort | Features | Test RMSE |
|-------|--------|----------|-----------|
| Predict-mean baseline | Total | — | 145.9 kg |
| Per-sex mean baseline | Total | — | 109.1 kg |
| Per-sex BW regression baseline | Total | — | 94.4 kg |
| OLS (normal equation) | Total | Base | 93.2 kg |
| OLS (normal equation) | Total | + Age | 87.1 kg |
| Gradient descent | Total | Base | 93.2 kg |
| Gradient descent | Total | + Age | 87.1 kg |

Per-lift (squat / bench / deadlift) results available in the notebook.

---

## Attribution

All competition data sourced from the OpenPowerlifting project
(https://openpowerlifting.gitlab.io) under CC BY 4.0.
Analysis and code by Kostas Rigatos, 2026.
