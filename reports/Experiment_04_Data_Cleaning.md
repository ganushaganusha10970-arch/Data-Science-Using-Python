
# DATA SCIENCE USING PYTHON

## EXPERIMENT 4 - Data Cleaning and Sampling for Modeling and Validation

### AIM:

To clean a raw customer dataset and prepare training, validation and test samples for modeling.

### QUESTION:

A telecom company has collected customer data for a churn prediction project. The raw data contains missing ages and incomes, duplicate records, inconsistent city names, invalid negative values and extreme outliers. Clean and transform the data, then draw samples suitable for model building and validation.

The program must:

1. Detect and report missing values, duplicates and inconsistent categories.
2. Detect outliers using the IQR rule and treat invalid values.
3. Impute missing values using the median and mean.
4. Encode categorical variables and standardise numeric features.
5. Draw a simple random sample and a stratified 70/15/15 train/validation/test split.
6. Explain why stratified sampling preserves the class ratio.

### ALGORITHM:

1. Import NumPy, pandas, train_test_split and StandardScaler libraries.
2. Generate a dataset containing 200 telecom customers with customer details and churn status.
3. Introduce missing values, duplicate records, invalid values and extreme outliers.
4. Display the raw dataset shape and identify missing values in each column.
5. Detect and remove duplicate records.
6. Standardise inconsistent city names using strip() and title().
7. Detect invalid negative income values and replace them with missing values.
8. Calculate the first quartile, third quartile and IQR to identify outliers.
9. Treat outliers using the IQR rule by applying clipping.
10. Impute missing age values using the median and income values using the mean.
11. Encode churn values and convert city categories using one-hot encoding.
12. Standardise numerical features using StandardScaler.
13. Draw a simple random sample containing 20% of the dataset.
14. Split the dataset into training, validation and testing sets using stratified sampling.
15. Display the number of records and churn rate for each dataset.

### CODE:

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

np.random.seed(42)
n = 200
df = pd.DataFrame(
    {
        "customer_id": range(1001, 1001 + n),
        "age": np.random.randint(18, 70, n).astype(float),
        "income": np.random.normal(45000, 12000, n).round(),
        "city": np.random.choice(
            ["Chennai", "chennai ", "Madurai", "Coimbatore"], n
        ),
        "purchases": np.random.poisson(6, n).astype(float),
        "churn": np.random.choice(["Yes", "No"], n, p=[0.3, 0.7]),
    }
)

# Inject typical data-quality problems

df.loc[np.random.choice(n, 15, replace=False), "age"] = np.nan
df.loc[np.random.choice(n, 10, replace=False), "income"] = np.nan
df.loc[[5, 50, 120], "income"] = [450000, 520000, -3000]

df = pd.concat(
    [df, df.iloc[[10, 20, 30, 40, 50]]], ignore_index=True
)

print("Customer Data: Cleaning and Sampling for Modeling")
print("=" * 50)
print("Raw shape:", df.shape)

# ---------- 1. Missing values ----------

print("\n[1] Missing values per column:")
print(df.isnull().sum().to_string())

# ---------- 2. Duplicates ----------

dups = df.duplicated().sum()
df = df.drop_duplicates()

print(f"\n[2] Duplicate rows removed: {dups}   -> shape {df.shape}")

# ---------- 3. Inconsistent categories ----------

print("\n[3] City values before:", sorted(df["city"].unique()))
df["city"] = df["city"].str.strip().str.title()

print("    City values after :", sorted(df["city"].unique()))

# ---------- 4. Invalid values and outliers (IQR rule) ----------

df.loc[df["income"] < 0, "income"] = np.nan
q1, q3 = df["income"].quantile([0.25, 0.75])
iqr = q3 - q1
low, high = q1 - 1.5 * iqr, q3 + 1.5 * iqr

outliers = df[(df["income"] < low) | (df["income"] > high)]

print(f"\n[4] Income IQR bounds: {low:.0f} to {high:.0f}")
print("    Outliers found:", len(outliers))
print(outliers[["customer_id", "income"]].to_string(index=False))

df["income"] = df["income"].clip(low, high)

# ---------- 5. Imputation ----------

df["age"] = df["age"].fillna(df["age"].median())
df["income"] = df["income"].fillna(df["income"].mean()).round()

print("\n[5] Missing values after imputation:",
      int(df.isnull().sum().sum()))

# ---------- 6. Encoding and scaling ----------

df["churn"] = df["churn"].map({"No": 0, "Yes": 1})
df = pd.get_dummies(df, columns=["city"], drop_first=True, dtype=int)

scaler = StandardScaler()
df[["age", "income", "purchases"]] = scaler.fit_transform(
    df[["age", "income", "purchases"]]
)

print("\n[6] Cleaned and transformed data (first 5 rows):")
print(df.head().round(3).to_string(index=False))

# ---------- 7. Sampling ----------

X = df.drop(columns=["customer_id", "churn"])
y = df["churn"]

simple = df.sample(frac=0.2, random_state=1)

print(
    "\n[7] Simple random sample (20%):",
    len(simple),
    "rows, churn rate =",
    round(simple["churn"].mean(), 3),
)

X_train, X_temp, y_train, y_temp = train_test_split(
    X, y, test_size=0.3, stratify=y, random_state=1
)

X_val, X_test, y_val, y_test = train_test_split(
    X_temp, y_temp, test_size=0.5, stratify=y_temp, random_state=1
)

print("\nStratified Train / Validation / Test split (70/15/15):")

for name, part in [
    ("Overall", y),
    ("Train", y_train),
    ("Validation", y_val),
    ("Test", y_test),
]:
    print(f" {name:<11} rows = {len(part):>3}    churn rate = {part.mean():.3f}")
```

### OUTPUT:

```text
Customer Data: Cleaning and Sampling for Modeling
==================================================

Raw shape: (205, 6)

[1] Missing values per column:
customer_id     0
age            16
income         10
city            0
purchases       0
churn           0

[2] Duplicate rows removed: 5   -> shape (200, 6)

[3] City values before: ['Chennai', 'Coimbatore', 'Madurai', 'chennai ']
    City values after : ['Chennai', 'Coimbatore', 'Madurai']

[4] Income IQR bounds: 11224 to 79240
    Outliers found: 4
 customer_id   income
        1006 450000.0
        1051 520000.0
        1073 10447.0
        1171 83244.0

[5] Missing values after imputation: 0

[6] Cleaned and transformed data (first 5 rows):
 customer_id    age  income  purchases  churn  city_Coimbatore  city_Madurai
        1001  0.877   0.095      0.760      0                0             1
        1002  1.785  -1.211     -0.418      0                0             0
        1003  0.179   1.821     -1.203      0                1             0
        1004 -0.799  -0.171     -0.418      0                1             0
        1005  0.039   0.000      0.367      0                0             0

[7] Simple random sample (20%): 40 rows, churn rate = 0.375

Stratified Train / Validation / Test split (70/15/15):
 Overall     rows = 200    churn rate = 0.320
 Train       rows = 140    churn rate = 0.321
 Validation  rows =  30    churn rate = 0.300
 Test        rows =  30    churn rate = 0.333
```

### RESULT:

Thus, the telecom customer dataset was successfully cleaned by identifying missing values, removing duplicates, correcting inconsistent categories and detecting outliers using the IQR method.

Missing values were imputed, categorical variables were encoded and numerical features were standardised. A simple random sample and stratified training, validation and testing datasets were successfully generated.

Stratified sampling preserves approximately the same churn class proportion in each subset, making the datasets suitable for model development and validation.
