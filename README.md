# Household Data Analysis

Analysis of the DemoStats 2024 and HouseholdSpend 2024 datasets, with one row per Canadian postal code. The notebook cleans and merges both datasets, clusters postal codes, and predicts the share of household income spent on insurance and pension contributions. It also exports tables for a Power BI dashboard.

## Pipeline

1. **Cleaning**: removes rows and columns that are all zeros, the `GEO` column, and columns with missing values. Checks for negative values. Merges the two datasets on `CODE`.
2. **Split**: 70/15/15 train/validation/test split with seed 2025. Outliers are handled with z-scores: they are replaced with medians from the training set, and the same medians are applied to the validation and test sets.
3. **Clustering**: K-means (k=2, chosen with the elbow method), shown with PCA and UMAP projections.
4. **Regression**: the target is `insurance_pension_ratio = HSEP001S / HSHNIAGG`.
   - Elastic Net, run both with and without PCA as a baseline.
   - XGBoost tuned with `RandomizedSearchCV`. Eight columns that can rebuild the target exactly are dropped (`LEAKY_COLS`) to prevent leakage. Test-set R² ≈ 0.86.
   - 95% confidence intervals from bootstrapping.
5. **Interpretability**: SHAP summary and dependence plots.
6. **Exports**: CSVs for Power BI are written to `Data/exports_v2/`:
   - `dashboard_data.csv`
   - `model_predictions.csv`
   - `shap_importance.csv`
   - `cluster_assignments.csv`

## Setup

```sh
python -m venv .venv
.venv\Scripts\activate
pip install numpy pandas polars pyarrow scikit-learn xgboost matplotlib seaborn shap yellowbrick umap-learn jupyter
```

## Data

`Data/` is not included in the repo because the files are about 5 GB and are too large for GitHub. Put these files in `Data/` before running the notebook:

- `DemoStats.csv`, `HouseholdSpend.csv`
- `DemoStats 2024 - Metadata.csv`, `HouseholdSpend 2024 - Metadata.csv`

The notebook reads from absolute Windows paths. Update them to match your machine.
