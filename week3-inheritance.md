# Week 3: Inheritance – Building on Existing Classes

## Learning Objectives
- Understand what inheritance is and why it is used
- Define a subclass (child class) that extends a superclass (parent class)
- Use `super()` to call the parent class constructor and methods
- Override parent methods in a subclass
- Use `isinstance()` and `issubclass()` to inspect type hierarchies
- Understand multilevel inheritance and the Method Resolution Order (MRO) for single-inheritance chains
- Know when inheritance is the right design choice
- *(Extension)* Understand multiple inheritance and how the MRO resolves method lookups across multiple parent classes

## 1. What is Inheritance?

### The Core Idea
**Inheritance** allows a class (the *child* or *derived* class) to acquire the attributes and methods of another class (the *parent* or *base* class). The child class inherits everything from the parent and can:
- **Use** inherited methods unchanged
- **Override** (replace) inherited methods with its own version
- **Extend** the parent by adding new attributes and methods

This supports code reuse and models "is-a" relationships (a Dog *is-a* Animal).

```python
# Without inheritance — lots of repeated code
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def eat(self):
        return f"{self.name} is eating."
    def sleep(self):
        return f"{self.name} is sleeping."

class Cat:
    def __init__(self, name, age):
        self.name = name    # duplicated!
        self.age = age
    def eat(self):
        return f"{self.name} is eating."   # duplicated!
    def sleep(self):
        return f"{self.name} is sleeping."  # duplicated!
```

```python
# With inheritance — shared behaviour lives in the parent class
class Animal:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        return f"{self.name} is eating."

    def sleep(self):
        return f"{self.name} is sleeping."

    def __str__(self):
        return f"{self.__class__.__name__}(name={self.name!r}, age={self.age})"


class Dog(Animal):
    """Dog inherits from Animal — gets eat() and sleep() for free."""
    def bark(self):
        return f"{self.name} says: Woof!"


class Cat(Animal):
    """Cat inherits from Animal — gets eat() and sleep() for free."""
    def meow(self):
        return f"{self.name} says: Meow!"


d = Dog("Buddy", 3)
c = Cat("Whiskers", 5)

print(d.eat())   # Buddy is eating.   ← inherited from Animal
print(d.bark())  # Buddy says: Woof!  ← defined in Dog
print(c.sleep()) # Whiskers is sleeping. ← inherited from Animal
print(c.meow())  # Whiskers says: Meow!
print(d)         # Dog(name='Buddy', age=3) ← inherited __str__
```

## 2. The `super()` Function

### Extending the Parent Constructor
When a child class needs its own attributes *in addition to* the parent's, use `super().__init__()` to call the parent constructor rather than rewriting all the shared attribute assignments.

```python
class Animal:
    def __init__(self, name, age, sound):
        self.name = name
        self.age = age
        self.sound = sound

    def speak(self):
        return f"{self.name} says: {self.sound}"

    def describe(self):
        return f"{self.name} is {self.age} years old."


class Dog(Animal):
    def __init__(self, name, age, breed):
        super().__init__(name, age, "Woof")  # call Animal's __init__
        self.breed = breed                   # Dog-specific attribute

    def fetch(self):
        return f"{self.name} fetches the ball!"

    def __str__(self):
        return f"Dog: {self.name} ({self.breed}), age {self.age}"


class GuideDog(Dog):
    def __init__(self, name, age, breed, owner):
        super().__init__(name, age, breed)  # call Dog's __init__
        self.owner = owner                  # GuideDog-specific

    def guide(self):
        return f"{self.name} is guiding {self.owner}."

    def __str__(self):
        return f"GuideDog: {self.name}, owner: {self.owner}"


d = Dog("Rex", 4, "German Shepherd")
g = GuideDog("Lassie", 6, "Collie", "Mr. Jones")

print(d.speak())    # Rex says: Woof  ← inherited from Animal via Dog
print(d.fetch())    # Rex fetches the ball!
print(g.guide())    # Lassie is guiding Mr. Jones.
print(g.describe()) # Lassie is 6 years old. ← inherited from Animal
print(g)            # GuideDog: Lassie, owner: Mr. Jones
```

### Calling Overridden Parent Methods with `super()`
`super()` can also call an overridden parent *method* (not just `__init__`) when you want to extend its behaviour.

```python
class Shape:
    def __init__(self, colour="black"):
        self.colour = colour

    def describe(self):
        return f"A {self.colour} shape."


class Rectangle(Shape):
    def __init__(self, width, height, colour="black"):
        super().__init__(colour)
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def describe(self):
        # Extend the parent's describe() output — don't replace it entirely
        parent_desc = super().describe()
        return f"{parent_desc} Rectangle {self.width}×{self.height}, area={self.area()}"


r = Rectangle(5, 3, "blue")
print(r.describe())
# A blue shape. Rectangle 5×3, area=15
```

## 3. Method Overriding

### Replacing Parent Behaviour
A child class can **override** any parent method simply by defining a method with the same name.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def calculate_bonus(self):
        """Standard bonus: 5% of salary."""
        return self.salary * 0.05

    def annual_cost(self):
        return self.salary + self.calculate_bonus()

    def __str__(self):
        return f"{self.__class__.__name__}: {self.name}, Salary: £{self.salary:,.2f}"


class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)
        self.team_size = team_size

    def calculate_bonus(self):
        """Managers get 10% + £500 per team member — overrides parent."""
        return self.salary * 0.10 + self.team_size * 500

    def __str__(self):
        return super().__str__() + f", Team size: {self.team_size}"


class SalesRep(Employee):
    def __init__(self, name, salary, sales_total):
        super().__init__(name, salary)
        self.sales_total = sales_total

    def calculate_bonus(self):
        """Sales bonus: 5% salary + 2% of total sales."""
        return self.salary * 0.05 + self.sales_total * 0.02


emp = Employee("Alice", 30000)
mgr = Manager("Bob", 60000, 8)
rep = SalesRep("Carol", 25000, 150000)

for person in [emp, mgr, rep]:
    print(f"{person.name}: bonus = £{person.calculate_bonus():,.2f}")

# Alice: bonus = £1,500.00
# Bob: bonus = £10,000.00
# Carol: bonus = £4,250.00
```

> **AQA exam language**: Questions may ask you to *"explain what is meant by overriding a method"* or *"write a subclass that overrides the `calculate_bonus()` method."* The correct definition is: **overriding** means a subclass provides its own implementation of a method with the same name as one in the parent class; when the method is called on an object of the subclass, Python runs the subclass version instead of the parent version.

> **AQA Note**: The AQA specification also calls an overrideable parent method a *virtual method* — see Week 4 for a full explanation of the term, including a Python example.

## 4. `isinstance()` and `issubclass()`

### Inspecting the Type Hierarchy at Runtime
```python
class Vehicle:
    pass

class Car(Vehicle):
    pass

class ElectricCar(Car):
    pass

my_car = ElectricCar()

# isinstance checks if an object is an instance of a class OR any of its parents
print(isinstance(my_car, ElectricCar))  # True
print(isinstance(my_car, Car))          # True  ← ElectricCar is-a Car
print(isinstance(my_car, Vehicle))      # True  ← ElectricCar is-a Vehicle
print(isinstance(my_car, list))         # False

# issubclass checks the class hierarchy (not an object — the class itself)
print(issubclass(ElectricCar, Car))     # True
print(issubclass(ElectricCar, Vehicle)) # True
print(issubclass(Car, ElectricCar))     # False — wrong direction!
print(issubclass(Car, Car))             # True — a class is a subclass of itself

# Practical use: polymorphic function that handles different types
def process_vehicle(v):
    if isinstance(v, ElectricCar):
        print("Charging the electric car...")
    elif isinstance(v, Car):
        print("Filling up with petrol...")
    elif isinstance(v, Vehicle):
        print("Servicing the vehicle...")
```

## 5. Multilevel Inheritance and the MRO

### Multilevel Inheritance
```python
class LivingThing:
    def breathe(self):
        return "Breathing..."

class Animal(LivingThing):
    def __init__(self, name):
        self.name = name

    def eat(self):
        return f"{self.name} is eating."

class Mammal(Animal):
    def feed_young(self):
        return f"{self.name} feeds its young with milk."

class Dog(Mammal):
    def bark(self):
        return f"{self.name} barks."


d = Dog("Rex")
print(d.breathe())     # Breathing...    ← from LivingThing
print(d.eat())         # Rex is eating.  ← from Animal
print(d.feed_young())  # Rex feeds its young with milk. ← from Mammal
print(d.bark())        # Rex barks.      ← from Dog
```

### Method Resolution Order (MRO)

> **AQA core:** Python searches the child class first, then its parent(s) in order, then grandparents. For single-inheritance chains (the common case in AQA questions) this is simply "look in the child, then the parent, then the grandparent". The example above with `LivingThing → Animal → Mammal → Dog` illustrates this.

> **Extension — Multiple Inheritance and the C3 Algorithm:** The material below covers *multiple inheritance* (a class inheriting from two or more parents simultaneously). Multiple inheritance is **not required by the AQA specification** and will not appear in exams as an implementation task, but understanding the MRO in this context is useful enrichment for students who want to explore Python more deeply.

```python
class A:
    def hello(self):
        return "Hello from A"

class B(A):
    def hello(self):
        return "Hello from B"

class C(A):
    def hello(self):
        return "Hello from C"

class D(B, C):  # multiple inheritance — B comes before C
    pass

d = D()
print(d.hello())  # Hello from B  ← B is searched before C

print(D.__mro__)
# (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>,
#  <class '__main__.A'>, <class 'object'>)
```

> Python uses the **C3 linearisation** algorithm to build the MRO for multiple-inheritance hierarchies. You are not expected to know the algorithm name for the AQA exam.

## 6. When to Use Inheritance

### The "is-a" Test
Use inheritance only when the child class genuinely *is a* kind of the parent:
- ✅ `Dog` **is a** `Animal` — use inheritance
- ✅ `Manager` **is an** `Employee` — use inheritance
- ❌ `Car` **is not** `Engine` — `Car` *has an* `Engine` — use composition instead

```python
# Poor design — using inheritance where composition is better
class Engine:
    def start(self):
        return "Engine started."

class Car(Engine):  # Wrong! A Car IS-NOT an Engine
    pass


# Correct design — composition (covered in Week 6)
class Car:
    def __init__(self):
        self.engine = Engine()  # Car HAS-AN Engine

    def start(self):
        return self.engine.start()
```

## Practice Exercises

### Exercise 1: Animal → Dog/Cat Hierarchy
Create a base class `Animal` with attributes `name`, `age`, and `sound`, and methods `speak()`, `eat()`, and `__str__`. Create subclasses `Dog` and `Cat`. `Dog` should add a `fetch(item)` method and override `speak()` to include enthusiasm (e.g. "Woof!! Woof!!"). `Cat` should add a `purr()` method and override `speak()` appropriately.

```python
# Your code here
class Animal:
    pass

class Dog(Animal):
    pass

class Cat(Animal):
    pass
```

### Exercise 2: Shape → Circle/Rectangle
Create a base `Shape` class with a `colour` attribute and abstract-style method `area()` that returns `0` (to be overridden). Create `Circle` (add `radius`) and `Rectangle` (add `width`, `height`) subclasses. Both must override `area()` and `perimeter()`. Add a `describe()` method in the base class that calls `area()` and `perimeter()` polymorphically.

```python
# Your code here
import math

class Shape:
    pass

class Circle(Shape):
    pass

class Rectangle(Shape):
    pass
```

### Exercise 3: Employee → Manager/Engineer
Build an `Employee` base class with `name`, `salary`, and `department`. Add `calculate_bonus()` returning 5% of salary, and `annual_cost()`. Create:
- `Manager(Employee)` — adds `team_size`; bonus is 10% salary + £1000 per team member
- `Engineer(Employee)` — adds `specialisation`; bonus is 7% salary + £2000 if specialisation is `"AI"` or `"Security"`

Override `__str__` in each subclass.

```python
# Your code here
class Employee:
    pass

class Manager(Employee):
    pass

class Engineer(Employee):
    pass
```

### Exercise 4: Vehicle Hierarchy
Create a hierarchy: `Vehicle` → `Car`, `Truck`, `Motorbike`. `Vehicle` has `make`, `model`, `year`, `speed` (default 0). Add methods `accelerate(amount)` and `brake(amount)`. Each subclass adds one unique attribute and overrides `__str__`. Add a top-level function `race(vehicles)` that calls `accelerate(30)` on each vehicle in a list and prints the result polymorphically.

```python
# Your code here
class Vehicle:
    pass

class Car(Vehicle):
    pass

class Truck(Vehicle):
    pass

class Motorbike(Vehicle):
    pass
```

### Exercise 5: `isinstance` and `issubclass` Practice
Using the hierarchy from Exercise 4, write a function `classify_vehicle(v)` that:
- Uses `isinstance` to print what type of vehicle it is (most specific first)
- Also checks with `issubclass` whether `Car`, `Truck`, and `Motorbike` are all subclasses of `Vehicle`
- Demonstrates that `isinstance` returns `True` for parent classes too

```python
# Your code here
def classify_vehicle(v):
    pass
```

## Key Concepts to Remember
- **Inheritance**: a child class acquires attributes and methods from a parent class automatically
- **Parent/Base/Superclass**: the class being inherited from
- **Child/Derived/Subclass**: the class that inherits; defined with `class Child(Parent):`
- **`super()`**: refers to the parent class; used to call the parent's `__init__` or other methods
- **Method overriding**: a child class defines a method with the same name as the parent's, replacing its behaviour
- **MRO (Method Resolution Order)**: the order Python searches the class hierarchy when looking up a method; for single-inheritance chains, this is simply child → parent → grandparent; inspectable via `ClassName.__mro__`
- **`isinstance(obj, Class)`**: returns `True` if `obj` is an instance of `Class` or any of its subclasses
- **`issubclass(Child, Parent)`**: returns `True` if `Child` is derived from `Parent`
- **"is-a" relationship**: the correct test for whether inheritance is appropriate

## Common Mistakes to Avoid
1. **Calling `super()` incorrectly** — always use `super().__init__(...)` inside the child's `__init__`; do not call `ParentClass.__init__(self, ...)` directly (it works but is less Pythonic and breaks with multiple inheritance).
2. **Overriding without calling `super()` when needed** — if the parent method does important setup, forgetting `super().method()` means that setup is lost.
3. **Creating deep inheritance chains (more than 2–3 levels)** — this makes code hard to follow and is usually a sign that composition would be better.
4. **Using inheritance for "has-a" relationships** — a `Car` having an `Engine` should use composition, not inheritance; only use inheritance for genuine "is-a" relationships.
5. **Assuming `isinstance` only matches exact types** — `isinstance(dog_object, Animal)` returns `True` even though `dog_object` was created as a `Dog`; this is intentional and useful.

## Extension Challenge
Build a full `SchoolMember` system. `SchoolMember` is the base class with `name`, `age`, and `school_id`. Subclasses:
- `Teacher(SchoolMember)` — adds `subject`, `salary`; method `teach()` returns what they teach; override `__str__`
- `Student(SchoolMember)` — adds `year_group` and `grades` (dict of subject → grade); methods `add_grade(subject, grade)`, `average_grade()`, `report()`; override `__str__`
- `HeadTeacher(Teacher)` — adds `years_in_post`; override `teach()` to say they manage the school; add `appoint_teacher(teacher)` that adds to a staff list

Write a function `school_assembly(members)` that takes a mixed list of `SchoolMember` objects and prints a different message for each type using `isinstance`.

```python
# Your code here
class SchoolMember:
    pass

class Teacher(SchoolMember):
    pass

class Student(SchoolMember):
    pass

class HeadTeacher(Teacher):
    pass

def school_assembly(members):
    pass
```
