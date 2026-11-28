# EXPERIMENT 15 – Matrix-Vector Multiplication using MapReduce
**Date:** 24-11-26
## AIM:

To compute the product of a sparse matrix and a vector using MapReduce, as used in one PageRank iteration.

## QUESTION:

A search engine ranks five web pages using PageRank, which repeatedly multiplies the link matrix M by the rank vector v. Because M is very large in practice, it is stored as (i, j, m_ij) records and the product x = Mv is computed with MapReduce. Implement the mapper and reducer and verify the result.

The program must:

1. Store the non-zero entries of M as (i, j, m_ij) records and v in a separate file.
2. Mapper: for each entry emit the key i with the value m_ij × v_j.
3. Shuffle and sort the values by row i.
4. Reducer: add the values for each i to obtain x_i.
5. Verify the result with NumPy and repeat the job to show PageRank converging.

## ALGORITHM:

1. Import the required system, groupby and NumPy modules.
2. Define a function to load the PageRank vector from a separate file.
3. Define the mapper to multiply each non-zero matrix entry by its corresponding vector value.
4. Define the reducer to group values by row and calculate their sum.
5. Create a 5 × 5 link matrix and an initial PageRank vector.
6. Store the non-zero matrix entries in `matrix.txt` and the vector values in `vector.txt`.
7. Divide the matrix records into two input splits and execute the mapper.
8. Perform shuffle and sort to group the intermediate values by row.
9. Execute the reducer to calculate the final matrix-vector product.
10. Verify the MapReduce result using NumPy.
11. Repeat the matrix-vector multiplication for 30 iterations and identify the most important page.

## CODE:

```python
"""
Matrix-vector multiplication x = M.v with MapReduce (Hadoop Streaming style).
Input records: i<TAB>j<TAB>m_ij (sparse matrix entries)
"""

import sys
from itertools import groupby
import numpy as np


def load_vector(path="vector.txt"):
    with open(path) as f:
        return {int(j): float(v) for j, v in (line.split() for line in f)}


def mapper(lines, v):
    for line in lines:
        i, j, m = line.split()
        yield f"{int(i)}\t{float(m) * v[int(j)]}"


def reducer(lines):
    pairs = (l.rstrip("\n").split("\t") for l in lines)
    for i, grp in groupby(pairs, key=lambda kv: int(kv[0])):
        yield f"{i}\t{sum(float(x) for _, x in grp)}"


if len(sys.argv) > 1 and sys.argv[1] in ("map", "reduce"):
    if sys.argv[1] == "map":
        out = mapper(sys.stdin, load_vector())
    else:
        out = reducer(sys.stdin)

    for o in out:
        print(o)
    sys.exit()


# ---------------- Local simulation ----------------
# Link matrix of a 5-page website
M = np.array(
    [
        [0, 0.5, 0, 0, 1 / 3],
        [1 / 3, 0, 0, 0.5, 0],
        [1 / 3, 0, 0, 0.5, 1 / 3],
        [1 / 3, 0, 1.0, 0, 1 / 3],
        [0, 0.5, 0, 0, 0],
    ]
)

v = np.array([0.2, 0.2, 0.2, 0.2, 0.2])

entries = [
    f"{i}\t{j}\t{M[i, j]}"
    for i in range(5)
    for j in range(5)
    if M[i, j] != 0
]

with open("matrix.txt", "w") as f:
    f.write("\n".join(entries))

with open("vector.txt", "w") as f:
    f.write("\n".join(f"{j} {val}" for j, val in enumerate(v)))

vec = load_vector()

print("Matrix-Vector Multiplication using MapReduce (PageRank step)")
print("=" * 60)
print("Matrix M (5 x 5):")
print(np.round(M, 3))
print("Vector v:", v)
print(f"Non-zero entries stored as (i, j, m_ij): {len(entries)} of {M.size}")

# ---------- MAP PHASE ----------
splits = [entries[:5], entries[5:]]

print("\n--- MAP PHASE: emit (i, m_ij * v_j) ---")
mapped = []

for s, chunk in enumerate(splits, 1):
    out = list(mapper(chunk, vec))
    mapped += out
    print(f"Mapper {s}:")

    for rec, o in zip(chunk, out):
        i, j, m = rec.split("\t")
        print(
            f" ({i},{j}, {float(m):.3f}) -> key {o.split()[0]}, "
            f"value {float(o.split()[1]):.4f}"
        )

# ---------- SHUFFLE AND SORT ----------
print("\n--- SHUFFLE AND SORT: group values by row i ---")

shuffled = sorted(mapped, key=lambda s: int(s.split("\t")[0]))

for i, grp in groupby(shuffled, key=lambda s: s.split("\t")[0]):
    print(f" row {i}: {[round(float(g.split()[1]), 4) for g in grp]}")

# ---------- REDUCE PHASE ----------
print("\n--- REDUCE PHASE: x_i = sum of values ---")

x = np.zeros(5)

for r in reducer(shuffled):
    i, val = r.split("\t")
    x[int(i)] = float(val)
    print(f" x[{i}] = {float(val):.4f}")

print("\nMapReduce result x :", np.round(x, 4))
print("NumPy check   M @ v :", np.round(M @ v, 4))
print("Results match       :", np.allclose(x, M @ v))

# ---------- PAGERANK ITERATIONS ----------
print("\nRepeating the job (power iteration) gives PageRank:")

r = v.copy()

for it in range(1, 31):
    r_new = M @ r

    if it in (1, 2, 5, 30):
        print(f" iteration {it:>2}: {np.round(r_new, 4)}")

    r = r_new

print("Most important page:", int(np.argmax(r)))
```

## OUTPUT:

```text
Matrix-Vector Multiplication using MapReduce (PageRank step)
============================================================
Matrix M (5 x 5):
[[0.    0.5   0.    0.    0.333]
 [0.333 0.    0.    0.5   0.   ]
 [0.333 0.    0.    0.5   0.333]
 [0.333 0.    1.    0.    0.333]
 [0.    0.5   0.    0.    0.   ]]

Vector v: [0.2 0.2 0.2 0.2 0.2]
Non-zero entries stored as (i, j, m_ij): 11 of 25

--- MAP PHASE: emit (i, m_ij * v_j) ---
Mapper 1:
 (0,1, 0.500) -> key 0, value 0.1000
 (0,4, 0.333) -> key 0, value 0.0667
 (1,0, 0.333) -> key 1, value 0.0667
 (1,3, 0.500) -> key 1, value 0.1000
 (2,0, 0.333) -> key 2, value 0.0667
Mapper 2:
 (2,3, 0.500) -> key 2, value 0.1000
 (2,4, 0.333) -> key 2, value 0.0667
 (3,0, 0.333) -> key 3, value 0.0667
 (3,2, 1.000) -> key 3, value 0.2000
 (3,4, 0.333) -> key 3, value 0.0667
 (4,1, 0.500) -> key 4, value 0.1000

--- SHUFFLE AND SORT: group values by row i ---
 row 0: [0.1, 0.0667]
 row 1: [0.0667, 0.1]
 row 2: [0.0667, 0.1, 0.0667]
 row 3: [0.0667, 0.2, 0.0667]
 row 4: [0.1]

--- REDUCE PHASE: x_i = sum of values ---
 x[0] = 0.1667
 x[1] = 0.1667
 x[2] = 0.2333
 x[3] = 0.3333
 x[4] = 0.1000

MapReduce result x : [0.1667 0.1667 0.2333 0.3333 0.1]
NumPy check   M @ v : [0.1667 0.1667 0.2333 0.3333 0.1]
Results match       : True

Repeating the job (power iteration) gives PageRank:
 iteration  1: [0.1667 0.1667 0.2333 0.3333 0.1   ]
 iteration  2: [0.1167 0.2222 0.2556 0.3222 0.0833]
 iteration  5: [0.137  0.2012 0.2346 0.3235 0.1037]
 iteration 30: [0.1364 0.2045 0.2386 0.3182 0.1023]
Most important page: 3
```

## RESULT:

Thus, matrix-vector multiplication was successfully implemented using the MapReduce programming model. The mapper generated intermediate values, shuffle and sort grouped them by row, and the reducer calculated the final vector. The result was verified using NumPy, and 30 PageRank iterations identified page index 3 (P3) as the most important page.