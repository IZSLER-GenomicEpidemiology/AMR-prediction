# Genotype-Based Antimicrobial Resistance Prediction
Exploring how machine learning can help us understand and predict antimicrobial resistance from microbial genomic data.

## Overview

This repository contains two machine learning approaches for predicting antimicrobial resistance (AMR) from genomic features:

- **Random Forest (RF):** a classification approach that predicts whether an isolate is susceptible or resistant to a given antimicrobial.
- **XGBoost:** a regression approach that predicts the quantitative minimum inhibitory concentration (MIC) of an isolate.

Both scripts use genomic features as predictors and evaluate model performance using repeated train/test splits.

The two approaches address different prediction tasks and should be selected according to the type of outcome available.

## Prediction Approaches

| Feature | Random Forest | XGBoost |
|---|---|---|
| Task | Binary classification | Regression |
| Target variable | Binary resistance phenotype | Quantitative MIC |
| Outcome encoding | 0 = susceptible, 1 = resistant | Numeric MIC expressed on a log2 scale |
| Model evaluation | Accuracy, precision, recall, and F1-score | Accuracy within ±1 MIC dilution and R² |
| Hyperparameter optimization | RandomizedSearchCV | Factorial design and GridSearchCV |

For more details about each implementation, see the dedicated documentation:

- [Random Forest documentation](scripts/README_random_forest.md)
- [XGBoost documentation](scripts/README_xgboost.md)

## Repository Structure

```text
repository/
├── README.md
├── requirements.txt
├── scripts/
│   ├── random_forest.py
│   ├── xgboost.py
│   ├── README_random_forest.md
│   └── README_xgboost.md
├── data/
└── results/
```

## Input Data

Input datasets must be provided as CSV files and placed directly in the `data/` directory.

Each CSV file represents a dataset to be processed independently.

The following columns are required:

- `sample_id`: unique identifier for each isolate.
- `outcome`: the target variable to be predicted.
- All remaining columns: genomic features used as predictors.

The meaning and encoding of `outcome` depend on the selected model:

- For Random Forest, `outcome` must represent a binary phenotype: `0` for susceptible and `1` for resistant.
- For XGBoost, `outcome` must contain numeric MIC values expressed on a log2 scale. A difference of one unit corresponds to one twofold dilution.

Genomic features should be encoded numerically, for example as binary presence/absence variables.

Make sure that the input data are appropriately prepared for the selected prediction task.

## Installation

Install the required Python packages using:

```bash
pip install -r requirements.txt
```

## Running the Scripts

Run the scripts from the repository root:

```bash
python scripts/random_forest.py
```

```bash
python scripts/xgboost.py
```

Each script automatically identifies and processes the CSV files in `data/`.

The scripts use different model-specific evaluation procedures and parameters. Consult the dedicated documentation before changing these settings.

## Output Files

All results are saved directly in the `results/` directory.

For each input dataset, the scripts generate the following results.

### Random Forest

- `accuracy_global_<dataset>.csv`: classification performance metrics for each run.
- `prediction_per_sample_<dataset>.csv`: real and predicted classes for each test sample, including an agreement indicator.
- `feature_importance_<dataset>.csv`: feature importance scores for each run.

### XGBoost

- `accuracy_global_<dataset>.csv`: global prediction performance for each run.
- `prediction_per_sample_<dataset>.csv`: real and predicted MIC values for each test sample, including an agreement indicator.
- `accuracy_per_dilution_<dataset>.csv`: prediction performance grouped by real MIC value for each run.
- `feature_importance_<dataset>.csv`: feature importance scores for each run.

The `<dataset>` placeholder corresponds to the input CSV filename without its extension.

## Notes

- Both scripts evaluate predictions on held-out test sets rather than on the training data.
- The reported performance can vary across runs because the train/test split and model optimization involve randomization.
- Feature importance scores indicate the contribution assigned to predictors by the fitted model; they should not automatically be interpreted as evidence of a causal relationship.
- The `results/` directory must exist before execution.
- Output files with the same names may be overwritten when the scripts are run again on the same datasets.
