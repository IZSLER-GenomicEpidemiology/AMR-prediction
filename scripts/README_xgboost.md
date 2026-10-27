# XGBoost for Quantitative MIC Prediction

The `xgboost.py` script predicts quantitative minimum inhibitory concentration (MIC) values from genomic features using XGBoost regression. It evaluates model performance across repeated train/test splits, optimizes hyperparameters through a factorial design and grid search, and records feature importance.

## Features

- **MIC regression:** predicts quantitative MIC values on a log2 scale.
- **Repeated evaluation:** assesses performance across different train/test partitions.
- **Hyperparameter optimization:** combines a factorial design with `GridSearchCV`.
- **Performance assessment:** calculates prediction accuracy within ±1 MIC dilution and the coefficient of determination (R²).
- **Feature importance:** records feature importance for each run.

## 1. Configurable Parameters

The main settings can be modified directly in `xgboost.py`.

### Evaluation settings

| Parameter | Default | Description |
|---|---:|---|
| `n_runs` | `2` | Number of repeated train/test evaluations. Each run uses a different random seed. |
| `test_size` | `0.20` | Fraction of samples reserved for testing in each run. |

The train/test split is stratified by MIC value to preserve the distribution of observed target values as far as the available sample counts allow.

Increasing `n_runs` provides a broader assessment of variability across data splits, at the cost of additional computational time.

### Factorial design

The factorial design evaluates combinations of the following hyperparameters:

| Parameter | Values | Description |
|---|---|---|
| `max_depth` | `1`, `6` | Maximum depth of each tree. |
| `colsample_bytree` | `0.25`, `1` | Fraction of features sampled for each tree. |
| `subsample` | `0.25`, `1` | Fraction of training samples used to build each tree. |

The combinations are evaluated using five-fold cross-validation and R² as the scoring metric. The base model uses `n_estimators = 100` and `learning_rate = 0.1`.

The best-performing combination determines the `colsample_bytree` and `subsample` values used in the subsequent grid search.

### Grid search

The grid search is performed using `GridSearchCV`, exploring the following hyperparameters:

| Parameter | Values | Description |
|---|---|---|
| `max_depth` | `3`, `4`, `5`, `6` | Maximum depth of each tree. |
| `learning_rate` | `0.01`, `0.05`, `0.1` | Step size used to update the model during boosting. |
| `n_estimators` | `100`, `600` | Number of boosting rounds. |
| `colsample_bytree` | Selected through the factorial design | Fraction of features sampled for each tree. |
| `subsample` | Selected through the factorial design | Fraction of training samples used to build each tree. |

Five-fold cross-validation and R² are used to select the best model.

## 2. Output Files

All output files are saved directly in the `results/` directory. Filenames are based on the input dataset name, without its `.csv` extension.

For each dataset, the script generates four CSV files.

### 2.1. Global Performance Metrics

**Filename:** `accuracy_global_<dataset_name>.csv`

One row is generated for each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `accuracy_1_diluition` | Proportion of test samples whose predicted MIC differs from the observed MIC by no more than one log2 unit. |
| `R2` | Coefficient of determination, measuring how well the predictions explain the variability in observed MIC values. |

The `accuracy_1_diluition` column measures agreement within ±1 MIC dilution and is not equivalent to conventional classification accuracy.

### 2.2. Predictions per Sample

**Filename:** `prediction_per_sample_<dataset_name>.csv`

One row is generated for each test sample in each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `sample_id` | Identifier of the test sample. |
| `real_mic` | Observed MIC value. |
| `predicted_mic` | MIC value predicted by XGBoost. |
| `agreement` | `1` if the predicted MIC differs from the observed MIC by no more than one log2 unit; otherwise `0`. |

A sample may appear in multiple runs because the test sets are generated independently.

### 2.3. Accuracy per MIC Dilution

**Filename:** `accuracy_per_dilution_<dataset_name>.csv`

This file summarizes prediction performance for each observed MIC value in each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `real_mic` | Observed MIC value defining the group. |
| `predicted_mic` | Mean predicted MIC among samples in that group. |
| `n_samples` | Number of test samples in the group. |
| `accuracy_1_diluition` | Proportion of samples in the group predicted within ±1 MIC dilution. |

Accuracy is calculated from individual predictions before aggregation by observed MIC. The `predicted_mic` column reports the mean predicted value for each group and is not used to calculate the group's accuracy.

### 2.4. Feature Importance

**Filename:** `feature_importance_<dataset_name>.csv`

Feature importance is saved in **long format**, with one row per feature per run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `feature` | Name of the genomic feature. |
| `importance` | Feature importance score reported by the fitted XGBoost model. |

Comparing feature importance across runs can help identify genomic features that consistently contribute to model predictions and assess the stability of their relative importance.
