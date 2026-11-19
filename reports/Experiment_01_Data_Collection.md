# DATA SCIENCE USING PYTHON

## EXPERIMENT 1 - Data Collection, Storage and Retrieval from Files

### AIM:

To collect retail order data from multiple file sources, integrate it, store it in different formats and retrieve it for analysis.

### QUESTION:

A consumer-electronics retailer receives order data from three channels: the in-store billing system exports a CSV file, the online store returns JSON from its web API, and the mobile app writes a pipe-separated text log. As the first step of the data science process, the analytics team must collect these files into one dataset, store the result in commonly used formats and retrieve it for simple business queries.

The program must:

1. Create and read data from CSV, JSON and plain-text files using Python and pandas.
2. Integrate the three sources into a single DataFrame and derive an order amount column.
3. Store the integrated dataset as CSV, JSON and Excel files and report their sizes.
4. Retrieve the stored data and compute revenue by branch and high-value orders.
5. Explain the role of data collection and storage in the data science process.

### ALGORITHM:

1. Import the required libraries: CSV, JSON, OS and pandas.
2. Create a data folder to store the input and output files.
3. Collect retail order data from three sources: CSV, JSON and text files.
4. Read the CSV, JSON and text files using pandas.
5. Combine the three datasets into a single DataFrame using concatenation.
6. Calculate the order amount by multiplying quantity by unit price.
7. Display the integrated dataset, its shape and data types.
8. Store the integrated dataset in CSV, JSON and Excel formats.
9. Retrieve the stored Excel dataset using pandas.
10. Calculate revenue by branch and display orders with an amount greater than 10000.

### CODE:

```python
import csv
import json
import os
import pandas as pd

os.makedirs("data", exist_ok=True)

# ---------- Step 1: Data collection from three different sources ----------
# Source 1: CSV file exported from the in-store billing (POS) system

pos_rows = [
    ["order_id", "customer", "product", "qty", "unit_price", "branch"],
    [101, "Anitha", "Laptop", 1, 54000, "Chennai"],
    [102, "Rahul", "Mouse", 3, 450, "Chennai"],
    [103, "Priya", "Keyboard", 2, 1200, "Chennai"],
    [104, "Karthik", "Monitor", 1, 9500, "Chennai"],
]

with open("data/pos_orders.csv", "w", newline="") as f:
    csv.writer(f).writerows(pos_rows)

# Source 2: JSON response from the online store (web API)

web_orders = [
    {
        "order_id": 201,
        "customer": "Divya",
        "product": "Laptop",
        "qty": 2,
        "unit_price": 52500,
        "branch": "Online",
    },
    {
        "order_id": 202,
        "customer": "Arjun",
        "product": "Headset",
        "qty": 1,
        "unit_price": 2300,
        "branch": "Online",
    },
    {
        "order_id": 203,
        "customer": "Meena",
        "product": "Mouse",
        "qty": 5,
        "unit_price": 430,
        "branch": "Online",
    },
]

with open("data/web_orders.json", "w") as f:
    json.dump(web_orders, f, indent=2)

# Source 3: Plain-text log written by the mobile app (pipe separated)

log_lines = [
    "301|Suresh|Monitor|2|9300|Mobile",
    "302|Lakshmi|Keyboard|1|1150|Mobile",
    "303|Vignesh|Laptop|1|55000|Mobile",
]

with open("data/mobile_orders.txt", "w") as f:
    f.write("\n".join(log_lines))

print("Retail Order Data: Collection, Storage and Retrieval")
print("=" * 55)

# ---------- Step 2: Reading the data back from each file ----------

df_pos = pd.read_csv("data/pos_orders.csv")
print("\n[1] CSV source -> rows:", len(df_pos))
print(df_pos.to_string(index=False))

with open("data/web_orders.json") as f:
    df_web = pd.DataFrame(json.load(f))

print("\n[2] JSON source -> rows:", len(df_web))
print(df_web.to_string(index=False))

cols = ["order_id", "customer", "product", "qty", "unit_price", "branch"]

df_mob = pd.read_csv(
    "data/mobile_orders.txt", sep="|", header=None, names=cols
)

print("\n[3] TXT source -> rows:", len(df_mob))
print(df_mob.to_string(index=False))

# ---------- Step 3: Integration into a single dataset ----------

orders = pd.concat([df_pos, df_web, df_mob], ignore_index=True)
orders["amount"] = orders["qty"] * orders["unit_price"]

print("\n--- Integrated Dataset ---")
print(orders.to_string(index=False))
print("\nShape        :", orders.shape)
print("Data types   :")
print(orders.dtypes.to_string())

# ---------- Step 4: Storage in different formats ----------

orders.to_csv("data/all_orders.csv", index=False)
orders.to_json("data/all_orders.json", orient="records", indent=2)
orders.to_excel("data/all_orders.xlsx", index=False)

print("\nFiles stored :")

for name in ["all_orders.csv", "all_orders.json", "all_orders.xlsx"]:
    print(f" data/{name:<16} {os.path.getsize('data/' + name):>6} bytes")

# ---------- Step 5: Retrieval and a quick query ----------

back = pd.read_excel("data/all_orders.xlsx")

print("\nRetrieved from Excel, rows:", len(back))
print("\nRevenue by branch:")

print(
    back.groupby("branch")["amount"]
    .sum()
    .sort_values(ascending=False)
    .to_string()
)

print("\nOrders with amount > 10000:")

print(
    back.loc[
        back["amount"] > 10000,
        ["order_id", "customer", "product", "amount"]
    ].to_string(index=False)
)
```

### OUTPUT:

```text
Retail Order Data: Collection, Storage and Retrieval
=======================================================

[1] CSV source -> rows: 4
 order_id customer product    qty  unit_price branch
      101   Anitha   Laptop     1       54000 Chennai
      102    Rahul    Mouse     3         450 Chennai
      103    Priya Keyboard     2        1200 Chennai
      104  Karthik  Monitor     1        9500 Chennai

[2] JSON source -> rows: 3
 order_id customer product  qty  unit_price branch
      201    Divya  Laptop    2       52500 Online
      202    Arjun Headset    1        2300 Online
      203    Meena   Mouse    5         430 Online

[3] TXT source -> rows: 3
 order_id customer  product  qty  unit_price branch
      301   Suresh  Monitor    2        9300 Mobile
      302  Lakshmi Keyboard    1        1150 Mobile
      303  Vignesh   Laptop    1       55000 Mobile

--- Integrated Dataset ---
 order_id customer  product    qty  unit_price  branch  amount
      101   Anitha   Laptop      1       54000 Chennai   54000
      102    Rahul    Mouse      3         450 Chennai    1350
      103    Priya Keyboard      2        1200 Chennai    2400
      104  Karthik  Monitor      1        9500 Chennai    9500
      201    Divya   Laptop      2       52500  Online  105000
      202    Arjun  Headset      1        2300  Online    2300
      203    Meena    Mouse      5         430  Online    2150
      301   Suresh  Monitor      2        9300  Mobile   18600
      302  Lakshmi Keyboard      1        1150  Mobile    1150
      303  Vignesh   Laptop      1       55000  Mobile   55000

Shape        : (10, 7)

Data types:
order_id       int64
customer         str
product          str
qty            int64
unit_price     int64
branch           str
amount         int64

Files stored :
 data/all_orders.csv      449 bytes
 data/all_orders.json    1565 bytes
 data/all_orders.xlsx    5353 bytes

Retrieved from Excel, rows: 10

Revenue by branch:
branch
Online     109450
Mobile      74750
Chennai     67250

Orders with amount > 10000:
 order_id customer product  amount
      101   Anitha  Laptop   54000
      201    Divya  Laptop  105000
      301   Suresh Monitor   18600
      303  Vignesh  Laptop   55000
```

### RESULT:

Thus, retail order data was successfully collected from CSV, JSON and text files, integrated into a single DataFrame, and stored in CSV, JSON and Excel formats. The stored data was retrieved successfully, and branch-wise revenue and high-value orders were calculated.

Data collection gathers information from multiple sources, while data storage preserves the integrated dataset for future retrieval, analysis and decision-making.