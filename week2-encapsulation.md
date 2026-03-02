# Week 2: Encapsulation – Protecting and Controlling Data

## Learning Objectives
- Understand what encapsulation means and why it is a core OOP principle
- Use single underscore `_` (protected) and double underscore `__` (private) naming conventions
- Understand Python's name mangling for double-underscore attributes
- Write getter (accessor) and setter (mutator) methods
- Use Python's `@property` decorator for clean, Pythonic encapsulation
- Implement data validation inside setters to enforce constraints

## 1. What is Encapsulation?

### The Core Idea
**Encapsulation** means bundling an object's data (attributes) and the code that operates on it (methods) together inside a class, while **restricting direct access** to the internal data from outside the class.

Think of it like a vending machine:
- You can see the items (interface)
- You interact through buttons (methods)
- You cannot reach in and grab items directly (private internals)

```python
# WITHOUT encapsulation — data is exposed and can be corrupted
class BadBankAccount:
    def __init__(self, balance):
        self.balance = balance  # fully public — anyone can change it

account = BadBankAccount(1000)
account.balance = -99999  # nothing stops this — disaster!
print(account.balance)    # -99999
```

```python
# WITH encapsulation — data is protected, only valid changes allowed
class GoodBankAccount:
    def __init__(self, balance):
        self.__balance = balance  # private — cannot be accessed directly

    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount

    def get_balance(self):
        return self.__balance

account = GoodBankAccount(1000)
account.deposit(500)
# account.__balance = -99999  # this WON'T affect the real attribute
print(account.get_balance())  # 1500 — safe
```

### Why Encapsulation Matters
- **Data integrity**: prevents attributes being set to invalid values
- **Flexibility**: the internal implementation can change without breaking code that uses the class
- **Clarity**: the public interface (methods) makes clear what operations are allowed
- **Security**: sensitive data (passwords, balances) is hidden from direct access

## 2. Access Modifiers in Python

### Public, Protected, and Private
Python does not enforce access control at the language level (unlike Java or C++). Instead, it uses **naming conventions** that signal intent to other programmers.

| Convention | Syntax | Meaning |
|---|---|---|
| Public | `self.name` | Accessible from anywhere |
| Protected | `self._name` | Convention: "internal use only" — accessible but treat with care |
| Private | `self.__name` | Name-mangled — genuinely harder to access from outside |

```python
class AccessDemo:
    def __init__(self):
        self.public = "I am public"
        self._protected = "I am protected (convention only)"
        self.__private = "I am private (name-mangled)"

    def show_all(self):
        # All three accessible inside the class
        print(self.public)
        print(self._protected)
        print(self.__private)


obj = AccessDemo()
print(obj.public)       # works fine
print(obj._protected)   # works (but signals: you shouldn't do this)
# print(obj.__private)  # AttributeError!
```

### Name Mangling
Python transforms `__attr` into `_ClassName__attr` behind the scenes. This makes accidental access from outside harder (but not impossible — it is a deterrent, not a lock).

```python
class Secret:
    def __init__(self):
        self.__pin = 1234

s = Secret()
# print(s.__pin)              # AttributeError: 'Secret' has no attribute '__pin'
print(s._Secret__pin)         # 1234 — name-mangled form (avoid using this!)
```

> For AQA A-Level: use `__` for genuinely private data you want to protect. Use `_` to indicate "internal" but still somewhat accessible data.

## 3. Getters and Setters

### Traditional Getter/Setter Methods
The classic OOP pattern is to make attributes private and provide controlled access via `get_` and `set_` methods.

```python
class Student:
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    # Getter (accessor) methods
    def get_name(self):
        return self.__name

    def get_age(self):
        return self.__age

    # Setter (mutator) methods with validation
    def set_name(self, name):
        if isinstance(name, str) and len(name) > 0:
            self.__name = name
        else:
            raise ValueError("Name must be a non-empty string.")

    def set_age(self, age):
        if isinstance(age, int) and 5 <= age <= 25:
            self.__age = age
        else:
            raise ValueError("Age must be an integer between 5 and 25.")

    def __str__(self):
        return f"Student: {self.__name}, Age: {self.__age}"


s = Student("Alice", 16)
print(s.get_name())     # Alice
s.set_age(17)
print(s)                # Student: Alice, Age: 17

try:
    s.set_age(200)      # raises ValueError
except ValueError as e:
    print(e)            # Age must be an integer between 5 and 25.
```

## 4. The `@property` Decorator

### Pythonic Encapsulation
Python's `@property` decorator lets you write getter and setter methods that are accessed like plain attributes — no parentheses needed. This gives you the clean syntax of attribute access with the power of validation.

```python
class Temperature:
    def __init__(self, celsius):
        self.__celsius = celsius  # store internally as Celsius

    @property
    def celsius(self):
        """Getter — accessed as obj.celsius"""
        return self.__celsius

    @celsius.setter
    def celsius(self, value):
        """Setter — called when you write obj.celsius = value"""
        if value < -273.15:
            raise ValueError("Temperature cannot be below absolute zero (-273.15°C).")
        self.__celsius = value

    @property
    def fahrenheit(self):
        """Read-only computed property — no setter defined."""
        return self.__celsius * 9/5 + 32

    @property
    def kelvin(self):
        return self.__celsius + 273.15

    def __str__(self):
        return f"{self.__celsius}°C / {self.fahrenheit}°F / {self.kelvin}K"


t = Temperature(100)
print(t)            # 100°C / 212.0°F / 373.15K

t.celsius = 0       # calls the setter — clean attribute-style syntax
print(t)            # 0°C / 32.0°F / 273.15K

print(t.fahrenheit) # 32.0 — read-only computed property

try:
    t.celsius = -300  # triggers validation
except ValueError as e:
    print(e)

# t.fahrenheit = 100  # AttributeError: can't set attribute (no setter defined)
```

### Property Without a Setter (Read-Only)
```python
class Circle:
    def __init__(self, radius):
        self.__radius = radius

    @property
    def radius(self):
        return self.__radius

    @radius.setter
    def radius(self, value):
        if value <= 0:
            raise ValueError("Radius must be positive.")
        self.__radius = value

    @property
    def area(self):
        """Read-only: computed from radius, no setter needed."""
        import math
        return math.pi * self.__radius ** 2

    @property
    def diameter(self):
        return 2 * self.__radius

    def __str__(self):
        return f"Circle(radius={self.__radius:.2f}, area={self.area:.4f})"


c = Circle(5)
print(c.radius)    # 5
print(c.area)      # 78.5398...
print(c.diameter)  # 10
c.radius = 10
print(c)           # Circle(radius=10.00, area=314.1593)
```

## 5. Data Validation Patterns

### Validation Strategies
```python
class Product:
    """Models a product in an inventory system."""

    VALID_CATEGORIES = {"electronics", "clothing", "food", "books", "other"}

    def __init__(self, name, price, stock, category="other"):
        # Use setters so validation always runs — even in __init__
        self.name = name          # uses the setter
        self.price = price        # uses the setter
        self.stock = stock        # uses the setter
        self.category = category  # uses the setter

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, value):
        if not isinstance(value, str) or len(value.strip()) == 0:
            raise ValueError("Product name must be a non-empty string.")
        self.__name = value.strip()

    @property
    def price(self):
        return self.__price

    @price.setter
    def price(self, value):
        if not isinstance(value, (int, float)) or value < 0:
            raise ValueError("Price must be a non-negative number.")
        self.__price = round(float(value), 2)

    @property
    def stock(self):
        return self.__stock

    @stock.setter
    def stock(self, value):
        if not isinstance(value, int) or value < 0:
            raise ValueError("Stock must be a non-negative integer.")
        self.__stock = value

    @property
    def category(self):
        return self.__category

    @category.setter
    def category(self, value):
        if value.lower() not in self.VALID_CATEGORIES:
            raise ValueError(f"Category must be one of: {self.VALID_CATEGORIES}")
        self.__category = value.lower()

    def apply_discount(self, percent):
        """Reduce price by a percentage (0–100)."""
        if not 0 <= percent <= 100:
            raise ValueError("Discount must be between 0 and 100.")
        self.price = self.__price * (1 - percent / 100)

    def __str__(self):
        return f"{self.__name} | £{self.__price:.2f} | Stock: {self.__stock} | {self.__category}"

    def __repr__(self):
        return (f"Product(name={self.__name!r}, price={self.__price}, "
                f"stock={self.__stock}, category={self.__category!r})")


p = Product("Laptop", 999.99, 50, "electronics")
print(p)                # Laptop | £999.99 | Stock: 50 | electronics
p.apply_discount(10)
print(p)                # Laptop | £899.99 | Stock: 50 | electronics
p.stock = 45
print(p.stock)          # 45
```

## Practice Exercises

### Exercise 1: BankAccount with Private Balance
Create a `BankAccount` class where `balance` is a private attribute (`__balance`). Use `@property` for a read-only `balance` property. Implement `deposit(amount)` and `withdraw(amount)` methods with full validation (positive amounts, sufficient funds). Add a transaction history list that records every deposit and withdrawal.

```python
# Your code here
class BankAccount:
    pass
```

### Exercise 2: Student with Validated Age Setter
Create a `Student` class with private attributes `__name`, `__age`, and `__year_group`. Use `@property` with setters for all three. Validate that:
- `name` is a non-empty string
- `age` is an integer between 11 and 19
- `year_group` is an integer between 7 and 13

Include a `promote()` method that increments `year_group` by 1 (with validation).

```python
# Your code here
class Student:
    pass
```

### Exercise 3: Temperature Class
Create a `Temperature` class that stores temperature internally in Celsius as a private attribute. Use `@property` to expose:
- `celsius` — with a setter that validates ≥ −273.15
- `fahrenheit` — read-only computed property
- `kelvin` — read-only computed property

Add a `convert_to(scale)` method that returns the temperature in the requested scale (`"C"`, `"F"`, or `"K"`).

```python
# Your code here
class Temperature:
    pass
```

### Exercise 4: Password Validator Class
Create a `UserAccount` class with `username` and `password` as private attributes. For the password:
- Store it as a **hashed** value (use Python's built-in `hash()` for simplicity)
- Never expose the raw password via a getter
- Provide a `check_password(attempt)` method that returns `True`/`False`
- Validate that passwords are at least 8 characters, contain at least one digit, and at least one uppercase letter
- Provide `change_password(old_password, new_password)` which validates the old password first

```python
# Your code here
import re

class UserAccount:
    pass
```

### Exercise 5: Product Class with Stock Validation
Create a `Product` class with `__name`, `__price`, and `__stock` as private attributes. Use `@property` with setters. Add:
- `restock(quantity)` — adds to stock, quantity must be positive
- `sell(quantity)` — reduces stock; raises an error if stock would go below zero
- `is_available` — read-only property returning `True` if stock > 0
- `total_value` — read-only property returning `price × stock`
- `__str__` and `__repr__`

```python
# Your code here
class Product:
    pass
```

## Key Concepts to Remember
- **Encapsulation**: bundling data and methods together while restricting direct access to internals
- **Public** (`name`): accessible from anywhere — no restriction
- **Protected** (`_name`): convention signals "internal use" — still accessible but should be treated carefully
- **Private** (`__name`): name-mangled to `_ClassName__name` — genuinely harder to access from outside the class
- **Name mangling**: Python transforms `__attr` to `_ClassName__attr`, preventing accidental clashes in subclasses
- **Getter (accessor)**: a method (or property) that retrieves a private attribute's value
- **Setter (mutator)**: a method (or property) that validates and updates a private attribute's value
- **`@property`**: Python decorator that lets you define getters, setters, and deleters with clean attribute-like syntax
- **Data validation**: checks inside setters ensure the object always remains in a valid state

## Common Mistakes to Avoid
1. **Accessing private attributes directly from outside the class** — `obj.__private` raises `AttributeError`; always use the provided getter/property instead.
2. **Forgetting to call setters inside `__init__`** — if you assign `self.__x = value` directly in `__init__` instead of using `self.x = value` (the setter), your validation is bypassed on construction.
3. **Incorrect `@property` / `@x.setter` syntax** — the setter decorator must be `@propertyname.setter`, where `propertyname` matches exactly the name used in `@property`.
4. **Making computed properties into setters** — derived values like `area` or `fahrenheit` should usually be read-only properties; providing a setter for them is usually a design error.
5. **Over-engineering** — not every attribute needs a property with validation; use plain attributes for data that genuinely has no constraints.

## Extension Challenge
Design a `Person` class with full encapsulation. Private attributes: `__first_name`, `__last_name`, `__dob` (date of birth as a string `"YYYY-MM-DD"`), `__email`, and `__phone`. Use `@property` with setters for all attributes. Validate:
- Names are non-empty strings containing only letters and hyphens
- DOB is a valid date string in `"YYYY-MM-DD"` format and represents a date in the past
- Email contains `@` and `.` in a plausible position
- Phone is a string of 10–15 digits (optionally starting with `+`)

Add read-only properties:
- `full_name` — returns `"First Last"`
- `age` — calculates age from DOB using today's date

```python
# Your code here
from datetime import date
import re

class Person:
    pass


# Test your class:
# p = Person("Alice", "Smith", "2007-06-15", "alice@example.com", "07700900123")
# print(p.full_name)   # Alice Smith
# print(p.age)         # (depends on today's date)
# p.email = "invalid"  # should raise ValueError
```
