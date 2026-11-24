# EXPERIMENT 16 – Relational Join and Aggregation using MapReduce

## AIM:

To join customer and order data and compute city-wise sales using chained MapReduce jobs.

## QUESTION:

An online store keeps CUSTOMERS(cust_id, name, city) and ORDERS(order_id, cust_id, amount) as separate files in HDFS. Management wants the total revenue, number of orders, maximum order and average order value for each city. Implement this as two chained MapReduce jobs: a reduce-side join followed by grouping and aggregation.

The program must:

1. Job 1 mapper: emit cust_id as the key and a record tagged with its source relation.
2. Job 1 reducer: combine customer and order records with the same key (inner join).
3. Job 2 mapper: emit (city, amount) from each joined record.
4. Job 2 reducer: compute total, count, maximum and average per city.
5. Show the output of every phase and explain why an order without a matching customer is dropped.

## ALGORITHM:

1. Import the groupby function from the itertools module.
2. Create separate customer and order records.
3. Define the first mapper to emit customer IDs as keys with tagged customer or order values.
4. Define the first reducer to group matching customer and order records and perform an inner join.
5. Define the second mapper to emit city and order amount.
6. Sort and group the mapped records by city.
7. Define the second reducer to calculate total revenue, number of orders and maximum order.
8. Calculate the average order value for each city.
9. Display the results of both MapReduce jobs.
10. Calculate and display the grand total revenue.

## CODE:

```python
from itertools import groupby

customers = [
    "C1,Kavya,Chennai",
    "C2,Ravi,Bengaluru",
    "C3,Sana,Chennai",
    "C4,Tarun,Hyderabad",
    "C5,Uma,Bengaluru",
]

orders = [
    "O1,C1,2500",
    "O2,C2,1800",
    "O3,C1,700",
    "O4,C3,4200",
    "O5,C4,1500",
    "O6,C2,3100",
    "O7,C5,950",
    "O8,C3,600",
    "O9,C9,800",
]


# ---------------- Job 1: Join ----------------
def join_mapper(source, line):
    f = line.split(",")
    if source == "customers":
        return f[0], ("C", f[1], f[2])
    return f[1], ("O", f[0], int(f[2]))


def join_reducer(cust_id, values):
    cust = [v for v in values if v[0] == "C"]
    ords = [v for v in values if v[0] == "O"]

    for c in cust:
        for o in ords:
            yield cust_id, c[1], c[2], o[1], o[2]


# ---------------- Job 2: Group by city ----------------
def agg_mapper(joined_row):
    _, _, city, _, amount = joined_row
    return city, amount


def agg_reducer(city, amounts):
    return city, sum(amounts), len(amounts), max(amounts)


def shuffle(pairs):
    pairs = sorted(pairs, key=lambda kv: kv[0])
    return [
        (k, [v for _, v in g])
        for k, g in groupby(pairs, key=lambda kv: kv[0])
    ]


print("Sales Analytics: Join and Aggregation using MapReduce")
print("=" * 55)

# ---------- JOB 1: MAP ----------
print("\nJOB 1 - MAP (key = cust_id, value tagged with relation):")

mapped = [
    join_mapper("customers", line) for line in customers
] + [
    join_mapper("orders", line) for line in orders
]

for k, v in mapped:
    print(f" {k} -> {v}")

# ---------- JOB 1: SHUFFLE AND SORT ----------
print("\nJOB 1 - SHUFFLE AND SORT:")

groups = shuffle(mapped)

for k, vals in groups:
    print(f" {k}: {vals}")

# ---------- JOB 1: REDUCE ----------
print("\nJOB 1 - REDUCE (joined records):")

joined = [
    row
    for k, vals in groups
    for row in join_reducer(k, vals)
]

print(f" {'cust':<5}{'name':<7}{'city':<11}{'order':<7}amount")

for row in joined:
    print(f" {row[0]:<5}{row[1]:<7}{row[2]:<11}{row[3]:<7}{row[4]}")

print(" Note: order O9 (cust C9) has no matching customer and is dropped.")

# ---------- JOB 2: MAP ----------
print("\nJOB 2 - MAP (key = city, value = amount):")

mapped2 = [agg_mapper(r) for r in joined]
print(" ", mapped2)

# ---------- JOB 2: SHUFFLE AND SORT ----------
print("\nJOB 2 - SHUFFLE AND SORT:")

groups2 = shuffle(mapped2)

for city, vals in groups2:
    print(f" {city}: {vals}")

# ---------- JOB 2: REDUCE ----------
print("\nJOB 2 - REDUCE (aggregates per city):")

print(f" {'City':<11}{'Revenue':>8}{'Orders':>8}{'Max':>7}{'Avg':>9}")

results = [agg_reducer(c, v) for c, v in groups2]

for city, total, cnt, mx in sorted(results, key=lambda r: -r[1]):
    print(f" {city:<11}{total:>8}{cnt:>8}{mx:>7}{total / cnt:>9.2f}")

print("\nGrand total revenue:", sum(r[1] for r in results))
```

## OUTPUT:

```text
Sales Analytics: Join and Aggregation using MapReduce
=======================================================

JOB 1 - MAP (key = cust_id, value tagged with relation):
 C1 -> ('C', 'Kavya', 'Chennai')
 C2 -> ('C', 'Ravi', 'Bengaluru')
 C3 -> ('C', 'Sana', 'Chennai')
 C4 -> ('C', 'Tarun', 'Hyderabad')
 C5 -> ('C', 'Uma', 'Bengaluru')
 C1 -> ('O', 'O1', 2500)
 C2 -> ('O', 'O2', 1800)
 C1 -> ('O', 'O3', 700)
 C3 -> ('O', 'O4', 4200)
 C4 -> ('O', 'O5', 1500)
 C2 -> ('O', 'O6', 3100)
 C5 -> ('O', 'O7', 950)
 C3 -> ('O', 'O8', 600)
 C9 -> ('O', 'O9', 800)

JOB 1 - SHUFFLE AND SORT:
 C1: [('C', 'Kavya', 'Chennai'), ('O', 'O1', 2500), ('O', 'O3', 700)]
 C2: [('C', 'Ravi', 'Bengaluru'), ('O', 'O2', 1800), ('O', 'O6', 3100)]
 C3: [('C', 'Sana', 'Chennai'), ('O', 'O4', 4200), ('O', 'O8', 600)]
 C4: [('C', 'Tarun', 'Hyderabad'), ('O', 'O5', 1500)]
 C5: [('C', 'Uma', 'Bengaluru'), ('O', 'O7', 950)]
 C9: [('O', 'O9', 800)]

JOB 1 - REDUCE (joined records):
 cust name   city       order amount
 C1   Kavya  Chennai    O1     2500
 C1   Kavya  Chennai    O3     700
 C2   Ravi   Bengaluru  O2     1800
 C2   Ravi   Bengaluru  O6     3100
 C3   Sana   Chennai    O4     4200
 C3   Sana   Chennai    O8     600
 C4   Tarun  Hyderabad  O5     1500
 C5   Uma    Bengaluru  O7     950

 Note: order O9 (cust C9) has no matching customer and is dropped.

JOB 2 - MAP (key = city, value = amount):
 [('Chennai', 2500), ('Chennai', 700), ('Bengaluru', 1800), ('Bengaluru', 3100), ('Chennai', 4200), ('Chennai', 600), ('Hyderabad', 1500), ('Bengaluru', 950)]

JOB 2 - SHUFFLE AND SORT:
 Bengaluru: [1800, 3100, 950]
 Chennai: [2500, 700, 4200, 600]
 Hyderabad: [1500]

JOB 2 - REDUCE (aggregates per city):
 City        Revenue  Orders    Max      Avg
 Chennai        8000       4   4200  2000.00
 Bengaluru      5850       3   3100  1950.00
 Hyderabad      1500       1   1500  1500.00

Grand total revenue: 15350
```

## RESULT:

Thus, relational join and city-wise sales aggregation were successfully implemented using two chained MapReduce jobs. Customer and order records were joined using a reduce-side inner join, followed by city-wise aggregation. The unmatched order O9 was dropped because customer C9 was not present in the customer records. The total revenue calculated was Rs.15,350.
