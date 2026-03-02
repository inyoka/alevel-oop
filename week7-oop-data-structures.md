# Week 7: OOP Data Structures – Linked Lists, Stacks, and Queues

## Learning Objectives
- Implement a `Node` class as the fundamental building block
- Build a `LinkedList` class with full traversal, search, insert, and delete operations
- Implement a `Stack` class (LIFO) with push, pop, and peek
- Implement a `Queue` class (FIFO) with enqueue, dequeue, and peek
- Understand when to use each data structure
- Appreciate how encapsulation applies to data structure design

## 1. The Node Class

### Building Blocks
A **node** is the fundamental unit of linked data structures. Each node holds a piece of data and one or more **pointers** (references) to the next node(s).

```
Visualising a singly linked list:

  HEAD
   │
   ▼
 ┌────┬────┐   ┌────┬────┐   ┌────┬────┐   ┌────┬──────┐
 │ 10 │  ──┼──►│ 20 │  ──┼──►│ 30 │  ──┼──►│ 40 │ None │
 └────┴────┘   └────┴────┘   └────┴────┘   └────┴──────┘
  data  next    data  next    data  next    data  next (tail)
```

```python
class Node:
    """A single node in a linked list."""

    def __init__(self, data):
        self.data = data
        self.next = None   # pointer to the next node (None = end of list)

    def __str__(self):
        return str(self.data)

    def __repr__(self):
        return f"Node(data={self.data!r})"


# Creating nodes manually (before we have a LinkedList)
n1 = Node(10)
n2 = Node(20)
n3 = Node(30)

# Link them together by hand
n1.next = n2
n2.next = n3

# Traverse: follow .next pointers until None
current = n1
while current is not None:
    print(current.data, end=" → ")
    current = current.next
print("None")
# 10 → 20 → 30 → None
```

## 2. Linked List

### The `LinkedList` Class
```python
class LinkedList:
    """A singly linked list with full CRUD operations."""

    def __init__(self):
        self.__head = None   # private — external code should use methods
        self.__size = 0

    # ── Core operations ────────────────────────────────────────────────

    def append(self, data):
        """Add a node at the END of the list — O(n)."""
        new_node = Node(data)
        if self.__head is None:
            self.__head = new_node
        else:
            current = self.__head
            while current.next is not None:
                current = current.next
            current.next = new_node
        self.__size += 1

    def prepend(self, data):
        """Add a node at the START of the list — O(1)."""
        new_node = Node(data)
        new_node.next = self.__head
        self.__head = new_node
        self.__size += 1

    def insert_at(self, index, data):
        """Insert a node at a specific index — O(n)."""
        if index < 0 or index > self.__size:
            raise IndexError(f"Index {index} out of range (size={self.__size}).")
        if index == 0:
            self.prepend(data)
            return
        new_node = Node(data)
        current = self.__head
        for _ in range(index - 1):
            current = current.next
        new_node.next = current.next
        current.next = new_node
        self.__size += 1

    def delete(self, data):
        """Remove the first node with the given data — O(n)."""
        if self.__head is None:
            raise ValueError("Cannot delete from an empty list.")
        # Special case: deleting the head node
        if self.__head.data == data:
            self.__head = self.__head.next
            self.__size -= 1
            return
        current = self.__head
        while current.next is not None:
            if current.next.data == data:
                current.next = current.next.next  # skip over the target node
                self.__size -= 1
                return
            current = current.next
        raise ValueError(f"{data!r} not found in list.")

    def search(self, data):
        """Return the index of the first occurrence, or -1 if not found — O(n)."""
        current = self.__head
        index = 0
        while current is not None:
            if current.data == data:
                return index
            current = current.next
            index += 1
        return -1

    def get(self, index):
        """Return the data at a given index — O(n)."""
        if index < 0 or index >= self.__size:
            raise IndexError(f"Index {index} out of range.")
        current = self.__head
        for _ in range(index):
            current = current.next
        return current.data

    def to_list(self):
        """Return all data values as a Python list."""
        result = []
        current = self.__head
        while current is not None:
            result.append(current.data)
            current = current.next
        return result

    def reverse(self):
        """Reverse the list in place — O(n)."""
        prev = None
        current = self.__head
        while current is not None:
            next_node = current.next   # save next
            current.next = prev        # reverse the pointer
            prev = current             # move prev forward
            current = next_node        # move current forward
        self.__head = prev

    def is_empty(self):
        return self.__size == 0

    # ── Dunder methods ─────────────────────────────────────────────────

    def __len__(self):
        return self.__size

    def __contains__(self, data):
        return self.search(data) != -1

    def __str__(self):
        """Display as: 10 → 20 → 30 → None"""
        parts = []
        current = self.__head
        while current is not None:
            parts.append(str(current.data))
            current = current.next
        return " → ".join(parts) + " → None"

    def __repr__(self):
        return f"LinkedList({self.to_list()})"

    def __iter__(self):
        current = self.__head
        while current is not None:
            yield current.data
            current = current.next


# ── Usage ────────────────────────────────────────────────────────────────
ll = LinkedList()
ll.append(10)
ll.append(20)
ll.append(30)
ll.append(40)

print(ll)           # 10 → 20 → 30 → 40 → None
print(len(ll))      # 4
print(20 in ll)     # True
print(50 in ll)     # False

ll.prepend(5)
print(ll)           # 5 → 10 → 20 → 30 → 40 → None

ll.insert_at(3, 25)
print(ll)           # 5 → 10 → 20 → 25 → 30 → 40 → None

ll.delete(25)
print(ll)           # 5 → 10 → 20 → 30 → 40 → None

ll.reverse()
print(ll)           # 40 → 30 → 20 → 10 → 5 → None

for item in ll:     # uses __iter__
    print(item, end=" ")
print()             # 40 30 20 10 5
```

## 3. Stack (LIFO)

### What is a Stack?
A **stack** is a **Last-In, First-Out (LIFO)** data structure. Think of a pile of plates: you add to the top and take from the top.

```
Operations:
  push(item)  — add to the top
  pop()       — remove and return the top item
  peek()      — look at the top item without removing it
  is_empty()  — check if the stack has no items

Visual:
  push(1), push(2), push(3)

     ┌───┐
  3  │ 3 │  ← top (most recently pushed)
     ├───┤
  2  │ 2 │
     ├───┤
  1  │ 1 │  ← bottom (first pushed)
     └───┘
  pop() returns 3, then stack is [2, 1]
```

```python
class Stack:
    """LIFO stack implemented using a Python list internally."""

    def __init__(self):
        self.__items = []   # private: index -1 is the "top"

    def push(self, item):
        """Add an item to the top of the stack — O(1)."""
        self.__items.append(item)

    def pop(self):
        """Remove and return the top item — O(1)."""
        if self.is_empty():
            raise IndexError("pop from an empty stack.")
        return self.__items.pop()

    def peek(self):
        """Return the top item WITHOUT removing it — O(1)."""
        if self.is_empty():
            raise IndexError("peek at an empty stack.")
        return self.__items[-1]

    def is_empty(self):
        return len(self.__items) == 0

    def size(self):
        return len(self.__items)

    def __len__(self):
        return len(self.__items)

    def __contains__(self, item):
        return item in self.__items

    def __str__(self):
        if self.is_empty():
            return "Stack: (empty)"
        top_indicator = " ← top"
        lines = []
        for item in reversed(self.__items):
            line = f"  | {item}"
            if item == self.__items[-1]:
                line += top_indicator
            lines.append(line)
        return "Stack:\n" + "\n".join(lines)

    def __repr__(self):
        return f"Stack(items={self.__items})"


# ── Practical example: balanced brackets checker ─────────────────────────
def is_balanced(expression):
    """Use a stack to verify that brackets are correctly balanced."""
    stack = Stack()
    pairs = {")": "(", "]": "[", "}": "{"}
    opening = set("([{")

    for char in expression:
        if char in opening:
            stack.push(char)
        elif char in pairs:
            if stack.is_empty() or stack.pop() != pairs[char]:
                return False

    return stack.is_empty()


tests = [
    ("(a + b) * (c - d)", True),
    ("{[()]}", True),
    ("(()", False),
    ("([)]", False),
    ("", True),
]

for expr, expected in tests:
    result = is_balanced(expr)
    status = "✓" if result == expected else "✗"
    print(f"{status}  is_balanced({expr!r}) = {result}")
```

## 4. Queue (FIFO)

### What is a Queue?
A **queue** is a **First-In, First-Out (FIFO)** data structure. Think of a queue at a shop: the first person in line is the first to be served.

```
Operations:
  enqueue(item) — add to the back
  dequeue()     — remove and return the front item
  peek()        — look at the front item without removing it
  is_empty()    — check if empty

Visual:
  enqueue(1), enqueue(2), enqueue(3)

  FRONT                        BACK
   ┌───┬───┬───┐
   │ 1 │ 2 │ 3 │
   └───┴───┴───┘
  dequeue() returns 1, then queue is [2, 3]
```

```python
class Queue:
    """FIFO queue — enqueue at the back, dequeue from the front."""

    def __init__(self):
        self.__items = []

    def enqueue(self, item):
        """Add an item to the back of the queue — O(1)."""
        self.__items.append(item)

    def dequeue(self):
        """Remove and return the front item — O(n) with a list."""
        if self.is_empty():
            raise IndexError("dequeue from an empty queue.")
        return self.__items.pop(0)

    def peek(self):
        """Return the front item without removing it — O(1)."""
        if self.is_empty():
            raise IndexError("peek at an empty queue.")
        return self.__items[0]

    def is_empty(self):
        return len(self.__items) == 0

    def size(self):
        return len(self.__items)

    def __len__(self):
        return len(self.__items)

    def __contains__(self, item):
        return item in self.__items

    def __str__(self):
        if self.is_empty():
            return "Queue: (empty)"
        items_str = " | ".join(str(i) for i in self.__items)
        return f"Queue: FRONT [ {items_str} ] BACK"

    def __repr__(self):
        return f"Queue(items={self.__items})"


# ── Practical example: print queue simulation ────────────────────────────
class PrintJob:
    """Represents a document in the print queue."""
    def __init__(self, document, pages, owner):
        self.document = document
        self.pages = pages
        self.owner = owner

    def __str__(self):
        return f'"{self.document}" ({self.pages}pp) for {self.owner}'


class PrintQueue:
    """Models a printer's document queue using a Queue."""

    def __init__(self, printer_name):
        self.printer_name = printer_name
        self.__queue = Queue()

    def submit(self, document, pages, owner):
        job = PrintJob(document, pages, owner)
        self.__queue.enqueue(job)
        print(f"[{self.printer_name}] Queued: {job}")

    def print_next(self):
        if self.__queue.is_empty():
            print(f"[{self.printer_name}] No jobs in queue.")
            return
        job = self.__queue.dequeue()
        print(f"[{self.printer_name}] Printing: {job}")

    def pending_count(self):
        return len(self.__queue)

    def __str__(self):
        return f"{self.printer_name}: {self.pending_count()} job(s) pending"


printer = PrintQueue("Office Printer")
printer.submit("Report.pdf", 12, "Alice")
printer.submit("Letter.docx", 2, "Bob")
printer.submit("Slides.pptx", 35, "Carol")

print(printer)        # Office Printer: 3 job(s) pending
printer.print_next()  # Printing: "Report.pdf" (12pp) for Alice
printer.print_next()  # Printing: "Letter.docx" (2pp) for Bob
print(printer)        # Office Printer: 1 job(s) pending
```

## 5. Stack Implemented Using a Linked List

### Avoiding the List Internally
```python
class StackNode:
    def __init__(self, data):
        self.data = data
        self.next = None


class LinkedStack:
    """Stack backed by a linked list — all operations are O(1)."""

    def __init__(self):
        self.__top = None   # points to the top StackNode
        self.__size = 0

    def push(self, item):
        new_node = StackNode(item)
        new_node.next = self.__top   # new node points to current top
        self.__top = new_node        # new node becomes the top
        self.__size += 1

    def pop(self):
        if self.is_empty():
            raise IndexError("pop from an empty stack.")
        data = self.__top.data
        self.__top = self.__top.next  # top moves down one
        self.__size -= 1
        return data

    def peek(self):
        if self.is_empty():
            raise IndexError("peek at an empty stack.")
        return self.__top.data

    def is_empty(self):
        return self.__top is None

    def __len__(self):
        return self.__size

    def __str__(self):
        items = []
        current = self.__top
        while current is not None:
            items.append(str(current.data))
            current = current.next
        return "LinkedStack (top→bottom): " + " → ".join(items) if items else "LinkedStack: (empty)"


ls = LinkedStack()
ls.push(1)
ls.push(2)
ls.push(3)
print(ls)         # LinkedStack (top→bottom): 3 → 2 → 1
print(ls.pop())   # 3
print(ls.peek())  # 2
print(len(ls))    # 2
```

## Practice Exercises

### Exercise 1: Extend LinkedList with `insert_at()`
The `LinkedList` class above already has `insert_at()`. Extend it further with:
- `delete_at(index)` — remove the node at a given index
- `find_middle()` — return the middle element (use the two-pointer technique: one pointer moves one step at a time, one moves two)
- `has_cycle()` — detect if the list contains a circular reference (Floyd's algorithm)
- `count(data)` — count how many times `data` appears

```python
# Your code here
class Node:
    pass

class LinkedList:
    pass
```

### Exercise 2: Stack to Check Balanced Brackets
Using the `Stack` class above, extend `is_balanced()` to also:
- Handle multi-line input (a string with newlines)
- Report the *position* (line and column) of the first unmatched bracket
- Handle string literals (brackets inside `"..."` or `'...'` should be ignored)

```python
# Your code here
class Stack:
    pass

def is_balanced_detailed(code):
    """Returns (True, None) or (False, (line, col, char))."""
    pass
```

### Exercise 3: Queue to Simulate a Print Queue
Build a more detailed `PrintQueue` simulation:
- Each `PrintJob` has a `priority` (1=low, 2=normal, 3=high)
- The queue serves jobs in priority order (highest first); equal priority is FIFO
- `PrintQueue.status()` — shows all pending jobs with their priorities
- Add `cancel(document_name)` — remove a job from the queue

```python
# Your code here
class PrintJob:
    pass

class PriorityPrintQueue:
    pass
```

### Exercise 4: Stack Using a Linked List
Implement a `LinkedStack` (as shown in Section 5) with these additional methods:
- `peek_bottom()` — returns the item at the bottom of the stack
- `to_list()` — returns all items from top to bottom as a list
- `reverse()` — reverses the stack in-place using no other data structure
- `__iter__` — iterates from top to bottom

```python
# Your code here
class StackNode:
    pass

class LinkedStack:
    pass
```

### Exercise 5: Circular Queue
Implement a `CircularQueue` using a fixed-size array (list of `None` values). A circular queue uses `front` and `rear` pointers that wrap around when they reach the end of the array:
- `enqueue(item)` — add to rear; raise error if full
- `dequeue()` — remove from front; raise error if empty
- `is_full()`, `is_empty()`, `size()`
- `__str__` — shows the queue contents in order

```python
# Your code here
class CircularQueue:
    def __init__(self, capacity):
        self.__capacity = capacity
        self.__data = [None] * capacity
        self.__front = 0
        self.__rear = 0
        self.__size = 0

    # Your methods here
    pass
```

## Key Concepts to Remember
- **Node**: a fundamental unit holding data and a pointer (`next`) to the next node
- **Linked list**: a sequence of nodes; each points to the next; no random access — must traverse from head
- **Stack (LIFO)**: Last-In, First-Out; push adds to top, pop removes from top; used for undo, recursion, bracket matching
- **Queue (FIFO)**: First-In, First-Out; enqueue adds to back, dequeue removes from front; used for scheduling, print queues, BFS
- **`peek()`**: inspect the top/front item without removing it — present in both Stack and Queue
- **`is_empty()`**: always check before pop/dequeue to avoid errors
- **Linked implementation vs list implementation**: a linked list gives O(1) push/pop for a stack; Python's built-in list gives O(1) append/pop-from-end (suitable for a stack) but O(n) pop-from-front (use `collections.deque` for O(1) queue operations in production)
- **Encapsulation in data structures**: `__head`, `__items`, `__top` etc. are private; users interact only via the defined methods

## Common Mistakes to Avoid
1. **Not handling the empty case** — always check `is_empty()` before calling `pop()`, `dequeue()`, or `peek()`; accessing an empty structure should raise a clear `IndexError` or custom exception.
2. **Losing the chain in linked list deletion** — to delete a node, update `current.next = current.next.next` (skip over the target); not doing this orphans the rest of the list.
3. **Confusing stack and queue operations** — LIFO (stack): both insert and remove happen at the *same* end (top). FIFO (queue): insert at the back, remove from the front.
4. **Forgetting to update `__size`** — if you maintain a size counter, every insert and delete must update it; an incorrect size breaks `__len__` and any range checks.
5. **Off-by-one errors in `insert_at()`** — the valid index range for insertion is `0` to `size` (inclusive); for deletion it is `0` to `size - 1`.

## Extension Challenge
Implement a **double-ended queue (deque)** class — a data structure where items can be added or removed from *either* end:
- `add_front(item)` and `add_rear(item)`
- `remove_front()` and `remove_rear()`
- `peek_front()` and `peek_rear()`
- `is_empty()`, `size()`, `__len__`, `__contains__`, `__str__`

Implement it using a **doubly linked list** (each node has `next` and `prev` pointers). Keep track of both `head` and `tail` pointers for O(1) operations at both ends.

```python
# Your code here
class DoublyNode:
    def __init__(self, data):
        self.data = data
        self.next = None
        self.prev = None


class Deque:
    def __init__(self):
        self.__head = None
        self.__tail = None
        self.__size = 0

    # Your methods here
    pass

# Test:
# d = Deque()
# d.add_rear(1)
# d.add_rear(2)
# d.add_front(0)
# print(d)              # 0 ↔ 1 ↔ 2
# print(d.remove_rear()) # 2
# print(d.remove_front()) # 0
# print(d)              # 1
```
