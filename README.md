# A-Level OOP – 8 Week Guide (AQA 7517)

Comprehensive week-by-week Object-Oriented Programming course for Year 13 AQA A-Level Computer Science (7517). From classes to design patterns! 🚀

## Course Overview

This guide covers all OOP content required for AQA A-Level Computer Science, aligned to the *Fundamentals of Programming* section of the specification. It follows the same rhythm and format as the [GCSE Python guide](https://github.com/nuast-dev/gcse-python).

### 📚 Weekly Topics

#### [Week 1: Classes and Objects](/week1-classes-and-objects.md)

The foundations of OOP

- What OOP is and why it matters
- Defining a class with `class` and the `__init__` constructor
- Instance attributes and methods; understanding `self`
- Class attributes vs instance attributes
- `__str__` and `__repr__` for readable output
- Creating multiple objects from the same class

#### [Week 2: Encapsulation](/week2-encapsulation.md)

Information hiding and controlled access

- Public, protected (`_`), and private (`__`) attributes
- Name mangling and why it exists
- Getters (accessors) and setters (mutators)
- The `@property` decorator
- Data validation inside setters
- Why encapsulation improves robustness

#### [Week 3: Inheritance](/week3-inheritance.md)

Building on existing classes

- Base (parent) and derived (child) classes
- `super()` to call parent constructors and methods
- Overriding parent methods in a subclass
- `isinstance()` and `issubclass()`
- Multilevel inheritance and the Method Resolution Order (MRO)
- When inheritance is the right design choice

#### [Week 4: Polymorphism](/week4-polymorphism.md)

One interface, many forms

- Method overriding as run-time polymorphism
- Duck typing in Python
- Operator overloading (`__add__`, `__eq__`, `__lt__`, `__len__`, etc.)
- Writing flexible, interchangeable objects
- Why polymorphism simplifies complex programs

#### [Week 5: Abstract Classes](/week5-abstract-classes.md)

Enforcing a common interface

- What abstraction means in OOP
- The `abc` module: `ABC` and `@abstractmethod`
- Why you cannot instantiate an abstract class
- Concrete subclasses that implement abstract methods
- Python's interface-by-convention approach
- Using abstract classes to enforce contracts

#### [Week 6: Object Associations](/week6-associations.md)

Modelling relationships between classes

- Association ("uses-a")
- Aggregation ("has-a", independent lifecycle)
- Composition ("has-a", dependent lifecycle)
- ASCII / UML class diagram notation
- "has-a" vs "is-a": choosing relationships over inheritance
- Real-world design examples

#### [Week 7: OOP Data Structures](/week7-oop-data-structures.md)

Implementing classic structures with classes

- `Node` class as the building block
- `LinkedList` class (append, prepend, delete, traverse, `__str__`, `__len__`)
- `Stack` class – LIFO with push, pop, peek, is_empty
- `Queue` class – FIFO with enqueue, dequeue, peek, is_empty
- When to use each structure

#### [Week 8: Consolidation](/week8-consolidation.md)

Exam-ready OOP design

- Reviewing all four OOP pillars (encapsulation, inheritance, polymorphism, abstraction)
- Reading and writing UML class diagrams for AQA exams
- SOLID principles overview
- Common AQA exam question patterns for OOP
- Full worked example: designing a larger OOP system from scratch

---

## 🎯 Learning Outcomes

By the end of this course, students will be able to:

- ✓ Design and implement classes with appropriate attributes and methods
- ✓ Apply encapsulation to protect data and enforce validation
- ✓ Use inheritance and method overriding to reuse and extend code
- ✓ Exploit polymorphism to write flexible, maintainable programs
- ✓ Define and implement abstract classes as contracts
- ✓ Model object relationships using association, aggregation, and composition
- ✓ Implement linked lists, stacks, and queues as OOP classes
- ✓ Tackle AQA OOP exam questions with confidence

## 📝 Course Structure

Each week includes:

- **Concept explanations** with clear, commented Python examples
- **Practice exercises** to reinforce learning
- **Extension challenges** for advanced students
- **Common mistakes** to avoid
- **Key concepts** summary

## 🚀 Getting Started

1. Start with [Week 1](/week1-classes-and-objects.md) and work through sequentially
2. Complete all practice exercises before moving to the next week
3. Try extension challenges to deepen understanding
4. Review key concepts regularly — OOP concepts build on each other

## 💡 Study Tips

- **Type the code**: don't just read it, write it yourself
- **Experiment**: modify examples to see what happens
- **Draw diagrams**: sketching class hierarchies aids understanding
- **Test thoroughly**: create objects and call every method
- **Relate to real life**: OOP models real-world systems — think of analogies
- **Review previous weeks**: each week builds on the last

## 📖 Prerequisites

- Solid understanding of Python fundamentals (variables, loops, functions, lists)
- Completion of (or equivalent to) the [GCSE Python guide](https://github.com/nuast-dev/gcse-python)
- Python 3.6+ installed
- A text editor or IDE (VS Code, PyCharm, IDLE, etc.)

## 🎓 AQA Specification Coverage

This course aligns with AQA A-Level Computer Science 7517, *Fundamentals of Programming* (Section 4.1), specifically the OOP content:

- Classes, objects, instantiation
- Encapsulation and information hiding
- Inheritance and method overriding
- Polymorphism
- Abstract classes
- Object relationships and UML diagrams

## 📚 Additional Resources

- [AQA A-Level Computer Science specification (7517)](https://www.aqa.org.uk/subjects/computer-science/a-level/computer-science-7517)
- [Python 3 documentation](https://docs.python.org/3/)
- [Python `abc` module](https://docs.python.org/3/library/abc.html)
