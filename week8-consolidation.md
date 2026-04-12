# Week 8: Consolidation – Exam-Ready OOP Design

## Learning Objectives
- Review all four OOP pillars: encapsulation, inheritance, polymorphism, and abstraction
- Apply the three core AQA design principles: encapsulate what varies, favour composition over inheritance, and program to interfaces not implementations
- Apply class design best practices to larger systems
- Read and interpret UML class diagrams as presented in AQA exams
- Understand the SOLID principles at a high level
- Understand and apply three key AQA object-oriented design principles: encapsulate what varies, favour composition over inheritance, and program to interfaces not implementations
- Recognise and answer common AQA OOP exam question patterns
- Build a complete multi-class OOP system from a specification

## Key Terminology

This consolidation week uses terms from all previous weeks. The key terms most likely to appear in AQA exam questions are listed here for quick reference.

- **Encapsulation**: bundling data and methods inside a class while restricting direct external access to internal data.
- **Inheritance**: a subclass acquiring attributes and methods from a parent class via `class Child(Parent):`.
- **Polymorphism**: the ability for the same method call to behave differently depending on the actual type of the object at runtime.
- **Abstraction**: hiding implementation details; exposing only the essential interface via abstract classes and methods.
- **Composition**: a "has-a" (strong) relationship — the whole creates its parts; the parts cannot exist without the whole.
- **Aggregation**: a "has-a" (weak) relationship — the whole contains references to parts that can exist independently.
- **Virtual method**: a method defined in a parent class that is intended to be overridden by subclasses; Python's dynamic dispatch calls the most derived version at runtime.
- **Abstract class**: a class that cannot be instantiated directly; defines a contract that concrete subclasses must fulfil.
- **UML**: Unified Modelling Language — a standard notation for class diagrams. Key symbols: open triangle = inheritance; ◇ = aggregation; ◆ = composition.
- **SOLID**: a set of five design principles for maintainable OOP (extension — not required by AQA specification).

## 1. The Four OOP Pillars — Quick Review

### Encapsulation
Bundle data and behaviour; restrict direct access to internals.
```python
class BankAccount:
    def __init__(self, owner, balance=0.0):
        self.owner = owner
        self.__balance = balance          # private

    @property
    def balance(self):                    # controlled read access
        return self.__balance

    def deposit(self, amount):            # controlled write via method
        if amount > 0:
            self.__balance += amount

    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
        else:
            raise ValueError("Invalid withdrawal amount.")

    def __str__(self):
        return f"{self.owner}: £{self.__balance:.2f}"
```

### Inheritance
Child classes acquire and extend parent class behaviour.
```python
from abc import ABC, abstractmethod

class Account(ABC):
    def __init__(self, owner, balance=0.0):
        self.owner = owner
        self.__balance = balance

    @property
    def balance(self):
        return self.__balance

    def _set_balance(self, value):
        """Protected helper for subclasses to update balance."""
        self.__balance = value

    @abstractmethod
    def account_type(self):
        pass

    def __str__(self):
        return f"[{self.account_type()}] {self.owner}: £{self.balance:.2f}"


class CurrentAccount(Account):
    def __init__(self, owner, overdraft_limit=500.0):
        super().__init__(owner)
        self.overdraft_limit = overdraft_limit

    def account_type(self):
        return "Current"

    def withdraw(self, amount):
        if amount <= self.balance + self.overdraft_limit:
            self._set_balance(self.balance - amount)
        else:
            raise ValueError("Exceeds overdraft limit.")


class SavingsAccount(Account):
    def __init__(self, owner, interest_rate=0.03):
        super().__init__(owner)
        self.interest_rate = interest_rate

    def account_type(self):
        return "Savings"

    def apply_interest(self):
        self._set_balance(self.balance * (1 + self.interest_rate))
```

### Polymorphism
Same method call, different behaviour depending on object type.
```python
def display_accounts(accounts):
    """Works with ANY Account subclass — no isinstance checks needed."""
    for acc in accounts:
        print(acc)          # calls the appropriate __str__
        print(acc.account_type())  # calls the correct override

accounts = [CurrentAccount("Alice", 1000), SavingsAccount("Bob", 0.05)]
display_accounts(accounts)
```

### Abstraction
Hide complexity; expose only the essential interface.
```python
# End users of Account only need to know the public interface:
# .balance, .deposit(), .withdraw(), .account_type(), str()
# They do NOT need to know how balance is stored or calculated internally.
```

## 2. AQA-Aligned Class Design Principles

These three design principles appear widely in A-Level OOP resources and underpin good class design. They complement the four pillars and can help you structure answers to "explain the design choices" questions.

### Encapsulate What Varies
Identify the parts of your design that are likely to change and separate them from the parts that stay stable. Put variation behind a method or attribute so the rest of the code is insulated from change.

```python
# BAD: the discount calculation is buried inside the order — hard to change
class Order:
    def total(self, subtotal):
        return subtotal * 0.9   # hardcoded 10% discount — varies!

# GOOD: encapsulate what varies (the discount rule) in its own class
class FixedDiscount:
    def apply(self, subtotal):
        return subtotal * 0.9

class SeasonalDiscount:
    def apply(self, subtotal):
        return subtotal * 0.8   # 20% during sale

class Order:
    def __init__(self, discount=None):
        self.__discount = discount

    def total(self, subtotal):
        if self.__discount:
            return self.__discount.apply(subtotal)
        return subtotal
```

### Favour Composition Over Inheritance
```python
# Use composition when the relationship is "has-a", not "is-a"
class Engine:
    def __init__(self, horsepower):
        self.horsepower = horsepower

    def start(self):
        return f"Engine ({self.horsepower}hp) started."


class Car:
    def __init__(self, make, model, horsepower):
        self.make = make
        self.model = model
        self.__engine = Engine(horsepower)   # HAS-A engine

    def start(self):
        return self.__engine.start()

    def __str__(self):
        return f"{self.make} {self.model}"
```

### Program to Interfaces, Not Implementations
Write code that depends on an abstract class (or consistent method signatures) rather than on a specific concrete class. This means your calling code remains unchanged when you swap one implementation for another.

```python
from abc import ABC, abstractmethod

class Notifier(ABC):
    """Abstract interface — calling code depends on this, not on Email/SMS."""

    @abstractmethod
    def send(self, message):
        pass


class EmailNotifier(Notifier):
    def send(self, message):
        print(f"[Email] {message}")

class SMSNotifier(Notifier):
    def send(self, message):
        print(f"[SMS] {message}")


def alert_user(notifier: Notifier, message: str):
    """Works with ANY Notifier — depends on the interface, not the implementation."""
    notifier.send(message)

# Swap implementations without touching alert_user
alert_user(EmailNotifier(), "Your order has shipped.")
alert_user(SMSNotifier(),   "Your order has shipped.")
```

### Single Responsibility
Each class should have **one reason to change**. A class that handles both business logic and file I/O is doing too much.

```python
# BAD: one class does too much
class StudentReport:
    def __init__(self, student):
        self.student = student

    def calculate_average(self):
        return sum(self.student.grades) / len(self.student.grades)

    def save_to_file(self, filename):          # ← different responsibility!
        with open(filename, "w") as f:
            f.write(f"{self.student.name}: {self.calculate_average()}")


# BETTER: separate responsibilities
class GradeCalculator:
    @staticmethod
    def average(grades):
        return sum(grades) / len(grades) if grades else 0.0


class ReportWriter:
    @staticmethod
    def save(filename, content):
        with open(filename, "w") as f:
            f.write(content)
```

## 3. Reading UML Class Diagrams for AQA

### AQA Exam Diagram Format
AQA often presents class diagrams like this. You need to be able to:
1. Identify class names, attributes (with types), and method signatures
2. Spot inheritance arrows and understand the hierarchy
3. Identify associations/aggregations and their multiplicities

```
AQA-style class diagram (written as ASCII):

┌─────────────────────────┐
│ <<abstract>>            │
│ Animal                  │
├─────────────────────────┤
│ - name : str            │
│ - age : int             │
├─────────────────────────┤
│ + __init__(name, age)   │
│ + speak() : str  <<abs>>│
│ + eat() : str           │
│ + __str__() : str       │
└────────────┬────────────┘
             △   (inheritance — open triangle pointing to parent)
    ┌────────┴────────┐
    │                 │
┌───┴──────┐   ┌──────┴─────┐
│  Dog     │   │   Cat      │
├──────────┤   ├────────────┤
│- breed   │   │- indoor    │
├──────────┤   ├────────────┤
│+ speak() │   │+ speak()   │
│+ fetch() │   │+ purr()    │
└──────────┘   └────────────┘

Relationships:
  △ or ▷  = inheritance (open arrowhead points to parent)
  ◇       = aggregation (open diamond on the "whole" side)
  ◆       = composition (filled diamond)
  ——      = association

Visibility:
  + public
  - private
  # protected
```

### Translating a Diagram to Python
```python
# From the UML above:
from abc import ABC, abstractmethod

class Animal(ABC):
    def __init__(self, name: str, age: int):
        self._name = name    # protected
        self._age = age

    @abstractmethod
    def speak(self) -> str:
        pass

    def eat(self) -> str:
        return f"{self._name} is eating."

    def __str__(self) -> str:
        return f"{self.__class__.__name__}(name={self._name!r}, age={self._age})"


class Dog(Animal):
    def __init__(self, name: str, age: int, breed: str):
        super().__init__(name, age)
        self.__breed = breed

    def speak(self) -> str:
        return f"{self._name} says: Woof!"

    def fetch(self) -> str:
        return f"{self._name} fetches the ball!"


class Cat(Animal):
    def __init__(self, name: str, age: int, indoor: bool):
        super().__init__(name, age)
        self.__indoor = indoor

    def speak(self) -> str:
        return f"{self._name} says: Meow!"

    def purr(self) -> str:
        return f"{self._name} purrs contentedly."
```

## 4. SOLID Principles — Brief Overview (Extension)

> **Teacher note (extension — beyond core AQA):** The SOLID principles are **not named or tested in the AQA specification**. You will not be asked to name, define, or apply them in an AQA exam. They are included here as enrichment because they underpin good professional OOP design and complement the concepts you have studied. Read this section to broaden your understanding; do not prioritise it over the examinable content in Sections 1–3 and 5.

The **SOLID** principles are guidelines for writing maintainable OOP code.

| Letter | Principle | One-line summary |
|---|---|---|
| **S** | Single Responsibility | A class should have only one reason to change |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subclasses must be usable wherever the parent is used |
| **I** | Interface Segregation | Prefer small, specific interfaces over large general ones |
| **D** | Dependency Inversion | Depend on abstractions, not concrete implementations |

```python
# Open/Closed Principle in practice:
# Instead of modifying existing code when adding new behaviour,
# extend it by creating new subclasses.

class Discount(ABC):
    @abstractmethod
    def apply(self, price):
        pass

class PercentageDiscount(Discount):
    def __init__(self, percent):
        self.percent = percent
    def apply(self, price):
        return price * (1 - self.percent / 100)

class FixedDiscount(Discount):
    def __init__(self, amount):
        self.amount = amount
    def apply(self, price):
        return max(0, price - self.amount)

# To add a new discount type, add a new subclass — no existing code changes.
```

## 5. AQA Object-Oriented Design Principles

These three principles are explicitly relevant to AQA A-Level Computer Science. They build directly on what you have already studied in earlier weeks and give you precise language to use in exam answers.

---

### Principle 1 — Encapsulate What Varies

**What it means:**  
Identify the parts of your design that are likely to *change*, and hide them behind a stable, unchanging interface. The rest of the system can then remain untouched even when the varying part is modified.

**Link to Week 2 (Encapsulation):**  
You already know how to make attributes private and expose them through getters, setters, and `@property`. *Encapsulate what varies* applies the same thinking at the level of whole methods or behaviours: if one part of a class is more likely to change than the rest, isolate it so changes cannot ripple outwards.

**Example — payment calculation that might change:**

```python
class Invoice:
    """The tax calculation is encapsulated privately.
    If the tax rules change, only _calculate_tax() needs updating —
    nothing that uses Invoice changes at all."""

    VAT_RATE = 0.20

    def __init__(self, subtotal):
        self.__subtotal = subtotal

    def __calculate_tax(self):             # private — the part that varies
        return self.__subtotal * self.VAT_RATE

    def total(self):                        # stable public interface
        return self.__subtotal + self.__calculate_tax()

    def __str__(self):
        return (f"Subtotal: £{self.__subtotal:.2f}  "
                f"Tax: £{self.__calculate_tax():.2f}  "
                f"Total: £{self.total():.2f}")


inv = Invoice(100.00)
print(inv)  # Subtotal: £100.00  Tax: £20.00  Total: £120.00
```

> If the government changes the VAT rate or the formula, you update `__calculate_tax()` in one place. Every call to `total()` automatically benefits.

**Exam scenario:**  
A `Game` class has a `calculate_score()` method. The scoring system is likely to change as the game is updated. By making the calculation a private method called from the stable `get_score()` public method, external code never needs to change.

---

### Principle 2 — Favour Composition Over Inheritance

**What it means:**  
When you want to reuse or extend behaviour, prefer *containing* an object (composition/aggregation — "has-a") over *inheriting* from a class ("is-a"). Composition gives you flexibility because you can swap the contained object at runtime; inheritance creates a permanent, rigid coupling.

**Link to Week 6 (Object Relationships — Aggregation and Composition):**  
Week 6 showed you the difference between aggregation and composition. This principle says: when you feel tempted to inherit just to reuse code, ask *"is this genuinely an is-a relationship?"* If not, use composition instead.

**Example — inheritance gone wrong vs composition done right:**

```python
# ── WRONG: inheriting just to reuse start() ──────────────────────────────
class Engine:
    def start(self):
        return "Engine started."

class Car(Engine):               # A Car IS-A Engine? No — that's wrong.
    pass                         # But it works... and that's the danger.

c = Car()
print(c.start())   # "Engine started." — but the design is misleading


# ── RIGHT: composition — a Car HAS-A Engine ──────────────────────────────
class Engine:
    def __init__(self, horsepower):
        self.__horsepower = horsepower

    def start(self):
        return f"Engine ({self.__horsepower} hp) started."


class Car:
    def __init__(self, make, model, horsepower):
        self.make = make
        self.model = model
        self.__engine = Engine(horsepower)    # HAS-A relationship

    def start(self):
        return self.__engine.start()          # delegates to the engine

    def upgrade_engine(self, new_horsepower):
        self.__engine = Engine(new_horsepower)  # swap engine at runtime!

    def __str__(self):
        return f"{self.make} {self.model}"


car = Car("Ford", "Mustang", 450)
print(car.start())             # Engine (450 hp) started.
car.upgrade_engine(600)
print(car.start())             # Engine (600 hp) started.
```

> You cannot "swap" an inherited method at runtime; you can always replace a contained object.

**Exam scenario:**  
A `Logger` class exists that writes log messages to a file. A `Database` class wants logging. Rather than `Database(Logger)` (wrong — a database is not a logger), compose: `Database` holds a `Logger` object and calls `self.__logger.log(...)`. If you later want to log to a screen instead, you swap the `Logger` object; the `Database` class is unchanged.

---

### Principle 3 — Program to Interfaces, Not Implementations

**What it means:**  
Write your code to depend on the *abstract type* (the interface — what something can do) rather than on a specific *concrete class* (how it does it). This means your code works with any object that satisfies the interface, making it easy to swap implementations.

**Link to Week 5 (Abstract Classes):**  
In Week 5 you created abstract base classes with `@abstractmethod`. Those abstract classes *are* the interfaces in Python. When you write a function that accepts a `PaymentProcessor` (abstract) rather than a `StripeProcessor` (concrete), that function works with any payment processor you ever write.

**Example — depending on an abstraction, not a concrete class:**

```python
from abc import ABC, abstractmethod

# ── The interface (abstract class) ───────────────────────────────────────
class Notifier(ABC):
    """Defines WHAT a notifier does, not HOW."""

    @abstractmethod
    def send(self, recipient, message):
        """Send a notification. Returns True on success."""
        pass


# ── Concrete implementations ──────────────────────────────────────────────
class EmailNotifier(Notifier):
    def send(self, recipient, message):
        print(f"[Email → {recipient}] {message}")
        return True


class SMSNotifier(Notifier):
    def send(self, recipient, message):
        print(f"[SMS → {recipient}] {message}")
        return True


class PushNotifier(Notifier):
    def send(self, recipient, message):
        print(f"[Push → {recipient}] {message}")
        return True


# ── Code that depends on the INTERFACE, not a concrete class ──────────────
def notify_user(notifier: Notifier, user_email, message):
    """Works with ANY Notifier — email, SMS, push, or one not yet written."""
    success = notifier.send(user_email, message)
    if success:
        print("Notification sent.")


# Swap the implementation without changing notify_user at all:
notify_user(EmailNotifier(), "alice@example.com", "Your order has shipped!")
notify_user(SMSNotifier(),   "alice@example.com", "Your order has shipped!")
notify_user(PushNotifier(),  "alice@example.com", "Your order has shipped!")
```

> `notify_user` never mentions `EmailNotifier` or `SMSNotifier`. It only knows about `Notifier`. Swapping implementations requires changing a single line at the call site.

**Exam scenario:**  
A `ReportGenerator` function takes a `DataStore` parameter (an abstract class with `save` and `load` methods). In tests you pass an `InMemoryStore`; in production you pass a `DatabaseStore`. The `ReportGenerator` code never changes.

---

### Connecting the Three Principles

| Principle | Key question to ask | Week it connects to |
|---|---|---|
| **Encapsulate what varies** | *"Which part of this class is most likely to change?"* | Week 2 — Encapsulation |
| **Favour composition over inheritance** | *"Is this genuinely an is-a relationship, or am I just reusing code?"* | Week 6 — Associations |
| **Program to interfaces** | *"Can I write this to accept an abstract type instead of a concrete one?"* | Week 5 — Abstract Classes |

---

## 6. Common AQA Exam Question Patterns

### Pattern 1 — "Define a class"
> *"Write a Python class `Circle` with a private attribute `__radius`. Include a constructor, a `area()` method, and a `__str__` method."*

```python
import math

class Circle:
    def __init__(self, radius):
        self.__radius = radius   # private

    @property
    def radius(self):
        return self.__radius

    def area(self):
        return math.pi * self.__radius ** 2

    def __str__(self):
        return f"Circle(radius={self.__radius})"
```

### Pattern 2 — "Extend a class"
> *"The class `Shape` is defined above. Write a subclass `Rectangle` that inherits from `Shape` and overrides the `area()` method."*

```python
class Rectangle(Shape):
    def __init__(self, width, height, colour="black"):
        super().__init__(colour)   # always call super().__init__()
        self.width = width
        self.height = height

    def area(self):                # override the parent method
        return self.width * self.height
```

### Pattern 3 — "Trace the output"
> *"What is the output of the following code?"*

```python
class Vehicle:
    count = 0
    def __init__(self, make):
        self.make = make
        Vehicle.count += 1

    def __str__(self):
        return f"Vehicle: {self.make}"

class Car(Vehicle):
    def __init__(self, make, doors):
        super().__init__(make)
        self.doors = doors

    def __str__(self):
        return f"Car: {self.make}, {self.doors} doors"

v = Vehicle("Generic")
c = Car("Ford", 4)
print(v)             # Vehicle: Generic
print(c)             # Car: Ford, 4 doors
print(Vehicle.count) # 2  ← both Vehicle() and Car() incremented it
print(isinstance(c, Vehicle))  # True
```

### Pattern 4 — "Explain encapsulation"
> *"Explain why the `__balance` attribute is declared as private in the BankAccount class."*

**Model answer**: The attribute is declared private (using double underscore name mangling) so that it cannot be accessed or modified directly from outside the class. This prevents the balance being set to an invalid value (e.g. negative). Instead, all changes go through the `deposit()` and `withdraw()` methods, which contain validation logic. This protects the integrity of the object's data.

## 7. Worked Example: A Full OOP System

### Design: `OnlineShop`
```python
from abc import ABC, abstractmethod


# ── Product hierarchy ────────────────────────────────────────────────────

class Product(ABC):
    """Abstract base for all products."""

    def __init__(self, product_id, name, price, stock):
        self.__product_id = product_id
        self.__name = name
        self.__price = price
        self.__stock = stock

    @property
    def product_id(self):
        return self.__product_id

    @property
    def name(self):
        return self.__name

    @property
    def price(self):
        return self.__price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Price cannot be negative.")
        self.__price = value

    @property
    def stock(self):
        return self.__stock

    def reduce_stock(self, quantity):
        if quantity > self.__stock:
            raise ValueError(f"Only {self.__stock} units available.")
        self.__stock -= quantity

    @abstractmethod
    def category(self):
        pass

    def __str__(self):
        return f"[{self.category()}] {self.name} — £{self.price:.2f} (stock: {self.stock})"


class PhysicalProduct(Product):
    def __init__(self, product_id, name, price, stock, weight_kg):
        super().__init__(product_id, name, price, stock)
        self.weight_kg = weight_kg

    def category(self):
        return "Physical"

    def shipping_cost(self):
        return round(self.weight_kg * 1.50, 2)


class DigitalProduct(Product):
    def __init__(self, product_id, name, price, stock, file_size_mb):
        super().__init__(product_id, name, price, stock)
        self.file_size_mb = file_size_mb

    def category(self):
        return "Digital"

    def shipping_cost(self):
        return 0.0   # no shipping for digital products


# ── Order system (composition) ───────────────────────────────────────────

class OrderLine:
    """Composed by Order — does not exist independently."""

    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity

    def line_total(self):
        return self.product.price * self.quantity

    def __str__(self):
        return f"  {self.product.name} x{self.quantity} = £{self.line_total():.2f}"


class Order:
    _next_id = 1

    def __init__(self, customer):
        self.order_id = Order._next_id
        Order._next_id += 1
        self.customer = customer
        self.__lines = []
        self.__status = "pending"
        self.__discount = None

    def add_product(self, product, quantity):
        product.reduce_stock(quantity)   # validates stock
        self.__lines.append(OrderLine(product, quantity))

    def apply_discount(self, percent):
        if not 0 < percent <= 100:
            raise ValueError("Discount must be between 0 and 100.")
        self.__discount = percent

    def subtotal(self):
        return sum(line.line_total() for line in self.__lines)

    def total(self):
        sub = self.subtotal()
        if self.__discount:
            sub *= (1 - self.__discount / 100)
        return round(sub, 2)

    def advance_status(self):
        transitions = {"pending": "processing", "processing": "shipped",
                       "shipped": "delivered"}
        if self.__status not in transitions:
            raise ValueError(f"Order is already {self.__status}.")
        self.__status = transitions[self.__status]

    @property
    def status(self):
        return self.__status

    def receipt(self):
        lines = [f"Order #{self.order_id} | Customer: {self.customer.name} | Status: {self.__status}"]
        lines += [str(line) for line in self.__lines]
        if self.__discount:
            lines.append(f"  Discount: {self.__discount}%")
        lines.append(f"  TOTAL: £{self.total():.2f}")
        return "\n".join(lines)


# ── Customer (aggregation with Order) ────────────────────────────────────

class Customer:
    def __init__(self, customer_id, name, email):
        self.customer_id = customer_id
        self.name = name
        self.__email = email
        self.__orders = []

    @property
    def email(self):
        return self.__email

    def place_order(self):
        order = Order(self)
        self.__orders.append(order)
        return order

    def order_history(self):
        return self.__orders[:]

    def total_spent(self):
        return sum(o.total() for o in self.__orders)

    def __str__(self):
        return f"Customer({self.customer_id}): {self.name} — {len(self.__orders)} orders"


# ── Demo ─────────────────────────────────────────────────────────────────

laptop = PhysicalProduct("P001", "Laptop", 899.99, 10, 2.1)
ebook = DigitalProduct("D001", "Python Guide", 14.99, 999, 45)

alice = Customer("C001", "Alice", "alice@example.com")
order = alice.place_order()
order.add_product(laptop, 1)
order.add_product(ebook, 2)
order.apply_discount(10)
print(order.receipt())
order.advance_status()
print(f"Status: {order.status}")
print(alice)
print(f"Total spent: £{alice.total_spent():.2f}")
```

## Practice Tasks

### Exercise 1: Zoo System from a Class Diagram
Implement the following class hierarchy:

```
<<abstract>>
Animal
  - name: str
  - age: int
  + sound(): str  <<abstract>>
  + feed(food): str
      ▲
      |
  ┌───┴──────────────┐
  │                  │
Mammal           Bird
- fur_colour        - wingspan
+ warm_blooded()    + fly(): str
      ▲                 ▲
      |                 |
  ┌───┴───┐         ┌───┴───┐
Lion    Elephant  Parrot   Penguin
```

Add a `Zoo` class that aggregates animals and supports `add_animal`, `feeding_time()` (calls `feed` for each), `all_sounds()`, and `find_by_name(name)`.

```python
# Your code here
from abc import ABC, abstractmethod

class Animal(ABC):
    pass

class Mammal(Animal):
    pass

class Bird(Animal):
    pass

class Lion(Mammal):
    pass

class Elephant(Mammal):
    pass

class Parrot(Bird):
    pass

class Penguin(Bird):
    pass

class Zoo:
    pass
```

### Exercise 2: Cinema Booking System
Design and implement a `Cinema` booking system:
- `Film` — title, duration, rating (U/PG/12/15/18), available_seats
- `Screening` — composes a Film, has date, time, screen_number; manage seat bookings
- `Booking` — created by Screening; has customer_name, seats_booked, booking_ref
- `Cinema` — aggregates Screenings; `add_screening`, `find_screenings(title)`, `book_seats(screening_id, name, seats)`
- Encapsulate all private data; validate seats > 0 and not exceeding available

```python
# Your code here
class Film:
    pass

class Screening:
    pass

class Booking:
    pass

class Cinema:
    pass
```

### Exercise 3: Bank System with Accounts and Transactions
Build a `Bank` system:
- Abstract `Account` class — owner, balance (private), `deposit`, `withdraw`, abstract `account_type`
- `CurrentAccount(Account)` — overdraft limit; `withdraw` allows going negative to the limit
- `SavingsAccount(Account)` — interest rate; `apply_interest()` method
- `Transaction` — composed by Account; stores amount, type ("deposit"/"withdrawal"), timestamp
- `Bank` — aggregates Accounts; `open_account`, `find_account(id)`, `total_deposits()`, `generate_report()`

```python
# Your code here
from abc import ABC, abstractmethod
from datetime import datetime

class Transaction:
    pass

class Account(ABC):
    pass

class CurrentAccount(Account):
    pass

class SavingsAccount(Account):
    pass

class Bank:
    pass
```

### Exercise 4: Game with Player/Enemy/Boss Hierarchy
Create a simple text-based game hierarchy:
- Abstract `Character` — name, health, attack_power; abstract `attack(target)`, `take_damage(amount)`
- `Player(Character)` — adds `level`, `experience`; `gain_xp(amount)` levels up at 100 XP; override `attack`
- `Enemy(Character)` — adds `reward_xp`; override `attack`
- `Boss(Enemy)` — adds `phase` (1 or 2); at < 50% health switches to phase 2 doubling attack power
- `Game` — manages a Player and a list of Enemies; `run_battle(player, enemy)` simulates combat turn by turn

```python
# Your code here
from abc import ABC, abstractmethod

class Character(ABC):
    pass

class Player(Character):
    pass

class Enemy(Character):
    pass

class Boss(Enemy):
    pass

class Game:
    pass
```

### Exercise 5: Library Management System
Design a full `Library Management System`:
- `Book` — isbn, title, author, year, genre, available
- `Member` — member_id, name, email, max_loans (default 3), list of current loans
- `Loan` — composed by Member; book reference, loan_date, due_date (14 days), return_date
- `Librarian` — name, staff_id; methods `issue_book(member, book)`, `return_book(member, book)`, `renew_loan(member, book)`
- `Library` — aggregates Books, Members, and Librarians; `search_books(query)`, `overdue_loans()`, `member_report(member_id)`, `catalogue()`

```python
# Your code here
from datetime import date, timedelta

class Book:
    pass

class Loan:
    pass

class Member:
    pass

class Librarian:
    pass

class Library:
    pass
```

## Exam Focus — Quick Reference

Use these definitions word-for-word (or close to them) in AQA exam answers.

| Principle / Concept | Exam-ready definition |
|---|---|
| **Encapsulation** | Bundling an object's data and the methods that operate on it inside a class, while restricting direct access to internal data from outside the class. |
| **Encapsulate what varies** | Identifying the parts of a design that are likely to change and hiding them behind a stable interface, so that changes to one part do not affect the rest of the system. |
| **Favour composition over inheritance** | Preferring a "has-a" relationship (where one object contains another) over an "is-a" relationship (inheritance) when the aim is code reuse rather than a genuine type hierarchy, because composition produces more flexible and loosely coupled designs. |
| **Program to interfaces, not implementations** | Writing code that depends on an abstract type (interface or abstract class) rather than a specific concrete class, so that implementations can be swapped without changing the calling code. |
| **Abstraction** | Hiding implementation details and exposing only the essential interface, achieved in Python using abstract base classes (`ABC`) and `@abstractmethod`. |
| **Polymorphism** | The ability for objects of different classes to respond to the same method call in their own way, allowing code to be written that works with any object in a class hierarchy without needing to know its specific type. |
| **Inheritance** | A mechanism by which a subclass acquires the attributes and methods of a superclass, supporting code reuse and an "is-a" type relationship. |
| **Composition** | A "has-a" relationship in which one object creates and owns other objects; the parts cannot exist independently of the whole. |
| **Aggregation** | A "has-a" relationship in which one object contains references to other objects that can exist independently of the containing object. |
| **Abstract class** | A class that cannot be instantiated directly; it defines a contract (a set of abstract methods) that concrete subclasses must implement. |

---

## Key Concepts to Remember
- **Encapsulation**: use `__private` attributes with `@property`/setters; validate in setters; bundle data with behaviour
- **Inheritance**: use `class Child(Parent):`; call `super().__init__(...)` in the child constructor; override methods when the child needs different behaviour
- **Polymorphism**: write code that calls methods by name on objects; Python dispatches to the correct version at runtime; virtual methods make this work through dynamic dispatch; duck typing — the principle that any object with the right method works regardless of type — is an enrichment concept, not a named AQA term
- **Abstraction**: use `ABC` and `@abstractmethod` to define contracts; concrete subclasses must implement all abstract methods; cannot instantiate abstract classes directly
- **Encapsulate what varies**: identify the parts of a class most likely to change; make them private; expose only a stable public interface — the rest of the system is unaffected when the internal detail changes
- **Favour composition over inheritance**: when the relationship is "has-a", compose objects rather than inherit; the whole creates or holds its parts; contained objects can be swapped at runtime, giving greater flexibility than inheritance
- **Program to interfaces, not implementations**: write functions and classes that accept abstract types (abstract base classes); concrete implementations can then be swapped freely without changing calling code
- **UML notation for AQA**: `+` public, `-` private, `#` protected; open triangle for inheritance; open/filled diamond for aggregation/composition; multiplicity labels (1, *, 1..*)
- **`super()`**: always call in child constructors to ensure parent initialisation runs
- **`isinstance` / `issubclass`**: use to safely check types at runtime without breaking polymorphism

> **Exam focus:** AQA consolidation questions often combine multiple OOP concepts in a single scenario. When answering: (1) identify what relationship type applies (is-a → inheritance; has-a with independent lifecycle → aggregation; has-a with dependent lifecycle → composition); (2) use precise terminology (encapsulation, polymorphism, virtual method, abstract class, etc.); (3) in code questions, always include `__init__`, private attributes with appropriate accessors, and `__str__`. Refer to the "Exam Focus — Quick Reference" table above for exam-ready definitions.

## Common Mistakes to Avoid
1. **Forgetting `super().__init__()` in child class constructors** — without it, the parent's `__init__` never runs and its attributes are not set up.
2. **Accessing `__private` attributes from a subclass** — double-underscore name mangling means `self.__balance` in `Account` becomes `_Account__balance`; a subclass cannot access it as `self.__balance`.
3. **Returning `None` from operator overloads** — `__add__`, `__str__`, `__lt__` etc. must always `return` a value.
4. **Instantiating abstract classes** — forgetting `@abstractmethod` on one method means the class is not truly abstract; always verify by trying to instantiate it.
5. **Not validating in setters / `__init__`** — the object should always be in a valid state; if validation logic exists in the setter, route `__init__` assignments through the setter (`self.price = price`) not directly to the private attribute.
6. **Using `isinstance` instead of polymorphism** — long `if isinstance(obj, A): ... elif isinstance(obj, B): ...` chains should usually be replaced by polymorphic method calls.
7. **Mutable default arguments in `__init__`** — `def __init__(self, items=[])` shares the list across all instances; use `def __init__(self, items=None): self.items = items if items is not None else []`.

## Extension
Design and implement a **Hospital Management System** from scratch using all OOP concepts covered in this course. The system must include:

**Classes:**
- Abstract `Person` — name, dob, contact details; abstract `role()` property
- `Patient(Person)` — patient_id, NHS number, medical history (list of `Diagnosis` objects — composition), current admissions
- `Doctor(Person)` — GMC number, specialisation, ward; methods `diagnose(patient, condition)`, `prescribe(patient, medication)`
- `Nurse(Person)` — NMC number, ward; method `administer(patient, medication)`
- `Diagnosis` — condition, date, doctor; composed by Patient
- `Prescription` — medication, dosage, prescribed_by, date; composed by Patient
- `Ward` — ward_name, capacity; aggregates Patients and has Doctor/Nurse staff
- `Hospital` — composes Wards; aggregates Doctors and Nurses; methods `admit_patient`, `discharge_patient`, `find_patient(id)`, `available_beds()`, `staff_report()`

**Requirements:**
- Full encapsulation with `@property` and setters where appropriate
- Correct use of composition (Ward composes Beds, Patient owns Diagnoses/Prescriptions) and aggregation (Hospital aggregates staff)
- Polymorphism: `role()` property returns `"Patient"`, `"Doctor"`, or `"Nurse"` for the correct subclass
- All data validated: negative ages rejected, capacity never exceeded, etc.

```python
# Your code here
from abc import ABC, abstractmethod
from datetime import date

class Person(ABC):
    pass

class Patient(Person):
    pass

class Doctor(Person):
    pass

class Nurse(Person):
    pass

class Diagnosis:
    pass

class Prescription:
    pass

class Ward:
    pass

class Hospital:
    pass
```
