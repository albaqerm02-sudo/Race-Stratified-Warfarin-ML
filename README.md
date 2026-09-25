# AI-Based Clinical Decision Support System for Precision Warfarin Dosing

**Grant:** TÜBİTAK 2209-A Academic R&D Project  
**Domain:** Pharmacogenomics · Clinical machine learning · Race-stratified dosing · Algorithmic fairness

## Overview

This project investigates machine-learning methods for predicting the stable maintenance dose of warfarin, a widely used anticoagulant with a notoriously narrow therapeutic index. It deliberately introduces a paradigm shift from a monolithic global modeling approach to a **Race-Stratified Architecture** across four demographic cohorts: **White**, **Asian**, **Black**, and **Unknown**. The system combines cohort-specific preprocessing, localized missing-data imputation, advanced pharmacogenetic feature engineering (including group-mean centering), feature ablation, and hyperparameter-optimized regressors to mitigate algorithmic bias and improve clinical safety margins.

**Scope:** Academic research and R&D prototype. Model predictions are not independently validated clinical diagnoses and should not be used for patient care without physician oversight.

## Dataset and access

The training and evaluation data are derived from the **International Warfarin Pharmacogenetics Consortium (IWPC)** dataset. Each row describes a patient using clinical, demographic, anthropometric, and pharmacogenetic features, with the target variable being the stable weekly warfarin dose (mg/week). 

The key feature groups include:
*   **Clinical & Demographic:** Age, Gender, Weight, Height, Comorbidities, Target INR, Concomitant medications (e.g., Amiodarone, Statins).
*   **Pharmacogenetic:** `CYP2C9` consensus haplotypes and `VKORC1 -1639G>A` genotypes.

### Final analytical cohort composition

After removing records with missing target values (stable dose), the analytical cohort of 5,528 patients was stratified as follows:

| Cohort | Total Patients |
|:--|--:|
| White | 2,969 |
| Asian | 1,619 |
| Black / African American | 462 |
| Unknown / Unclassified | 478 |
| **Total** | **5,528** |

**Data availability:** To comply with patient privacy and ethical standards, the raw IWPC clinical and genetic CSV files are **not included in this repository**. This repository documents the dataset structure and training methodology without redistributing those files. Researchers can request official access via [PharmGKB](https://www.pharmgkb.org/). A `sample_data.csv` containing randomized, dummy records is provided in the `data/` folder for code testing and reproduction purposes.

## Methodology

The final training workflow is implemented in the Jupyter notebooks within the `notebooks/` directory. The pipeline evaluates the dataset first using a global monolithic baseline (Stacking Ensemble), followed by the superior race-stratified pipelines.

### Preprocessing and feature engineering

1. **Genetic Imputation:** Missing `VKORC1 -1639G>A` genotypes were imputed using Linkage Disequilibrium (LD) logic based on available auxiliary `VKORC1` variants.
2. **Missing Data Handling:** Deployed **MICE** for the White cohort to leverage multivariate relationships, and distance-weighted **KNN Imputer** ($k=5$) for Asian and Black cohorts to resolve localized non-random missingness (e.g., unreported cardiovascular medications).
3. **Data Transformation:** Applied $log_{1p}$ transformation to dose and weight distributions to handle right-skewness. Continuous variables were scaled using `RobustScaler`.
4. **Encoding:** Categorical genetic variables were handled via Ordinal Encoding for tree-based models and drop-first One-Hot Encoding (OHE) for linear models to avoid the dummy variable trap.
5. **Feature Engineering:** 
   * Calculated BMI and BSA.
   * Created an explicit pharmacogenetic-anthropometric cross-term (`VKORC1 × BSA`).
   * Applied **Group-Mean Centering** ($X_{centered} = X_i - X_{group}$) to isolate individual variations from genetic baseline traits.

### Cohort-specific models

| Cohort | Final Regressor | Key Preprocessing / Architectural Choice |
|:--|:--|:--|
| **White** | CatBoost | Leveraging symmetric trees to capture complex non-linear interactions across a large, high-variance dataset. |
| **Asian** | CatBoost | KNN imputation coupled with tree-based learning to map highly sensitive, low-dose physiological profiles. |
| **Black** | L2-Regularized Ridge | Strict penalization to prevent coefficient explosion caused by sparse categorical OHE genetics, avoiding severe overfitting. |
| **Unknown** | SVR (RBF Kernel) | **Feature Ablation:** Systematic removal of the 'Weight' feature to eliminate unstable weight-to-genotype assumptions in unclassified demographics. |

## Evaluation

The primary clinical safety metric is **PW20** (Percentage of predictions within $\pm20\%$ of the actual therapeutic dose). The overall evaluation demonstrates how the global monolithic model suffers from "accuracy dilution" and underperforms on minorities (e.g., Black PW20 = 41.77%), whereas the race-stratified models eliminate this cross-talk.

| Cohort | Optimal Algorithm | Test MAE (mg/week) | Test PW20 (%) | Test $R^2$ (%) |
|:--|:--|--:|--:|--:|
| **White** | CatBoost Regressor | 8.75 | 51.18% | 39.50% |
| **Asian** | CatBoost Regressor | 6.23 | 47.22% | 35.63% |
| **Black** | Ridge Regression | 10.80 | 52.69% | 17.37% |
| **Unknown** | SVR (RBF Kernel) | 7.23 | 61.46% | 41.43% |

*Note: Results reflect the isolated test set evaluation (80/20 split) following Isolation Forest outlier filtering and Optuna hyperparameter tuning.*

## Repository structure

```text
.
├── data/                             # Contains sample_data.csv and README_DATA.md
├── models/                           # Trained artifacts (pkl files)
│   ├── Warfarin_White_Package.pkl    
│   ├── Warfarin_Asian_Package.pkl    
│   ├── Warfarin_Black_Package.pkl    
│   └── Warfarin_Unknown_Package.pkl  
├── notebooks/                        # Training, OOF evaluation, and EDA
│   ├── 01_Data_Preprocessing.ipynb
│   ├── 02_Global_Baseline_Model.ipynb
│   └── 03_Race_Stratified_Optuna.ipynb
├── requirements.txt                  # Python dependencies for environment reproduction
├── README.md
└── LICENSE                           # MIT License
