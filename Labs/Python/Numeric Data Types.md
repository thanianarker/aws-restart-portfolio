# Working with Numeric Data Types in Python

## Overview

I completed a comprehensive hands-on lab exploring Python's numeric data types and type system. Through interactive exercises using both the Python shell and scripted programs, I gained practical experience with integers, floating-point numbers, complex numbers, and Boolean values. I learned how to inspect data types using built-in functions, perform basic arithmetic operations, create and manipulate variables, and understand how Python internally represents different numeric types. By the end of this lab, I demonstrated proficiency with Python's core numeric type system and the foundational concepts necessary for data manipulation and mathematical programming.

## Topics Covered

After completing this lab, I was able to:

- Use the Python shell for interactive programming
- Perform basic arithmetic operations (addition, subtraction, multiplication, division)
- Understand the `int` (integer) data type and its use cases
- Understand the `float` (floating-point) data type and decimal precision
- Understand the `complex` (complex number) data type and imaginary units
- Understand the `bool` (Boolean) data type and its numeric representation
- Use the `type()` built-in function to inspect data types
- Create and assign variables to store numeric values
- Use the `print()` function to display values and information
- Use the `str()` function to convert data types to strings
- Concatenate strings using the `+` operator
- Write and execute Python scripts from files
- Run Python programs from both the IDE and command line
- Understand the Python Standard Library and built-in functions

## Introducing the Technologies

### Python Numeric Data Types

Python provides multiple data types for storing numeric values, each optimized for different use cases.

| Data Type | Purpose | Range | Example | Memory |
|-----------|---------|-------|---------|--------|
| **int** | Whole numbers | Unlimited precision | `42`, `-17`, `0` | Variable (grows with value) |
| **float** | Decimal numbers | ~15-17 decimal digits | `3.14`, `-2.5`, `0.0` | 8 bytes (IEEE 754) |
| **complex** | Complex numbers (real + imaginary) | Real and imaginary parts | `5j`, `3+4j`, `2-1j` | 16 bytes (two floats) |
| **bool** | Truth values | True or False (1 or 0) | `True`, `False` | 1 byte (subset of int) |

### Understanding Each Data Type

#### Integer (`int`)
The integer type stores whole numbers without decimal points. Python integers have unlimited precision, meaning they can represent arbitrarily large numbers.

```python
myValue = 42          # Positive integer
myValue = -17         # Negative integer
myValue = 0           # Zero
```

#### Float (`float`)
The float type stores numbers with decimal points. Internally, floats use IEEE 754 double-precision format, providing approximately 15-17 significant decimal digits.

```python
myValue = 3.14        # Positive decimal
myValue = -2.5        # Negative decimal
myValue = 1.0         # Decimal representation of whole number
```

💭 **Consider**: Even `1.0` is a float, not an integer. The decimal point determines the type, not the value.

#### Complex (`complex`)
The complex type represents complex numbers with real and imaginary parts. The imaginary unit is represented by `j` (not `i` as in mathematics).

```python
myValue = 5j          # Pure imaginary (real part is 0)
myValue = 3 + 4j      # Complex with both parts
myValue = 2 - 1j      # Complex with negative imaginary
```

📝 **Note**: Complex numbers are essential in advanced mathematics, signal processing, electrical engineering, and scientific computing.

#### Boolean (`bool`)
The bool type represents truth values: `True` (equivalent to 1) and `False` (equivalent to 0). Technically, bool is a subset of int, not a separate type.

```python
myValue = True        # Boolean true (internally 1)
myValue = False       # Boolean false (internally 0)
```

### Built-in Functions for Type Inspection

| Function | Purpose | Example |
|----------|---------|---------|
| **`type()`** | Returns the data type of a value | `type(42)` → `<class 'int'>` |
| **`print()`** | Outputs values to the console | `print("Hello")` → displays "Hello" |
| **`str()`** | Converts value to string representation | `str(42)` → `"42"` |

## Tasks

### Task 1: Access Cloud9 IDE and Create Python File

#### Step 1.1: Launch Lab and Open Cloud9

I started the lab by clicking **Start Lab** and waited for the lab status to display "Lab status: ready."

I clicked the **AWS** button to open the AWS Management Console, then navigated to **Services** → **Cloud9**.

I selected the **reStart-python-cloud9** environment and clicked **Open IDE**.

✅ **Task complete**: Cloud9 IDE successfully opened.

#### Step 1.2: Create Python Exercise File

From the Cloud9 menu bar, I selected **File** → **New From Template** → **Python File**.

I deleted the template boilerplate code and saved the file as `numeric-data.py` in the `/home/ec2-user/environment` directory.

✅ **Task complete**: Python file created and ready for exercises.

### Task 2: Use the Python Shell for Interactive Programming

#### Step 2.1: Start Python Shell

I opened a new terminal in Cloud9 and started the Python shell:

```bash
python3
```

**Shell prompt displayed:**
```
Python 3.6.12 (default, Aug 31 2020, 18:56:18)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

The three greater-than symbols (`>>>`) represent the Python prompt, where I can enter commands interactively.

#### Step 2.2: Perform Basic Arithmetic

I performed addition in the Python shell:

```python
2 + 2
```

**Output:** `4`

I performed subtraction:

```python
4 - 2
```

**Output:** `2`

I performed multiplication:

```python
2 * 2
```

**Output:** `4`

I performed division:

```python
4 / 2
```

**Output:** `2.0`

⚠️ **Caution**: Notice that division returns `2.0` (a float), not `2` (an integer). In Python 3, the `/` operator always returns a float, even when dividing evenly. In Python 2, integer division would return an integer.

#### Step 2.3: Exit Python Shell

I exited the Python shell:

```python
quit()
```

✅ **Task complete**: Demonstrated interactive arithmetic operations in the Python shell.

### Task 3: Explore the Integer (`int`) Data Type

#### Step 3.1: Write and Execute Program

In the `numeric-data.py` file, I entered:

```python
print("Python has three numeric types: int, float, and complex")
```

I saved the file and clicked **Run** to execute it.

**Console output:**
```
Python has three numeric types: int, float, and complex
```

#### Step 3.2: Create Integer Variable

On a new line, I added:

```python
myValue = 1
print(myValue)
print(type(myValue))
print(str(myValue) + " is of the data type " + str(type(myValue)))
```

**Code explanation:**
- `myValue = 1`: Creates a variable and assigns the integer `1` to it
- `print(myValue)`: Displays the value (`1`)
- `type(myValue)`: Returns the data type (`<class 'int'>`)
- String concatenation: Combines the value, text, and data type into a single output string

I saved and ran the file.

**Console output:**
```
Python has three numeric types: int, float, and complex
1
<class 'int'>
1 is of the data type <class 'int'>
```

📝 **Note**: The `type()` function is essential for debugging—it reveals what Python considers the data type of any value, which may differ from what you intended.

✅ **Task complete**: Successfully created and inspected an integer variable.

### Task 4: Explore the Float (`float`) Data Type

#### Step 4.1: Create Float Variable

I added the following lines to the file:

```python
myValue = 3.14
print(myValue)
print(type(myValue))
print(str(myValue) + " is of the data type " + str(type(myValue)))
```

I saved and ran the file.

**Console output (final section):**
```
3.14
<class 'float'>
3.14 is of the data type <class 'float'>
```

#### Step 4.2: Understanding Float Precision

💭 **Consider**: Floats use IEEE 754 double-precision format, which provides about 15-17 significant decimal digits. Very large or very small numbers may lose precision due to the limited number of bits available for representation.

```python
# Examples of precision limitations:
0.1 + 0.2  # May not equal exactly 0.3 due to binary representation
```

✅ **Task complete**: Successfully created and inspected a float variable.

### Task 5: Explore the Complex (`complex`) Data Type

#### Step 5.1: Create Complex Variable

I added the following lines to the file:

```python
myValue = 5j
print(myValue)
print(type(myValue))
print(str(myValue) + " is of the data type " + str(type(myValue)))
```

I saved and ran the file.

**Console output (final section):**
```
5j
<class 'complex'>
5j is of the data type <class 'complex'>
```

#### Step 5.2: Understanding Complex Numbers

The value `5j` represents a pure imaginary number (5 times the imaginary unit). A complex number with both real and imaginary parts would be represented as `3 + 4j`.

📝 **Note**: Python uses `j` for the imaginary unit instead of the mathematical convention `i`. This avoids confusion with the variable name `i`, which is commonly used as a loop counter.

✅ **Task complete**: Successfully created and inspected a complex variable.

### Task 6: Explore the Boolean (`bool`) Data Type

#### Step 6.1: Create True Boolean Variable

I added the following lines to the file:

```python
myValue = True
print(myValue)
print(type(myValue))
print(str(myValue) + " is of the data type " + str(type(myValue)))
```

I saved and ran the file.

**Console output (final section):**
```
True
<class 'bool'>
True is of the data type <class 'bool'>
```

#### Step 6.2: Create False Boolean Variable

I added the following lines to the file:

```python
myValue = False
print(myValue)
print(type(myValue))
print(str(myValue) + " is of the data type " + str(type(myValue)))
```

I saved and ran the file.

**Complete final output:**
```
Python has three numeric types: int, float, and complex
1
<class 'int'>
1 is of the data type <class 'int'>
3.14
<class 'float'>
3.14 is of the data type <class 'float'>
5j
<class 'complex'>
5j is of the data type <class 'complex'>
True
<class 'bool'>
True is of the data type <class 'bool'>
False
<class 'bool'>
False is of the data type <class 'bool'>
```

#### Step 6.3: Understanding Boolean as Subset of Integer

The bool type is technically a subset of int in Python. Booleans can be used in arithmetic operations:

```python
True + True  # Results in 2 (True is 1, True is 1)
False + 5    # Results in 5 (False is 0)
```

⚠️ **Caution**: While bool is a numeric type, it's best practice to use booleans for truth values (conditions, flags) rather than arithmetic operations. Using booleans for math can make code confusing and error-prone.

✅ **Task complete**: Successfully created and inspected Boolean variables.

## Python Type System Summary

I demonstrated the complete numeric type hierarchy:

```
Numeric Types
├── int (integers: ..., -2, -1, 0, 1, 2, ...)
├── float (decimals: 3.14, -2.5, 0.0)
├── complex (imaginary: 5j, 3+4j)
└── bool (truth values: True, False)
    └── [bool is actually a subset of int]
```

## Data Type Selection Guide

| Scenario | Best Type | Reason |
|----------|-----------|--------|
| Counting items, array indices | `int` | No decimal needed; unlimited precision |
| Money, measurements, percentages | `float` | Decimals needed for fractional values |
| Mathematical operations with imaginary components | `complex` | Represents both real and imaginary parts |
| Conditional logic, flags, comparisons | `bool` | Represents true/false states |

## Conclusion

I successfully completed a comprehensive exploration of Python's numeric type system:

- ✅ Used the Python shell for interactive command execution
- ✅ Performed basic arithmetic operations (addition, subtraction, multiplication, division)
- ✅ Created variables and assigned numeric values to them
- ✅ Inspected data types using the `type()` built-in function
- ✅ Converted data to strings using the `str()` function
- ✅ Concatenated strings to create informative output messages
- ✅ Explored the `int` data type for whole numbers
- ✅ Explored the `float` data type for decimal numbers
- ✅ Explored the `complex` data type for imaginary numbers
- ✅ Explored the `bool` data type for truth values
- ✅ Understood that `bool` is technically a subset of `int`
- ✅ Executed Python programs both interactively and from files
- ✅ Gained foundational knowledge of Python's type system

This lab provided essential knowledge about Python's numeric types and built-in functions, establishing the foundation for more complex programming tasks involving data manipulation, mathematical operations, and type-aware code design.

## Additional Resources

- [Python Data Types Documentation](https://docs.python.org/3/library/stdtypes.html)
- [Numeric Types in Python](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex)
- [Built-in Functions Reference](https://docs.python.org/3/library/functions.html)
- [Type System and Type Checking](https://docs.python.org/3/library/typing.html)
- [Python Integer Arithmetic](https://docs.python.org/3/reference/lexical_analysis.html#integer-and-float-literals)
- [IEEE 754 Floating Point Format](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
- [Complex Numbers in Mathematics and Python](https://en.wikipedia.org/wiki/Complex_number)
- [Boolean Values and Operations](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)
- [AWS Cloud9 Python Development Guide](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-python.html)
- [Python Standard Library Overview](https://docs.python.org/3/library/)
