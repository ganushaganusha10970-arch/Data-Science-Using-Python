
# EXPERIMENT 9 – Python Data Types, Variables, Expressions, Objects and Functions
**Date:** 21-11-26
## AIM:

To implement a patient health indicator system using Python basics.

## QUESTION:

A clinic wants a small Python program that computes health indicators for its patients. Use this scenario to demonstrate Python data types, variables, expressions, user-defined functions (including default, variable-length and keyword arguments, and lambda) and objects created from a class.

The program must:

1. Declare variables of type int, float, str, bool, None and complex and display their types.
2. Evaluate arithmetic, relational, logical and type-conversion expressions.
3. Write functions to compute BMI, BMI category and blood-pressure status.
4. Use default arguments, *args, **kwargs and a lambda function.
5. Create a Patient class with a class attribute, methods and __str__, and print a report for each patient.

## ALGORITHM:

1. Declare variables using different Python data types.
2. Display the values and types of the declared variables.
3. Demonstrate arithmetic, relational, logical and type-conversion expressions.
4. Define functions to calculate BMI, BMI category and blood-pressure status.
5. Demonstrate default arguments, variable-length arguments, keyword arguments and lambda functions.
6. Create a Patient class with attributes and methods.
7. Create patient objects and generate individual health reports.
8. Display the number of objects, average age and patient summary.

## CODE:

```python
# ---------- Variables and basic data types ----------
hospital = "City Care Clinic" # str
patients_today = 4 # int
consultation_fee = 350.0 # float
is_open = True # bool
emergency_contact = None # NoneType
complex_reading = 3 + 4j # complex (used here only to show the type)

print("Patient Health Indicator System")
print("=" * 40)
print("Variables and their types:")
for name, value in [
    ("hospital", hospital),
    ("patients_today", patients_today),
    ("consultation_fee", consultation_fee),
    ("is_open", is_open),
    ("emergency_contact", emergency_contact),
    ("complex_reading", complex_reading),
]:
    print(f" {name:<18} = {str(value):<18} type = {type(value).__name__}")

# ---------- Expressions and operators ----------
print("\nExpressions:")
print(" Total fee (arithmetic)         :", patients_today * consultation_fee)
print(" 17 // 5, 17 % 5, 2 ** 10       :", 17 // 5, 17 % 5, 2**10)
print(" Fee > 300 and clinic open      :", consultation_fee > 300 and is_open)
print(" Type conversion int('45') + 5 :", int("45") + 5)
print(" abs(complex_reading)           :", abs(complex_reading))


# ---------- Functions ----------
def bmi(weight_kg, height_m):
    """Return Body Mass Index rounded to 2 decimals."""
    return round(weight_kg / height_m**2, 2)


def bmi_category(value):
    if value < 18.5:
        return "Underweight"
    elif value < 25:
        return "Normal"
    elif value < 30:
        return "Overweight"
    return "Obese"


def bp_status(systolic, diastolic=80): # default argument
    if systolic >= 140 or diastolic >= 90:
        return "High"
    if systolic < 90 or diastolic < 60:
        return "Low"
    return "Normal"


def average(*values): # variable-length arguments
    return round(sum(values) / len(values), 2)


def summary(**details): # keyword arguments
    return ", ".join(f"{k}={v}" for k, v in details.items())


risk_score = lambda age, bmi_val: round(0.03 * age + 0.05 * bmi_val, 2)


# ---------- Objects (class) ----------
class Patient:
    count = 0 # class attribute

    def __init__(self, name, age, weight, height, systolic, diastolic):
        self.name, self.age = name, age
        self.weight, self.height = weight, height
        self.systolic, self.diastolic = systolic, diastolic
        Patient.count += 1

    def report(self):
        b = bmi(self.weight, self.height)
        return (
            f"{self.name:<8} Age {self.age:>2} | BMI {b:>5} ({bmi_category(b):<11}) | "
            f"BP {self.systolic}/{self.diastolic} "
            f"({bp_status(self.systolic, self.diastolic)}) | "
            f"Risk {risk_score(self.age, b)}"
        )

    def __str__(self):
        return f"Patient({self.name}, {self.age})"


patients = [
    Patient("Arun", 52, 82, 1.70, 145, 92),
    Patient("Sneha", 28, 54, 1.62, 112, 74),
    Patient("Gopal", 65, 60, 1.68, 88, 58),
    Patient("Nisha", 41, 95, 1.60, 130, 85),
]

print("\nPatient reports (objects + functions):")
for p in patients:
    print(" ", p.report())

print("\nObjects created     :", Patient.count)
print("str() of an object :", patients[0])
print("isinstance check    :", isinstance(patients[0], Patient))
print("Average age (*args) :", average(*[p.age for p in patients]))
print(
    "Summary (**kwargs) :",
    summary(clinic=hospital, patients=Patient.count, fee=consultation_fee),
)
print("bp_status(120) uses default diastolic ->", bp_status(120))
print("Docstring of bmi() :", bmi.__doc__)
```

## OUTPUT:

```text
Patient Health Indicator System
========================================
Variables and their types:
 hospital           = City Care Clinic      type = str
 patients_today     = 4                    type = int
 consultation_fee   = 350.0                type = float
 is_open            = True                 type = bool
 emergency_contact  = None                 type = NoneType
 complex_reading    = (3+4j)               type = complex

Expressions:
 Total fee (arithmetic)         : 1400.0
 17 // 5, 17 % 5, 2 ** 10       : 3 2 1024
 Fee > 300 and clinic open      : True
 Type conversion int('45') + 5 : 50
 abs(complex_reading)           : 5.0

Patient reports (objects + functions):
 Arun     Age 52 | BMI 28.37 (Overweight ) | BP 145/92 (High) | Risk 2.98
 Sneha    Age 28 | BMI 20.58 (Normal     ) | BP 112/74 (Normal) | Risk 1.87
 Gopal    Age 65 | BMI 21.26 (Normal     ) | BP 88/58 (Low) | Risk 3.01
 Nisha    Age 41 | BMI 37.11 (Obese      ) | BP 130/85 (Normal) | Risk 3.09

Objects created     : 4
str() of an object : Patient(Arun, 52)
isinstance check    : True
Average age (*args) : 46.5
Summary (**kwargs) : clinic=City Care Clinic, patients=4, fee=350.0
bp_status(120) uses default diastolic -> Normal
Docstring of bmi() : Return Body Mass Index rounded to 2 decimals.
```

## RESULT:

Thus, the patient health indicator system was successfully implemented using Python data types, variables, expressions, functions, lambda expressions and object-oriented programming concepts. BMI, BMI categories and blood-pressure status were calculated, and patient reports were generated successfully.
