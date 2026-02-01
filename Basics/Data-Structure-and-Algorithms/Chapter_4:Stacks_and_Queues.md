# Chapter 4: Stacks & Queues

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [Stacks — Theory & Implementation](#stacks--theory--implementation)
3. [Queues — Theory & Implementation](#queues--theory--implementation)
4. [Deque (Double-Ended Queue)](#deque-double-ended-queue)
5. [Priority Queue & Heaps](#priority-queue--heaps)
6. [Monotonic Stack](#monotonic-stack)
7. [Practice Problems](#practice-problems)
8. [Mini-Project: Expression Evaluator](#mini-project-expression-evaluator)
9. [ML Relevance & Summary](#ml-relevance--summary)

---

## Introduction

### What Are Stacks and Queues?

Both are **restricted-access** sequences — you can only add or remove elements at specific ends, not at arbitrary indices. That restriction is exactly what makes them powerful: it forces a particular *order of processing* that solves entire families of problems.

| Structure | Access rule | Real-world analogy |
|---|---|---|
| **Stack** | Last In, First Out (LIFO) | A pile of plates — you always take the top one |
| **Queue** | First In, First Out (FIFO) | People waiting in line — first to arrive is first served |

### Why Learn These?

- **Stacks** appear everywhere: expression evaluation, undo systems, browser back-buttons, recursive call simulation, syntax validation, and the monotonic-stack pattern that solves "next greater element" problems in O(n).
- **Queues** are the backbone of BFS (breadth-first search), task schedulers, and streaming pipelines.
- **Priority Queues** (heaps) power Dijkstra's shortest-path, top-k queries, and merge-k-sorted-lists — all common in ML pipelines.

---

## Stacks — Theory & Implementation

### Mental Model

```
Push 1        Push 2        Push 3        Pop → 3       Pop → 2
                                          
  ┌───┐       ┌───┐         ┌───┐        ┌───┐         ┌───┐
  │ 1 │       │ 2 │  ← top  │ 3 │ ← top  │ 2 │ ← top   │ 1 │ ← top
  └───┘       │ 1 │         │ 2 │        │ 1 │         └───┘
              └───┘         │ 1 │        └───┘
                            └───┘
```

### Complexity

| Operation | Time | Notes |
|---|---|---|
| `push(x)` | O(1) | append to end of list |
| `pop()` | O(1) | remove from end of list |
| `peek()` | O(1) | read last element |
| `is_empty()` | O(1) | check length |

### Implementation in Python

Python lists already behave as a stack via `append` / `pop()`. For clarity we can also wrap them in a class:

```python
class Stack:
    """
    LIFO structure built on a Python list.
    All operations are O(1).
    """
    def __init__(self):
        self._data = []

    def push(self, val):
        self._data.append(val)

    def pop(self):
        if self.is_empty():
            raise IndexError("pop from empty stack")
        return self._data.pop()        # removes & returns last element

    def peek(self):
        if self.is_empty():
            raise IndexError("peek on empty stack")
        return self._data[-1]          # read without removing

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

    def __repr__(self):
        return f"Stack({self._data})  ← top is rightmost"


# ── Demo ──
s = Stack()
s.push(10)
s.push(20)
s.push(30)
print(s)          # Stack([10, 20, 30])  ← top is rightmost
print(s.peek())   # 30
print(s.pop())    # 30
print(s.pop())    # 20
print(s)          # Stack([10])
```

### When to Use a Stack
- You need to process things in **reverse** order.
- You need to **match pairs** (open/close brackets, tags).
- You need to simulate **recursion** iteratively.
- You need the **nearest greater/smaller** element (monotonic stack).

---

## Queues — Theory & Implementation

### Mental Model

```
Enqueue A     Enqueue B     Enqueue C     Dequeue → A    Dequeue → B

┌───┐         ┌───┬───┐     ┌───┬───┬───┐  ┌───┬───┐     ┌───┐
│ A │ front   │ A │ B │     │ A │ B │ C │  │ B │ C │     │ C │
└───┘         └───┴───┘     └───┴───┴───┘  └───┴───┘     └───┘
              front  back   front   back   front  back
```

### Complexity

| Operation | Time (list) | Time (deque) |
|---|---|---|
| `enqueue(x)` | O(1) append | O(1) |
| `dequeue()` | **O(n)** pop(0) | **O(1)** popleft |
| `peek()` | O(1) | O(1) |

> **Critical point:** `list.pop(0)` is O(n) because it shifts every remaining element left. For a real queue always use `collections.deque`.

### Implementation

```python
from collections import deque

class Queue:
    """
    FIFO structure.  Uses deque internally so enqueue AND dequeue are O(1).
    """
    def __init__(self):
        self._data = deque()

    def enqueue(self, val):
        self._data.append(val)         # add to back

    def dequeue(self):
        if self.is_empty():
            raise IndexError("dequeue from empty queue")
        return self._data.popleft()    # remove from front — O(1)!

    def peek(self):
        if self.is_empty():
            raise IndexError("peek on empty queue")
        return self._data[0]

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

    def __repr__(self):
        return f"Queue({list(self._data)})  front→back"


# ── Demo ──
q = Queue()
q.enqueue('A')
q.enqueue('B')
q.enqueue('C')
print(q)           # Queue(['A', 'B', 'C'])  front→back
print(q.dequeue()) # A
print(q.dequeue()) # B
print(q)           # Queue(['C'])
```

### When to Use a Queue
- **BFS** (breadth-first search) — process nodes level by level.
- **Task scheduling** — first submitted, first executed.
- **Streaming / buffering** — process data in arrival order.

---

## Deque (Double-Ended Queue)

A deque lets you push/pop from **both** ends in O(1). Python's `collections.deque` is implemented as a doubly-linked list of fixed-size blocks.

```
                ┌───┬───┬───┬───┐
  appendleft →  │ A │ B │ C │ D │  ← append
                └───┴───┴───┴───┘
  popleft   →                      ← pop
```

### Full API

```python
from collections import deque

d = deque([1, 2, 3])

d.append(4)        # add right   → [1, 2, 3, 4]
d.appendleft(0)    # add left    → [0, 1, 2, 3, 4]
d.pop()            # remove right → 4,  d = [0, 1, 2, 3]
d.popleft()        # remove left  → 0,  d = [1, 2, 3]

d.rotate(1)        # rotate right by 1 → [3, 1, 2]
d.rotate(-1)       # rotate left  by 1 → [1, 2, 3]

# All operations above are O(1)
# Indexed access d[i] is O(n) — deque is NOT an array!
```

### Bounded Deque (Max Size)

```python
# maxlen deque automatically discards from the opposite end when full
recent = deque(maxlen=3)
recent.append('a')   # deque(['a'])
recent.append('b')   # deque(['a', 'b'])
recent.append('c')   # deque(['a', 'b', 'c'])
recent.append('d')   # deque(['b', 'c', 'd'])  ← 'a' auto-dropped!

# Perfect for "keep last k items" — used in ML for sliding-window feature buffers.
```

---

## Priority Queue & Heaps

### What Is a Priority Queue?

Like a normal queue, but **dequeue always returns the highest-priority element**, regardless of insertion order.

### How It Works — The Binary Heap

A **min-heap** is a complete binary tree where every parent ≤ its children. The minimum element is always at the root (index 0). Stored as a flat array:

```
        1                  arr = [1, 3, 2, 7, 4, 5, 6]
       / \
      3    2               Parent of i  →  (i−1) // 2
     / \  / \              Left child   →  2i + 1
    7  4 5   6             Right child  →  2i + 2

Index:  0  1  2  3  4  5  6
```

### Complexity

| Operation | Time |
|---|---|
| `heappush` | O(log n) — bubble up |
| `heappop` | O(log n) — bubble down |
| `peek` (min) | O(1) — just `heap[0]` |
| Build heap from list | O(n) — `heapify` |

### Python's `heapq` Module

Python only provides a **min-heap**. For a max-heap, negate the values.

```python
import heapq

# ── Min-Heap ──
h = []
heapq.heappush(h, 5)
heapq.heappush(h, 2)
heapq.heappush(h, 8)
heapq.heappush(h, 1)
print(h)                  # [1, 2, 8, 5]  — heap property, NOT sorted
print(heapq.heappop(h))   # 1  (smallest)
print(heapq.heappop(h))   # 2
print(h)                  # [5, 8]

# ── Max-Heap (negate values) ──
max_h = []
for x in [5, 2, 8, 1]:
    heapq.heappush(max_h, -x)
print(-heapq.heappop(max_h))   # 8  (largest)
print(-heapq.heappop(max_h))   # 5

# ── Build heap from existing list ──
data = [9, 4, 7, 1, 3]
heapq.heapify(data)            # O(n) — rearranges in place
print(data)                    # [1, 3, 7, 9, 4]
```

### Top-K Elements (Common Interview Pattern)

```python
import heapq

def top_k_frequent(nums, k):
    """
    Find the k most frequent elements.

    Strategy: count frequencies, then use a min-heap of size k.
    Keep only the k largest frequencies — when heap exceeds k, pop the smallest.

    nums = [1,1,1,2,2,3], k = 2
    freqs: {1:3, 2:2, 3:1}
    heap after processing: [(2,'2'), (3,'1')]   ← the 2 largest freqs
    Answer: [1, 2]

    Time O(n log k), Space O(n)
    """
    from collections import Counter
    freq = Counter(nums)

    # Min-heap of size k — stores (frequency, element)
    heap = []
    for elem, count in freq.items():
        heapq.heappush(heap, (count, elem))
        if len(heap) > k:
            heapq.heappop(heap)     # drop the smallest frequency

    return [elem for _, elem in heap]

print(top_k_frequent([1,1,1,2,2,3], 2))  # [1, 2]
```

---

## Monotonic Stack

### What Is It?

A stack that is kept **sorted** at all times — either always increasing from bottom to top, or always decreasing. When a new element would violate the order, you pop elements until the order is restored.

### Why It Exists

The "next greater element" problem asks: *for each element, what is the first element to its right that is larger?* Brute force is O(n²). A monotonic stack solves it in **O(n)**.

### Pattern — Next Greater Element

```
nums = [4, 5, 2, 10, 8]

We want:  for each index, the first element to its RIGHT that is greater.
Answer:   [5, 10, 10, -1, -1]

Walk left to right.  Maintain a stack of indices whose "next greater"
we haven't found yet.  When we see a new element, it is the answer for
every stack element it is greater than.

─────────────────────────────────────────────────────
i=0  val=4   stack=[]    nothing to pop.  push 0.         stack=[0]
i=1  val=5   stack=[0]   5 > nums[0]=4  → ans[0]=5  pop.  push 1.  stack=[1]
i=2  val=2   stack=[1]   2 < nums[1]=5  → can't pop.     push 2.  stack=[1,2]
i=3  val=10  stack=[1,2] 10>nums[2]=2  → ans[2]=10 pop.
                          10>nums[1]=5  → ans[1]=10 pop.  push 3.  stack=[3]
i=4  val=8   stack=[3]   8 < nums[3]=10 → can't pop.     push 4.  stack=[3,4]
End: stack=[3,4]  → ans[3]=-1, ans[4]=-1  (nothing greater to the right)

Result  [5, 10, 10, -1, -1]  ✓
─────────────────────────────────────────────────────
```

```python
def next_greater_element(nums):
    """
    For each element return the first element to its right that is greater.
    Return -1 if none exists.

    Time O(n) — each index is pushed once and popped at most once.
    Space O(n)
    """
    n   = len(nums)
    ans = [-1] * n
    stack = []             # stores indices, monotonically decreasing values

    for i in range(n):
        # Pop everything the current element is greater than
        while stack and nums[i] > nums[stack[-1]]:
            j = stack.pop()
            ans[j] = nums[i]
        stack.append(i)

    return ans

print(next_greater_element([4, 5, 2, 10, 8]))  # [5, 10, 10, -1, -1]
```

### Variant — Next Greater Element in Circular Array

```python
def next_greater_circular(nums):
    """
    Same problem but the array wraps around.
    Trick: iterate through the array TWICE (indices 0 … 2n−1),
    using modulo to wrap.

    nums = [1, 2, 1]
    Answer: [2, -1, 2]
      1's next greater going right (wrapping) → 2
      2 has nothing greater in the whole array → -1
      last 1 wraps around to find 2            → 2

    Time O(n), Space O(n)
    """
    n   = len(nums)
    ans = [-1] * n
    stack = []

    for i in range(2 * n):          # two passes
        idx = i % n
        while stack and nums[idx] > nums[stack[-1]]:
            ans[stack.pop()] = nums[idx]
        if i < n:                   # only push during first pass
            stack.append(idx)

    return ans

print(next_greater_circular([1, 2, 1]))  # [2, -1, 2]
```

---

## Practice Problems

### Problem 1 — Valid Parentheses (Easy)

**Statement:** Given a string of `()`, `[]`, `{}`, determine if every open bracket has a matching close bracket in correct order.

```
"({[]})"  → True
"({[}])"  → False   ([ closed before })
"((("     → False   (unclosed)
```

```python
def is_valid(s):
    """
    Push every opening bracket.
    On a closing bracket, check that the top of the stack matches.
    At the end the stack must be empty.

    Walkthrough  s = "({[]})"
    ────────────────────────────
    '('  open   → push    stack = ['(']
    '{'  open   → push    stack = ['(', '{']
    '['  open   → push    stack = ['(', '{', '[']
    ']'  close  → top='[' matches ']'  pop   stack = ['(', '{']
    '}'  close  → top='{' matches '}'  pop   stack = ['(']
    ')'  close  → top='(' matches ')'  pop   stack = []
    End  stack empty → True  ✓
    ────────────────────────────

    Time O(n), Space O(n)
    """
    matching = {')': '(', ']': '[', '}': '{'}
    stack = []

    for ch in s:
        if ch in '([{':
            stack.append(ch)
        else:                                  # closing bracket
            if not stack or stack[-1] != matching[ch]:
                return False
            stack.pop()

    return len(stack) == 0

print(is_valid("({[]})"))   # True
print(is_valid("({[}])"))   # False
print(is_valid("((("))      # False
print(is_valid(""))         # True
```

### Problem 2 — Evaluate Reverse Polish Notation (Medium)

**Statement:** Evaluate a math expression given in postfix (RPN) notation.

```
["2", "1", "+", "3", "*"]  →  (2 + 1) * 3 = 9

Walkthrough:
  "2"  → push 2       stack = [2]
  "1"  → push 1       stack = [2, 1]
  "+"  → pop 1, pop 2 → 2+1=3  push 3    stack = [3]
  "3"  → push 3       stack = [3, 3]
  "*"  → pop 3, pop 3 → 3*3=9  push 9    stack = [9]
  Answer = 9
```

```python
def eval_rpn(tokens):
    """
    Time O(n), Space O(n)
    """
    stack = []
    ops   = {'+', '-', '*', '/'}

    for t in tokens:
        if t in ops:
            b = stack.pop()          # second operand (pushed later)
            a = stack.pop()          # first operand
            if   t == '+': stack.append(a + b)
            elif t == '-': stack.append(a - b)
            elif t == '*': stack.append(a * b)
            else:          stack.append(int(a / b))  # truncate toward zero
        else:
            stack.append(int(t))

    return stack[0]

print(eval_rpn(["2","1","+","3","*"]))           # 9
print(eval_rpn(["4","13","5","/","+"]))          # 6   (4 + 13/5 = 4+2)
print(eval_rpn(["10","6","9","3","+","-11","*","/","*","17","+","5","+"]))  # 22
```

### Problem 3 — Implement Queue Using Two Stacks (Medium)

**Statement:** Build a FIFO queue using only two LIFO stacks.

```
Idea: one stack for enqueue ("input"), one for dequeue ("output").
When output is empty and we need to dequeue, dump all of input
into output — this reverses the order, turning LIFO into FIFO.

Enqueue 1  →  input=[1]  output=[]
Enqueue 2  →  input=[1,2] output=[]
Dequeue    →  output empty → flip input into output
              input=[]  output=[2,1]   pop output → 1  ✓  (FIFO!)
Dequeue    →  output=[2]  pop → 2  ✓
Enqueue 3  →  input=[3]  output=[]
Dequeue    →  output empty → flip again
              input=[]  output=[3]  pop → 3  ✓

Amortised O(1) per operation — each element is moved exactly twice
(once into input, once into output).
```

```python
class MyQueue:
    def __init__(self):
        self.input  = []   # stack for enqueue
        self.output = []   # stack for dequeue

    def enqueue(self, x):
        self.input.append(x)

    def _transfer(self):
        """Move all elements from input → output (reverses order)."""
        if not self.output:
            while self.input:
                self.output.append(self.input.pop())

    def dequeue(self):
        self._transfer()
        return self.output.pop()

    def peek(self):
        self._transfer()
        return self.output[-1]

    def is_empty(self):
        return not self.input and not self.output


# ── Demo ──
q = MyQueue()
q.enqueue(1)
q.enqueue(2)
print(q.dequeue())   # 1
print(q.dequeue())   # 2
q.enqueue(3)
print(q.dequeue())   # 3
```

### Problem 4 — Implement Stack Using Two Queues (Medium)

```python
class MyStack:
    """
    Use a single queue.  After each enqueue, rotate the queue so the
    newest element ends up at the front.

    Enqueue 1  →  q=[1]
    Enqueue 2  →  q=[1,2]  rotate 1 time  →  q=[2,1]
    Enqueue 3  →  q=[2,1,3] rotate 2 times →  q=[3,2,1]
    Pop        →  dequeue front → 3  ✓  (LIFO!)

    Each push is O(n), pop is O(1).  Overall amortised the same total
    work as the two-stack queue above.
    """
    def __init__(self):
        from collections import deque
        self.q = deque()

    def push(self, x):
        self.q.append(x)
        # Rotate so x is at the front
        for _ in range(len(self.q) - 1):
            self.q.append(self.q.popleft())

    def pop(self):
        return self.q.popleft()

    def top(self):
        return self.q[0]

    def is_empty(self):
        return len(self.q) == 0


# ── Demo ──
s = MyStack()
s.push(1)
s.push(2)
s.push(3)
print(s.pop())   # 3
print(s.pop())   # 2
print(s.top())   # 1
```

### Problem 5 — Sliding Window Maximum (Hard)

**Statement:** Given array and window size k, return the maximum in each window.

```
nums = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3

Windows and their max:
[1, 3,-1]  → 3
   [3,-1,-3] → 3
      [-1,-3, 5] → 5
         [-3, 5, 3] → 5
            [5, 3, 6] → 6
               [3, 6, 7] → 7

Answer: [3, 3, 5, 5, 6, 7]
```

```python
from collections import deque

def max_sliding_window(nums, k):
    """
    Monotonic decreasing deque — front always holds the index of the
    current window's maximum.

    Rules:
      1. Remove indices that have fallen outside the window (front).
      2. Remove indices from the back whose VALUES are ≤ current value
         (they can never be the max while current is in the window).
      3. Append current index.
      4. Once i ≥ k−1 the window is full — record front as the max.

    Walkthrough  k=3
    ───────────────────────────────────────────────────
    i=0 val=1   deque=[]       push 0     deque=[0]          (window not full)
    i=1 val=3   back nums[0]=1 ≤ 3  pop.  push 1  deque=[1]  (window not full)
    i=2 val=-1  back nums[1]=3 > -1 stop. push 2  deque=[1,2] → max=nums[1]=3
    i=3 val=-3  front=1  1 < 3−3+1=1? no.
                back nums[2]=-1 > -3 stop. push 3  deque=[1,2,3] → max=nums[1]=3
    i=4 val=5   front=1  1 < 4−3+1=2? yes → pop front.  deque=[2,3]
                back nums[3]=-3 ≤ 5 pop.  back nums[2]=-1 ≤ 5 pop.
                push 4  deque=[4]         → max=nums[4]=5
    i=5 val=3   front=4  4 < 5−3+1=3? no.
                back nums[4]=5 > 3 stop.  push 5  deque=[4,5] → max=nums[4]=5
    i=6 val=6   front=4  4 < 6−3+1=4? no.
                back nums[5]=3 ≤ 6 pop.  back nums[4]=5 ≤ 6 pop.
                push 6  deque=[6]         → max=nums[6]=6
    i=7 val=7   front=6  6 < 7−3+1=5? no.
                back nums[6]=6 ≤ 7 pop.  push 7  deque=[7] → max=nums[7]=7
    ───────────────────────────────────────────────────
    Result [3, 3, 5, 5, 6, 7]  ✓

    Time O(n) — each index pushed and popped at most once.
    Space O(k)
    """
    dq   = deque()     # stores indices
    out  = []

    for i in range(len(nums)):
        # 1. Remove out-of-window indices from front
        while dq and dq[0] < i - k + 1:
            dq.popleft()

        # 2. Remove smaller elements from back
        while dq and nums[dq[-1]] <= nums[i]:
            dq.pop()

        # 3. Add current index
        dq.append(i)

        # 4. Record max once window is full
        if i >= k - 1:
            out.append(nums[dq[0]])

    return out

print(max_sliding_window([1,3,-1,-3,5,3,6,7], 3))  # [3,3,5,5,6,7]
```

---

## Mini-Project: Expression Evaluator

### Project Description

Build a calculator that parses and evaluates **infix** expressions (the kind humans write: `3 + 5 * 2`) respecting operator precedence and parentheses, using stacks.

### How It Works — Two-Stack Shunting-Yard Algorithm

```
Convert infix → postfix (RPN) using a stack, then evaluate.

Precedence: * and /  >  + and -
Parentheses override precedence.

Input: "3 + 5 * 2 - 8 / 4"

Shunting-yard (operator stack + output queue):
  '3'  → output=[3]
  '+'  → op_stack=[+]
  '5'  → output=[3,5]
  '*'  → * has higher prec than +  → just push.  op_stack=[+,*]
  '2'  → output=[3,5,2]
  '-'  → - has same prec as +.  Pop * (higher) → output=[3,5,2,*]
         Pop + (same prec, left-assoc) → output=[3,5,2,*,+]
         Push -.  op_stack=[-]
  '8'  → output=[3,5,2,*,+,8]
  '/'  → / higher than -  → push.  op_stack=[-,/]
  '4'  → output=[3,5,2,*,+,8,4]
  End  → pop remaining: output=[3,5,2,*,+,8,4,/,-]

RPN: 3 5 2 * + 8 4 / -

Evaluate RPN:
  3 → [3]
  5 → [3,5]
  2 → [3,5,2]
  * → 5*2=10  [3,10]
  + → 3+10=13 [13]
  8 → [13,8]
  4 → [13,8,4]
  / → 8/4=2   [13,2]
  - → 13-2=11 [11]

Answer: 11  ✓   (3 + 10 - 2 = 11)
```

### Implementation

```python
class ExpressionEvaluator:
    """
    Evaluates infix math expressions with +, -, *, /,
    parentheses, and integer/float operands.
    """

    PRECEDENCE = {'+': 1, '-': 1, '*': 2, '/': 2}

    def __init__(self):
        pass

    # ── Tokeniser ─────────────────────────────────────
    def tokenize(self, expr):
        """
        Split expression string into tokens.
        Handles multi-digit numbers and negative literals.

        "3 + 52 * (2 - 8)" → ['3','+','52','*','(','2','-','8',')']
        """
        tokens = []
        i = 0
        while i < len(expr):
            if expr[i].isspace():
                i += 1
                continue
            if expr[i].isdigit() or expr[i] == '.':
                j = i
                while j < len(expr) and (expr[j].isdigit() or expr[j] == '.'):
                    j += 1
                tokens.append(expr[i:j])
                i = j
            else:
                tokens.append(expr[i])
                i += 1
        return tokens

    # ── Infix → Postfix (Shunting-Yard) ──────────────
    def to_postfix(self, tokens):
        """
        Convert infix token list to postfix (RPN) using operator stack.

        Rules:
          • Number → straight to output.
          • '('    → push onto op_stack.
          • ')'    → pop ops to output until '(' is found, then discard '('.
          • Operator → pop and output any ops on stack with ≥ precedence,
                       then push current op.
        """
        output   = []
        op_stack = []

        for tok in tokens:
            if tok == '(':
                op_stack.append(tok)
            elif tok == ')':
                while op_stack and op_stack[-1] != '(':
                    output.append(op_stack.pop())
                op_stack.pop()                       # discard the '('
            elif tok in self.PRECEDENCE:             # operator
                while (op_stack
                       and op_stack[-1] != '('
                       and op_stack[-1] in self.PRECEDENCE
                       and self.PRECEDENCE[op_stack[-1]] >= self.PRECEDENCE[tok]):
                    output.append(op_stack.pop())
                op_stack.append(tok)
            else:                                    # number
                output.append(tok)

        while op_stack:
            output.append(op_stack.pop())

        return output

    # ── Evaluate Postfix ──────────────────────────────
    def eval_postfix(self, rpn):
        """Evaluate RPN token list using a value stack."""
        stack = []
        for tok in rpn:
            if tok in self.PRECEDENCE:
                b = stack.pop()
                a = stack.pop()
                if   tok == '+': stack.append(a + b)
                elif tok == '-': stack.append(a - b)
                elif tok == '*': stack.append(a * b)
                else:            stack.append(a / b)
            else:
                stack.append(float(tok))
        return stack[0]

    # ── Main Entry Point ──────────────────────────────
    def evaluate(self, expr):
        """
        Full pipeline: tokenise → infix-to-postfix → evaluate.
        Returns the numeric result.
        """
        tokens  = self.tokenize(expr)
        postfix = self.to_postfix(tokens)
        result  = self.eval_postfix(postfix)
        # Return int if whole number
        return int(result) if result == int(result) else result


# ─── Demo ─────────────────────────────────────────────
calc = ExpressionEvaluator()

tests = [
    ("3 + 5 * 2",            13),
    ("(3 + 5) * 2",          16),
    ("10 - 2 * 3 + 1",        5),
    ("(2 + 3) * (4 - 1)",    15),
    ("100 / (5 * 2)",        10),
    ("2 + 3 * 4 - 6 / 2",   11),
]

for expr, expected in tests:
    result = calc.evaluate(expr)
    status = "✓" if result == expected else "✗"
    print(f"  {status}  {expr:30s} = {result}   (expected {expected})")
```

### Extension Ideas
1. **Support unary minus** — e.g. `(-3 + 5)`.
2. **Add power operator** `^` with right-associativity.
3. **Support variables** — store a symbol table, evaluate `x + 2` after setting `x = 5`.
4. **REPL loop** — read expressions from stdin continuously.
5. **Error reporting** — detect and describe mismatched parentheses, unknown tokens.

---

## ML Relevance & Summary

### Where Stacks & Queues Appear in ML

| Concept | ML use-case |
|---|---|
| **Stack** | Simulating recursion iteratively in tree/graph traversal for model evaluation |
| **Queue (BFS)** | Level-order traversal of decision trees; BFS in knowledge graphs |
| **Priority Queue** | Dijkstra for shortest path in recommendation-system graphs; top-k prediction selection |
| **Monotonic Stack** | Sliding-window max/min for time-series feature engineering |
| **Expression Eval** | Parsing computational graphs in TensorFlow / PyTorch (`tf.function` traces) |
| **Bounded Deque** | Keeping the last-k frames in reinforcement learning replay buffers |

### Complexity Quick-Reference

| Structure | Push/Enqueue | Pop/Dequeue | Peek |
|---|---|---|---|
| Stack (list) | O(1) | O(1) | O(1) |
| Queue (deque) | O(1) | O(1) | O(1) |
| Priority Queue (heap) | O(log n) | O(log n) | O(1) |
| Monotonic Stack | O(1) amortised | O(1) amortised | O(1) |

### When to Reach for Each
| Trigger | Use |
|---|---|
| "Match pairs / validate nesting" | Stack |
| "Reverse processing order" | Stack |
| "Process in arrival order" | Queue |
| "Always need the smallest / largest" | Priority Queue |
| "Next greater / smaller element" | Monotonic Stack |
| "Max/min in every sliding window" | Monotonic Deque |

### Common Pitfalls
- Using `list.pop(0)` for a queue — it's O(n). Use `deque.popleft()`.
- Forgetting that Python's `heapq` is a **min**-heap; negate values for max.
- In monotonic stack problems, forgetting to handle the **remaining** stack elements after the loop.
- In the two-stack queue, forgetting that the transfer only happens when the output stack is **empty**.

---

**Next → Chapter 5: Linked Lists**