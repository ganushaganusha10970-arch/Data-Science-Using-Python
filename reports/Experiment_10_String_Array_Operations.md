
# EXPERIMENT 10 – String and Array Operations

## AIM:

To analyse customer reviews using string operations and sensor readings using arrays.

## QUESTION:

An e-commerce company wants to clean and score product reviews, and a warehouse wants to analyse temperature readings from three sensors over a week. Use Python string methods and regular expressions for the reviews, and the array module and NumPy arrays for the sensor data.

The program must:

1. Apply strip, lower, upper, slicing, find, count, replace, split, join and format to strings.
2. Compute a simple keyword-based sentiment score for each review.
3. Extract e-mail addresses using a regular expression.
4. Create and modify an array using the array module.
5. Perform indexing, slicing, boolean filtering, aggregation, reshaping and vectorised arithmetic on a 2-D NumPy array.

## ALGORITHM:

1. Import the array, regular expression and NumPy libraries.
2. Store customer reviews in a list.
3. Apply string methods such as strip, lower, upper, slicing, find, count, replace and split.
4. Define positive and negative keywords to calculate sentiment scores.
5. Extract e-mail addresses using regular expressions.
6. Demonstrate joining and formatting of strings.
7. Create and modify a temperature array using the array module.
8. Create a two-dimensional NumPy array containing sensor readings.
9. Perform indexing, slicing, filtering, aggregation, reshaping and vectorised temperature conversion.
10. Display the results of string and array operations.

## CODE:

```python
import array
import re
import numpy as np

reviews = [
    " Excellent product! Battery life is GREAT and delivery was fast. ",
    "Poor quality. The battery drained in 2 hours. Very disappointed!!",
    "Good value for money, but the charger is slow.",
    "Delivery was late; product is average. Contact: support@shopmart.in",
    "Great camera, great display, great price!",
]

print("Customer Review and Sensor Analytics: Strings and Arrays")
print("=" * 58)

# ---------- Part A: String operations ----------
r = reviews[0]
print("\nPart A - String operations on review 1")
print(" Original           :", repr(r))
print(" strip()            :", r.strip())
print(" lower()            :", r.strip().lower())
print(" upper()            :", r.strip().upper())
print(" Length             :", len(r.strip()))
print(" Slice [0:17]       :", r.strip()[0:17])
print(" Reverse [::-1]     :", r.strip()[::-1][:25], "...")
print(" find('Battery')    :", r.strip().find("Battery"))
print(" count('a')         :", r.lower().count("a"))
print(" replace            :", r.strip().replace("GREAT", "good"))
print(" split()            :", r.strip().split()[:5], "...")
print(" startswith('Exc') :", r.strip().startswith("Exc"))

positive = {"excellent", "great", "good", "fast"}
negative = {"poor", "disappointed", "slow", "late", "drained"}

print("\nSentiment of each review:")
for i, text in enumerate(reviews, 1):
    words = re.findall(r"[a-z]+", text.lower())
    score = sum(w in positive for w in words) - sum(w in negative for w in words)
    label = "Positive" if score > 0 else "Negative" if score < 0 else "Neutral"
    print(f" Review {i}: words={len(words):>2} score={score:>2} -> {label}")

emails = re.findall(r"[\w.]+@[\w.]+", " ".join(reviews))
print("\nEmail extracted with regex:", emails)
print("Joined keywords :", " | ".join(sorted(positive)))
print("Formatted string: {:<10}|{:^10}|{:>10}".format("left", "centre", "right"))

# ---------- Part B: Arrays ----------
print("\nPart B - Arrays of temperature sensor readings")
temps = array.array("f", [31.5, 32.0, 33.8, 35.2, 34.1, 30.9, 36.4])
print(" array module       :", temps.tolist())

temps.append(37.0)
temps.insert(0, 30.2)

print(" after append/insert:", [round(t, 1) for t in temps])
print(
    " index of max       :",
    temps.index(max(temps)),
    "| typecode:",
    temps.typecode,
)

week = np.array(
    [
        [31.5, 32.0, 33.8, 35.2, 34.1, 30.9, 36.4],
        [29.8, 30.5, 31.2, 33.0, 32.7, 29.5, 34.0],
        [33.1, 34.2, 35.6, 37.1, 36.0, 32.8, 38.2],
    ]
)

print("\n NumPy 2-D array (3 sensors x 7 days):")
print(week)
print(" shape, ndim, size :", week.shape, week.ndim, week.size)
print(" Sensor means       :", week.mean(axis=1).round(2))
print(" Daily maximum      :", week.max(axis=0))
print(" Readings > 35      :", week[week > 35])
print(" Sensor 2, days 2-4:", week[1, 1:4])
print(" In Fahrenheit (vectorised):")
print((week * 9 / 5 + 32).round(1))
print(" Reshape to 7x3     :", week.reshape(7, 3).shape)
print(" Sorted sensor 1    :", np.sort(week[0]))
```

## OUTPUT:

```text
Customer Review and Sensor Analytics: Strings and Arrays
==========================================================

Part A - String operations on review 1
 Original           : ' Excellent product! Battery life is GREAT and delivery was fast. '
 strip()            : Excellent product! Battery life is GREAT and delivery was fast.
 lower()            : excellent product! battery life is great and delivery was fast.
 upper()            : EXCELLENT PRODUCT! BATTERY LIFE IS GREAT AND DELIVERY WAS FAST.
 Length             : 63
 Slice [0:17]       : Excellent product
 Reverse [::-1]     : .tsaf saw yreviled dna TA ...
 find('Battery')    : 19
 count('a')         : 5
 replace            : Excellent product! Battery life is good and delivery was fast.
 split()            : ['Excellent', 'product!', 'Battery', 'life', 'is'] ...
 startswith('Exc') : True

Sentiment of each review:
 Review 1: words=10 score= 3 -> Positive
 Review 2: words= 9 score=-3 -> Negative
 Review 3: words= 9 score= 0 -> Neutral
 Review 4: words=10 score=-1 -> Negative
 Review 5: words= 6 score= 3 -> Positive

Email extracted with regex: ['support@shopmart.in']
Joined keywords : excellent | fast | good | great
Formatted string: left      |  centre  |     right

Part B - Arrays of temperature sensor readings
 array module       : [31.5, 32.0, 33.79999923706055, 35.20000076293945, 34.099998474121094, 30.899999618530273, 36.400001525878906]
 after append/insert: [30.2, 31.5, 32.0, 33.8, 35.2, 34.1, 30.9, 36.4, 37.0]
 index of max       : 8 | typecode: f

 NumPy 2-D array (3 sensors x 7 days):
[[31.5 32.  33.8 35.2 34.1 30.9 36.4]
 [29.8 30.5 31.2 33.  32.7 29.5 34. ]
 [33.1 34.2 35.6 37.1 36.  32.8 38.2]]
 shape, ndim, size : (3, 7) 2 21
 Sensor means       : [33.41 31.53 35.29]
 Daily maximum      : [33.1 34.2 35.6 37.1 36.  32.8 38.2]
 Readings > 35      : [35.2 36.4 35.6 37.1 36.  38.2]
 Sensor 2, days 2-4: [30.5 31.2 33. ]
 In Fahrenheit (vectorised):
[[ 88.7  89.6  92.8  95.4  93.4  87.6  97.5]
 [ 85.6  86.9  88.2  91.4  90.9  85.1  93.2]
 [ 91.6  93.6  96.1  98.8  96.8  91.  100.8]]
 Reshape to 7x3     : (7, 3)
 Sorted sensor 1    : [30.9 31.5 32.  33.8 34.1 35.2 36.4]
```

## RESULT:

Thus, customer reviews were successfully analysed using Python string operations and regular expressions. Temperature sensor readings were processed using the array module and NumPy, demonstrating indexing, slicing, filtering, aggregation, reshaping and vectorised arithmetic.
