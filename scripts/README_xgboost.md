# XGBoost for Quantitative MIC Prediction

The `xgboost.py` script predicts quantitative minimum inhibitory concentration (MIC) values from genomic features using XGBoost regression.

## Features

- **MIC regression:** predicts quantitative MIC values on a log2 scale.
- **Repeated evaluation:** assesses performance across different train/test partitions.
- **Hyperparameter optimization:** combines a factorial design with `GridSearchCV`.
- **Performance assessment:** calculates prediction accuracy within ±1 MIC dilution and the coefficient of determination (R²).
- **Feature importance:** records feature importance for each run.

## 1. Input Data

The script processes all CSV files in the `data/` directory independently. Each file must contain the following columns:

| Column | Description |
|---|---|
| `sample_id` | Unique identifier for each sample or isolate. |
| `outcome` | Quantitative MIC value expressed on a log2 scale. |
| Feature columns | Numeric genomic predictors, such as gene presence/absence profiles or k-mer frequencies. |

**Input requirements**

- `outcome` must contain numeric MIC values on a log2 scale.
- All columns except `sample_id` and `outcome` are treated as numeric genomic features.
- Rows with missing values or non-numeric MIC values are excluded.
- MIC values occurring only once are excluded.
- `sample_id` is used for identification, not as a predictor.
- Datasets must contain enough samples for train/test splitting and cross-validation.

## 2. Configurable Parameters

The main settings can be modified directly in `xgboost.py`.

### Evaluation settings

| Parameter | Default | Description |
|---|---:|---|
| `n_runs` | `100` | Number of repeated train/test evaluations. Each run uses a different random seed. |
| `test_size` | `0.20` | Fraction of samples reserved for testing in each run. |

The train/test split is stratified by MIC value to preserve the distribution of observed target values as far as the available sample counts allow.

Increasing `n_runs` provides a broader assessment of variability across data splits, at the cost of additional computational time.

### Factorial design

The factorial design evaluates combinations of the following hyperparameters:

| Parameter | Description |
|---|---|
| `max_depth` | Maximum depth of each tree. |
| `colsample_bytree` | Fraction of features sampled for each tree. |
| `subsample` | Fraction of training samples used to build each tree. |

The combinations are evaluated using five-fold cross-validation and R² as the scoring metric. The base model uses `n_estimators = 100` and `learning_rate = 0.1`.

The best-performing combination determines the `colsample_bytree` and `sub
