# Week 4: Polymorphism – One Interface, Many Forms

## Learning Objectives
- Understand what polymorphism means and the two main types
- Understand what a virtual method is and how dynamic dispatch works (AQA terminology)
- Implement run-time polymorphism through method overriding
- Overload operators using Python's dunder (magic) methods
- Implement comparison operators to make objects sortable and comparable
- Implement `__len__`, `__contains__`, and other sequence-style dunder methods
- *(Enrichment)* Understand duck typing as Python's informal approach to polymorphism
- *(Extension)* Use `@total_ordering` to auto-generate comparison methods

## 1. What is Polymorphism?

### The Core Idea
**Polymorphism** (from Greek: "many forms") means that the same operation can behave differently on different types of objects. In practice, you write code that calls a method by name, and the correct version runs automatically depending on the actual type of the object.

```python
# Classic example: the same function call produces different behaviour
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

class Duck:
    def speak(self):
        return "Quack!"


animals = [Dog(), Cat(), Duck()]

for animal in animals:
    # Each call to .speak() dispatches to a DIFFERENT method body
    print(animal.speak())

# Woof!
# Meow!
# Quack!
```

### Compile-Time vs Run-Time Polymorphism
| Type | Also called | Python support |
|---|---|---|
| Compile-time | Method overloading, static dispatch | Not directly (Python resolves at runtime) |
| Run-time | Method overriding, dynamic dispatch | ✅ Core to Python's design |

> **AQA Note**: Python is dynamically typed, so polymorphism is almost always run-time. The concept of "compile-time polymorphism" (different methods with the same name but different signatures) exists in Java/C++ but not natively in Python — instead Python uses default arguments and `*args`.

## 2. Run-Time Polymorphism: Method Overriding

### Virtual Methods — AQA Terminology
In OOP theory (and in AQA exam questions), an overridable method is called a **virtual method**. When you call a virtual method through a reference to the base class, Python automatically calls the most-derived version — this is called **dynamic dispatch** or **late binding**.

> **AQA exam language**: Questions may ask you to *"explain what is meant by a virtual method"* or *"describe how virtual methods support polymorphism."* The correct answer is: a virtual method is a method defined in a parent class that can be overridden in a subclass; when it is called at run-time, Python dispatches to the overriding version in the most-derived class.

In Python, **all instance methods are virtual by default** — every method can be overridden in a subclass. (In C++ or Java you must explicitly mark a method `virtual` or `override`; in Python no extra keyword is needed.) This is why Python code naturally supports run-time polymorphism through plain method overriding.

```python
class Animal:
    def speak(self):            # virtual — can be overridden
        return "(silence)"

class Dog(Animal):
    def speak(self):            # overrides the virtual method
        return "Woof!"

class Cat(Animal):
    def speak(self):            # overrides the virtual method
        return "Meow!"

# Dynamic dispatch: the correct override is selected at run-time
def make_animal_speak(animal):  # works with ANY Animal subclass
    print(animal.speak())

make_animal_speak(Dog())   # Woof!
make_animal_speak(Cat())   # Meow!
make_animal_speak(Animal()) # (silence)  ← base class version
```

### Polymorphism in a Class Hierarchy
```python
import math

class Shape:
    def __init__(self, colour="black"):
        self.colour = colour

    def area(self):
        """To be overridden — returns 0 as a sensible default."""
        return 0

    def perimeter(self):
        return 0

    def describe(self):
        """Calls area() and perimeter() — works polymorphically."""
        return (f"{self.__class__.__name__} [{self.colour}]: "
                f"area={self.area():.2f}, perimeter={self.perimeter():.2f}")


class Circle(Shape):
    def __init__(self, radius, colour="black"):
        super().__init__(colour)
        self.radius = radius

    def area(self):
        return math.pi * self.radius ** 2

    def perimeter(self):
        return 2 * math.pi * self.radius


class Rectangle(Shape):
    def __init__(self, width, height, colour="black"):
        super().__init__(colour)
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)


class Triangle(Shape):
    def __init__(self, a, b, c, colour="black"):
        super().__init__(colour)
        self.a, self.b, self.c = a, b, c

    def area(self):
        # Heron's formula
        s = (self.a + self.b + self.c) / 2
        return math.sqrt(s * (s - self.a) * (s - self.b) * (s - self.c))

    def perimeter(self):
        return self.a + self.b + self.c


shapes = [
    Circle(5, "red"),
    Rectangle(4, 6, "blue"),
    Triangle(3, 4, 5, "green"),
]

# Polymorphic loop — same method call, different behaviour
for shape in shapes:
    print(shape.describe())

# Circle [red]: area=78.54, perimeter=31.42
# Rectangle [blue]: area=24.00, perimeter=20.00
# Triangle [green]: area=6.00, perimeter=12.00

# Polymorphic function
def total_area(shapes):
    return sum(s.area() for s in shapes)

print(f"Total area: {total_area(shapes):.2f}")  # Total area: 108.54
```

> **Note:** In the `Shape` example above, `area()` and `perimeter()` are **virtual methods** — they are defined in `Shape` with placeholder return values and are intended to be overridden by every concrete subclass. Calling `shape.area()` on any item in the list automatically runs the correct subclass version (dynamic dispatch). This is exactly what examiners mean when they ask you to *"identify the virtual methods in a class"* or *"explain how polymorphism is achieved through virtual methods"*.

## 3. Duck Typing

> **Enrichment — Beyond Core AQA:** The concept below is the mechanism Python uses to achieve polymorphism without a strict inheritance hierarchy. AQA examiners expect you to understand polymorphism through method overriding (Section 2). The term **duck typing** and its use without inheritance are useful background knowledge that deepen your understanding, but will not be tested as a named concept in the exam.

### "If It Walks Like a Duck..."
Duck typing is Python's approach to polymorphism: Python does not care about the *type* of an object — only whether the object has the method or attribute you need. If it has `.speak()`, you can call `.speak()` on it.

```python
class Robot:
    def speak(self):
        return "Beep boop."

class Parrot:
    def speak(self):
        return "Polly wants a cracker!"

class Person:
    def speak(self):
        return "Hello there!"


def make_noise(entity):
    """Works with ANY object that has a speak() method — no inheritance needed."""
    print(entity.speak())


# These three classes share NO inheritance relationship
for thing in [Robot(), Parrot(), Person()]:
    make_noise(thing)

# Beep boop.
# Polly wants a cracker!
# Hello there!
```

```python
# Practical duck typing: any object with .area() works
def print_areas(objects):
    for obj in objects:
        try:
            print(f"{type(obj).__name__}: area = {obj.area():.2f}")
        except AttributeError:
            print(f"{type(obj).__name__}: no area() method")
```

## 4. Operator Overloading

### Dunder (Magic) Methods for Operators
Python calls special dunder methods when you use operators on objects. By implementing these methods, you can make your objects work with `+`, `-`, `*`, `==`, `<`, `len()`, `in`, and more.

| Operator | Dunder method | Notes |
|---|---|---|
| `+` | `__add__(self, other)` | Also `__radd__` for right-side |
| `-` | `__sub__(self, other)` | |
| `*` | `__mul__(self, other)` | |
| `==` | `__eq__(self, other)` | Returns bool |
| `!=` | `__ne__(self, other)` | Auto-derived from `__eq__` if not defined |
| `<` | `__lt__(self, other)` | |
| `<=` | `__le__(self, other)` | |
| `>` | `__gt__(self, other)` | |
| `>=` | `__ge__(self, other)` | |
| `len()` | `__len__(self)` | Must return a non-negative int |
| `in` | `__contains__(self, item)` | Returns bool |
| `str()` | `__str__(self)` | Human-readable |

### Vector Class with Arithmetic Operators
```python
class Vector:
    """2D mathematical vector."""

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        """v1 + v2"""
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        """v1 - v2"""
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        """v * scalar — scale the vector"""
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        """scalar * v — Python tries __rmul__ if left operand doesn't handle it"""
        return self.__mul__(scalar)

    def __eq__(self, other):
        return isinstance(other, Vector) and self.x == other.x and self.y == other.y

    def __neg__(self):
        """Unary negation: -v"""
        return Vector(-self.x, -self.y)

    def magnitude(self):
        import math
        return math.sqrt(self.x ** 2 + self.y ** 2)

    def dot(self, other):
        """Dot product."""
        return self.x * other.x + self.y * other.y

    def __str__(self):
        return f"Vector({self.x}, {self.y})"

    def __repr__(self):
        return f"Vector(x={self.x}, y={self.y})"


v1 = Vector(2, 3)
v2 = Vector(1, -1)

print(v1 + v2)          # Vector(3, 2)
print(v1 - v2)          # Vector(1, 4)
print(v1 * 3)           # Vector(6, 9)
print(3 * v1)           # Vector(6, 9)  ← uses __rmul__
print(-v1)              # Vector(-2, -3)
print(v1 == Vector(2, 3))  # True
print(v1.magnitude())   # 3.605...
print(v1.dot(v2))       # 2*1 + 3*(-1) = -1
```

### Comparison Operators — Playing Cards

> **Extension:** The `@total_ordering` decorator is a useful Python standard-library shortcut, but it is **not required by the AQA specification**. You are expected to know that you can implement comparison operators via dunder methods; `@total_ordering` is a Python-specific convenience on top of that.

```python
from functools import total_ordering

SUIT_RANK = {"clubs": 1, "diamonds": 2, "hearts": 3, "spades": 4}

@total_ordering  # auto-generates missing comparison methods from __eq__ and __lt__
class Card:
    """A playing card with value (2–14) and suit."""

    FACE_NAMES = {11: "Jack", 12: "Queen", 13: "King", 14: "Ace"}

    def __init__(self, value, suit):
        if not 2 <= value <= 14:
            raise ValueError("Card value must be between 2 and 14.")
        if suit.lower() not in SUIT_RANK:
            raise ValueError(f"Suit must be one of: {list(SUIT_RANK.keys())}")
        self.value = value
        self.suit = suit.lower()

    def __eq__(self, other):
        if not isinstance(other, Card):
            return NotImplemented
        return self.value == other.value and self.suit == other.suit

    def __lt__(self, other):
        if not isinstance(other, Card):
            return NotImplemented
        # Compare by value first, then suit rank
        if self.value != other.value:
            return self.value < other.value
        return SUIT_RANK[self.suit] < SUIT_RANK[other.suit]

    def __str__(self):
        name = self.FACE_NAMES.get(self.value, str(self.value))
        return f"{name} of {self.suit.capitalize()}"

    def __repr__(self):
        return f"Card(value={self.value}, suit={self.suit!r})"


cards = [Card(10, "hearts"), Card(7, "spades"), Card(14, "clubs"), Card(7, "diamonds")]

# @total_ordering provides <=, >, >= automatically from __eq__ and __lt__
print(sorted(cards))   # sorted from lowest to highest
print(max(cards))      # Ace of Clubs
print(Card(7, "spades") > Card(7, "diamonds"))  # True (spades > diamonds)
```

### `__len__` and `__contains__` — Custom Collection
```python
class Bag:
    """A simple collection that supports len() and 'in'."""

    def __init__(self):
        self.__items = []

    def add(self, item):
        self.__items.append(item)

    def remove(self, item):
        if item in self.__items:
            self.__items.remove(item)
        else:
            raise ValueError(f"{item!r} not in bag.")

    def __len__(self):
        """Called by len(bag)."""
        return len(self.__items)

    def __contains__(self, item):
        """Called by 'item in bag'."""
        return item in self.__items

    def __str__(self):
        return f"Bag({self.__items})"

    def __iter__(self):
        """Allows 'for item in bag:' to work."""
        return iter(self.__items)


b = Bag()
b.add("apple")
b.add("banana")
b.add("cherry")

print(len(b))            # 3
print("banana" in b)     # True
print("mango" in b)      # False

for item in b:
    print(item)          # apple / banana / cherry
```

## Practice Exercises

### Exercise 1: Shape Hierarchy with Polymorphic `area()`
Create a `Shape` base class and at least four subclasses: `Circle`, `Rectangle`, `Triangle`, and `Square`. Each subclass must override `area()` and `perimeter()`. Write a function `largest_shape(shapes)` that returns the shape with the greatest area, and `total_perimeter(shapes)` that sums all perimeters. Demonstrate that the functions work without any `if isinstance(...)` checks.

```python
# Your code here
import math

class Shape:
    pass

class Circle(Shape):
    pass

class Rectangle(Shape):
    pass

class Triangle(Shape):
    pass

class Square(Rectangle):  # hint: Square IS-A Rectangle
    pass
```

### Exercise 2: Vector Class with `__add__` and `__mul__`
Extend the `Vector` class above to also support:
- `__truediv__(self, scalar)` — divide both components by scalar
- `__abs__()` — returns the magnitude (so `abs(v)` works)
- `__bool__()` — returns `False` if the zero vector, otherwise `True`
- `normalise()` — returns a unit vector (magnitude 1) in the same direction
- `__iter__()` — allows unpacking: `x, y = vector`

```python
# Your code here
import math

class Vector:
    pass
```

### Exercise 3: Fraction Class with Arithmetic
Create a `Fraction` class supporting:
- `__add__`, `__sub__`, `__mul__`, `__truediv__` — returns a new `Fraction` in lowest terms
- `__eq__`, `__lt__`, `__le__`, `__gt__`, `__ge__`
- `__neg__` — unary negation
- `__str__` returning `"3/4"`, `__repr__` returning `"Fraction(3, 4)"`
- Use `math.gcd` to keep fractions in lowest terms

```python
# Your code here
import math

class Fraction:
    def __init__(self, numerator, denominator):
        if denominator == 0:
            raise ValueError("Denominator cannot be zero.")
        # Your code here — reduce to lowest terms
        pass
```

### Exercise 4: Playing Card with Comparison Operators
Expand the `Card` class with:
- A `Hand` class that holds a list of cards
- `__len__` returning the number of cards
- `__contains__` checking if a card is in the hand
- `add_card(card)` and `play_card(card)` methods
- `best_card()` returning the highest card in the hand
- `__str__` showing all cards in the hand

```python
# Your code here
class Card:
    pass

class Hand:
    pass
```

### Exercise 5: Custom List Class
Create a `NumberList` class wrapping a Python list that supports:
- `append(value)`, `remove(value)`, `__len__`, `__contains__`
- `__add__(other)` — merges two `NumberList` objects into a new one
- `__mul__(n)` — repeats the list `n` times
- `__getitem__(index)` — allows `my_list[i]` syntax
- `minimum()`, `maximum()`, `mean()` methods
- `__str__` showing the list contents

```python
# Your code here
class NumberList:
    pass
```

## Key Concepts to Remember
- **Polymorphism**: the ability for the same method call to produce different behaviour depending on the object's type
- **Virtual method**: a method defined in a parent class that is intended to be overridden by subclasses; in Python all instance methods are effectively virtual by default
- **Method overriding**: a subclass replaces a parent method with its own version; Python calls the most derived version at runtime
- **Duck typing**: Python's informal polymorphism — an object is usable wherever its methods match what is expected, regardless of its actual type; "if it walks like a duck…" (enrichment; not a named AQA term)
- **Operator overloading**: implementing dunder methods (`__add__`, `__eq__`, etc.) lets your objects work with built-in Python operators
- **Dunder methods**: special methods surrounded by double underscores (e.g. `__str__`, `__len__`) that Python calls in specific circumstances
- **`@total_ordering`**: a `functools` decorator (extension) that auto-generates missing comparison methods from `__eq__` and one of `__lt__`/`__le__`/`__gt__`/`__ge__`
- **`__len__`**: called by `len()`; must return a non-negative integer
- **`__contains__`**: called by the `in` operator; should return a bool

## Common Mistakes to Avoid
1. **Forgetting to `return` from `__add__`** — operator overloads that build new objects must `return` that new object; forgetting `return` gives you `None`.
2. **Mutating `self` in `__add__`** — arithmetic operators should return a *new* object, not modify the existing one; `v1 + v2` should not change `v1`.
3. **Not handling `NotImplemented`** — when `other` is the wrong type, return `NotImplemented` (not `False` or raise an error) so Python can try the reflected operation on `other`.
4. **Missing `__rmul__` when needed** — `3 * vector` calls `int.__mul__(vector)` first, which returns `NotImplemented`, then tries `vector.__rmul__(3)`; without `__rmul__`, `3 * v` fails.
5. **Implementing comparison operators inconsistently** — if `a == b` is `True`, then `a < b` should be `False`; use `@total_ordering` or be very careful with the logic.

## Extension Challenge
Create a `Matrix` class representing an m×n matrix of numbers. Implement:
- `__init__(self, rows)` where `rows` is a list of lists
- `__add__` and `__sub__` (matrices must be the same shape)
- `__mul__` for both scalar multiplication (`Matrix * 3`) and matrix multiplication (`Matrix * Matrix`)
- `__eq__` to compare two matrices element-by-element
- `__len__` returning the number of rows
- `transpose()` returning a new Matrix with rows and columns swapped
- `__str__` displaying the matrix in a grid format
- `__getitem__` allowing `matrix[row][col]` syntax

```python
# Your code here
class Matrix:
    def __init__(self, rows):
        # rows is a list of lists, e.g. [[1,2,3],[4,5,6]]
        pass

# Test:
# a = Matrix([[1, 2], [3, 4]])
# b = Matrix([[5, 6], [7, 8]])
# print(a + b)      # [[6, 8], [10, 12]]
# print(a * b)      # Matrix multiplication: [[19, 22], [43, 50]]
# print(a * 2)      # [[2, 4], [6, 8]]
# print(a.transpose())  # [[1, 3], [2, 4]]
```
