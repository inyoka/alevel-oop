# Week 1: Classes and Objects – The Foundations of OOP

## Learning Objectives
- Understand what Object-Oriented Programming (OOP) is and why it is used
- Define a class using the `class` keyword
- Write an `__init__` constructor to initialise instance attributes
- Define and call instance methods using `self`
- Distinguish between class attributes and instance attributes
- Implement `__str__` and `__repr__` for readable object output
- Create multiple objects from the same class

## 1. What is Object-Oriented Programming?

### The Core Idea
OOP is a programming paradigm that organises code around **objects** — data structures that bundle together related **attributes** (data) and **methods** (behaviour). Rather than writing a collection of loose functions and variables, you model real-world (or abstract) entities as self-contained objects.

A **class** is the blueprint or template. An **object** (also called an **instance**) is a specific thing created from that blueprint.

```python
# Analogy: a class is like an architect's plan for a house.
# Each house built from that plan is a separate object (instance).
# Every house has the same structure (rooms, doors) but its own values
# (colour, address, owner).
```

### Why OOP Matters for AQA A-Level
The AQA specification (7517) requires you to:
- Write classes with constructors, attributes, and methods
- Apply encapsulation, inheritance, and polymorphism
- Model systems using objects and their relationships

OOP advantages:
- **Modularity**: classes keep related code together
- **Reusability**: write once, instantiate many times
- **Maintainability**: changes inside a class do not break unrelated code
- **Modelling**: maps naturally onto real-world systems

## 2. Defining a Class

### Basic Class Syntax
```python
class Dog:
    """A simple class representing a dog."""

    # Class attribute — shared by ALL instances of Dog
    species = "Canis familiaris"

    def __init__(self, name, breed, age):
        """Constructor: called automatically when a new Dog is created."""
        # Instance attributes — unique to each object
        self.name = name
        self.breed = breed
        self.age = age

    def bark(self):
        """An instance method."""
        return f"{self.name} says: Woof!"

    def description(self):
        """Returns a description of this dog."""
        return f"{self.name} is a {self.age}-year-old {self.breed}."
```

### The `__init__` Constructor
`__init__` is a **special method** (also called a *dunder* or *magic* method) that Python calls automatically when you create a new object. Its job is to set up the object's initial state by assigning values to instance attributes.

```python
class Person:
    def __init__(self, name, age):
        self.name = name   # 'self.name' is the instance attribute
        self.age = age     # 'age' (without self) would be a local variable only
```

### Understanding `self`
`self` is a reference to the **current object**. Python passes it automatically as the first argument to every instance method. You never pass it yourself when calling a method.

```python
class Counter:
    def __init__(self):
        self.count = 0        # 'self' refers to this particular Counter object

    def increment(self):
        self.count += 1       # modifies THIS object's count attribute

    def reset(self):
        self.count = 0

    def get_count(self):
        return self.count


c1 = Counter()
c2 = Counter()

c1.increment()
c1.increment()
c2.increment()

print(c1.get_count())  # 2 — c1 has its own count
print(c2.get_count())  # 1 — c2 is completely independent
```

## 3. Creating Objects (Instantiation)

### Instantiation Syntax
```python
# Creating objects (instances) from the Dog class
dog1 = Dog("Buddy", "Labrador", 3)
dog2 = Dog("Max", "Poodle", 5)
dog3 = Dog("Bella", "Beagle", 2)

# Accessing instance attributes
print(dog1.name)    # Buddy
print(dog2.breed)   # Poodle
print(dog3.age)     # 2

# Calling instance methods
print(dog1.bark())           # Buddy says: Woof!
print(dog2.description())    # Max is a 5-year-old Poodle.

# Accessing a class attribute through an instance or the class itself
print(dog1.species)    # Canis familiaris
print(Dog.species)     # Canis familiaris
```

### Class Attributes vs Instance Attributes
```python
class Student:
    # Class attribute: shared across ALL Student objects
    school_name = "Greenfield Academy"
    student_count = 0  # often used to track how many objects exist

    def __init__(self, name, year):
        # Instance attributes: unique to each Student
        self.name = name
        self.year = year
        Student.student_count += 1  # update the class attribute on creation

    def info(self):
        return f"{self.name} (Year {self.year}) at {Student.school_name}"


s1 = Student("Alice", 12)
s2 = Student("Bob", 13)

print(s1.info())                 # Alice (Year 12) at Greenfield Academy
print(Student.student_count)     # 2
print(s1.student_count)          # 2 — same value via instance
```

> **Key distinction**: modifying an instance attribute with `self.x = ...` only affects that one object. Modifying a class attribute should be done via the class name (`ClassName.attr = ...`) to avoid accidentally shadowing it with an instance attribute.

## 4. `__str__` and `__repr__`

### Why These Methods Matter
When you `print()` an object or display it in the REPL, Python needs to know how to represent it as a string.

- `__str__`: the **human-readable** representation (used by `print()` and `str()`)
- `__repr__`: the **developer/debugging** representation (used by `repr()` and the REPL)

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __str__(self):
        """Human-friendly string — what end users would see."""
        return f'"{self.title}" by {self.author} ({self.pages} pages)'

    def __repr__(self):
        """Unambiguous representation — useful for debugging.
        Convention: return a string that looks like a constructor call."""
        return f"Book(title={self.title!r}, author={self.author!r}, pages={self.pages})"


b = Book("1984", "George Orwell", 328)

print(b)       # "1984" by George Orwell (328 pages)
print(str(b))  # "1984" by George Orwell (328 pages)
print(repr(b)) # Book(title='1984', author='George Orwell', pages=328)
```

### Without `__str__`
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p)  # <__main__.Point object at 0x7f3b1c0d4a90>  ← not very helpful!

# After adding __str__:
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"Point({self.x}, {self.y})"

    def __repr__(self):
        return f"Point(x={self.x}, y={self.y})"

p = Point(3, 4)
print(p)       # Point(3, 4)
print(repr(p)) # Point(x=3, y=4)
```

## 5. A Complete Example

```python
class BankAccount:
    """Models a simple bank account."""

    bank_name = "Python Bank"  # class attribute

    def __init__(self, owner, account_number, initial_balance=0.0):
        self.owner = owner
        self.account_number = account_number
        self.balance = initial_balance

    def deposit(self, amount):
        """Add funds to the account."""
        if amount > 0:
            self.balance += amount
            return f"Deposited £{amount:.2f}. New balance: £{self.balance:.2f}"
        return "Deposit amount must be positive."

    def withdraw(self, amount):
        """Remove funds from the account."""
        if amount <= 0:
            return "Withdrawal amount must be positive."
        if amount > self.balance:
            return "Insufficient funds."
        self.balance -= amount
        return f"Withdrew £{amount:.2f}. New balance: £{self.balance:.2f}"

    def get_balance(self):
        return self.balance

    def __str__(self):
        return f"Account [{self.account_number}] — Owner: {self.owner}, Balance: £{self.balance:.2f}"

    def __repr__(self):
        return (f"BankAccount(owner={self.owner!r}, "
                f"account_number={self.account_number!r}, "
                f"initial_balance={self.balance})")


# Using the class
acc1 = BankAccount("Alice", "ACC001", 1000.00)
acc2 = BankAccount("Bob", "ACC002")

print(acc1)                          # Account [ACC001] — Owner: Alice, Balance: £1000.00
print(acc1.deposit(500))             # Deposited £500.00. New balance: £1500.00
print(acc1.withdraw(200))            # Withdrew £200.00. New balance: £1300.00
print(acc1.withdraw(5000))           # Insufficient funds.
print(repr(acc2))                    # BankAccount(owner='Bob', ...)
print(BankAccount.bank_name)         # Python Bank
```

## Practice Exercises

### Exercise 1: BankAccount Class
Create a `BankAccount` class with the attributes `owner`, `account_number`, and `balance`. Add methods `deposit(amount)`, `withdraw(amount)`, and `get_balance()`. Include `__str__` to display account information clearly.

```python
# Your code here
class BankAccount:
    pass
```

### Exercise 2: Student Class
Create a `Student` class with attributes `name`, `student_id`, `year_group`, and a list of `grades` (initially empty). Add methods:
- `add_grade(grade)` — appends a grade to the list
- `average_grade()` — returns the mean of all grades (handle the empty case)
- `highest_grade()` — returns the maximum grade
- `__str__` — returns a nicely formatted summary

```python
# Your code here
class Student:
    pass
```

### Exercise 3: Rectangle Class
Create a `Rectangle` class with `width` and `height` attributes. Add methods:
- `area()` — returns width × height
- `perimeter()` — returns 2 × (width + height)
- `is_square()` — returns `True` if width equals height
- `scale(factor)` — multiplies both dimensions by `factor`
- `__str__` — displays the dimensions

```python
# Your code here
class Rectangle:
    pass
```

### Exercise 4: Car Class
Create a `Car` class with attributes `make`, `model`, `year`, `mileage` (default 0), and `fuel_level` (default 100). Add methods:
- `drive(km)` — reduces fuel by `km * 0.08`, increases mileage by `km` (if enough fuel)
- `refuel(litres)` — adds litres to fuel (cap at 100)
- `service_due()` — returns `True` if mileage > 10000
- `__str__` and `__repr__`

```python
# Your code here
class Car:
    pass
```

### Exercise 5: `__str__` and `__repr__` Practice
Create a `Fraction` class (just the data representation for now — no arithmetic yet) with `numerator` and `denominator` attributes. Implement:
- `__str__` that returns `"3/4"` style output
- `__repr__` that returns `"Fraction(numerator=3, denominator=4)"` style output
- A `to_decimal()` method that returns the float value
- A class attribute `instances_created` that tracks how many Fraction objects have been made

```python
# Your code here
class Fraction:
    instances_created = 0
    pass
```

## Key Concepts to Remember
- **Class**: a blueprint/template defining the structure and behaviour of objects
- **Object / Instance**: a specific entity created from a class blueprint
- **`__init__`**: the constructor method; called automatically when an object is created
- **`self`**: a reference to the current object instance; must be the first parameter of every instance method
- **Instance attribute**: a variable belonging to one specific object (`self.x = ...`)
- **Class attribute**: a variable shared by all instances of a class (defined outside `__init__`)
- **`__str__`**: defines the human-readable string representation (used by `print()`)
- **`__repr__`**: defines the developer/debugging representation (used by `repr()` and the REPL)
- **Instantiation**: the act of creating an object from a class using `ClassName(args)`

## Common Mistakes to Avoid
1. **Forgetting `self` as the first parameter** — every instance method must have `self` as its first argument, even if it looks unused; Python passes the object automatically.
2. **Calling methods without parentheses** — `obj.method` gives you the method object; `obj.method()` actually calls it.
3. **Confusing class attributes and instance attributes** — modifying a class attribute via `self.attr = value` creates a new *instance* attribute that shadows the class attribute; use `ClassName.attr = value` to modify the class attribute for all instances.
4. **Not initialising attributes in `__init__`** — attributes should almost always be created in `__init__` so every instance is in a known state from the start.
5. **Using a mutable class attribute (like a list)** — all instances share the same list, leading to unexpected shared state. Mutable defaults should be instance attributes created inside `__init__`.

## Extension Challenge
Design a `Library` and `Book` system. A `Book` should have `title`, `author`, `isbn`, and `available` (bool) attributes. A `Library` should hold a list of `Book` objects and support:
- `add_book(book)` — adds a Book to the collection
- `checkout(isbn)` — marks a book as unavailable
- `return_book(isbn)` — marks a book as available again
- `search_by_author(author)` — returns all books by that author
- `available_books()` — returns all currently available books
- `__str__` — shows the library name and total book count

Both classes must have proper `__str__` and `__repr__` methods.

```python
# Your code here
class Book:
    pass


class Library:
    pass


# Test your system:
# lib = Library("City Library")
# lib.add_book(Book("1984", "George Orwell", "978-0-14-103614-4"))
# lib.add_book(Book("Animal Farm", "George Orwell", "978-0-14-103613-7"))
# lib.add_book(Book("Brave New World", "Aldous Huxley", "978-0-06-085052-4"))
# print(lib)
# lib.checkout("978-0-14-103614-4")
# print(lib.available_books())
# print(lib.search_by_author("George Orwell"))
```
