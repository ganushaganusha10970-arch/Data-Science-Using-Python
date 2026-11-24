
# EXPERIMENT 11 – List and Tuple Operations

## AIM:

To manage a pharmacy inventory using lists and tuples.

## QUESTION:

A pharmacy stores a fixed product catalogue (code, name, price) that must not change, and stock quantities that change every day. Use tuples for the immutable catalogue and lists for the changing stock, and combine them to produce inventory reports.

The program must:

1. Create a tuple of product records and demonstrate indexing, slicing, unpacking, count and index.
2. Show that tuples are immutable.
3. Use list methods append, extend, insert, remove, pop, sort, reverse, copy and clear.
4. Use list comprehensions to update stock and to build inventory records.
5. Identify items to reorder, compute total stock value and summarise a nested sales list.

## ALGORITHM:

1. Create a tuple containing product codes, names and prices.
2. Demonstrate tuple indexing, slicing, unpacking, count and index operations.
3. Show tuple immutability using an assignment attempt.
4. Create a list to store changing stock quantities.
5. Update stock after restocking and daily sales using list operations and comprehensions.
6. Demonstrate append, extend, insert, remove, pop, sort and reverse methods.
7. Combine product tuples and stock lists to generate inventory records.
8. Identify products that need reordering and calculate their stock values.
9. Calculate total stock value and find the highest-value item.
10. Use a nested list to calculate weekly sales and demonstrate copy and clear operations.

## CODE:

```python
print("Pharmacy Inventory Management: Lists and Tuples")
print("=" * 50)

# ---------- Tuples: fixed product records (immutable) ----------
# (product_code, name, unit_price)
products = (
    ("P101", "Paracetamol", 25.0),
    ("P102", "Amoxicillin", 120.0),
    ("P103", "Cetirizine", 40.0),
    ("P104", "Vitamin D3", 210.0),
    ("P105", "Insulin", 450.0),
)

print("\nProduct catalogue (tuple of tuples):")
for code, name, price in products:
    print(f" {code} {name:<12} Rs.{price:>7.2f}")

print("Number of products :", len(products))
print("Second product      :", products[1])
print("Codes (slice)       :", tuple(p[0] for p in products[:3]))

print(
    "Count / index      :",
    products.count(products[0]),
    "/",
    products.index(("P103", "Cetirizine", 40.0)),
)

try:
    products[0][2] = 30.0
except TypeError as e:
    print("Attempt to change a tuple ->", "TypeError:", e)

# ---------- Lists: stock that changes every day ----------
stock = [120, 35, 80, 15, 8]
print("\nInitial stock list :", stock)

stock[3] += 50
print("After restocking P104 (+50):", stock)

sold_today = [20, 10, 5, 12, 3]
stock = [s - d for s, d in zip(stock, sold_today)]
print("After today's sales:", stock)

expiring = ["Amoxicillin", "Insulin"]
expiring.append("Cetirizine")
expiring.extend(["Paracetamol"])
expiring.insert(0, "Vitamin D3")

print("\nExpiring list after append/extend/insert:", expiring)

expiring.remove("Paracetamol")
last = expiring.pop()

print("After remove('Paracetamol') and pop() ->", expiring, "| popped:", last)

expiring.sort()
print("Sorted             :", expiring)

expiring.reverse()
print("Reversed           :", expiring)

# ---------- Combining lists and tuples ----------
inventory = [
    (code, name, price, qty)
    for (code, name, price), qty in zip(products, stock)
]

reorder = [name for _, name, _, qty in inventory if qty < 30]
value = [round(price * qty, 2) for _, _, price, qty in inventory]

print("\nInventory records (list of tuples):")
for item, v in zip(inventory, value):
    print(f" {item} stock value = Rs.{v:,.2f}")

print("Items to reorder (qty < 30):", reorder)
print("Total stock value : Rs.{:,.2f}".format(sum(value)))
print("Max / Min quantity :", max(stock), "/", min(stock))

by_value = sorted(inventory, key=lambda t: t[2] * t[3], reverse=True)
print("Highest value item :", by_value[0][1])

# ---------- Nested list as a weekly sales matrix ----------
weekly = [
    [20, 10, 5, 12, 3],
    [18, 8, 7, 10, 2],
    [25, 12, 6, 9, 4]
]

totals = [sum(day[i] for day in weekly) for i in range(len(products))]

print("\nWeekly sales matrix (3 days x 5 products):", weekly)
print("Units sold per product:", dict(zip([p[1] for p in products], totals)))

copy_list = stock.copy()
copy_list.clear()

print("Copied then cleared list:", copy_list, "| original stock still:", stock)
```

## OUTPUT:

```text
Pharmacy Inventory Management: Lists and Tuples
==================================================

Product catalogue (tuple of tuples):
 P101 Paracetamol Rs.  25.00
 P102 Amoxicillin Rs. 120.00
 P103 Cetirizine Rs.  40.00
 P104 Vitamin D3 Rs. 210.00
 P105 Insulin Rs. 450.00
Number of products : 5
Second product      : ('P102', 'Amoxicillin', 120.0)
Codes (slice)       : ('P101', 'P102', 'P103')
Count / index      : 1 / 2
Attempt to change a tuple -> TypeError: 'tuple' object does not support item assignment

Initial stock list : [120, 35, 80, 15, 8]
After restocking P104 (+50): [120, 35, 80, 65, 8]
After today's sales: [100, 25, 75, 53, 5]

Expiring list after append/extend/insert: ['Vitamin D3', 'Amoxicillin', 'Insulin', 'Cetirizine', 'Paracetamol']
After remove('Paracetamol') and pop() -> ['Vitamin D3', 'Amoxicillin', 'Insulin'] | popped: Cetirizine
Sorted             : ['Amoxicillin', 'Insulin', 'Vitamin D3']
Reversed           : ['Vitamin D3', 'Insulin', 'Amoxicillin']

Inventory records (list of tuples):
 ('P101', 'Paracetamol', 25.0, 100) stock value = Rs.2,500.00
 ('P102', 'Amoxicillin', 120.0, 25) stock value = Rs.3,000.00
 ('P103', 'Cetirizine', 40.0, 75) stock value = Rs.3,000.00
 ('P104', 'Vitamin D3', 210.0, 53) stock value = Rs.11,130.00
 ('P105', 'Insulin', 450.0, 5) stock value = Rs.2,250.00
Items to reorder (qty < 30): ['Amoxicillin', 'Insulin']
Total stock value : Rs.21,880.00
Max / Min quantity : 100 / 5
Highest value item : Vitamin D3

Weekly sales matrix (3 days x 5 products): [[20, 10, 5, 12, 3], [18, 8, 7, 10, 2], [25, 12, 6, 9, 4]]
Units sold per product: {'Paracetamol': 63, 'Amoxicillin': 30, 'Cetirizine': 18, 'Vitamin D3': 31, 'Insulin': 9}
Copied then cleared list: [] | original stock still: [100, 25, 75, 53, 5]
```

## RESULT:

Thus, the pharmacy inventory management system was successfully implemented using Python lists and tuples. Tuple immutability, list operations, list comprehensions, stock valuation, reorder identification and weekly sales summarisation were demonstrated successfully.
