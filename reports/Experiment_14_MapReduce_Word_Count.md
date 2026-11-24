# EXPERIMENT 14 – Word Count using MapReduce (Map, Shuffle-Sort and Reduce Phases)

## AIM:

To count word frequencies in news headlines using a Hadoop Streaming MapReduce program.

## QUESTION:

A news agency wants to find the most frequent words in its headlines, which are stored as input splits on three nodes. Write the mapper and reducer in Python in Hadoop Streaming style (read stdin, write key<TAB>value). Show the output of the map phase, the combiner, shuffling and sorting, and the reduce phase.

The program must:

1. Write a mapper that emits (word, 1) for every word in a line.
2. Write a reducer that sums the counts of sorted keys.
3. Use the reducer as a combiner on each mapper and show the reduction in pairs.
4. Show the grouped values after shuffling and sorting.
5. Display the final word counts, the number of distinct words and the top words.

## ALGORITHM:

1. Import the required regular expression, system and groupby modules.
2. Define a mapper function to convert each word into a key-value pair.
3. Define a reducer function to group identical words and calculate their counts.
4. Divide the news headlines into three input splits.
5. Execute the mapper independently on each split.
6. Apply the combiner to reduce repeated word pairs locally.
7. Perform shuffle and sort to group identical keys.
8. Execute the reducer to calculate final word frequencies.
9. Sort the results by frequency and display the top three words.
10. Display the total number of words and distinct words.

## CODE:

```python
"""
Word count in Hadoop Streaming style.
Local simulation: python Experiment_14_MapReduce_Word_Count.py
"""

import re
import sys
from itertools import groupby


def mapper(lines):
    for line in lines:
        for word in re.findall(r"[a-z]+", line.lower()):
            yield f"{word}\t1"


def reducer(lines):
    pairs = (line.rstrip("\n").split("\t") for line in lines)
    for word, group in groupby(pairs, key=lambda kv: kv[0]):
        yield f"{word}\t{sum(int(c) for _, c in group)}"


if len(sys.argv) > 1 and sys.argv[1] == "map":
    for out in mapper(sys.stdin):
        print(out)
    sys.exit()

if len(sys.argv) > 1 and sys.argv[1] == "reduce":
    for out in reducer(sys.stdin):
        print(out)
    sys.exit()


# ---------------- Local simulation of the Hadoop job ----------------
splits = {
    "split-1 (node1)": [
        "Data science uses data to create value",
        "Big data needs distributed storage",
    ],
    "split-2 (node2)": [
        "Hadoop stores big data in HDFS",
        "MapReduce processes data in parallel",
    ],
    "split-3 (node3)": [
        "Spark and Hadoop process big data",
        "Data drives science",
    ],
}

print("MapReduce Word Count on News Headlines")
print("=" * 42)

# ---------- MAP PHASE ----------
print("\n--- MAP PHASE ---")
mapped = {}

for split, lines in splits.items():
    out = list(mapper(lines))
    mapped[split] = out
    print(
        f"{split}: {len(out)} pairs ->",
        ", ".join(o.replace("\t", ":") for o in out[:6]),
        "...",
    )

# ---------- COMBINER PHASE ----------
print("\n--- COMBINER (local aggregation on each mapper) ---")
combined = []

for split, out in mapped.items():
    local = list(reducer(sorted(out)))
    combined.extend(local)
    print(f"{split}: {len(out)} pairs reduced to {len(local)}")

# ---------- SHUFFLE AND SORT PHASE ----------
print("\n--- SHUFFLE AND SORT ---")
shuffled = sorted(combined)

for key, grp in groupby(shuffled, key=lambda s: s.split("\t")[0]):
    values = [g.split("\t")[1] for g in grp]
    if len(values) > 1:
        print(f" {key:<12} -> {values}")

print(" (keys appearing on only one mapper are not shown)")

# ---------- REDUCE PHASE ----------
print("\n--- REDUCE PHASE ---")
result = [r.split("\t") for r in reducer(shuffled)]
result.sort(key=lambda kv: (-int(kv[1]), kv[0]))

print(f"{'WORD':<12}COUNT")

for word, count in result:
    print(f"{word:<12}{count}")

print("\nDistinct words :", len(result))
print("Total words    :", sum(int(c) for _, c in result))
print("Top 3 words    :", [w for w, _ in result[:3]])
```

## OUTPUT:

```text
MapReduce Word Count on News Headlines
==========================================

--- MAP PHASE ---
split-1 (node1): 12 pairs -> data:1, science:1, uses:1, data:1, to:1, create:1 ...
split-2 (node2): 11 pairs -> hadoop:1, stores:1, big:1, data:1, in:1, hdfs:1 ...
split-3 (node3): 9 pairs -> spark:1, and:1, hadoop:1, process:1, big:1, data:1 ...

--- COMBINER (local aggregation on each mapper) ---
split-1 (node1): 12 pairs reduced to 10
split-2 (node2): 11 pairs reduced to 9
split-3 (node3): 9 pairs reduced to 8

--- SHUFFLE AND SORT ---
 big          -> ['1', '1', '1']
 data         -> ['2', '2', '3']
 hadoop       -> ['1', '1']
 science      -> ['1', '1']
 (keys appearing on only one mapper are not shown)

--- REDUCE PHASE ---
WORD        COUNT
data        7
big         3
hadoop      2
in          2
science     2
and         1
create      1
distributed 1
drives      1
hdfs        1
mapreduce   1
needs       1
parallel    1
process     1
processes   1
spark       1
storage     1
stores      1
to          1
uses        1
value       1

Distinct words : 21
Total words    : 32
Top 3 words    : ['data', 'big', 'hadoop']
```

## EQUIVALENT EXECUTION COMMANDS:

Windows local pipe:

```bash
type news.txt | python Experiment_14_MapReduce_Word_Count.py map | sort | python Experiment_14_MapReduce_Word_Count.py reduce
```

Linux local pipe:

```bash
cat news.txt | python Experiment_14_MapReduce_Word_Count.py map | sort | python Experiment_14_MapReduce_Word_Count.py reduce
```

Hadoop Streaming reference:

```bash
hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-*.jar -files Experiment_14_MapReduce_Word_Count.py -mapper "python Experiment_14_MapReduce_Word_Count.py map" -reducer "python Experiment_14_MapReduce_Word_Count.py reduce" -input /data/news -output /out/wc
```

## RESULT:

Thus, the word-count application was successfully implemented using the MapReduce programming model. The Map, Combiner, Shuffle-and-Sort and Reduce phases were demonstrated using three input splits. A total of 32 words and 21 distinct words were identified, with **data** as the most frequent word.