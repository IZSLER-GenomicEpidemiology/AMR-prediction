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

### Factorial design

The factorial design evaluates combinations of the following hyperparameters:

| Parameter | Description |
|---|---|
| `max_depth` | Maximum depth of each tree. |
| `colsample_bytree` | Fraction of features sampled for each tree. |
| `subsample` | Fraction of training samples used to build each tree. |

The combinations are evaluated using five-fold cross-validation and R² as the scoring metric. 

### Grid search

The grid search is performed using `GridSearchCV`, exploring the following hyperparameters:

| Parameter | Description |
|---|---|
| `max_depth` | Maximum depth of each tree. |
| `learning_rate` | Step size used to update the model during boosting. |
| `n_estimators` | Number of boosting rounds. |
| `colsample_bytree` | Fraction of features sampled for each tree, selected through the factorial design. |
| `subsample` | Fraction of training samples used for each tree, selected through the factorial design. |

Five-fold cross-validation and R² are used to select the best model.

## 3. Output Files

All output files are saved directly in the `results/` directory. Filenames are based on the input dataset name, without its `.csv` extension.

For each dataset, the script generates four CSV files.

### 3.1. Global Performance Metrics

**Filename:** `accuracy_global_<dataset_name>.csv`

One row is generated for each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `accuracy_1_diluition` | Proportion of test samples whose predicted MIC differs from the observed MIC by no more than one log2 unit. |
| `R2` | Coefficient of determination. |

### 3.2. Predictions per Sample

**Filename:** `prediction_per_sample_<dataset_name>.csv`

One row is generated for each test sample in each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `sample_id` | Identifier of the test sample. |
| `real_mic` | Observed MIC value on the log2 scale. |
| `predicted_mic` | MIC value predicted by XGBoost. |
| `agreement` | `1` if the prediction is within ±1 log2 unit of the observed MIC; otherwise `0`. |

### 3.3. Accuracy per MIC Dilution

**Filename:** `accuracy_per_dilution_<dataset_name>.csv`

Summarizes prediction performance for each observed MIC value in each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `real_mic` | Observed MIC value defining the group. |
| `predicted_mic` | Mean predicted MIC among samples in that group. |
| `n_samples` | Number of test samples in the group. |
| `accuracy_1_diluition` | Proportion of samples predicted within ±1 MIC dilution. |

### 3.4. Feature Importance

**Filename:** `feature_importance_<dataset_name>.csv`

Feature importance is saved in **long format**, with one row per feature per run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `feature` | Name of the genomic feature. |
| `importance` | Feature importance score reported by the fitted XGBoost model. |

