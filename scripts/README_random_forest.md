# Random Forest for Antimicrobial Resistance Classification

The `random_forest.py` script predicts antimicrobial resistance phenotypes from genomic features using Random Forest classification. It evaluates model performance across repeated train/test splits, optimizes hyperparameters through cross-validation, and records feature importance to support the interpretation of genomic predictors.

## Features

- **Binary classification:** predicts susceptible (`0`) or resistant (`1`) phenotypes.
- **Repeated evaluation:** assesses performance across different train/test partitions.
- **Hyperparameter optimization:** uses `RandomizedSearchCV` with stratified cross-validation, optimizing the F1 score of the resistant class.
- **Performance assessment:** calculates accuracy, precision, recall, and F1-score for both classes.
- **Feature importance:** records feature importance for each run.

## 1. Configurable Parameters

The main settings can be modified directly in `random_forest.py`.

### Evaluation settings

| Parameter | Default | Description |
|---|---:|---|
| `n_runs` | `100` | Number of repeated train/test evaluations. Each run uses a different random seed. |
| `test_size` | `0.20` | Fraction of samples reserved for testing in each run. |
| `n_iter_search` | `10` | Number of hyperparameter combinations sampled and evaluated in each run. |
| `n_splits_cv` | `4` | Number of stratified folds used for cross-validation during hyperparameter optimization. |

The train/test split is stratified by `outcome`, preserving the approximate class proportions in the training and test sets.

Increasing `n_runs` provides a broader assessment of variability across data splits, while increasing `n_iter_search` explores more hyperparameter combinations. Increasing `n_splits_cv` changes the cross-validation configuration and increases computational cost.

### Hyperparameter search space

The search space is defined by `param_dist`:

| Hyperparameter | Description |
|---|---|
| `n_estimators` | Number of decision trees in the Random Forest. |
| `max_features` | Number or fraction of features considered when searching for the best split at each node. |
| `max_depth` | Maximum depth of each decision tree. |
| `min_samples_split` | Minimum number of samples required to split an internal node. |
| `min_samples_leaf` | Minimum number of samples required in a leaf node. |
| `bootstrap` | Whether bootstrap samples are used to train individual trees. |
| `class_weight` | Controls class weighting during model fitting. The `balanced` setting adjusts weights inversely according to class frequencies in the training data. |

`RandomizedSearchCV` evaluates `n_iter_search` randomly selected hyperparameter combinations using stratified cross-validation. The best combination is selected based on the mean F1-score for the resistant class (`outcome = 1`).

The best-performing model is then used to predict the resistance classes of the test samples.

## 2. Output Files

All output files are saved directly in the `results/` directory. Filenames are based on the input dataset name, without its `.csv` extension.

For each dataset, the script generates three CSV files.

### 2.1. Global Performance Metrics

**Filename:** `accuracy_global_<dataset_name>.csv`

One row is generated for each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `accuracy` | Proportion of correctly classified test samples. |
| `S_precision` | Precision for the susceptible class (`0`). |
| `S_recall` | Recall for the susceptible class (`0`). |
| `S_F1_score` | F1-score for the susceptible class (`0`). |
| `R_precision` | Precision for the resistant class (`1`). |
| `R_recall` | Recall for the resistant class (`1`). |
| `R_F1_score` | F1-score for the resistant class (`1`). |

Precision, recall, and F1-score are calculated separately for each class. Undefined class-specific metrics are set to zero.

### 2.2. Feature Importance

**Filename:** `feature_importance_<dataset_name>.csv`

Feature importance is saved in **long format**, with one row per feature per run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `feature` | Name of the genomic feature. |
| `importance` | Feature importance score reported by the fitted Random Forest. |

Feature importance values represent the relative contribution of each feature to impurity reduction in the fitted model. They sum to approximately 1 within each run.

Comparing feature importance across runs can help identify genomic features that consistently contribute to classification and assess the stability of their relative importance.

### 2.3. Predictions per Sample

**Filename:** `prediction_per_sample_<dataset_name>.csv`

One row is generated for each test sample in each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `sample_id` | Identifier of the test sample. |
| `real_class` | Observed class from `outcome`. |
| `predicted_class` | Class predicted by the fitted Random Forest. |
| `agreement` | `1` if the predicted class matches the observed class; otherwise `0`. |

A sample may appear in multiple runs because the test sets are generated independently. The `agreement` column allows classification performance to be examined at the sample level.
