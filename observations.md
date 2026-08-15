# Model Selection Observations and Conclusions

Generated automatically from the notebook's live results — every number below comes
directly from `results_df`, `training_results_df`, and `final_test_results`.

## 1. Results table (validation set, sorted by F1)

| Model                              |   Accuracy |   Precision |   Recall |     F1 |   ROC-AUC |   Training Time (s) |   Inference Time (s) |
|:-----------------------------------|-----------:|------------:|---------:|-------:|----------:|--------------------:|---------------------:|
| Logistic Regression                |     0.824  |      0.6981 |   0.5936 | 0.6416 |    0.86   |              0.0917 |               0.0186 |
| XGBoost                            |     0.8141 |      0.6944 |   0.5348 | 0.6042 |    0.8554 |              0.1134 |               0.0187 |
| KNN (K=15)                         |     0.7906 |      0.6106 |   0.5829 | 0.5964 |    0.8268 |              0.0547 |               0.0357 |
| Decision Tree (shallow, depth=5)   |     0.7885 |      0.6086 |   0.5695 | 0.5884 |    0.8411 |              0.0574 |               0.0135 |
| Gradient Boosting                  |     0.8077 |      0.6833 |   0.5134 | 0.5863 |    0.8534 |              0.8854 |               0.0163 |
| KNN (K=5)                          |     0.7821 |      0.5944 |   0.5642 | 0.5789 |    0.7957 |              0.0512 |               0.0941 |
| SVM (with scaling)                 |     0.8048 |      0.6813 |   0.4973 | 0.575  |    0.8079 |              4.7442 |               0.5749 |
| Random Forest                      |     0.7928 |      0.6519 |   0.4706 | 0.5466 |    0.8294 |              0.5177 |               0.0662 |
| Decision Tree (deep, unrestricted) |     0.7317 |      0.4949 |   0.5241 | 0.5091 |    0.6663 |              0.0674 |               0.0131 |
| Decision Tree (underfit, depth=1)  |     0.7346 |      0      |   0      | 0      |    0.7377 |              0.053  |               0.0173 |
| SVM (without scaling)              |     0.7346 |      0      |   0      | 0      |    0.7995 |              7.6842 |               0.6341 |

## 2. Training-set results

| Model                              |   Accuracy |   Precision |   Recall |     F1 |
|:-----------------------------------|-----------:|------------:|---------:|-------:|
| Random Forest                      |     0.9983 |      0.9973 |   0.9964 | 0.9969 |
| Decision Tree (deep, unrestricted) |     0.9983 |      1      |   0.9938 | 0.9969 |
| KNN (K=5)                          |     0.8381 |      0.7115 |   0.6557 | 0.6825 |
| Gradient Boosting                  |     0.8383 |      0.7423 |   0.5986 | 0.6627 |
| XGBoost                            |     0.8301 |      0.7246 |   0.5798 | 0.6442 |
| KNN (K=15)                         |     0.8102 |      0.652  |   0.6102 | 0.6304 |
| SVM (with scaling)                 |     0.8256 |      0.7286 |   0.5459 | 0.6242 |
| Decision Tree (shallow, depth=5)   |     0.8062 |      0.6438 |   0.603  | 0.6228 |
| Logistic Regression                |     0.805  |      0.6581 |   0.5513 | 0.6    |
| Decision Tree (underfit, depth=1)  |     0.7347 |      0      |   0      | 0      |
| SVM (without scaling)              |     0.7347 |      0      |   0      | 0      |

## 3. Required experiments

**KNN — K=5 vs. K=15:**
K=5 validation F1 = 0.5789,
K=15 validation F1 = 0.5964.

**Decision Tree — shallow (depth=5) vs. deep (unrestricted):**
Shallow: train F1 = 0.6228, validation F1 = 0.5884.
Deep: train F1 = 0.9969, validation F1 = 0.5091.
The deep tree's much larger train/validation gap indicates overfitting (high variance).

**Decision Tree vs. Random Forest:**
Random Forest validation F1 = 0.5466
vs. the best single Decision Tree variant on validation.
Random Forest's ensembling reduces variance relative to the single deep tree
(train F1 = 0.9969 on training data).

**Random Forest vs. boosting (Gradient Boosting / XGBoost):**
Random Forest F1 = 0.5466,
Gradient Boosting F1 = 0.5863,
XGBoost F1 = 0.6042 (all on validation).

**SVM — with vs. without scaling:**
Without scaling: accuracy = 0.7346, F1 = 0.0000.
With scaling: accuracy = 0.8048, F1 = 0.5750.
This demonstrates why feature scaling matters for distance-based kernels.

**Underfitting vs. overfitting:**
- Underfit model: Decision Tree (depth=1) — train F1 = 0.0000, validation F1 = 0.0000. Poor on both — high bias.
- Overfit model: Decision Tree (unrestricted depth) — train F1 = 0.9969, validation F1 = 0.5091. Large gap — high variance.

**Class imbalance:**
Target distribution: No = 73.5%, Yes = 26.5%.
Stratified splits were used throughout. Models scoring zero recall on the minority
class in this run: ['Decision Tree (underfit, depth=1)', 'SVM (without scaling)'].
This is why accuracy alone is insufficient and precision/recall/F1/ROC-AUC are reported
for every model.

## 4. Required observations

- **Best on training data (F1):** Random Forest
- **Best on validation data (F1):** Logistic Regression
- **Selected & generalization on the held-out test set:** Logistic Regression
  — test Accuracy = 0.7878, Precision = 0.6176,
  Recall = 0.5267, F1 = 0.5685, ROC-AUC = 0.8324.
- **Most interpretable algorithm:** Logistic Regression — high (coefficients map directly to feature effects).
  General interpretability ranking (high → low): Logistic Regression / shallow Decision Tree
  > Random Forest / Gradient Boosting / XGBoost (feature importances only) > KNN / SVM (no
  explicit decision boundary).
- **Fastest at inference:** Decision Tree (deep, unrestricted) (0.01314s on the validation set).
- **If explainability is the requirement:** Logistic Regression or the shallow Decision Tree
  — both expose a direct, human-readable relationship between features and predictions.
- **If predictive performance is the primary objective:** Logistic Regression (highest validation F1).
- **Signs of high bias / high variance observed:**
  - High bias (underfitting): Decision Tree (depth=1) — flat, near-identical low
    train/validation performance.
  - High variance (overfitting): Decision Tree (unrestricted depth) and, to a lesser
    extent, Random Forest — both show train F1 far above validation F1
    (largest gap: Decision Tree (deep, unrestricted), gap = 0.4878).

## 5. Final model choice

**Logistic Regression** was selected based on validation F1 and evaluated once on the
held-out test set (Accuracy = 0.7878, F1 = 0.5685,
ROC-AUC = 0.8324). The drop from validation to test performance
is expected and reflects genuine generalization to unseen data, since the test set was
never used during model selection.
