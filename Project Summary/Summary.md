# Project Summary — Employee Performance Analysis (INX Future Inc.)

**Project Code:** 10281 | **Certification:** IABAC™ Certified Data Scientist

> This is a corrected revision. The original draft used one blended model for both the
> "explain performance" and "predict for hiring" asks; since several of its best predictors
> (satisfaction scores, tenure, salary-hike history) don't exist for an external candidate, that
> model could not actually do the hiring task the brief asked for. This revision splits the work
> into two purpose-built models, saves the actual encoding pipeline (not just its mapping),
> trains with class weighting, selects models by cross-validated macro-F1 instead of a single
> accuracy number, and corrects a reversed overtime finding. Full detail in `Analysis.md`.

## What Was Done

The INX Future Inc. employee dataset (1200 records, 28 attributes) was cleaned and encoded, with
every predictor classified as either **knowable about a hiring candidate** or **only observable
for an existing employee**. Two models were then trained on that split:

- **Hiring Model** (tuned XGBoost, 12 candidate-safe features): **41.7% test accuracy, 0.331
  macro-F1** — modest, but genuinely better than a majority-class baseline's 0.281 macro-F1.
  This is the model that fulfills the brief's "used to hire employees" requirement.
- **Diagnostic Model** (tuned Random Forest, all 26 features): **94.17% test accuracy, 0.901
  macro-F1** — used to explain what drives performance across the current workforce.

Both were trained with class weighting to counter the dataset's 73%/16%/11% rating imbalance,
and selected by 5-fold cross-validated macro-F1 rather than a single test-accuracy number.

## Key Findings

1. **Department-wise performance:** Development (3.09) and Data Science (3.05) rate highest on
   average; Finance (2.78) and Sales (2.86) rate lowest.
2. **Top 3 factors driving performance** (Diagnostic Model): `EmpLastSalaryHikePercent`,
   `EmpEnvironmentSatisfaction`, `YearsSinceLastPromotion`.
3. **Overtime workers rate slightly *higher* on average (2.99 vs. 2.93)** — correcting a reversed
   claim in the original draft.
4. **Candidate-only data is a weak-but-real signal** for hiring: department/role fit and prior
   job-hopping frequency (`NumCompaniesWorked`) were the strongest pre-hire predictors, but the
   model should support a hiring decision, not replace one.

## Where Everything Lives

- `Project Summary/Requirement.md`, `Analysis.md`, `Summary.md` — this write-up, in full.
- `data/raw/` — original dataset; `data/processed/` — cleaned and encoded modeling data.
- `src/Data Processing/` — cleaning, encoder-persistence, and EDA notebooks.
- `src/models/` — training (both models) and prediction notebooks.
- `models/` — saved models, scalers, feature lists, **and the fitted encoders**
  (`label_encoders.pkl`, `binary_maps.pkl`) plus `model_metadata.json` for both models.
- `src/visualization/` — consolidated results charts (also saved as PNGs in `references/`).
- `references/` — original PDF briefs, data dictionary, and all generated chart images.

Full methodology, all metrics, and every figure referenced above are reproduced in detail in
`Analysis.md` and in the executed notebook outputs themselves.
