# Week 6: Object Associations – Aggregation and Composition

## Learning Objectives
- Understand the three types of object relationships: association, aggregation, and composition
- Implement each relationship type in Python
- Read and write UML class diagram notation (in ASCII form)
- Apply the "has-a" test to distinguish relationships from "is-a" (inheritance)
- Know when to choose composition over inheritance
- Model dependent and independent object lifecycles
- Identify aggregation and composition as the primary AQA-assessed relationship types; recognise association as supporting design vocabulary

## 1. The Three Object Relationships

### Overview
When designing OOP systems you need to decide *how objects relate to each other*. There are three key types:

| Relationship | Keyword | Lifecycle | UML symbol |
|---|---|---|---|
| **Association** | "uses a" / "knows about" | Independent | `—` (plain line) |
| **Aggregation** | "has a" (weak) | Independent | `◇—` (open diamond) |
| **Composition** | "has a" (strong) | Dependent | `◆—` (filled diamond) |

```
Simple memory aid:
  ASSOCIATION  — objects know about each other but exist independently
  AGGREGATION  — one object contains others, but the parts can exist alone
  COMPOSITION  — one object owns others; the parts CANNOT exist without the whole
```

> **AQA exam focus:** **Aggregation** and **composition** are the two relationship types most likely to be examined in AQA questions. You must be able to:
> - Explain the difference between aggregation and composition (lifecycle dependency)
> - Implement each in Python and identify the correct UML symbol (◇ vs ◆)
> - Apply the "has-a" test to choose between them
>
> **Association** is useful supporting vocabulary for modelling and discussing system designs, but AQA questions are primarily centred on aggregation and composition. Understand association as the background concept from which aggregation and composition are distinguished.

## 2. Association

### Objects That Know About Each Other
An **association** is the loosest relationship. Object A holds a reference to Object B, but neither creates nor destroys the other. They exist independently.

```
UML (ASCII):
  Student ————————— Course
  "A student is enrolled on a course"
```

```python
class Student:
    def __init__(self, name, student_id):
        self.name = name
        self.student_id = student_id
        self.enrolled_courses = []   # holds references to Course objects

    def enrol(self, course):
        """Associate this student with an existing course."""
        if course not in self.enrolled_courses:
            self.enrolled_courses.append(course)
            course.add_student(self)

    def drop(self, course):
        if course in self.enrolled_courses:
            self.enrolled_courses.remove(course)
            course.remove_student(self)

    def __str__(self):
        return f"Student({self.name}, ID={self.student_id})"


class Course:
    def __init__(self, title, code):
        self.title = title
        self.code = code
        self.enrolled_students = []  # holds references to Student objects

    def add_student(self, student):
        if student not in self.enrolled_students:
            self.enrolled_students.append(student)

    def remove_student(self, student):
        if student in self.enrolled_students:
            self.enrolled_students.remove(student)

    def roster(self):
        return [s.name for s in self.enrolled_students]

    def __str__(self):
        return f"Course({self.title}, {self.code})"


# Association in action — objects created independently
cs101 = Course("Intro to CS", "CS101")
maths = Course("Further Maths", "MA201")

alice = Student("Alice", "S001")
bob = Student("Bob", "S002")

alice.enrol(cs101)
alice.enrol(maths)
bob.enrol(cs101)

print(cs101.roster())   # ['Alice', 'Bob']
print(maths.roster())   # ['Alice']

# If alice is deleted, cs101 still exists — independent lifecycles
del alice               # Student object is gone
print(cs101.roster())   # ['Bob'] — course is unaffected
```

## 3. Aggregation

### "Has-A" with Independent Parts
**Aggregation** is a stronger relationship where one object (the *whole*) contains other objects (the *parts*), but the parts can exist independently of the whole. If the whole is destroyed, the parts live on.

```
UML (ASCII):
  Library  ◇————  Book
  "A library has books; books can exist without the library"

  ◇ = open diamond on the 'whole' side
```

```python
class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.available = True

    def __str__(self):
        status = "available" if self.available else "on loan"
        return f'"{self.title}" by {self.author} [{status}]'

    def __repr__(self):
        return f"Book(title={self.title!r}, isbn={self.isbn!r})"


class Library:
    """Library AGGREGATES Books — books exist independently."""

    def __init__(self, name):
        self.name = name
        self.__books = []   # holds references to Book objects (not ownership)

    def add_book(self, book):
        """Add an existing Book to the library."""
        self.__books.append(book)

    def remove_book(self, isbn):
        """Remove a book by ISBN — it still exists outside the library."""
        self.__books = [b for b in self.__books if b.isbn != isbn]

    def checkout(self, isbn):
        for book in self.__books:
            if book.isbn == isbn and book.available:
                book.available = False
                return f'Checked out: {book.title}'
        return "Book not found or not available."

    def return_book(self, isbn):
        for book in self.__books:
            if book.isbn == isbn:
                book.available = True
                return f'Returned: {book.title}'
        return "Book not found."

    def available_books(self):
        return [b for b in self.__books if b.available]

    def search_author(self, author):
        return [b for b in self.__books if b.author.lower() == author.lower()]

    def __len__(self):
        return len(self.__books)

    def __str__(self):
        return f"{self.name} — {len(self)} books in catalogue"


# Books exist independently — they can be created before the library
b1 = Book("1984", "George Orwell", "978-0-14-103614-4")
b2 = Book("Brave New World", "Aldous Huxley", "978-0-06-085052-4")
b3 = Book("Fahrenheit 451", "Ray Bradbury", "978-1-4516-7331-9")

lib = Library("City Library")
lib.add_book(b1)
lib.add_book(b2)
lib.add_book(b3)

print(lib)                     # City Library — 3 books in catalogue
print(lib.checkout("978-0-14-103614-4"))  # Checked out: 1984
print(lib.available_books())   # [Brave New World, Fahrenheit 451]

# If the library is deleted, the Book objects still exist
del lib
print(b1)   # "1984" by George Orwell [on loan] — still alive!
```

## 4. Composition

### "Has-A" with Dependent Parts
**Composition** is the strongest "has-a" relationship. The *whole* **creates** the *parts*, and the parts **cannot meaningfully exist** without the whole. If the whole is destroyed, the parts are destroyed too.

```
UML (ASCII):
  House  ◆————  Room
  "A house is composed of rooms; rooms don't exist without a house"

  ◆ = filled diamond on the 'whole' side
```

```python
class Room:
    """A room — only makes sense as part of a House."""

    def __init__(self, room_type, area_sqm):
        self.room_type = room_type
        self.area_sqm = area_sqm

    def __str__(self):
        return f"{self.room_type} ({self.area_sqm} m²)"

    def __repr__(self):
        return f"Room(room_type={self.room_type!r}, area_sqm={self.area_sqm})"


class House:
    """House COMPOSES Rooms — rooms are created and owned by the house."""

    def __init__(self, address, bedrooms, bathrooms):
        self.address = address
        # Rooms are created BY the House — composition
        self.__rooms = []
        for _ in range(bedrooms):
            self.__rooms.append(Room("Bedroom", 12))
        for _ in range(bathrooms):
            self.__rooms.append(Room("Bathroom", 5))
        self.__rooms.append(Room("Kitchen", 15))
        self.__rooms.append(Room("Living Room", 25))

    def add_room(self, room_type, area_sqm):
        """Create and add a new room to this house."""
        self.__rooms.append(Room(room_type, area_sqm))  # house creates the room

    def total_area(self):
        return sum(r.area_sqm for r in self.__rooms)

    def rooms_of_type(self, room_type):
        return [r for r in self.__rooms if r.room_type == room_type]

    def __str__(self):
        return f"House at {self.address} — {len(self.__rooms)} rooms, {self.total_area()} m²"

    def __repr__(self):
        return f"House(address={self.address!r})"


h = House("10 Downing Street", bedrooms=3, bathrooms=2)
h.add_room("Study", 10)

print(h)                        # House at 10 Downing Street — 8 rooms, 104 m²
print(h.rooms_of_type("Bedroom"))  # [Bedroom (12 m²), Bedroom (12 m²), Bedroom (12 m²)]

# When the house goes, the rooms go — you can't access them from outside
```

### Composition vs Aggregation — Side by Side
```python
# AGGREGATION: the Team aggregates Player objects created externally
class Player:
    def __init__(self, name, position):
        self.name = name
        self.position = position

    def __str__(self):
        return f"{self.name} ({self.position})"


class Team:
    """Aggregates Players — players can exist on multiple teams or no team."""

    def __init__(self, team_name):
        self.team_name = team_name
        self.__players = []

    def sign(self, player):
        """Add an existing player to the team."""
        self.__players.append(player)

    def release(self, player):
        """Remove player — they still exist."""
        self.__players = [p for p in self.__players if p != player]

    def squad(self):
        return self.__players[:]

    def __str__(self):
        names = ", ".join(p.name for p in self.__players)
        return f"{self.team_name}: [{names}]"


p1 = Player("Alice", "Goalkeeper")
p2 = Player("Bob", "Striker")
t = Team("Pythons FC")
t.sign(p1)
t.sign(p2)
print(t)         # Pythons FC: [Alice, Bob]
t.release(p1)
print(p1)        # Alice (Goalkeeper) — still exists after leaving the team


# COMPOSITION: Order creates and owns OrderItems
class OrderItem:
    """Only exists as part of an Order."""

    def __init__(self, product_name, quantity, unit_price):
        self.product_name = product_name
        self.quantity = quantity
        self.unit_price = unit_price

    def line_total(self):
        return self.quantity * self.unit_price

    def __str__(self):
        return f"{self.product_name} x{self.quantity} @ £{self.unit_price:.2f} = £{self.line_total():.2f}"


class Order:
    """Composes OrderItems — items are created and destroyed with the order."""

    _next_id = 1

    def __init__(self, customer_name):
        self.customer_name = customer_name
        self.order_id = Order._next_id
        Order._next_id += 1
        self.__items = []

    def add_item(self, product_name, quantity, unit_price):
        """The Order creates the OrderItem — composition."""
        self.__items.append(OrderItem(product_name, quantity, unit_price))

    def remove_item(self, product_name):
        self.__items = [i for i in self.__items if i.product_name != product_name]

    def total(self):
        return sum(item.line_total() for item in self.__items)

    def receipt(self):
        lines = [f"Order #{self.order_id} for {self.customer_name}"]
        lines += [f"  {item}" for item in self.__items]
        lines.append(f"  TOTAL: £{self.total():.2f}")
        return "\n".join(lines)

    def __str__(self):
        return f"Order #{self.order_id} — {self.customer_name} — £{self.total():.2f}"


o = Order("Carol")
o.add_item("Python Book", 2, 29.99)
o.add_item("USB Hub", 1, 14.99)
o.add_item("Laptop Stand", 1, 39.99)
print(o.receipt())
```

## 5. UML Class Diagram Notation

### Reading and Writing UML for AQA
AQA exams may require you to read class diagrams. Here is the key notation in ASCII form:

```
CLASS BOX:
┌──────────────────────┐
│ ClassName            │  ← class name (bold/underlined in real UML)
├──────────────────────┤
│ - privateAttr: type  │  ← attributes  (- = private, + = public, # = protected)
│ + publicAttr: type   │
├──────────────────────┤
│ + method(): rtnType  │  ← methods
│ - helper(): void     │
└──────────────────────┘

RELATIONSHIPS:
  Association:   A ────────── B     (plain line, with optional label/multiplicity)
  Aggregation:   A ◇────────── B    (open diamond on whole side)
  Composition:   A ◆────────── B    (filled diamond on whole side)
  Inheritance:   A ────────▷ B     (open arrow on parent side)

MULTIPLICITY:
  1    exactly one
  *    zero or more
  1..* one or more
  0..1 zero or one

EXAMPLE DIAGRAM:
              1              *
  Library  ◇────────────  Book
                 has

              1              *
  Order    ◆────────────  OrderItem
              contains

              1              *
  Student  ─────────────  Course
              enrolled on
```

## Practice Exercises

### Exercise 1: Library and Book (Aggregation)
Extend the `Library`/`Book` example above. Add:
- A `Member` class (name, member_id, max_loans=3)
- `Member.borrow(book)` — if member has capacity and book is available; mark book unavailable
- `Member.return_book(book)` — marks book available again
- `Library.get_overdue_members()` — returns members who have borrowed books for more than 14 days (store borrow date using `datetime.date.today()`)

```python
# Your code here
from datetime import date

class Book:
    pass

class Member:
    pass

class Library:
    pass
```

### Exercise 2: House and Room (Composition)
Extend the `House`/`Room` example. Add:
- `Room` gets a `furniture` list (composition within composition: `Furniture` objects created by `Room`)
- `Room.add_furniture(name, value)` — creates a `Furniture` object internally
- `Room.total_furniture_value()` — sums furniture values
- `House.total_value()` — sums all furniture values across all rooms
- `House.most_valuable_room()` — returns the room with highest furniture value

```python
# Your code here
class Furniture:
    pass

class Room:
    pass

class House:
    pass
```

### Exercise 3: Student and Course (Association)
Model a university registration system. `Student` and `Course` have a bidirectional association. Add:
- `Course` has a `max_students` capacity; `enrol()` should fail if full
- `Student.transcript()` — lists all enrolled courses and grades
- `Course.average_grade()` — computes the class average
- `Grade` class to hold student-course-grade triples (association class)

```python
# Your code here
class Grade:
    pass

class Student:
    pass

class Course:
    pass
```

### Exercise 4: Team and Player (Aggregation)
Build a sports management system:
- `Player` has `name`, `position`, `goals_scored`, `matches_played`
- `Team` aggregates Players; add `manager` (a single `Person` object — association)
- `Team.top_scorer()` — player with most goals
- `Team.average_goals_per_match()` — total team goals / total matches
- `League` aggregates Teams; add `add_result(home, away, home_score, away_score)` that updates a table
- `League.table()` — returns teams sorted by points (3 for win, 1 for draw, 0 for loss)

```python
# Your code here
class Person:
    pass

class Player:
    pass

class Team:
    pass

class League:
    pass
```

### Exercise 5: Order and OrderItem (Composition)
Extend the `Order`/`OrderItem` example:
- Add a `Discount` class (composition: created by `Order`) with `code`, `percent`
- `Order.apply_discount(code, percent)` — creates a `Discount` internally
- `Order.total()` — applies the discount if one exists
- Add `status` to `Order` with valid transitions: `"pending"` → `"processing"` → `"shipped"` → `"delivered"`
- `Order.advance_status()` — moves to the next status; raises error if already delivered

```python
# Your code here
class OrderItem:
    pass

class Discount:
    pass

class Order:
    pass
```

## Key Concepts to Remember
- **Association**: a "uses-a" or "knows-about" relationship; objects are independent; neither creates the other
- **Aggregation**: a "has-a" (weak) relationship; the whole contains parts that can exist independently; open diamond (◇) in UML
- **Composition**: a "has-a" (strong) relationship; the whole creates the parts; parts cannot exist without the whole; filled diamond (◆) in UML
- **"has-a" vs "is-a"**: use composition/aggregation for "has-a" (a Car has an Engine); use inheritance for "is-a" (a Dog is an Animal)
- **Lifecycle dependency**: the key distinction between aggregation and composition — in composition, destroying the whole destroys the parts
- **UML multiplicity**: `1`, `*`, `1..*`, `0..1` notation indicates how many objects participate in a relationship
- **Favour composition over inheritance**: a commonly cited design principle — prefer "has-a" when there is no genuine "is-a" relationship, as it leads to more flexible designs

## Common Mistakes to Avoid
1. **Using inheritance when composition is better** — if you are adding "is-a" merely for code reuse rather than a genuine type relationship, composition avoids tight coupling.
2. **Confusing aggregation with composition** — the key question is: *can the part exist without the whole?* If yes → aggregation. If no → composition.
3. **Not modelling dependent lifecycles correctly in composition** — in composition, parts should be created *inside* the whole's methods, not passed in from outside.
4. **Making all associations bidirectional** — only add a back-reference when the design genuinely requires it; unnecessary bidirectional links add complexity and maintenance burden.
5. **Forgetting to clean up associations** — when an object is removed from an association, update both sides; e.g. when a student drops a course, remove the student from the course's list too.

## Extension Challenge
Design a **school management system** that uses all three relationship types:
- **Composition**: `School` composes `Department` objects (departments don't exist without the school); each `Department` composes `Classroom` objects
- **Aggregation**: `Department` aggregates `Teacher` objects (teachers can move between departments); `Course` aggregates `Textbook` objects
- **Association**: `Student` is associated with `Course` (many-to-many); `Teacher` is associated with `Course` they teach

Requirements:
- `School.add_department(name)` — creates a Department internally (composition)
- `Department.hire_teacher(teacher)` — adds an existing Teacher (aggregation)
- `Department.create_course(title, code, teacher)` — creates course internally (composition)
- `Student.enrol(course)` and `Course.enrol_student(student)` — association
- `School.summary()` — prints departments, courses, teacher counts, and student counts

```python
# Your code here
class Textbook:
    pass

class Classroom:
    pass

class Teacher:
    pass

class Student:
    pass

class Course:
    pass

class Department:
    pass

class School:
    pass
```
