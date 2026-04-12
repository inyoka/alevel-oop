# Week 5: Abstract Classes and Interfaces – Enforcing a Contract

## Learning Objectives
- Understand what abstraction means in OOP
- Use Python's `abc` module to create abstract base classes
- Declare abstract methods with `@abstractmethod`
- Understand why you cannot instantiate an abstract class directly
- Write concrete subclasses that implement all abstract methods
- Understand Python's "interface by convention" approach
- Use `@staticmethod` and `@classmethod` and explain the difference between instance, class, and static methods
- Explain the benefits of abstract classes for AQA exam questions
- *(Extension)* Understand `__subclasshook__` and virtual subclasses

## 1. What is Abstraction?

### The Core Idea
**Abstraction** means hiding *implementation details* and exposing only what is essential. You define *what* something must do without specifying *how* it does it.

In OOP, abstraction is achieved through:
- **Abstract classes**: classes that define a common interface but cannot be instantiated directly
- **Abstract methods**: method signatures with no body that subclasses *must* implement

Think of a TV remote control:
- You know the buttons (the interface): power, volume, channel
- You do not need to know the infrared signals being sent (the implementation)
- Every TV brand implements those buttons differently — same interface, different implementations

```python
# Problem without abstraction: no enforcement of the contract
class Shape:
    def area(self):
        pass  # Subclasses SHOULD override this, but nothing forces them to


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    # Oops — forgot to implement area()!

c = Circle(5)
print(c.area())  # None — no error, but wrong! The bug is silent.
```

```python
# With abstraction: Python enforces the contract
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass  # No implementation — subclasses MUST provide one

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    # No area() — Python will refuse to let us create a Circle

c = Circle(5)  # TypeError: Can't instantiate abstract class Circle
               # with abstract method area
```

## 2. The `abc` Module

### `ABC` and `@abstractmethod`
```python
from abc import ABC, abstractmethod
import math

class Shape(ABC):
    """Abstract base class for all shapes.
    
    Concrete subclasses MUST implement area() and perimeter().
    """

    def __init__(self, colour="black"):
        self.colour = colour

    @abstractmethod
    def area(self):
        """Return the area of the shape."""
        pass

    @abstractmethod
    def perimeter(self):
        """Return the perimeter of the shape."""
        pass

    def describe(self):
        """Concrete method — available to all subclasses as-is."""
        return (f"{self.__class__.__name__} [{self.colour}]: "
                f"area={self.area():.4f}, perimeter={self.perimeter():.4f}")

    def __str__(self):
        return self.describe()


# Cannot instantiate the abstract class directly
try:
    s = Shape()
except TypeError as e:
    print(e)  # Can't instantiate abstract class Shape with abstract methods area, perimeter


class Circle(Shape):
    def __init__(self, radius, colour="black"):
        super().__init__(colour)
        self.radius = radius

    def area(self):       # MUST implement — fulfils the abstract contract
        return math.pi * self.radius ** 2

    def perimeter(self):  # MUST implement
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


class EquilateralTriangle(Shape):
    def __init__(self, side, colour="black"):
        super().__init__(colour)
        self.side = side

    def area(self):
        return (math.sqrt(3) / 4) * self.side ** 2

    def perimeter(self):
        return 3 * self.side


shapes = [Circle(7, "red"), Rectangle(4, 9, "blue"), EquilateralTriangle(6, "green")]

for shape in shapes:
    print(shape)

# Circle [red]: area=153.9380, perimeter=43.9823
# Rectangle [blue]: area=36.0000, perimeter=26.0000
# EquilateralTriangle [green]: area=15.5885, perimeter=18.0000
```

### Partially Abstract Classes
An abstract class *can* have some concrete methods alongside abstract ones. Subclasses inherit the concrete methods for free.

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @abstractmethod
    def sound(self):
        """Must be implemented — different for each animal."""
        pass

    @abstractmethod
    def move(self):
        """Must be implemented — different for each animal."""
        pass

    def breathe(self):
        """Concrete — same for all animals."""
        return f"{self.name} breathes in and out."

    def eat(self, food):
        """Concrete — same for all animals."""
        return f"{self.name} eats {food}."

    def __str__(self):
        return f"{self.__class__.__name__}(name={self.name!r}, age={self.age})"


class Dog(Animal):
    def sound(self):
        return f"{self.name} says: Woof!"

    def move(self):
        return f"{self.name} runs on four legs."


class Eagle(Animal):
    def sound(self):
        return f"{self.name} screeches!"

    def move(self):
        return f"{self.name} soars through the air."


class Fish(Animal):
    def sound(self):
        return f"{self.name} makes no sound."

    def move(self):
        return f"{self.name} swims through the water."


for animal in [Dog("Rex", 3), Eagle("Aquila", 5), Fish("Nemo", 1)]:
    print(animal.sound())
    print(animal.move())
    print(animal.breathe())  # inherited concrete method
    print()
```

## 3. Abstract Properties

### Using `@property` with `@abstractmethod`
```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    def __init__(self, make, model):
        self.make = make
        self.model = model

    @property
    @abstractmethod
    def fuel_type(self):
        """Subclasses must expose what kind of fuel they use."""
        pass

    @abstractmethod
    def refuel(self, amount):
        pass

    def describe(self):
        return f"{self.make} {self.model} — fuel: {self.fuel_type}"


class PetrolCar(Vehicle):
    def __init__(self, make, model, tank_litres):
        super().__init__(make, model)
        self.__fuel = 0.0
        self.__tank_litres = tank_litres

    @property
    def fuel_type(self):  # implements the abstract property
        return "Petrol"

    def refuel(self, amount):
        space = self.__tank_litres - self.__fuel
        added = min(amount, space)
        self.__fuel += added
        return f"Added {added:.1f}L petrol."


class ElectricCar(Vehicle):
    def __init__(self, make, model, battery_kwh):
        super().__init__(make, model)
        self.__charge = 0.0
        self.__battery_kwh = battery_kwh

    @property
    def fuel_type(self):  # implements the abstract property
        return "Electric"

    def refuel(self, amount):
        added = min(amount, self.__battery_kwh - self.__charge)
        self.__charge += added
        return f"Charged {added:.1f} kWh."


petrol = PetrolCar("Ford", "Fiesta", 50)
electric = ElectricCar("Tesla", "Model 3", 75)

print(petrol.describe())   # Ford Fiesta — fuel: Petrol
print(electric.describe()) # Tesla Model 3 — fuel: Electric
```

## 4. Interfaces by Convention

### Python Has No `interface` Keyword
Unlike Java or C#, Python has no separate `interface` construct. Instead, an abstract class with *only* abstract methods (no concrete methods, no instance data) serves the same purpose.

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    """Interface: anything that can be drawn on screen."""

    @abstractmethod
    def draw(self):
        pass

    @abstractmethod
    def resize(self, factor):
        pass


class Serialisable(ABC):
    """Interface: anything that can be serialised."""

    @abstractmethod
    def serialise(self):
        """Return a dict representation."""
        pass

    @abstractmethod
    def deserialise(self, data):
        """Reconstruct the object from a dict."""
        pass


# A class can implement multiple "interfaces" via multiple inheritance
class GraphicShape(Drawable, Serialisable):
    def __init__(self, x, y, colour):
        self.x = x
        self.y = y
        self.colour = colour

    def draw(self):
        return f"Drawing {self.__class__.__name__} at ({self.x},{self.y}) in {self.colour}"

    def resize(self, factor):
        # Subclasses will define specific resize logic
        pass

    def serialise(self):
        return {"type": self.__class__.__name__, "x": self.x,
                "y": self.y, "colour": self.colour}

    def deserialise(self, data):
        self.x = data["x"]
        self.y = data["y"]
        self.colour = data["colour"]
```

### `__subclasshook__` — Virtual Subclasses

> **Extension — Beyond Core AQA:** Virtual subclasses and `__subclasshook__` are advanced Python ABC features. They are **not required by the AQA specification** and will not appear in exams. This section is included for students who want to understand how Python's ABC machinery works under the hood.

Python's ABCs support *virtual subclasses*: classes that are considered subclasses of an ABC without explicitly inheriting from it.

```python
from abc import ABC, abstractmethod

class Speakable(ABC):
    @abstractmethod
    def speak(self):
        pass

    @classmethod
    def __subclasshook__(cls, subclass):
        """Return True if subclass has a 'speak' method."""
        if cls is Speakable:
            return hasattr(subclass, "speak") and callable(subclass.speak)
        return NotImplemented


class Robot:
    def speak(self):
        return "Beep boop."

# Robot never explicitly inherits from Speakable
# but __subclasshook__ says it qualifies
print(isinstance(Robot(), Speakable))  # True — duck typing meets ABCs
```

## 5. Static Methods and Class Methods

### `@staticmethod` — Behaviour That Belongs to the Class, Not an Instance
A **static method** is a method defined inside a class that does **not** receive the instance (`self`) or the class (`cls`) as its first argument. It is a plain function that belongs to the class's namespace because it is logically related to the class.

> **AQA note**: AQA questions may ask you to explain the difference between instance methods, class methods, and static methods, or to write a class that includes a static method.

```python
class MathUtils:
    """A collection of utility functions — no instance needed."""

    @staticmethod
    def add(a, b):
        return a + b

    @staticmethod
    def multiply(a, b):
        return a * b

    @staticmethod
    def is_even(n):
        return n % 2 == 0


# Call without creating an instance
print(MathUtils.add(3, 4))       # 7
print(MathUtils.is_even(10))     # True

# Can also call on an instance (but that is unusual — class-level call is clearer)
utils = MathUtils()
print(utils.multiply(5, 6))      # 30
```

### `@classmethod` — Methods That Receive the Class Itself
A **class method** receives the class (`cls`) as its first argument instead of the instance. It can access and modify class-level attributes and is often used as an alternative constructor.

```python
class Temperature:
    """Represents a temperature, stored in Celsius internally."""

    ABSOLUTE_ZERO_C = -273.15

    def __init__(self, celsius):
        if celsius < Temperature.ABSOLUTE_ZERO_C:
            raise ValueError("Temperature below absolute zero.")
        self.__celsius = celsius

    @classmethod
    def from_fahrenheit(cls, fahrenheit):
        """Alternative constructor: create from a Fahrenheit value."""
        return cls((fahrenheit - 32) * 5 / 9)

    @classmethod
    def from_kelvin(cls, kelvin):
        """Alternative constructor: create from a Kelvin value."""
        return cls(kelvin + Temperature.ABSOLUTE_ZERO_C)

    @staticmethod
    def celsius_to_fahrenheit(c):
        """Utility conversion — no instance needed."""
        return c * 9 / 5 + 32

    @property
    def celsius(self):
        return self.__celsius

    @property
    def fahrenheit(self):
        return self.__celsius * 9 / 5 + 32

    def __str__(self):
        return f"{self.__celsius:.2f}°C / {self.fahrenheit:.2f}°F"


boiling = Temperature(100)
body_temp = Temperature.from_fahrenheit(98.6)
abs_zero = Temperature.from_kelvin(0)

print(boiling)                          # 100.00°C / 212.00°F
print(body_temp)                        # 37.00°C / 98.60°F
print(abs_zero)                         # -273.15°C / -459.67°F
print(Temperature.celsius_to_fahrenheit(0))  # 32.0
```

### When to Use Each
| Type | Decorator | First argument | Has access to | Typical use |
|---|---|---|---|---|
| Instance method | *(none)* | `self` | instance attributes + class attributes | Behaviour specific to one object |
| Class method | `@classmethod` | `cls` | class attributes only | Alternative constructors; factory methods |
| Static method | `@staticmethod` | *(none)* | neither instance nor class | Utility/helper functions logically related to the class |

```python
class Dog:
    species = "Canis lupus familiaris"   # class attribute

    def __init__(self, name, breed):
        self.name = name                 # instance attribute
        self.breed = breed

    def bark(self):                      # instance method — uses self
        return f"{self.name} says: Woof!"

    @classmethod
    def get_species(cls):               # class method — uses cls
        return f"All dogs are: {cls.species}"

    @staticmethod
    def is_valid_breed(breed):          # static method — no self or cls
        known_breeds = {"Labrador", "Poodle", "Beagle", "Bulldog", "Collie"}
        return breed in known_breeds


fido = Dog("Fido", "Labrador")

print(fido.bark())                 # Fido says: Woof!
print(Dog.get_species())           # All dogs are: Canis lupus familiaris
print(Dog.is_valid_breed("Poodle"))  # True
print(Dog.is_valid_breed("Dragon"))  # False
```

## 6. Benefits of Abstract Classes for AQA

### Summary of Benefits
```python
# 1. ENFORCEMENT: abstract classes prevent partial implementations
# 2. DOCUMENTATION: the abstract methods document the required contract
# 3. POLYMORPHISM: all concrete subclasses share the same interface
# 4. FLEXIBILITY: swap implementations without changing calling code

from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    """Abstract interface for payment processing."""

    @abstractmethod
    def process_payment(self, amount):
        """Process a payment. Returns True on success."""
        pass

    @abstractmethod
    def refund(self, transaction_id):
        """Refund a transaction. Returns True on success."""
        pass

    @abstractmethod
    def get_balance(self):
        """Return available balance."""
        pass


class StripeProcessor(PaymentProcessor):
    def __init__(self, api_key):
        self.__api_key = api_key
        self.__balance = 10000.00

    def process_payment(self, amount):
        if amount > self.__balance:
            return False
        self.__balance -= amount
        print(f"[Stripe] Processed £{amount:.2f}")
        return True

    def refund(self, transaction_id):
        print(f"[Stripe] Refunding transaction {transaction_id}")
        return True

    def get_balance(self):
        return self.__balance


class PayPalProcessor(PaymentProcessor):
    def __init__(self, email):
        self.__email = email
        self.__balance = 5000.00

    def process_payment(self, amount):
        self.__balance -= amount
        print(f"[PayPal] Processed £{amount:.2f} via {self.__email}")
        return True

    def refund(self, transaction_id):
        print(f"[PayPal] Refunding via {self.__email}, tx: {transaction_id}")
        return True

    def get_balance(self):
        return self.__balance


def checkout(processor, amount):
    """Works with ANY PaymentProcessor — the calling code is unaffected
    when you swap Stripe for PayPal."""
    success = processor.process_payment(amount)
    if success:
        print(f"Payment of £{amount:.2f} succeeded. Balance: £{processor.get_balance():.2f}")
    else:
        print("Payment failed.")


stripe = StripeProcessor("sk_test_abc123")
paypal = PayPalProcessor("shop@example.com")

checkout(stripe, 199.99)
checkout(paypal, 49.99)
```

## Practice Exercises

### Exercise 1: Abstract `Shape` with Concrete Subclasses
Define an abstract `Shape` class (using `ABC`) with abstract methods `area()` and `perimeter()`, plus a concrete method `is_larger_than(other)` that compares areas. Implement `Circle`, `Rectangle`, `Parallelogram`, and `RightAngledTriangle` as concrete subclasses. Demonstrate that attempting to instantiate `Shape` directly raises a `TypeError`.

```python
# Your code here
from abc import ABC, abstractmethod
import math

class Shape(ABC):
    pass

class Circle(Shape):
    pass

class Rectangle(Shape):
    pass

class Parallelogram(Shape):
    pass

class RightAngledTriangle(Shape):
    pass
```

### Exercise 2: Abstract `Animal` with `sound()`
Define an abstract `Animal` class with abstract methods `sound()` and `move()`, and a concrete method `daily_routine()` that calls both. Implement `Dog`, `Eagle`, `Salmon`, and `Snake`. Create a mixed list and call `daily_routine()` on each without any `isinstance` checks.

```python
# Your code here
from abc import ABC, abstractmethod

class Animal(ABC):
    pass

class Dog(Animal):
    pass

class Eagle(Animal):
    pass

class Salmon(Animal):
    pass

class Snake(Animal):
    pass
```

### Exercise 3: Abstract `PaymentProcessor`
Design an abstract `PaymentProcessor` with abstract methods: `authorise(amount)`, `capture(transaction_id)`, `refund(transaction_id)`. Add a concrete method `process(amount)` that calls `authorise` then `capture`. Implement two concrete processors: `CreditCardProcessor` and `CryptoProcessor`. Write a function `run_transactions(processor, amounts)` that processes a list of amounts.

```python
# Your code here
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    pass

class CreditCardProcessor(PaymentProcessor):
    pass

class CryptoProcessor(PaymentProcessor):
    pass
```

### Exercise 4: Abstract `DataStore`
Create an abstract `DataStore` class with abstract methods: `save(key, value)`, `load(key)`, `delete(key)`, `exists(key)`. Implement two concrete subclasses:
- `InMemoryStore` — uses a dictionary internally
- `FileStore` — uses a plain text file (one `key=value` per line)

Write a function `cache_demo(store)` that works with either implementation.

```python
# Your code here
from abc import ABC, abstractmethod

class DataStore(ABC):
    pass

class InMemoryStore(DataStore):
    pass

class FileStore(DataStore):
    pass
```

### Exercise 5: Polymorphic Collection of Abstract Shapes
Build a `Canvas` class that holds a list of `Shape` objects (using the abstract class from Exercise 1). Add methods:
- `add_shape(shape)` — validates the object is a `Shape` instance before adding
- `total_area()` — sum of all areas
- `largest_shape()` — the shape with maximum area
- `shapes_by_area()` — returns shapes sorted by area (smallest first)
- `__str__` — displays all shapes and a total

```python
# Your code here
class Canvas:
    pass
```

## Key Concepts to Remember
- **Abstraction**: hiding implementation details; exposing only the essential interface
- **Abstract class**: a class that cannot be instantiated directly; defines a contract for subclasses
- **`ABC`**: the base class from the `abc` module that your abstract class must inherit from
- **`@abstractmethod`**: a decorator marking a method that subclasses *must* implement; the class becomes abstract if it contains any
- **Concrete class**: a class that implements all abstract methods and *can* be instantiated
- **Interface (by convention)**: an abstract class with only abstract methods, no instance data — serves as a pure specification
- **`@staticmethod`**: a method that belongs to the class but receives neither `self` nor `cls`; used for utility/helper logic that is logically related to the class
- **`@classmethod`**: a method that receives the class (`cls`) as its first argument; used for alternative constructors and factory methods
- **`__subclasshook__`**: (extension) a class method on an ABC that enables virtual subclass registration based on duck typing; not required for AQA
- **Why abstract classes matter for AQA**: they enforce consistent interfaces across an inheritance hierarchy, making polymorphism reliable and safe

## Common Mistakes to Avoid
1. **Forgetting to import `ABC` and `abstractmethod`** — without `from abc import ABC, abstractmethod`, using `@abstractmethod` does nothing; the class is not actually abstract.
2. **Inheriting from `ABC` but forgetting `@abstractmethod`** — a class that inherits `ABC` without any `@abstractmethod` methods is just a regular class; it can be instantiated normally.
3. **Not implementing ALL abstract methods in a subclass** — if even one abstract method is missing, the subclass is also abstract and cannot be instantiated.
4. **Confusing abstract and concrete** — calling methods on the abstract class itself (rather than on instances of a concrete subclass) will raise a `TypeError`.
5. **Putting too much logic in abstract methods** — abstract methods typically have a `pass` body; occasionally you give them a default body (callable via `super()`), but this is advanced and should be used sparingly.

## Extension Challenge
Build a plugin system using abstract base classes. Define an abstract `Plugin(ABC)` class with abstract methods `name()` (property), `version()` (property), `execute(data)`, and `validate(data)`. Add a concrete `PluginManager` class that:
- Maintains a registry of plugins by name
- `register(plugin)` — validates the plugin is a `Plugin` instance and adds it
- `run(name, data)` — finds the plugin by name, calls `validate` then `execute`
- `list_plugins()` — returns all registered plugin names and versions

Implement at least two concrete plugins: `UpperCasePlugin` (converts text to uppercase) and `WordCountPlugin` (counts words). Demonstrate the manager running both.

```python
# Your code here
from abc import ABC, abstractmethod

class Plugin(ABC):
    pass

class PluginManager:
    pass

class UpperCasePlugin(Plugin):
    pass

class WordCountPlugin(Plugin):
    pass
```
