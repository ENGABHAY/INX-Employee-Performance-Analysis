# Analysis

This document is structured to directly answer Sections 3, 4, and 5 of the IABAC™ Project
Submission Guidelines (V1.2), in that order.

> **Revision note:** This is a corrected version of the analysis. A review identified that the
> original single-model approach used features (satisfaction scores, tenure, salary-hike
> history) that do not exist for an external hiring candidate — undermining the brief's stated
> hiring use case — plus three smaller issues (the encoding pipeline wasn't saved, no class
> weighting was applied despite the imbalance being discussed, and the best model was picked by
> single-split test accuracy rather than cross-validated macro-F1). All five points are fixed
> below and in the notebooks.

---

## Section 3 — Project Summary

### 3.1 Algorithm and training method(s) used
**Two separate models are trained**, because the project's two requests — a hiring-prediction
model, and an explanation of what drives performance — require different, non-overlapping
feature sets (see 3.2 and Section 5.3).

For both, five algorithms (Logistic Regression, SVM-RBF, Random Forest, Gradient Boosting,
XGBoost) were benchmarked with an 80/20 stratified split and 5-fold stratified cross-validation,
**all trained with class weighting** (`class_weight='balanced'` where the estimator supports it
natively; `sample_weight` via `compute_sample_weight('balanced', ...)` for Gradient Boosting and
XGBoost, which don't expose a `class_weight` parameter) to counter the 73%/16%/11% target
imbalance. The best algorithm was **selected by mean 5-fold CV macro-F1**, not single-split test
accuracy — accuracy alone would over-reward whichever model is best at the majority class alone.
The selected algorithm was then tuned with `GridSearchCV` (also scored on CV macro-F1), and the
held-out 20% test set was used only afterward, as a final unbiased check.

- **Hiring Model:** tuned **XGBoost** — CV macro-F1 **0.369**, holdout accuracy **41.7%**,
  holdout macro-F1 **0.331**.
- **Diagnostic Model:** tuned **Random Forest** (`n_estimators=300, max_depth=10,
  min_samples_leaf=2`) — CV macro-F1 **0.896**, holdout accuracy **94.17%**, holdout macro-F1
  **0.901**.

Full comparison tables, the class-weighting code, and the CV-based selection logic are in
`src/models/train_model.ipynb`.

### 3.2 Most important features selected for analysis, and why — was PCA/factorization used?
**No PCA or other dimensionality-reduction/factorization technique was used** — with at most 26
candidate predictors and tree-based models in the comparison, dimensionality was not a practical
problem, and PCA's abstract components would have sacrificed the interpretability the business
explicitly needs.

Instead, features were split by **availability at prediction time**:
- **Hiring Model (12 features):** only what is knowable about an external candidate —
  `Age, Gender, EducationBackground, MaritalStatus, EmpDepartment, EmpJobRole, EmpJobLevel,
  BusinessTravelFrequency, DistanceFromHome, EmpEducationLevel, NumCompaniesWorked,
  TotalWorkExperienceInYears`.
- **Diagnostic Model (26 features):** the full set, valid because it is applied to *existing*
  employees, including satisfaction, tenure, promotion history, and compensation-growth
  features that simply don't exist for a candidate.

This split — not a correlation or importance threshold — is the primary feature-selection
decision in this project, and it is what makes the Hiring Model actually usable for its stated
purpose. See `src/Data Processing/data_processing.ipynb` Section 4 for the full feature
classification and reasoning.

### 3.3 Other techniques and tools used
- **pandas / numpy** for data loading and cleaning.
- **scikit-learn** for preprocessing (`StandardScaler`, `LabelEncoder`), `compute_sample_weight`
  for class balancing, cross-validation, and `GridSearchCV`.
- **XGBoost** library for the gradient-boosted tree model in the comparison.
- **matplotlib / seaborn** for all exploratory and results visualizations.
- **joblib** to persist both trained models, both scalers, both feature lists, **and the fitted
  encoders themselves** (`label_encoders.pkl`, `binary_maps.pkl`) — so `predict_model.ipynb` can
  transform a brand-new raw record end-to-end, not just already-encoded test rows.

---

## Section 4 — Features Selection / Engineering

### 4.1 Most important features selected for analysis, and why
See 3.2 above for the primary split (hiring-safe vs. full feature set). Within each model, no
further manual feature elimination was performed — every available feature in that model's set
was passed to training, and the trained model's own feature importance was used to surface the
top predictors, which is more defensible for a business audience than an analyst's subjective
pre-filtering:
- **Hiring Model top predictors:** `EmpDepartment`, `EmpJobRole`, `NumCompaniesWorked`.
- **Diagnostic Model top 3 factors:** `EmpLastSalaryHikePercent`, `EmpEnvironmentSatisfaction`,
  `YearsSinceLastPromotion` (this is the answer to the brief's "Top 3 Factors" business
  question — see Section 5.3).

### 4.2 Important feature transformations
- **Encoding:** binary categoricals (`Gender`, `OverTime`, `Attrition`) mapped to 0/1 via a
  saved dictionary (`models/binary_maps.pkl`); nominal categoricals with more than two
  categories (`EducationBackground`, `MaritalStatus`, `EmpDepartment`, `EmpJobRole`,
  `BusinessTravelFrequency`) label-encoded with `sklearn.LabelEncoder` objects that are **fit
  once and persisted** (`models/label_encoders.pkl`), so every downstream notebook — and any
  future new data — uses an identical mapping instead of silently re-fitting.
- **Scaling:** `StandardScaler`, fit separately for the Hiring Model's 12 features and the
  Diagnostic Model's 26 features (two different scalers, `hiring_scaler.pkl` and
  `diagnostic_scaler.pkl`), applied only for the distance/gradient-sensitive candidate models
  (Logistic Regression, SVM); tree-based models used unscaled features.
- **Class weighting:** applied during training of every candidate algorithm (see 3.1) —
  genuinely present in the code now, not just discussed as a concept.
- **No imputation was needed** — the dataset had zero missing values.

### 4.3 Correlation / interactions among the selected features, and how they were considered
A full correlation matrix against `PerformanceRating` was computed
(`data_exploratory_analysis.ipynb`). The strongest correlates by magnitude were
`EmpEnvironmentSatisfaction` (+0.40), `EmpLastSalaryHikePercent` (+0.33), and
`YearsSinceLastPromotion` (−0.17). These linear correlations were used as a sanity check, not
the final answer, since they can miss non-linear or interaction effects. The Diagnostic Model's
top-3 factors (Section 4.1) come from the trained Random Forest's feature importances, which do
capture such interactions, and they closely agree with the correlation ranking — increasing
confidence in the result rather than relying on either method alone.

---

## Section 5 — Results, Analysis and Insights

### 5.1 Interesting relationships found
- The target is **imbalanced**: Rating 3 ("Excellent") ≈ 73%, Rating 2 ("Good") ≈ 16%, Rating 4
  ("Outstanding") ≈ 11%. A majority-class baseline (always predict Rating 3) scores **72.9%
  accuracy but only 0.281 macro-F1** on the same test split — this baseline is what the Hiring
  Model's macro-F1 of 0.331 should be judged against, not against the baseline's inflated
  accuracy figure (see `train_model.ipynb` for the full comparison).
- `EmpLastSalaryHikePercent` trends upward across performance-rating bands.
- **Corrected finding:** employees who work overtime have a *slightly higher* average
  performance rating (2.99) than those who don't (2.93) — an earlier draft of this analysis
  stated the opposite direction; the underlying notebook output was always correct, only the
  written summary was wrong, and that has been fixed here.

### 5.2 Most important technique used in this project
Splitting the feature set by **availability at prediction time** (hiring-safe vs. full) was the
single most consequential decision — it's what makes the Hiring Model's output something a
recruiter could actually act on, rather than a model that looks accurate on paper but is
unusable in practice because its best predictors don't exist for a candidate.

### 5.3 Clear answers to the business problems

| Business ask | Answer |
|---|---|
| Department-wise performance | Development (3.09) and Data Science (3.05) rate highest; Sales (2.86) and **Finance (2.78) rate lowest.** Full table: `data_exploratory_analysis.ipynb`, `visualize.ipynb` Section 2. |
| Top 3 factors affecting performance | **1. `EmpLastSalaryHikePercent`  2. `EmpEnvironmentSatisfaction`  3. `YearsSinceLastPromotion`** — from the Diagnostic Model's feature importance (existing-workforce data). |
| A trained model to predict performance, for hiring | **Hiring Model** (tuned XGBoost), using only candidate-knowable features: **41.7% test accuracy, 0.331 macro-F1** (vs. a 0.281 macro-F1 majority-class baseline). Modest but honest — see 5.4. Saved to `models/hiring_model.pkl`, demonstrated in `predict_model.ipynb`. |
| Recommendations to improve performance | See 5.4 below. |

### 5.4 Further business insights and recommendations
- **Be honest with the CEO about what candidate-only data can and can't tell you.** The Hiring
  Model's accuracy (41.7%) is well below the Diagnostic Model's (94.2%) — not because of a
  modeling error, but because *the strongest real predictors of performance are things you only
  learn after someone joins* (environment satisfaction, promotion history, compensation growth).
  A resume-stage model can meaningfully beat a naive guess (0.331 vs. 0.281 macro-F1) but should
  be used as one weak signal among several in a hiring decision, not a cutoff score.
- **Compensation growth and environment satisfaction are the two biggest levers for the existing
  workforce.** `EmpLastSalaryHikePercent` and `EmpEnvironmentSatisfaction` rank above department
  in the Diagnostic Model's feature importance — pacing of pay growth and day-to-day working
  conditions are more actionable than department-specific interventions.
- **Stalled promotions correlate with lower ratings**, and this shows up in both the correlation
  analysis and the Diagnostic Model's top-3 factors — worth a targeted promotion-pipeline review
  rather than a blanket policy, in line with the CEO's concern about broadly affecting morale.
- **Sales and Finance merit a department-specific review** (2.86 and 2.78 average rating vs.
  3.09 in Development).
- **For hiring**, department and role fit (`EmpDepartment`, `EmpJobRole`) and prior job-hopping
  frequency (`NumCompaniesWorked`) were the strongest signals available pre-hire — worth
  weighting role-fit and career-stability questions more heavily in interviews, since those are
  the traits the data can actually speak to before someone joins.
