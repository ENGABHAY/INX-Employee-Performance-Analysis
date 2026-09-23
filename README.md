# INX Future Inc. — Employee Performance Analysis
IABAC™ Certified Data Scientist | Project Code: 10281

> **Revision note:** This package corrects five issues found in an earlier draft — most
> importantly, that the original single model used features (satisfaction, tenure, salary-hike
> history) unavailable for an external hiring candidate. See `Project Summary/Analysis.md` for
> the full list of corrections.

## How to Read This Submission

1. Start with **`Project Summary/Summary.md`** for the executive summary.
2. **`Project Summary/Requirement.md`** — the business problem as given.
3. **`Project Summary/Analysis.md`** — full methodology, metrics, and insights (includes a
   revision note detailing what was corrected and why).
4. Notebooks (run in this order to reproduce everything from raw data):
   1. `src/Data Processing/data_processing.ipynb` — cleaning, encoding, **saves the fitted
      encoders**, classifies every feature as hiring-safe or not.
   2. `src/Data Processing/data_exploratory_analysis.ipynb`
   3. `src/models/train_model.ipynb` — trains **both** the Hiring Model and the Diagnostic
      Model, with class weighting and CV-macro-F1-based selection.
   4. `src/models/predict_model.ipynb` — scores a new candidate (Hiring Model, via the saved
      encoders) and reviews an existing employee (Diagnostic Model).
   5. `src/visualization/visualize.ipynb`

All notebooks have already been executed — outputs, tables, and charts are embedded, so no
re-run is required to review results.

## Directory Structure (per IABAC™ submission guidelines)

```
├── Project Summary/        <- Requirement, Analysis, Summary (top-level write-up)
├── data/
│   ├── external/            <- (unused — no third-party data beyond the provided dataset)
│   ├── processed/           <- Cleaned + encoded modeling-ready data
│   └── raw/                 <- Original, unmodified .xls dataset
├── src/
│   ├── Data Processing/      <- data_processing.ipynb, data_exploratory_analysis.ipynb
│   ├── models/               <- train_model.ipynb, predict_model.ipynb
│   └── visualization/        <- visualize.ipynb
├── models/                  <- Saved models, scalers, feature lists, fitted encoders, metadata
└── references/               <- Original project PDFs, data dictionary, all generated charts
```

## Headline Results

| Model | Purpose | Features | Algorithm | Test Accuracy | Macro-F1 |
|---|---|---|---|---|---|
| **Hiring Model** | Predict performance for candidate screening | 12 (hiring-safe only) | XGBoost | 41.7% | 0.331 (vs. 0.281 majority-class baseline) |
| **Diagnostic Model** | Explain what drives performance in the current workforce | 26 (all) | Random Forest | 94.17% | 0.901 |

**Top 3 factors affecting performance:** `EmpLastSalaryHikePercent`, `EmpEnvironmentSatisfaction`,
`YearsSinceLastPromotion`. **Department ranking:** Development and Data Science highest; Finance
and Sales lowest. Full reasoning and recommendations in `Project Summary/Analysis.md`.
