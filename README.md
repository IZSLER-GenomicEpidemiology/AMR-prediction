# Genotype-Based Antimicrobial Resistance Prediction

This repository provides two complementary machine learning approaches to predict antimicrobial resistance (AMR) phenotypes from microbial genomic features and identify the genomic features contributing to model predictions.

## Overview

The repository includes two models, designed for different prediction goals:

- **Random Forest:** predicts whether a microbial isolate is susceptible or resistant to a given antimicrobial.
- **XGBoost:** predicts the minimum inhibitory concentration (MIC).

## Choosing the Right Approach

| | Random Forest | XGBoost |
|---|---|---|
| **Goal** | Classify isolates as susceptible or resistant | Predict quantitative MIC values |
| **Prediction** | Susceptible (0) or resistant (1) | MIC on a log2 scale |
| **Performance evaluation** | Accuracy, precision, recall, and F1-score | Accuracy within ±1 MIC dilution and R² |
| **Hyperparameter optimization** | Randomized search with cross-validation | Factorial design and grid search |

For implementation details, configurable parameters, and output descriptions, see the dedicated documentation:

- [Random Forest documentation](scripts/README_random_forest.md)
- [XGBoost documentation](scripts/README_xgboost.md)

## Inputs and Outputs

The workflows use genomic data and antimicrobial resistance measurements to evaluate predictive performance and explore the contribution of genomic features.

**Inputs:** CSV datasets must be placed in the `data/` directory. Each dataset contains isolate identifiers, an antimicrobial resistance outcome, and genomic features. Depending on the selected model, the outcome represents either a binary susceptible/resistant phenotype or a quantitative MIC value.

**Outputs:** the scripts generate model performance metrics, predictions for individual test samples, and feature importance scores. XGBoost also provides a summary of prediction accuracy for each observed MIC value. All results are saved in the `results/` directory.

## Recommended Use

- **Choose the model according to your objective.** Use Random Forest to classify isolates as susceptible or resistant, and XGBoost to predict quantitative MIC values.
- **Set parameters carefully.** Consider available computational resources when configuring search settings. More extensive searches may improve model evaluation but increase computational costs.
- **Assess performance across repeated runs.** Results can vary with train/test partitions and hyperparameter optimization, so consider this variability when evaluating model performance.
- **Explore feature importance with care.** Features that repeatedly contribute to predictions may be useful candidates for further investigation, but feature importance alone does not demonstrate a causal relationship with antimicrobial resistance.
- **Prepare datasets consistently.** Use appropriate genomic feature encoding and ensure that the outcome is compatible with the selected model.
- **Keep your results organized.** The scripts write their output files to `results/`. Existing files with matching names may be overwritten when the scripts are run again, so preserve any results you want to retain.

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

## Installation

Make sure Python is installed, then clone or download this repository and navigate to its root directory.

Install the required Python packages using:

```bash
pip install -r requirements.txt
```


## Running the Scripts

Run the commands below from the repository root.

**Random Forest**

```bash
python scripts/random_forest.py
```

**XGBoost**

```bash
python scripts/xgboost.py
```

Each script automatically processes the CSV files in `data/` and saves the corresponding results in `results/`. Make sure both directories exist and that `results/` is available before running the scripts.

Before running either workflow, consult its dedicated README for the required input format and model-specific settings.
