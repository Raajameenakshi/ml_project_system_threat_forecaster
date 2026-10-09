# System Threat Forecaster

Predicting whether a Windows machine will be infected by malware, using system telemetry collected by antivirus software. Built for the **Machine Learning Project (MLP)**

**Competition:** [System Threat Forecaster on Kaggle](https://www.kaggle.com/t/84c52e4e6c0b4591a442913092481d5c)

## Problem

Each row in the data is one machine, described by about 75 properties: antivirus product and engine versions, OS version and build, hardware specs (RAM, disk, processor), security settings, region identifiers, and date fields. The goal is binary classification: predict `target`, which is 1 when malware was detected on the machine and 0 otherwise.

- **Metric:** accuracy
- **Training data:** 100,000 rows, 76 columns
- **Test data:** 10,000 rows, 75 columns
- **Classes:** nearly balanced (50,525 infected vs 49,475 not infected)

## Results

| Model | Description | Validation accuracy |
|---|---|---|
| 1 | AdaBoost | 0.614 |
| 2 | Voting classifier (Logistic Regression + Random Forest + XGBoost) | 0.620 |
| 3 | PCA (55 components) + Random Forest | 0.610 |
| 4 | RFE (45 features) + XGBoost | 0.628 |
| 4 (tuned) | RFE (45 features) + XGBoost with randomized hyperparameter search | **0.632** |

The final submission used the tuned XGBoost model (Model 4) and scored **0.633** accuracy on the Kaggle leaderboard, above the 0.55 cutoff. The top score in the competition was 0.696.

Validation accuracy comes from a random 80/20 split of the training data (`ran

## Approach

### 1. Exploratory data analysis
- Checked shape, data types, descriptive statistics, duplicates (167 duplicate rows), and missing values (small counts per column, for example 623 missing in `CityID`)
- Count plots for low-cardinality categorical features
- Correlation heatmap and correlation of each numerical feature with the target
- Histograms of the most correlated features

Key observations:
- The target is almost perfectly balanced, so accuracy is a fair metric.
- `AntivirusConfigID`, `TotalPhysicalRAMMB`, `IsSystemProtected`, and `IsGamer` correlate positively with infection. `IsAlwaysOnAlwaysConnectedCapable`, `IsTouchEnabled`, and `IsPassiveModeEnabled` correlate negatively.

### 2. Preprocessing
- Dropped the `MachineID` identifier
- Imputed missing values: mean for numerical features, most frequent for categorical features
- Scaled numerical features with `StandardScaler`
- Encoded categorical features with one-hot, label, or ordinal encoding, depending on the model

### 3. Models
**Model 1: AdaBoost**
A boosting model with default settings, trained on one-hot encoded features. Used as the first baseline.

**Model 2: Voting classifier**
Combines Logistic Regression, Random Forest, and XGBoost. Each model predicts a class and the majority vote becomes the final prediction.

**Model 3: PCA + Random Forest**
Categorical features are label encoded and numerical features are scaled. PCA reduces the features to 55 components, and a Random Forest is trained on them.

**Model 4: RFE + XGBoost**
Categorical features are ordinal encoded and numerical features are scaled. Recursive Feature Elimination (RFE) keeps the best 45 features, and an XGBoost model is trained on them. The model is then tuned with randomized search (10 candidates, 5-fold cross-validation) over `n_estimators`, `learning_rate`, `max_depth`, `subsample`, and `colsample_bytree`.

## Tech Stack

Python, NumPy, Pandas, Matplotlib, Seaborn, scikit-learn, XGBoost.

## Repository Contents

    ├── 21f2001081-notebook-t12025.ipynb   # EDA, preprocessing, models, submission
    └── README.md
