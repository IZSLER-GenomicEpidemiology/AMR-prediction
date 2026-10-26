# Random Forest for Antimicrobial Resistance Classification

A machine learning workflow for predicting antimicrobial resistance phenotypes from genomic features using Random Forest classification.

The script evaluates model performance across repeated train/test splits, optimizes hyperparameters through cross-validation, and records feature importance to support the interpretation of genomic predictors.

## Features

- **Binary classification:** predicts susceptible (0) or resistant (1) phenotypes.
- **Repeated evaluation:** assesses model performance across different train/test partitions.
- **Hyperparameter optimization:** uses `RandomizedSearchCV` with stratified cross-validation, optimizing the F1 score of the resistant class.
- **Performance assessment:** calculates accuracy, precision, recall, and F1 score for both classes.
- **Feature importance:** records the importance of each feature for every run.
- **Batch processing:** processes all CSV datasets found in the input directory.

## 1. Input Data

### File location

Place the input CSV files in the `data/` directory at the root of the repository.

The script automatically detects and processes all files with the `.csv` extension in this directory. Each dataset is analyzed independently.

```text
repository/
├── scripts/
│   └── random_forest.py
├── data/
│   ├── dataset_1.csv
│   └── dataset_2.csv
└── results/
```

### Required format

Each CSV file must contain the following columns:

| Column | Description |
|---|---|
| `sample_id` | Unique identifier for each sample or isolate. |
| `outcome` | Binary target variable: `0` = susceptible, `1` = resistant. |
| Feature columns | Numeric predictors used by the classifier, such as gene presence/absence or other genomic features. |

Example:

```csv
sample_id,outcome,gene_feature_1,gene_feature_2,gene_feature_3
SAMPLE_001,0,0,1,0
SAMPLE_002,1,1,1,0
SAMPLE_003,0,0,0,1
SAMPLE_004,1,1,0,1
```

**Input requirements**

- The CSV must contain both `sample_id` and `outcome`.
- `outcome` must contain integer-compatible binary labels: `0` and `1`.
- All feature columns must contain numeric values supported by scikit-learn. For gene presence/absence data, use `0` for absence and `1` for presence.
- Rows with missing `sample_id` or `outcome` are removed automatically. Missing feature values are not imputed.
- Each class must have enough samples to support the selected train/test split and cross-validation settings.
- The script does not use `sample_id` as a predictor. All columns other than `sample_id` and `outcome` are treated as features.

Use consistent feature definitions and encoding across datasets.

## 2. Configurable Parameters

The main settings can be modified directly in `random_forest.py`.

### Evaluation settings

| Parameter | Default | Description |
|---|---:|---|
| `n_runs` | `100` | Number of repeated train/test evaluations. Each run uses a different random seed. |
| `test_size` | `0.20` | Fraction of samples reserved for testing in each run. The remaining 80% is used for training. |
| `n_iter_search` | `10` | Number of hyperparameter combinations sampled and evaluated in each run. |
| `n_splits_cv` | `4` | Number of stratified folds used for cross-validation during hyperparameter optimization. |

The train/test split is stratified by `outcome`, preserving the approximate class proportions in the training and test sets.

Increasing `n_runs` provides a broader assessment of variability across data splits, while increasing `n_iter_search` explores more hyperparameter combinations. Increasing `n_splits_cv` provides a different cross-validation configuration, with a corresponding increase in computational cost.

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

`RandomizedSearchCV` samples `n_iter_search` combinations from the specified search space. The best combination is selected using the mean cross-validation F1 score for the resistant class (`outcome = 1`).

The search space can be customized by editing `param_dist`, allowing the user to adapt the model configuration to the dataset and analysis objectives.

## 3. Output Files

For each input dataset, the script generates three CSV files. Output filenames are based on the input filename, without its `.csv` extension.

### 3.1. Prediction metrics

**Location:** `results/prediction/prediction_<dataset_name>.csv`

One row is generated for each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `Accuracy` | Proportion of correctly classified test samples. |
| `S_precision` | Precision for the susceptible class (0). |
| `S_recall` | Recall for the susceptible class (0). |
| `S_F1_score` | F1 score for the susceptible class (0). |
| `R_precision` | Precision for the resistant class (1). |
| `R_recall` | Recall for the resistant class (1). |
| `R_F1_score` | F1 score for the resistant class (1). |
| `best_n_estimators` | Number of trees selected by the search. |
| `best_max_features` | Selected `max_features` value. |
| `best_max_depth` | Selected maximum tree depth. |
| `best_min_samples_split` | Selected minimum samples required to split a node. |
| `best_min_samples_leaf` | Selected minimum samples required in a leaf. |

Precision, recall, and F1 score are calculated separately for each class. Undefined class-specific metrics are set to zero.

### 3.2. Feature importance

**Location:** `results/features_importance/feature_importance_<dataset_name>.csv`

Feature importance is saved in **long format**, with one row per feature per run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `feature` | Name of the genomic feature. |
| `importance` | Feature importance reported by the fitted Random Forest. |

Example:

```csv
run,feature,importance
1,gene_feature_1,0.40
1,gene_feature_2,0.30
1,gene_feature_3,0.20
2,gene_feature_1,0.35
2,gene_feature_3,0.32
2,gene_feature_2,0.25
```

Feature importance values represent the relative contribution of each feature to impurity reduction in the fitted Random Forest. They sum to approximately 1 within each run.

Comparing feature importance across runs can help identify genomic features that consistently contribute to classification and assess the stability of their relative importance.

### 3.3. Predicted classes

**Location:** `results/predicted_classes/predicted_classes_<dataset_name>.csv`

One row is generated for each test sample in each run.

| Column | Description |
|---|---|
| `run` | Run number. |
| `sample_id` | Identifier of the test sample. |
| `real_class` | Observed class from `outcome`. |
| `predicted_class` | Class predicted by the fitted Random Forest. |

These predictions can be used to examine classification performance at the sample level, identify recurrent misclassifications, and compare predictions across repeated runs. A sample may appear in multiple runs because the test sets are generated independently.

## 4. Installation and Execution

Python 3 and the following packages are required:

```bash
pip install pandas scipy scikit-learn
```

Place one or more input CSV files in `data/` and run the script from the repository:

```bash
python scripts/random_forest.py
```

The script processes every CSV file in `data/` and writes the results to the corresponding subdirectories under `results/`.

## 5. Recommended Use

This workflow is designed to evaluate how well genomic features classify antimicrobial resistance phenotypes and to explore which features contribute to model predictions.

The output files support three complementary analyses:

- **Predictive performance:** use the metrics across runs to assess classification performance and its variability across different data partitions.
- **Genomic feature prioritization:** compare feature importance across runs to identify features that repeatedly contribute to the model.
- **Sample-level evaluation:** examine predicted and observed classes to identify patterns in misclassification and assess prediction consistency.

For antimicrobial resistance studies, these results can help prioritize candidate genomic determinants for further investigation and support the interpretation of genotype–phenotype relationships. Feature importance is most informative when considered alongside biological knowledge, genomic context, and independent evidence.

The workflow can be adapted to different organisms, antimicrobial agents, and genomic feature sets, provided that the input data follow the required format and the target phenotype is encoded consistently.
