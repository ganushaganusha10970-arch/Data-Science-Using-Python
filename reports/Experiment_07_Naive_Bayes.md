
# DATA SCIENCE USING PYTHON

## EXPERIMENT 7 - Naive Bayes and Memorization Methods for Classification

### AIM:

To compare Naive Bayes with memorization methods (k-Nearest Neighbours and Decision Tree) and select the best model.

### QUESTION:

A winery wants to identify the cultivar of a wine sample from 13 chemical measurements (UCI Wine dataset). Train a Gaussian Naive Bayes classifier and two memorization methods, k-Nearest Neighbours and a Decision Tree, then choose the best model using validation.

The program must:

1. Load the Wine dataset and create a stratified train/test split.
2. Train Gaussian Naive Bayes, kNN (k = 5) and Decision Tree (depth = 3) models.
3. Compare training accuracy, test accuracy and 5-fold cross-validation accuracy.
4. Report the confusion matrix and classification report of the selected model.
5. Study the effect of k in kNN and display the rules learnt by the decision tree.

### ALGORITHM:

1. Import NumPy, pandas and the required Scikit-learn libraries.
2. Load the UCI Wine dataset using load_wine().
3. Separate the 13 chemical features and target classes.
4. Display the number of samples, features and class distribution.
5. Split the dataset into training and testing sets using stratified sampling.
6. Create Gaussian Naive Bayes, k-Nearest Neighbours and Decision Tree classifiers.
7. Train each model using the training dataset.
8. Calculate training accuracy and test accuracy for each model.
9. Perform 5-fold stratified cross-validation.
10. Compare the models using their mean cross-validation accuracy and test accuracy.
11. Select the best-performing model.
12. Generate its confusion matrix and classification report.
13. Evaluate kNN using different values of k.
14. Display the decision tree rules using export_text().

### CODE:

```python
import numpy as np
import pandas as pd
from sklearn.datasets import load_wine
from sklearn.model_selection import (
    train_test_split,
    cross_val_score,
    StratifiedKFold,
)
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline
from sklearn.naive_bayes import GaussianNB
from sklearn.neighbors import KNeighborsClassifier
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.metrics import (
    accuracy_score,
    classification_report,
    confusion_matrix,
)

wine = load_wine()
X = pd.DataFrame(wine.data, columns=wine.feature_names)
y = wine.target

print("Wine Quality Classification: Naive Bayes vs Memorization Methods")
print("=" * 65)
print(
    "Samples:",
    X.shape[0],
    "| Features:",
    X.shape[1],
    "| Classes:",
    list(wine.target_names),
)
print("Class counts:", np.bincount(y).tolist())

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, stratify=y, random_state=42
)

models = {
    "Gaussian Naive Bayes": GaussianNB(),
    "k-Nearest Neighbours (k=5)": make_pipeline(
        StandardScaler(), KNeighborsClassifier(n_neighbors=5)
    ),
    "Decision Tree (depth=3)": DecisionTreeClassifier(
        max_depth=3, random_state=42
    ),
}

kfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
summary = []

for name, model in models.items():
    model.fit(X_train, y_train)
    train_acc = accuracy_score(y_train, model.predict(X_train))
    test_acc = accuracy_score(y_test, model.predict(X_test))
    cv = cross_val_score(model, X, y, cv=kfold)
    summary.append([name, train_acc, test_acc, cv.mean(), cv.std()])

table = pd.DataFrame(
    summary, columns=["Model", "Train Acc", "Test Acc", "CV Mean", "CV Std"]
)

print("\nModel comparison:")
print(table.round(4).to_string(index=False))

best = table.sort_values(["CV Mean", "Test Acc"], ascending=False).iloc[0][
    "Model"
]

print("\nSelected model (highest CV accuracy):", best)

pred = models[best].predict(X_test)

print("\nConfusion matrix of selected model:")
print(confusion_matrix(y_test, pred))

print("\nClassification report:")
print(classification_report(y_test, pred, target_names=wine.target_names))

# ---------- Effect of k in kNN (memorization) ----------

print("Effect of k on kNN (5-fold CV accuracy):")

for k in [1, 3, 5, 7, 11, 15]:
    knn = make_pipeline(
        StandardScaler(), KNeighborsClassifier(n_neighbors=k)
    )
    print(f" k = {k:>2} -> {cross_val_score(knn, X, y, cv=kfold).mean():.4f}")

# ---------- Rules learnt by the decision tree ----------

print("\nDecision tree rules:")
print(
    export_text(models["Decision Tree (depth=3)"], feature_names=list(X.columns))
)
```

### OUTPUT:

```text
Wine Quality Classification: Naive Bayes vs Memorization Methods
=================================================================

Samples: 178 | Features: 13 | Classes: ['class_0', 'class_1', 'class_2']
Class counts: [59, 71, 48]

Model comparison:
                     Model  Train Acc  Test Acc  CV Mean  CV Std
      Gaussian Naive Bayes     0.9677    1.0000   0.9719  0.0252
k-Nearest Neighbours (k=5)    0.9839    0.9444   0.9717  0.0181
   Decision Tree (depth=3)    0.9919    0.9630   0.8763  0.0465

Selected model (highest CV accuracy): Gaussian Naive Bayes

Confusion matrix of selected model:
[[18  0  0]
 [ 0 21  0]
 [ 0  0 15]]

Classification report:
              precision    recall  f1-score   support

     class_0       1.00      1.00      1.00        18
     class_1       1.00      1.00      1.00        21
     class_2       1.00      1.00      1.00        15

    accuracy                           1.00        54
   macro avg       1.00      1.00      1.00        54
weighted avg       1.00      1.00      1.00        54

Effect of k on kNN (5-fold CV accuracy):
 k =  1 -> 0.9551
 k =  3 -> 0.9662
 k =  5 -> 0.9717
 k =  7 -> 0.9719
 k = 11 -> 0.9662
 k = 15 -> 0.9776

Decision tree rules:
|--- color_intensity <= 3.82
|   |--- ash <= 3.00
|   |   |--- od280/od315_of_diluted_wines <= 3.73
|   |   |   |--- class: 1
|   |   |--- od280/od315_of_diluted_wines > 3.73
|   |   |   |--- class: 0
|   |--- ash > 3.00
|   |   |--- class: 0
|--- color_intensity > 3.82
|   |--- flavanoids <= 1.58
|   |   |--- alcalinity_of_ash <= 17.65
|   |   |   |--- class: 1
|   |   |--- alcalinity_of_ash > 17.65
|   |   |   |--- class: 2
|   |--- flavanoids > 1.58
|   |   |--- proline <= 724.50
|   |   |   |--- class: 1
|   |   |--- proline > 724.50
|   |   |   |--- class: 0
```

### RESULT:

Thus, Gaussian Naive Bayes, k-Nearest Neighbours and Decision Tree classifiers were successfully trained and evaluated using the UCI Wine dataset.

Gaussian Naive Bayes was selected as the best model based on its highest cross-validation accuracy of 0.9719. It achieved 1.0000 test accuracy with a correctly classified confusion matrix.

The experiment also demonstrated the effect of different k values in kNN and displayed the decision rules learned by the Decision Tree classifier.
```

This report follows the question, program and reference results from pages 9–12 of your PDF. <FileCite ref_id="turn12file0"/>

## Step 3 — Save and upload to GitHub

Press **Ctrl + S**.

There is no graph image to upload for this experiment.

Run these three commands separately in your existing PowerShell terminal.

**1. Add the files**

```powershell
git add experiments/Experiment_07_Naive_Bayes.py reports/Experiment_07_Naive_Bayes.md
```

**2. Commit**

```powershell
git commit -m "Add Experiment 7 Naive Bayes classification report"
```

**3. Push**

```powershell
git push
```

## Step 4 — Verify your GitHub repository

[Open Data-Science-Using-Python](https://github.com/ganushaganusha10970-arch/Data-Science-Using-Python)

Your repository should now contain Experiments 1–7.

Once the push succeeds, send me a screenshot. We'll then start **Experiment 8 — K-Means Clustering for Customer Segmentation**, the final experiment in your new PDF. ❤️