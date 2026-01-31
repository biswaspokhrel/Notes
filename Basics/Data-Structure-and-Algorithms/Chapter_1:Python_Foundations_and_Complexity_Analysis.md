# Chapter 1: Python Foundations & Complexity Analysis

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [Time Complexity](#time-complexity)
3. [Space Complexity](#space-complexity)
4. [Python-Specific Complexities](#python-specific-complexities)
5. [Practical Examples](#practical-examples)
6. [Practice Problems](#practice-problems)
7. [Mini-Project](#mini-project)

---

## Introduction

**Why Study Complexity Analysis?**
- Predict how algorithms scale with input size
- Compare different solutions objectively
- Make informed decisions in ML pipelines (e.g., choosing between algorithms for large datasets)
- Optimize bottlenecks in production systems

**Learning Objectives:**
- Understand Big O, Omega, and Theta notations
- Analyze time and space complexity of algorithms
- Recognize common complexity patterns
- Apply complexity analysis to Python code

---

## Time Complexity

### What is Time Complexity?

Time complexity measures how the runtime of an algorithm grows as the input size increases. We express this using **asymptotic notation**.

### Big O Notation (O)

**Definition:** Upper bound on the growth rate (worst-case scenario)

**Common Time Complexities (from best to worst):**

| Notation | Name | Example | Description |
|----------|------|---------|-------------|
| O(1) | Constant | Array access, hash table lookup | Time doesn't change with input size |
| O(log n) | Logarithmic | Binary search | Halves the problem each step |
| O(n) | Linear | Loop through array | Time grows proportionally with input |
| O(n log n) | Linearithmic | Merge sort, heap sort | Efficient sorting algorithms |
| O(n²) | Quadratic | Nested loops | Common in brute force solutions |
| O(n³) | Cubic | Triple nested loops | Matrix multiplication (naive) |
| O(2ⁿ) | Exponential | Recursive Fibonacci (naive) | Doubles with each additional input |
| O(n!) | Factorial | Generate all permutations | Explodes extremely quickly |

### Visual Comparison

```
Operations for different complexities (n = 100):

O(1):        1 operation
O(log n):    ~7 operations
O(n):        100 operations
O(n log n):  ~664 operations
O(n²):       10,000 operations
O(2ⁿ):       1.26 × 10³⁰ operations (basically impossible!)
```

### Examples with Code

#### O(1) - Constant Time
```python
def get_first_element(arr):
    """Access first element - always takes same time"""
    return arr[0]  # O(1)

def hash_lookup(dictionary, key):
    """Dictionary lookup is O(1) average case"""
    return dictionary.get(key)  # O(1)
```

**Explanation:** No matter if the array has 10 or 10 million elements, accessing `arr[0]` takes the same time.

#### O(log n) - Logarithmic Time
```python
def binary_search(arr, target):
    """Binary search on sorted array"""
    left, right = 0, len(arr) - 1
    
    while left <= right:  # Runs log₂(n) times
        mid = (left + right) // 2
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

# Example:
# n = 16: max 4 iterations (log₂(16) = 4)
# n = 1024: max 10 iterations (log₂(1024) = 10)
```

**Explanation:** Each iteration cuts the search space in half. If you have 1000 elements, you'll need at most ~10 comparisons.

#### O(n) - Linear Time
```python
def find_max(arr):
    """Find maximum element"""
    max_val = arr[0]
    for num in arr:  # Iterates n times
        if num > max_val:
            max_val = num
    return max_val

def sum_array(arr):
    """Sum all elements"""
    total = 0
    for num in arr:  # O(n)
        total += num
    return total
```

**Explanation:** Must check every element once. Double the input size, double the time.

#### O(n log n) - Linearithmic Time
```python
def merge_sort(arr):
    """Merge sort - efficient sorting"""
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])   # T(n/2)
    right = merge_sort(arr[mid:])  # T(n/2)
    
    return merge(left, right)       # O(n)

# Recurrence: T(n) = 2T(n/2) + O(n) = O(n log n)
```

**Explanation:** Divides the problem (log n levels) and merges (n operations per level).

#### O(n²) - Quadratic Time
```python
def bubble_sort(arr):
    """Bubble sort - nested loops"""
    n = len(arr)
    for i in range(n):          # Outer loop: n times
        for j in range(n - i - 1):  # Inner loop: n times
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr

def find_all_pairs(arr):
    """Print all pairs"""
    pairs = []
    for i in range(len(arr)):      # n times
        for j in range(len(arr)):  # n times
            pairs.append((arr[i], arr[j]))
    return pairs
# Total operations: n × n = n²
```

**Explanation:** For each element, we process all elements again. 10 elements = 100 operations.

#### O(2ⁿ) - Exponential Time
```python
def fibonacci_naive(n):
    """Naive recursive Fibonacci - exponential!"""
    if n <= 1:
        return n
    return fibonacci_naive(n-1) + fibonacci_naive(n-2)

# For n=5:
#         fib(5)
#        /      \
#    fib(4)    fib(3)
#    /    \    /    \
# fib(3) fib(2) ...
# Exponential tree of calls!
```

**Explanation:** Each call spawns 2 more calls. Grows like 2^n.

### Omega Notation (Ω)

**Definition:** Lower bound on growth rate (best-case scenario)

```python
def linear_search(arr, target):
    """Search for element in array"""
    for i, num in enumerate(arr):
        if num == target:
            return i  # Best case: O(1) if first element
    return -1  # Worst case: O(n) if not found

# Best case: Ω(1)
# Worst case: O(n)
# Average case: Θ(n)
```

### Theta Notation (Θ)

**Definition:** Tight bound (both upper and lower bound are the same)

```python
def sum_array(arr):
    """Always visits every element"""
    total = 0
    for num in arr:  # Must iterate through all n elements
        total += num
    return total

# Best, Average, Worst case all: Θ(n)
```

### Analyzing Complex Code

#### Example 1: Multiple Loops
```python
def complex_function(arr):
    # Loop 1: O(n)
    for num in arr:
        print(num)
    
    # Loop 2: O(n)
    for num in arr:
        print(num * 2)
    
    # Loop 3: O(n²)
    for i in arr:
        for j in arr:
            print(i + j)
    
    # Total: O(n) + O(n) + O(n²) = O(n²)
    # Why? We drop lower order terms

complex_function([1, 2, 3])
```

**Rule:** Drop constants and lower-order terms.
- O(2n + 5) → O(n)
- O(n² + n) → O(n²)
- O(n² + 100n + 1000) → O(n²)

#### Example 2: Nested Loops with Different Ranges
```python
def print_pairs(arr1, arr2):
    for i in arr1:           # O(n) where n = len(arr1)
        for j in arr2:       # O(m) where m = len(arr2)
            print(i, j)
    
    # Total: O(n × m) = O(nm)

# If arr1 and arr2 are same size n: O(n²)
```

#### Example 3: Logarithmic in Loop
```python
def function(n):
    i = 1
    while i < n:
        print(i)
        i *= 2  # i doubles each iteration: 1, 2, 4, 8, 16...
    
    # How many iterations?
    # 2^k < n  →  k < log₂(n)
    # Answer: O(log n)
```

---

## Space Complexity

### What is Space Complexity?

Space complexity measures the total memory used by an algorithm as a function of input size.

**Components:**
1. **Input space**: Memory to store input (usually not counted)
2. **Auxiliary space**: Extra memory used by algorithm
3. **Output space**: Memory for output

### Examples

#### O(1) - Constant Space
```python
def sum_array(arr):
    """Uses only a few variables"""
    total = 0      # O(1) space
    count = 0      # O(1) space
    for num in arr:
        total += num
        count += 1
    return total / count if count > 0 else 0

# Space: O(1) - only stores total and count
```

#### O(n) - Linear Space
```python
def create_copy(arr):
    """Creates a copy of array"""
    new_arr = []
    for num in arr:
        new_arr.append(num)  # Storing n elements
    return new_arr

# Space: O(n) - new array of size n

def fibonacci_memo(n, memo={}):
    """Fibonacci with memoization"""
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci_memo(n-1, memo) + fibonacci_memo(n-2, memo)
    return memo[n]

# Space: O(n) - memo dictionary stores up to n values
```

#### O(log n) - Logarithmic Space
```python
def binary_search_recursive(arr, target, left, right):
    """Recursive binary search"""
    if left > right:
        return -1
    
    mid = (left + right) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, right)
    else:
        return binary_search_recursive(arr, target, left, mid - 1)

# Space: O(log n) - call stack depth is log(n)
```

#### O(n) - Call Stack Space
```python
def factorial(n):
    """Recursive factorial"""
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Space: O(n) - n recursive calls on stack
# Stack: factorial(5) → factorial(4) → ... → factorial(1)
```

---

## Python-Specific Complexities

### List Operations

```python
# Common List Operations Complexity

# Access
my_list = [1, 2, 3, 4, 5]
x = my_list[2]                    # O(1) - direct index access

# Append
my_list.append(6)                 # O(1) - amortized
my_list.extend([7, 8, 9])         # O(k) where k is length of extended list

# Insert
my_list.insert(0, 0)              # O(n) - must shift all elements
my_list.insert(len(my_list), 10)  # O(1) - append to end

# Delete
del my_list[0]                    # O(n) - must shift all elements
my_list.pop()                     # O(1) - remove from end
my_list.pop(0)                    # O(n) - remove from start
my_list.remove(5)                 # O(n) - must search then shift

# Search
if 5 in my_list:                  # O(n) - linear search
    pass

# Slicing
sub = my_list[1:4]                # O(k) where k is slice length

# Sorting
my_list.sort()                    # O(n log n) - Timsort
sorted_list = sorted(my_list)     # O(n log n) - creates new list

# Reverse
my_list.reverse()                 # O(n) - in-place
reversed_list = list(reversed(my_list))  # O(n) - creates new list
```

### Dictionary Operations

```python
# Common Dictionary Operations Complexity

my_dict = {'a': 1, 'b': 2, 'c': 3}

# Access
value = my_dict['a']              # O(1) average
value = my_dict.get('a')          # O(1) average

# Insert/Update
my_dict['d'] = 4                  # O(1) average
my_dict.update({'e': 5, 'f': 6})  # O(k) where k is number of items

# Delete
del my_dict['a']                  # O(1) average
my_dict.pop('b')                  # O(1) average

# Search
if 'c' in my_dict:                # O(1) average (checking key)
    pass

if 3 in my_dict.values():         # O(n) (checking values)
    pass

# Iteration
for key in my_dict:               # O(n)
    print(key, my_dict[key])
```

### Set Operations

```python
# Common Set Operations Complexity

my_set = {1, 2, 3, 4, 5}

# Add/Remove
my_set.add(6)                     # O(1) average
my_set.remove(1)                  # O(1) average
my_set.discard(10)                # O(1) average (no error if not exists)

# Search
if 3 in my_set:                   # O(1) average
    pass

# Set Operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}

union = set1 | set2               # O(len(set1) + len(set2))
intersection = set1 & set2        # O(min(len(set1), len(set2)))
difference = set1 - set2          # O(len(set1))
```

### String Operations

```python
# Strings are IMMUTABLE in Python!

s = "hello"

# Access
char = s[0]                       # O(1)

# Concatenation
s1 = "hello"
s2 = "world"
s3 = s1 + s2                      # O(n + m) - creates new string!

# Multiple concatenations (BAD!)
result = ""
for i in range(n):
    result += str(i)              # O(n²) - creates new string each time!

# Multiple concatenations (GOOD!)
result = []
for i in range(n):
    result.append(str(i))
result = "".join(result)          # O(n) - join is efficient!

# Slicing
sub = s[1:4]                      # O(k) where k is slice length

# Search
if "ll" in s:                     # O(n × m) where m is pattern length
    pass

s.find("ll")                      # O(n × m)
s.index("ll")                     # O(n × m)

# Replace
new_s = s.replace("l", "L")       # O(n)

# Split
words = s.split()                 # O(n)
```

### List Comprehensions vs Loops

```python
# List comprehension
squares = [x**2 for x in range(n)]  # O(n) time, O(n) space

# Equivalent loop
squares = []
for x in range(n):
    squares.append(x**2)              # O(n) time, O(n) space

# Both have same complexity!
# Comprehensions are more Pythonic and often slightly faster
```

### Generator Expressions (Memory Efficient!)

```python
# List comprehension - creates entire list
squares_list = [x**2 for x in range(1000000)]  # O(n) space - 1M integers in memory!

# Generator expression - lazy evaluation
squares_gen = (x**2 for x in range(1000000))   # O(1) space - generates on demand!

# Use generators when you only need to iterate once
for square in squares_gen:
    print(square)  # Computes one at a time
```

---

## Practical Examples

### Example 1: Finding Duplicates

```python
# Approach 1: Brute Force - O(n²) time, O(1) space
def has_duplicates_v1(arr):
    for i in range(len(arr)):
        for j in range(i + 1, len(arr)):
            if arr[i] == arr[j]:
                return True
    return False

# Approach 2: Sorting - O(n log n) time, O(1) space (if in-place)
def has_duplicates_v2(arr):
    arr.sort()
    for i in range(len(arr) - 1):
        if arr[i] == arr[i + 1]:
            return True
    return False

# Approach 3: Hash Set - O(n) time, O(n) space
def has_duplicates_v3(arr):
    seen = set()
    for num in arr:
        if num in seen:
            return True
        seen.add(num)
    return False

# ML Context: Checking for duplicate samples in dataset
# For 1M samples: v1 = hours, v2 = seconds, v3 = milliseconds!
```

### Example 2: Two Sum Problem

**Problem:** Find two numbers in array that add up to target.

```python
# Approach 1: Brute Force - O(n²)
def two_sum_v1(nums, target):
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []

# Approach 2: Hash Map - O(n)
def two_sum_v2(nums, target):
    seen = {}  # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

# Example:
nums = [2, 7, 11, 15]
target = 9
print(two_sum_v2(nums, target))  # [0, 1] because nums[0] + nums[1] = 9
```

### Example 3: Fibonacci Comparison

```python
import time

# O(2ⁿ) - Exponential (TERRIBLE!)
def fib_recursive(n):
    if n <= 1:
        return n
    return fib_recursive(n-1) + fib_recursive(n-2)

# O(n) - Linear with memoization
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]

# O(n) - Linear iterative
def fib_iterative(n):
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b

# Benchmark
n = 35
start = time.time()
result1 = fib_recursive(n)
print(f"Recursive: {time.time() - start:.2f}s")  # ~3 seconds

start = time.time()
result2 = fib_memo(n)
print(f"Memoized: {time.time() - start:.4f}s")   # ~0.0001 seconds

start = time.time()
result3 = fib_iterative(n)
print(f"Iterative: {time.time() - start:.4f}s")  # ~0.00001 seconds
```

---

## Practice Problems

### Basic Problems

**Problem 1: Constant Time Check**
```python
"""
Write a function that returns True if a list is empty, False otherwise.
What is its time complexity?
"""

def is_empty(arr):
    return len(arr) == 0  # or: return not arr

# Answer: O(1) - just checking length
```

**Problem 2: Linear Search**
```python
"""
Find the index of target in array. Return -1 if not found.
Analyze best, worst, average case.
"""

def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

# Best case: Ω(1) - target is first element
# Worst case: O(n) - target not in array or at end
# Average case: Θ(n) - expected to check n/2 elements
```

**Problem 3: Count Operations**
```python
"""
How many operations does this code perform for n=10?
What is the Big O?
"""

def mystery_function(n):
    count = 0
    for i in range(n):           # n iterations
        for j in range(i):       # 0, 1, 2, ..., n-1 iterations
            count += 1
    return count

# Operations: 0 + 1 + 2 + ... + (n-1) = n(n-1)/2 = (n² - n)/2
# Big O: O(n²) - we drop constants and lower terms
```

### Intermediate Problems

**Problem 4: Analyze This Code**
```python
"""
What is the time and space complexity?
"""

def process_data(arr):
    # Step 1
    result = []
    for num in arr:              # O(n)
        result.append(num * 2)
    
    # Step 2
    result.sort()                # O(n log n)
    
    # Step 3
    lookup = set(result)         # O(n)
    
    # Step 4
    for i in range(len(result)):      # O(n)
        for j in range(len(result)):  # O(n)
            if result[i] + result[j] in lookup:  # O(1)
                pass
    
    return result

# Time: O(n) + O(n log n) + O(n) + O(n²) = O(n²)
# Space: O(n) + O(n) = O(n)
```

**Problem 5: Compare Approaches**
```python
"""
Which is faster for finding if array contains duplicates?
"""

# Option A
def option_a(arr):
    return len(arr) != len(set(arr))
# Time: O(n), Space: O(n)

# Option B
def option_b(arr):
    arr_sorted = sorted(arr)
    for i in range(len(arr_sorted) - 1):
        if arr_sorted[i] == arr_sorted[i + 1]:
            return True
    return False
# Time: O(n log n), Space: O(n) for sorted copy

# Answer: Option A is faster! O(n) < O(n log n)
```

### Advanced Problems

**Problem 6: Nested Structure**
```python
"""
Analyze the complexity of this nested loop pattern:
"""

def complex_pattern(n):
    for i in range(n):           # n iterations
        j = i
        while j > 0:             # log(i) iterations (j /= 2 each time)
            j //= 2
            for k in range(n):   # n iterations
                print(k)

# Outer loop: n times
# Middle loop: log(i) times for each i
# Inner loop: n times
# Total: n × average(log(i)) × n ≈ O(n² log n)
```

**Problem 7: Amortized Analysis**
```python
"""
Why is list.append() O(1) amortized?
"""

# When you append to a Python list:
# - If there's space: O(1) to add element
# - If list is full: O(n) to create new larger list and copy all elements
# 
# BUT: Python doubles the size each time, so copying happens less frequently
# Sequence of appends: 1, 2, 4, 8, 16, 32, ...
# Total copies for n appends: 1 + 2 + 4 + ... + n/2 = n - 1
# Average cost per append: (n-1)/n ≈ O(1) amortized!

import sys

lst = []
for i in range(20):
    lst.append(i)
    print(f"Length: {len(lst)}, Capacity: {sys.getsizeof(lst)}")
```

---

## Mini-Project: Algorithm Performance Analyzer

### Project Description
Build a CLI tool that compares different algorithms and visualizes their performance.

### Requirements
1. Implement multiple algorithms for same problem
2. Measure actual execution time
3. Plot complexity graphs
4. Compare theoretical vs actual performance

### Implementation

```python
import time
import matplotlib.pyplot as plt
import random

class AlgorithmAnalyzer:
    """Compare different algorithms empirically"""
    
    def __init__(self):
        self.results = {}
    
    def measure_time(self, func, *args):
        """Measure execution time of function"""
        start = time.perf_counter()
        result = func(*args)
        end = time.perf_counter()
        return end - start, result
    
    def run_experiment(self, func, input_sizes, generate_input):
        """Run algorithm on different input sizes"""
        times = []
        for size in input_sizes:
            data = generate_input(size)
            exec_time, _ = self.measure_time(func, data)
            times.append(exec_time)
        return times
    
    def compare_algorithms(self, algorithms, input_sizes, generate_input):
        """Compare multiple algorithms"""
        plt.figure(figsize=(12, 6))
        
        for name, func in algorithms.items():
            print(f"Testing {name}...")
            times = self.run_experiment(func, input_sizes, generate_input)
            self.results[name] = times
            plt.plot(input_sizes, times, marker='o', label=name)
        
        plt.xlabel('Input Size (n)')
        plt.ylabel('Time (seconds)')
        plt.title('Algorithm Performance Comparison')
        plt.legend()
        plt.grid(True)
        plt.savefig('algorithm_comparison.png')
        plt.show()
        
        print("\n" + "="*50)
        print("Results Summary:")
        print("="*50)
        for name, times in self.results.items():
            print(f"{name:20} | {times[-1]:.6f}s for n={input_sizes[-1]}")

# Example Usage: Comparing Search Algorithms
def linear_search(arr, target=-1):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

def binary_search(arr, target=-1):
    arr = sorted(arr)  # Ensure sorted
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

def set_search(arr, target=-1):
    return target in set(arr)

# Run comparison
analyzer = AlgorithmAnalyzer()
algorithms = {
    'Linear Search': linear_search,
    'Binary Search': binary_search,
    'Set Search': set_search
}

input_sizes = [100, 500, 1000, 5000, 10000]
generate_input = lambda n: list(range(n))

analyzer.compare_algorithms(algorithms, input_sizes, generate_input)
```

### Extension Ideas
1. Add sorting algorithm comparison (bubble, merge, quick)
2. Visualize space complexity
3. Compare different data structures (list vs set vs dict)
4. Test with different input types (random, sorted, reverse sorted)
5. Generate complexity reports (PDF/HTML)

### ML Relevance
- Choosing between algorithms for large datasets
- Optimizing data preprocessing pipelines
- Understanding trade-offs in feature engineering
- Profiling bottlenecks in training loops

---

## Summary & Key Takeaways

### Master These Concepts
✅ Big O notation represents worst-case growth rate  
✅ Drop constants and lower-order terms  
✅ Space complexity is equally important  
✅ Python operations have specific complexities  
✅ Choose data structures based on operations needed  

### Common Patterns to Recognize
- Single loop → O(n)
- Nested loops → O(n²)
- Halving problem → O(log n)
- Recursion tree → count nodes and depth
- Hash table operations → O(1) average

### When Analyzing Code
1. Identify the input size (what is n?)
2. Count basic operations in loops
3. Analyze recursive calls
4. Consider best, average, worst cases
5. Don't forget space complexity!

### Next Chapter Preview
In Chapter 2, we'll dive deep into **Recursion**, where we'll:
- Build intuition for recursive thinking
- Master the call stack
- Solve classic recursive problems
- Learn when recursion is better than iteration
- Optimize recursive solutions with memoization

---

## Additional Resources

**Books:**
- "Introduction to Algorithms" by CLRS - Chapter 3
- "Grokking Algorithms" by Aditya Bhargava - Chapters 1-2

**Online:**
- Big O Cheat Sheet: https://www.bigocheatsheet.com/
- Python Time Complexity: https://wiki.python.org/moin/TimeComplexity
- Visualgo: https://visualgo.net/en/sorting (visualize algorithms)

**Practice:**
- LeetCode: Easy problems tagged "Array"
- HackerRank: Practice > Data Structures > Arrays
- Practice analyzing complexity before submitting solutions!

---

**Next Steps:** Complete the practice problems, build the mini-project, then move on to Chapter 2: Recursion!