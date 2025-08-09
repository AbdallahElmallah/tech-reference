# Python Scripting - Level 1: Basic Syntax and Simple Scripts

## Prerequisites
- Basic computer literacy
- Python 3.9+ installed on your system
- Basic understanding of what programming is

## Problem It Solves

This level focuses on **fundamental Python syntax and writing simple scripts**. You'll learn to:

- Understand Python basic syntax and structure
- Write simple scripts that perform basic operations
- Work with variables, data types, and basic control structures
- Create scripts that can read input and produce output
- Handle basic errors in your scripts

## Key Concepts

### 1. Python Basics

#### Variables and Data Types
```python
# Basic variables
name = "Ahmed"
age = 25
height = 1.75
is_working = True

# Printing variables
print("Name:", name)
print("Age:", age)
print("Height:", height)
print("Working:", is_working)

# Simple string operations
full_name = "Ahmed" + " " + "Ali"
print("Full name:", full_name)
print("Name length:", len(name))
```

#### Basic Lists
```python
# Creating lists
fruits = ["apple", "banana", "orange"]
numbers = [1, 2, 3, 4, 5]

# Basic list operations
fruits.append("grape")  # Add item
print("First fruit:", fruits[0])
print("All fruits:", fruits)

# Simple loop through list
for fruit in fruits:
    print("I like", fruit)
```

#### Basic Dictionaries
```python
# Simple dictionary
student = {
    "name": "Sara",
    "age": 22,
    "grade": 85
}

# Accessing dictionary values
print("Student name:", student["name"])
print("Student age:", student["age"])

# Adding new information
student["email"] = "sara@example.com"
print("Updated student:", student)
```

### 2. Control Flow

#### Simple Conditions
```python
# Basic if statements
age = 20

if age >= 18:
    print("You are an adult")
else:
    print("You are a minor")

# Multiple conditions
score = 85

if score >= 90:
    print("Excellent!")
elif score >= 70:
    print("Good job!")
else:
    print("Keep trying!")
```

#### Simple Loops
```python
# For loop with list
fruits = ["apple", "banana", "orange"]

for fruit in fruits:
    print("I like", fruit)

# For loop with numbers
for i in range(5):
    print("Number:", i)

# While loop
count = 0
while count < 3:
    print("Count is:", count)
    count = count + 1
```

#### Simple Functions
```python
# Basic function
def greet(name):
    print("Hello,", name)

# Call the function
greet("Ahmed")

# Function that returns a value
def add_numbers(a, b):
    result = a + b
    return result

# Use the function
total = add_numbers(5, 3)
print("Total:", total)
```

### 3. Working with Files

#### Reading Files
```python
# Reading a text file
with open("example.txt", "r") as file:
    content = file.read()
    print(content)

# Reading line by line
with open("example.txt", "r") as file:
    for line in file:
        print(line.strip())  # strip() removes newline characters
```

#### Writing Files
```python
# Writing to a file
with open("output.txt", "w") as file:
    file.write("Hello, World!")
    file.write("\nThis is a new line")

# Writing multiple lines
lines = ["First line", "Second line", "Third line"]
with open("output.txt", "w") as file:
    for line in lines:
        file.write(line + "\n")
```

#### Simple File Operations
```python
# Check if file exists
import os

if os.path.exists("example.txt"):
    print("File exists")
else:
    print("File does not exist")

# Get file size
if os.path.exists("example.txt"):
    size = os.path.getsize("example.txt")
    print(f"File size: {size} bytes")
```

### 4. Getting User Input

#### Simple Input
```python
# Getting input from user
name = input("What is your name? ")
print("Hello,", name)

# Getting numbers
age_text = input("What is your age? ")
age = int(age_text)  # Convert text to number
print("You are", age, "years old")

# Simple calculator
num1 = float(input("Enter first number: "))
num2 = float(input("Enter second number: "))
result = num1 + num2
print("Sum:", result)
```

### 5. Basic Error Handling

#### Simple Try-Except
```python
# Handling errors when converting input
try:
    age = int(input("Enter your age: "))
    print("Your age is:", age)
except ValueError:
    print("Please enter a valid number")

# Handling file errors
try:
    with open("data.txt", "r") as file:
        content = file.read()
        print(content)
except FileNotFoundError:
    print("File not found")
except Exception as e:
    print("An error occurred:", e)
```

#### Safe Division Function
```python
def safe_divide(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        print("Cannot divide by zero")
        return None
    except TypeError:
        print("Please provide numbers only")
        return None

# Using the function
result = safe_divide(10, 2)
if result is not None:
    print("Result:", result)
```

## Simple Examples

### Example 1: Personal Information Script
```python
# Simple script to collect and display personal information
print("Personal Information Collector")
print("=" * 30)

name = input("Enter your name: ")
age = int(input("Enter your age: "))
city = input("Enter your city: ")

print("\n--- Your Information ---")
print(f"Name: {name}")
print(f"Age: {age}")
print(f"City: {city}")

if age >= 18:
    print("You are an adult")
else:
    print("You are a minor")
```

### Example 2: Simple Grade Calculator
```python
# Calculate average grade from multiple subjects
print("Grade Calculator")
print("=" * 20)

subjects = ["Math", "Science", "English"]
grades = []

for subject in subjects:
    grade = float(input(f"Enter your {subject} grade: "))
    grades.append(grade)

average = sum(grades) / len(grades)

print(f"\nYour grades: {grades}")
print(f"Average grade: {average:.2f}")

if average >= 90:
    print("Excellent!")
elif average >= 80:
    print("Very Good!")
elif average >= 70:
    print("Good!")
else:
    print("Keep trying!")
```

### Example 3: Simple File Reader
```python
# Read a file and count words
filename = "story.txt"

try:
    with open(filename, "r") as file:
        content = file.read()
        words = content.split()
        
        print(f"File: {filename}")
        print(f"Total words: {len(words)}")
        print(f"Total characters: {len(content)}")
        
except FileNotFoundError:
    print(f"File {filename} not found")
```

### Example 4: Simple To-Do List
```python
# Simple to-do list manager
todo_list = []

while True:
    print("\n--- To-Do List ---")
    print("1. Add task")
    print("2. View tasks")
    print("3. Remove task")
    print("4. Exit")
    
    choice = input("Choose an option (1-4): ")
    
    if choice == "1":
        task = input("Enter a new task: ")
        todo_list.append(task)
        print("Task added!")
        
    elif choice == "2":
        if todo_list:
            print("\nYour tasks:")
            for i, task in enumerate(todo_list, 1):
                print(f"{i}. {task}")
        else:
            print("No tasks yet!")
            
    elif choice == "3":
        if todo_list:
            for i, task in enumerate(todo_list, 1):
                print(f"{i}. {task}")
            try:
                task_num = int(input("Enter task number to remove: "))
                if 1 <= task_num <= len(todo_list):
                    removed = todo_list.pop(task_num - 1)
                    print(f"Removed: {removed}")
                else:
                    print("Invalid task number")
            except ValueError:
                print("Please enter a valid number")
        else:
            print("No tasks to remove!")
            
    elif choice == "4":
        print("Goodbye!")
        break
        
    else:
        print("Invalid choice, please try again")
```

## Best Practices

1. **Use descriptive variable names**
   ```python
   # Good
   student_name = "Ahmed"
   total_score = 85
   
   # Bad
   n = "Ahmed"
   s = 85
   ```

2. **Always use proper indentation**
   ```python
   # Good
   if age >= 18:
       print("You can vote")
   else:
       print("You cannot vote yet")
   ```

3. **Use comments to explain your code**
   ```python
   # Calculate the area of a rectangle
   length = 10
   width = 5
   area = length * width  # Area = length × width
   print(f"Area: {area}")
   ```

## Common Mistakes to Avoid

1. **Forgetting to convert input to numbers**
   ```python
   # Wrong
   age = input("Enter age: ")
   if age > 18:  # This won't work!
       print("Adult")
   
   # Correct
   age = int(input("Enter age: "))
   if age > 18:
       print("Adult")
   ```

2. **Not handling file errors**
   ```python
   # Better approach
   try:
       with open("data.txt", "r") as file:
           content = file.read()
           print(content)
   except FileNotFoundError:
       print("File not found!")
   ```

## Simple Practice Projects

### Project 1: Number Guessing Game
Create a game where:
- Computer picks a random number between 1-100
- User tries to guess it
- Give hints ("too high" or "too low")
- Count the number of attempts

### Project 2: Simple Calculator
Build a calculator that:
- Takes two numbers from user
- Asks for operation (+, -, *, /)
- Shows the result
- Handles division by zero

### Project 3: Student Grade Book
Create a program that:
- Stores student names and grades
- Calculates average grade
- Shows highest and lowest grades
- Saves data to a file

## Related Levels

### Previous Level
None - This is the foundation level

### Next Level
**[Level 2: Intermediate Scripting](level-2.md)**
- Advanced data structures and algorithms
- API integration and web scraping
- Database connectivity
- Unit testing and debugging
- Configuration management

### Related Topics
- **[Linux](../linux/level-1.md)**: Command-line proficiency
- **[Testing](../testing/level-1.md)**: Basic testing concepts
- **[SQL](../sql/level-1.md)**: Database fundamentals

## Q&A Section

### Basic Questions

<details>
<summary><strong>Q1: What's the difference between a list and a dictionary in Python?</strong></summary>

**Answer:**
- **List**: Ordered collection of items accessed by index (0, 1, 2, ...)
  ```python
# Create a list of fruits
fruits = ["apple", "banana", "orange"]
# Access the first element using index 0
first_fruit = fruits[0]  # "apple"
```

- **Dictionary**: Collection of key-value pairs accessed by key
  ```python
# Create a dictionary containing student data
student = {"name": "Ahmed", "age": 25}
# Access the name using the "name" key
student_name = student["name"]  # "Ahmed"
```

Use lists for ordered data, dictionaries for key-based lookups.
</details>

<details>
<summary><strong>Q2: When should I use try/except blocks?</strong></summary>

**Answer:**
Use try/except for operations that might fail:

1. **File operations** (file might not exist)
2. **User input conversion** (invalid format)
3. **Division by zero**

```python
# Good example of using try/except
try:
    # Attempt to convert text to number
    age = int(input("Enter your age: "))
except ValueError:
    # If conversion fails, print error message
    print("Please enter a valid number")
    age = 0  # Set default value
```

Don't use try/except for normal program flow.
</details>

<details>
<summary><strong>Q3: How do I read a file safely?</strong></summary>

**Answer:**
Always use `with` statement to ensure files are closed properly:

```python
# Correct way - file closes automatically
try:
    with open("data.txt", "r") as file:
        content = file.read()  # Read file content
        print(content)  # Print the content
except FileNotFoundError:
    print("File not found!")  # Error message if file doesn't exist

# Wrong way - might forget to close the file
file = open("data.txt", "r")
content = file.read()
file.close()  # Easy to forget!
```
</details>

<details>
<summary><strong>Q4: What's the difference between `print()` and `return` in functions?</strong></summary>

**Answer:**
- **`print()`**: Shows output to the user, doesn't give back a value
- **`return`**: Gives back a value that can be used by other code

```python
# Function that prints directly and returns nothing
def greet_print(name):
    print(f"Hello, {name}")  # Prints greeting directly

# Function that returns text instead of printing it
def greet_return(name):
    return f"Hello, {name}"  # Returns text for later use

# Usage
greet_print("Ahmed")  # Prints: Hello, Ahmed
message = greet_return("Sara")  # Stores text in variable
print(message)  # Prints: Hello, Sara
```
</details>

<details>
<summary><strong>Q5: How do I check if a number is even or odd?</strong></summary>

**Answer:**
Use the modulo operator `%` to check the remainder:

```python
# Get number from user and convert to integer
number = int(input("Enter a number: "))

# Check remainder when divided by 2
if number % 2 == 0:
    print(f"{number} is even")  # If remainder is 0, number is even
else:
    print(f"{number} is odd")   # If remainder is 1, number is odd

# Or as a separate function
def is_even(num):
    return num % 2 == 0  # Returns True if even, False if odd

print(is_even(4))  # True - 4 is even
print(is_even(5))  # False - 5 is odd
```
</details>

---

**Next Steps**: Once you're comfortable with these fundamentals, proceed to **[Level 2: Intermediate Scripting](level-2.md)** to learn about APIs, databases, and testing.

**Need Help?** Review the mini-projects above or practice with the Q&A examples to reinforce your understanding.