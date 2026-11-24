
# EXPERIMENT 12 – Set and Dictionary Operations

## AIM:

To analyse course enrolments and student marks using sets and dictionaries.

## QUESTION:

A university wants to analyse which students are enrolled in its Python, Machine Learning and Big Data courses, and to process student marks and course feedback. Use sets for the enrolment analysis and dictionaries for marks and word frequencies.

The program must:

1. Perform union, intersection, difference and symmetric difference on enrolment sets.
2. Check subset and disjoint relations; add and discard elements.
3. Create a nested dictionary of marks; access, insert, update and delete entries.
4. Use dictionary comprehension, get(), defaultdict and sorting by value.
5. Count word frequencies in feedback using Counter.

## ALGORITHM:

1. Import Counter and defaultdict from the collections module.
2. Create sets containing students enrolled in Python, Machine Learning and Big Data courses.
3. Perform union, intersection, difference and symmetric difference operations.
4. Check subset and disjoint relationships and demonstrate add() and discard().
5. Create a nested dictionary to store student marks.
6. Access, insert, update and delete dictionary entries.
7. Calculate student averages using dictionary comprehension and identify the topper.
8. Use defaultdict to calculate subject-wise statistics.
9. Use Counter to count feedback word frequencies.
10. Display the most common words and sort the frequencies by value.

## CODE:

```python
from collections import Counter, defaultdict

print("University Course Enrolment Analysis: Sets and Dictionaries")
print("=" * 60)

# ---------- Sets ----------
python_course = {"Anu", "Bala", "Chitra", "Dinesh", "Esha", "Farhan"}
ml_course = {"Chitra", "Dinesh", "Gokul", "Harini", "Esha"}
big_data = {"Bala", "Esha", "Harini", "Ishaan"}

print("\nPython course :", sorted(python_course))
print("ML course     :", sorted(ml_course))
print("Big Data      :", sorted(big_data))

print(
    "\nUnion (all students)          :",
    sorted(python_course | ml_course | big_data),
)
print("Intersection (Python & ML)    :", sorted(python_course & ml_course))
print("Difference (Python only, not ML):", sorted(python_course - ml_course))
print("Symmetric difference (Py ^ ML):", sorted(python_course ^ ml_course))

print(
    "In all three courses          :",
    sorted(python_course & ml_course & big_data),
)

print(
    "Is {'Esha','Bala'} subset of Python? :",
    {"Esha", "Bala"}.issubset(python_course),
)

print("Are ML and Big Data disjoint?        :", ml_course.isdisjoint(big_data))

ml_course.add("Jaya")
ml_course.discard("Gokul")

print("ML after add('Jaya'), discard('Gokul'):", sorted(ml_course))

duplicates = ["Anu", "Bala", "Anu", "Chitra", "Bala", "Anu"]
print("Unique names from a list with duplicates:", sorted(set(duplicates)))

# ---------- Dictionaries ----------
marks = {
    "Anu": {"Python": 88, "ML": None},
    "Bala": {"Python": 72, "BigData": 81},
    "Chitra": {"Python": 95, "ML": 91},
    "Dinesh": {"Python": 64, "ML": 70},
}

marks["Anu"]["ML"] = 79
marks["Esha"] = {"Python": 85, "ML": 88, "BigData": 90}

print("\nStudent marks dictionary:")
for student, sub in marks.items():
    print(f" {student:<7} {sub}")

print("\nKeys   :", list(marks.keys()))
print("Chitra's Python mark   :", marks["Chitra"]["Python"])
print("get() for missing key :", marks.get("Zara", "Not enrolled"))

averages = {
    s: round(sum(v.values()) / len(v), 2) for s, v in marks.items()
}
print("Average per student    :", averages)

topper = max(averages, key=averages.get)
print("Topper                 :", topper, averages[topper])

subject_scores = defaultdict(list)

for sub in marks.values():
    for subject, m in sub.items():
        subject_scores[subject].append(m)

print("\nSubject-wise statistics:")
for subject, scores in subject_scores.items():
    print(
        f" {subject:<8} count={len(scores)} "
        f"avg={sum(scores)/len(scores):.2f}   max={max(scores)}"
    )

removed = marks.pop("Dinesh")
print("\nRemoved Dinesh ->", removed, "| students left:", len(marks))

marks.update({"Farhan": {"Python": 77}})
print("After update() with Farhan:", sorted(marks))

feedback = (
    "good course good faculty excellent labs good projects excellent support"
)

freq = Counter(feedback.split())
print("\nWord frequency of feedback:", dict(freq))
print("Most common 2 words       :", freq.most_common(2))

print(
    "Sorted by value (desc)   :",
    dict(sorted(freq.items(), key=lambda kv: kv[1], reverse=True)),
)
```

## OUTPUT:

```text
University Course Enrolment Analysis: Sets and Dictionaries
============================================================

Python course : ['Anu', 'Bala', 'Chitra', 'Dinesh', 'Esha', 'Farhan']
ML course     : ['Chitra', 'Dinesh', 'Esha', 'Gokul', 'Harini']
Big Data      : ['Bala', 'Esha', 'Harini', 'Ishaan']

Union (all students)          : ['Anu', 'Bala', 'Chitra', 'Dinesh', 'Esha', 'Farhan', 'Gokul', 'Harini', 'Ishaan']
Intersection (Python & ML)    : ['Chitra', 'Dinesh', 'Esha']
Difference (Python only, not ML): ['Anu', 'Bala', 'Farhan']
Symmetric difference (Py ^ ML): ['Anu', 'Bala', 'Farhan', 'Gokul', 'Harini']
In all three courses          : ['Esha']
Is {'Esha','Bala'} subset of Python? : True
Are ML and Big Data disjoint?        : False
ML after add('Jaya'), discard('Gokul'): ['Chitra', 'Dinesh', 'Esha', 'Harini', 'Jaya']
Unique names from a list with duplicates: ['Anu', 'Bala', 'Chitra']

Student marks dictionary:
 Anu     {'Python': 88, 'ML': 79}
 Bala    {'Python': 72, 'BigData': 81}
 Chitra  {'Python': 95, 'ML': 91}
 Dinesh  {'Python': 64, 'ML': 70}
 Esha    {'Python': 85, 'ML': 88, 'BigData': 90}

Keys   : ['Anu', 'Bala', 'Chitra', 'Dinesh', 'Esha']
Chitra's Python mark   : 95
get() for missing key : Not enrolled
Average per student    : {'Anu': 83.5, 'Bala': 76.5, 'Chitra': 93.0, 'Dinesh': 67.0, 'Esha': 87.67}
Topper                 : Chitra 93.0

Subject-wise statistics:
 Python   count=5 avg=80.80   max=95
 ML       count=4 avg=82.00   max=91
 BigData  count=2 avg=85.50   max=90

Removed Dinesh -> {'Python': 64, 'ML': 70} | students left: 4
After update() with Farhan: ['Anu', 'Bala', 'Chitra', 'Esha', 'Farhan']

Word frequency of feedback: {'good': 3, 'course': 1, 'faculty': 1, 'excellent': 2, 'labs': 1, 'projects': 1, 'support': 1}
Most common 2 words       : [('good', 3), ('excellent', 2)]
Sorted by value (desc)   : {'good': 3, 'excellent': 2, 'course': 1, 'faculty': 1, 'labs': 1, 'projects': 1, 'support': 1}
```

## RESULT:

Thus, course enrolment analysis and student mark processing were successfully implemented using Python sets and dictionaries. Set operations, nested dictionary manipulation, dictionary comprehension, defaultdict and Counter were demonstrated successfully.
