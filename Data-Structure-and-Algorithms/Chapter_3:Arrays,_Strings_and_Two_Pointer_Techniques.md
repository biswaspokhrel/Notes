# Chapter 3: Arrays, Strings & Two-Pointer Techniques

## 📚 Table of Contents
1. [Introduction](#introduction)
2. [Arrays — Deep Dive](#arrays--deep-dive)
3. [Strings — Deep Dive](#strings--deep-dive)
4. [Two-Pointer Technique](#two-pointer-technique)
5. [Sliding Window Pattern](#sliding-window-pattern)
6. [Prefix Sums](#prefix-sums)
7. [Kadane's Algorithm](#kadanes-algorithm)
8. [Practice Problems](#practice-problems)
9. [Mini-Project: Text Processing Toolkit](#mini-project-text-processing-toolkit)
10. [ML Relevance & Summary](#ml-relevance--summary)

---

## Introduction

### Why Arrays and Strings?

Every dataset you will ever touch in ML — a feature matrix, an image, a sentence — lives in memory as an array or a string. Mastering how these structures work *internally* and what operations cost what lets you write code that actually scales.

On top of raw mechanics this chapter teaches three high-value *thinking patterns*:

| Pattern | What it does | Typical gain |
|---|---|---|
| Two-Pointer | Replaces a nested loop with two moving indices | O(n²) → O(n) |
| Sliding Window | Processes every fixed- or variable-size subarray in one pass | O(n·k) → O(n) |
| Prefix Sum | Pre-computes cumulative totals so any range-sum is O(1) | O(n) per query → O(1) per query |

### Learning Goals
- Predict the cost of any Python list or string operation.
- Recognise when two-pointer, sliding-window, or prefix-sum applies.
- Solve O(n) problems that brute force handles in O(n²).
- Build a reusable NLP text-processing toolkit.

---

## Arrays — Deep Dive

### How Python Lists Work Internally

A Python list is a **dynamic array**: a contiguous block of *pointers* to objects, with extra slots pre-allocated so that most `append` calls do not need to copy.

```
┌──────────────────────────────────┐
│  list object                     │
│  ├─ ptr to buffer ───────────────┼──► [ *  |  *  |  *  |  *  | NULL | NULL ]
│  ├─ length      = 4              │       │     │     │     │
│  └─ allocated   = 6              │       ▼     ▼     ▼     ▼
└──────────────────────────────────┘      10    20    30    40
                                     ^^^^^^^^^^^^^^^^^^^^^^
                                     only 4 of 6 slots used
```

When the buffer is full, Python allocates a new one roughly **1.125×** the old size, copies everything, then continues. The occasional O(n) copy is spread over many O(1) appends, giving **O(1) amortised** append.

### Complete Operation-Cost Table

| Operation | Time | Why |
|---|---|---|
| `a[i]` | O(1) | Pointer offset |
| `a[i] = x` | O(1) | Overwrite one pointer |
| `a.append(x)` | O(1) amortised | Amortised over growth copies |
| `a.pop()` | O(1) | Remove last — no shifting |
| `a.pop(i)` | O(n) | Shift every element after i left |
| `a.insert(i, x)` | O(n) | Shift every element after i right |
| `a.remove(x)` | O(n) | Linear scan + shift |
| `x in a` | O(n) | Linear scan |
| `a.sort()` | O(n log n) | Timsort |
| `a[i:j]` | O(j−i) | Copy slice into new list |
| `len(a)` | O(1) | Stored as attribute |
| `a.extend(b)` | O(k), k=len(b) | Append k items |

### Creating Arrays — Every Variant

```python
# Literal
a = [1, 2, 3, 4, 5]

# range
b = list(range(10))              # [0, 1, …, 9]

# Fill with one value
c = [0] * 5                      # [0, 0, 0, 0, 0]

# ──── 2-D matrix ─────────────────────────────────
# CORRECT — each row is an independent list
matrix = [[0] * 4 for _ in range(3)]
matrix[0][0] = 9
# [[9,0,0,0],
#  [0,0,0,0],   ← unaffected
#  [0,0,0,0]]

# WRONG — all rows are the SAME list object!
bad = [[0] * 4] * 3
bad[0][0] = 9
# [[9,0,0,0],
#  [9,0,0,0],   ← also changed!
#  [9,0,0,0]]
# ─────────────────────────────────────────────────

# Comprehensions
squares = [x**2 for x in range(10)]
evens   = [x for x in range(20) if x % 2 == 0]
```

### Pattern A — Reverse In Place

```
[1, 2, 3, 4, 5]
 L           R      swap(1,5) → [5, 2, 3, 4, 1]
    L     R         swap(2,4) → [5, 4, 3, 2, 1]
       L=R          stop  ✓

Time  O(n)   — n/2 swaps
Space O(1)   — two variables
```

```python
def reverse_array(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left  += 1
        right -= 1

arr = [1, 2, 3, 4, 5]
reverse_array(arr)
print(arr)   # [5, 4, 3, 2, 1]
```

### Pattern B — Rotate Right by k (Triple-Reverse Trick)

```
arr = [1, 2, 3, 4, 5, 6, 7],  k = 3   want → [5,6,7,1,2,3,4]

Step 1  reverse all        [7, 6, 5, 4, 3, 2, 1]
Step 2  reverse first k    [5, 6, 7, 4, 3, 2, 1]
Step 3  reverse the rest   [5, 6, 7, 1, 2, 3, 4]  ✓

Time O(n), Space O(1)
```

```python
def rotate_right(arr, k):
    n = len(arr)
    k = k % n                          # k > n is the same as k % n

    def rev(lo, hi):
        while lo < hi:
            arr[lo], arr[hi] = arr[hi], arr[lo]
            lo += 1; hi -= 1

    rev(0, n - 1)       # whole array
    rev(0, k - 1)       # first k
    rev(k, n - 1)       # rest

arr = [1, 2, 3, 4, 5, 6, 7]
rotate_right(arr, 3)
print(arr)   # [5, 6, 7, 1, 2, 3, 4]
```

### Pattern C — Remove Duplicates from Sorted Array (Write Pointer)

```
[1, 1, 2, 2, 3, 4, 4]
 W  R                       arr[R]==arr[R-1]  skip
 W     R                    arr[R]!=arr[R-1]  arr[W]=2  W++  → [1,2,2,2,3,4,4]
    W     R                 skip
    W        R              arr[W]=3  W++     → [1,2,3,2,3,4,4]
       W        R           arr[W]=4  W++     → [1,2,3,4,3,4,4]
          W        R        skip
Result  arr[:W] = [1, 2, 3, 4]
```

```python
def remove_duplicates(arr):
    if not arr:
        return 0
    write = 1
    for read in range(1, len(arr)):
        if arr[read] != arr[read - 1]:
            arr[write] = arr[read]
            write += 1
    return write

arr = [1, 1, 2, 2, 3, 4, 4]
n   = remove_duplicates(arr)
print(arr[:n])   # [1, 2, 3, 4]
```

### Pattern D — Find Duplicates (Hash Set)

```python
def find_duplicates(arr):
    seen, dups = set(), set()
    for x in arr:
        if x in seen:
            dups.add(x)
        seen.add(x)
    return list(dups)

print(find_duplicates([1, 2, 3, 2, 4, 3, 5]))  # [2, 3]
# Time O(n), Space O(n)
```

---

## Strings — Deep Dive

### Immutability — The Key Fact

Python strings are **immutable**. You can never change a character in place; every "modification" silently creates a brand-new string object.

```python
s = "hello"
# s[0] = 'H'   ← TypeError!

s = s + " world"
# Under the hood:
#   1. allocate len("hello")+len(" world") bytes
#   2. copy "hello" into it
#   3. copy " world" into it
#   4. rebind name s to the new object
#   5. old "hello" is garbage-collected
```

#### The Concatenation Trap

```python
# BAD  —  O(n²) total
result = ""
for i in range(n):
    result += str(i)      # copies everything built so far, every iteration
# work = 1 + 2 + 3 + … + n  =  O(n²)

# GOOD  —  O(n) total
parts = []
for i in range(n):
    parts.append(str(i))  # O(1) amortised
result = "".join(parts)   # single allocation, single pass
```

### String-Method Cost Table

| Method | Time | Notes |
|---|---|---|
| `s[i]` | O(1) | |
| `s + t` | O(n+m) | new string |
| `"".join(lst)` | O(total chars) | best multi-concat |
| `s.split()` | O(n) | |
| `s.find(sub)` | O(n·m) | m = len(sub) |
| `s.replace(a,b)` | O(n) | new string |
| `s.upper()/lower()` | O(n) | new string |
| `s.startswith(p)` | O(len(p)) | |
| `s.strip()` | O(n) | |

### Palindrome — Three Levels

```python
# Level 1 — slice (O(n) space)
def palindrome_slice(s):
    return s == s[::-1]

# Level 2 — two pointers (O(1) space)
def palindrome_two_ptr(s):
    """
    "racecar"
     L     R   r==r ✓
      L   R    a==a ✓
       L R     c==c ✓
        L=R    done → True

    "hello"
     L   R     h!=o → False  (instant)
    """
    L, R = 0, len(s) - 1
    while L < R:
        if s[L] != s[R]:
            return False
        L += 1; R -= 1
    return True

# Level 3 — ignore punctuation & case
def palindrome_clean(s):
    """
    "A man, a plan, a canal: Panama"  →  True
    Skip non-alphanumeric chars, compare case-insensitively.
    """
    L, R = 0, len(s) - 1
    while L < R:
        while L < R and not s[L].isalnum():  L += 1
        while L < R and not s[R].isalnum():  R -= 1
        if s[L].lower() != s[R].lower():
            return False
        L += 1; R -= 1
    return True

print(palindrome_clean("A man, a plan, a canal: Panama"))  # True
```

### Anagrams — Detect & Group

```python
from collections import Counter, defaultdict

# ── Detect (two strings) ──
def is_anagram(s1, s2):
    """
    "listen" vs "silent"
    freq after s1: {l:1, i:1, s:1, t:1, e:1, n:1}
    subtract s2:   {l:0, i:0, s:0, t:0, e:0, n:0}  all zero → True

    Time O(n), Space O(1) — alphabet is fixed at 26
    """
    if len(s1) != len(s2):
        return False
    freq = {}
    for c in s1:
        freq[c] = freq.get(c, 0) + 1
    for c in s2:
        freq[c] = freq.get(c, 0) - 1
        if freq[c] < 0:
            return False
    return True

# ── Group (list of words) ──
def group_anagrams(words):
    """
    Key idea: sorted characters are the same for all anagrams.
    "eat" → "aet",  "tea" → "aet",  "tan" → "ant"

    Time O(n · k log k), Space O(n · k)   where k = max word length
    """
    groups = defaultdict(list)
    for w in words:
        groups[''.join(sorted(w))].append(w)
    return list(groups.values())

print(group_anagrams(["eat","tea","tan","ate","nat","bat"]))
# [['eat','tea','ate'], ['tan','nat'], ['bat']]
```

### First Non-Repeating Character

```python
def first_unique(s):
    """
    Two-pass approach.
    Pass 1 — build frequency map.
    Pass 2 — return first char whose freq is 1.

    "aabbcdc"  freqs: a2 b2 c2 d1
    Scan: a(2) b(2) c(2) d(1) ← index 5

    Time O(n), Space O(1)
    """
    freq = {}
    for c in s:
        freq[c] = freq.get(c, 0) + 1
    for i, c in enumerate(s):
        if freq[c] == 1:
            return i
    return -1

print(first_unique("aabbcdc"))  # 5
```

---

## Two-Pointer Technique

### What It Is

Two index variables move through the same collection. A condition decides which pointer advances at each step. This eliminates the inner loop in many O(n²) brute-force solutions.

### When to Reach for Two Pointers
- Sorted array → find a pair/triplet with a target sum.
- Filter or compact an array in place.
- Merge two sorted sequences.
- Detect a palindrome.

---

### Pattern 1 — Opposite Ends (sorted pair-sum)

```
Target = 10    arr = [1, 2, 3, 4, 5, 6, 7, 8]
                      L                    R
  1+8 = 9  < 10  → L is too small  → L++
                      L              R
  2+8 = 10 == 10 → FOUND  return [1, 7]

Decision rule:
  sum < target  →  need bigger  →  L++
  sum > target  →  need smaller →  R--
  sum == target →  done
```

```python
def two_sum_sorted(arr, target):
    L, R = 0, len(arr) - 1
    while L < R:
        s = arr[L] + arr[R]
        if   s == target: return [L, R]
        elif s <  target: L += 1
        else:             R -= 1
    return []

print(two_sum_sorted([1,2,3,4,5,6,7,8], 10))  # [1, 7]
```

### Pattern 2 — Fast & Slow (filter in place)

```
Remove every 3 from [1, 3, 2, 3, 4, 3, 5]

S=0 F=0  val=1 ≠ 3  → arr[0]=1  S=1
S=1 F=1  val=3 == 3 → skip
S=1 F=2  val=2 ≠ 3  → arr[1]=2  S=2
S=2 F=3  val=3 == 3 → skip
S=2 F=4  val=4 ≠ 3  → arr[2]=4  S=3
S=3 F=5  val=3 == 3 → skip
S=3 F=6  val=5 ≠ 3  → arr[3]=5  S=4

arr[:S] = [1, 2, 4, 5]   Time O(n), Space O(1)
```

```python
def remove_element(arr, val):
    slow = 0
    for fast in range(len(arr)):
        if arr[fast] != val:
            arr[slow] = arr[fast]
            slow += 1
    return slow

a = [1, 3, 2, 3, 4, 3, 5]
print(a[:remove_element(a, 3)])  # [1, 2, 4, 5]
```

### Pattern 3 — Same Direction (merge two sorted arrays)

```
A = [1, 3, 5, 7]     B = [2, 4, 6, 8]
i=0 j=0   1<2  take 1   → [1]
i=1 j=0   3>2  take 2   → [1,2]
i=1 j=1   3<4  take 3   → [1,2,3]
i=2 j=1   5>4  take 4   → [1,2,3,4]
…                         [1,2,3,4,5,6,7,8]
```

```python
def merge_sorted(a, b):
    res, i, j = [], 0, 0
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            res.append(a[i]); i += 1
        else:
            res.append(b[j]); j += 1
    res.extend(a[i:])
    res.extend(b[j:])
    return res

print(merge_sorted([1,3,5,7], [2,4,6,8]))  # [1,2,3,4,5,6,7,8]
```

### Pattern 4 — Three Sum (fix one element, two-pointer the rest)

```python
def three_sum(nums):
    """
    Find every unique triplet that sums to 0.

    Strategy
    ────────
    1. Sort.
    2. Loop i over each element as the "anchor".
    3. Two-pointer (L, R) on the slice after i to find pairs summing to -nums[i].
    4. Skip duplicates at every level to avoid repeats.

    Walkthrough   nums = [-4, -1, -1, 0, 1, 2]  (sorted)
    ─────────────────────────────────────────────────────
    i=0  anchor=-4  need pair = 4
         L=1 R=5   -1+2=1 < 4  L++
         L=2 R=5   -1+2=1 < 4  L++
         L=3 R=5    0+2=2 < 4  L++
         L=4 R=5    1+2=3 < 4  L++   → no triplet

    i=1  anchor=-1  need pair = 1
         L=2 R=5   -1+2=1 == 1  → [-1,-1,2]  ← found!   skip dups
         L=3 R=4    0+1=1 == 1  → [-1, 0,1]  ← found!   skip dups
         done

    i=2  nums[2]==nums[1]  → skip (duplicate anchor)
    i=3  anchor=0   need pair = 0
         L=4 R=5    1+2=3 > 0   R--    → no triplet

    Result  [[-1,-1,2], [-1,0,1]]
    ─────────────────────────────────────────────────────
    Time O(n²), Space O(1) ignoring output
    """
    nums.sort()
    out = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i-1]:
            continue                          # skip duplicate anchor
        L, R = i + 1, len(nums) - 1
        while L < R:
            s = nums[i] + nums[L] + nums[R]
            if s == 0:
                out.append([nums[i], nums[L], nums[R]])
                while L < R and nums[L] == nums[L+1]: L += 1   # skip dup L
                while L < R and nums[R] == nums[R-1]: R -= 1   # skip dup R
                L += 1; R -= 1
            elif s < 0: L += 1
            else:       R -= 1
    return out

print(three_sum([-1, 0, 1, 2, -1, -4]))  # [[-1,-1,2], [-1,0,1]]
```

---

## Sliding Window Pattern

### Core Idea

Place a "window" over the array. Slide it one step right each iteration. Instead of recomputing the window's value from scratch, **remove the element that left** and **add the element that entered**.

```
Naive — recompute every window of size 3
[2, 1, 5, 1, 3, 2]
 ———       sum=8     ← 3 additions
    ———    sum=7     ← 3 additions   total work = n·k
       ———  sum=9
          ——— sum=6

Sliding — update incrementally
[2, 1, 5, 1, 3, 2]
 ———       sum=8
    ———    sum = 8 − 2 + 1 = 7   ← 1 sub + 1 add
       ———  sum = 7 − 1 + 3 = 9
          ——— sum = 9 − 5 + 2 = 6   total work = O(n)
```

### Fixed-Size Window — Max Subarray Sum of Size k

```python
def max_sum_subarray(arr, k):
    """
    Time O(n), Space O(1)
    """
    if len(arr) < k:
        return 0
    window = sum(arr[:k])       # first window
    best   = window
    for i in range(k, len(arr)):
        window += arr[i] - arr[i - k]   # slide
        best   = max(best, window)
    return best

print(max_sum_subarray([2, 1, 5, 1, 3, 2], 3))  # 9
```

### Variable-Size Window — Longest Substring Without Repeating Characters

```
s = "abcabcbb"

Expand right until a duplicate appears, then shrink left just enough
to remove the duplicate.  Use a dict to remember each char's last index
so the left pointer can jump directly instead of crawling.

L=0 R=0  'a'   seen={a:0}          window="a"      max=1
L=0 R=1  'b'   seen={a:0,b:1}      window="ab"     max=2
L=0 R=2  'c'   seen={a:0,b:1,c:2}  window="abc"    max=3
L=0 R=3  'a'   'a' last at 0 ≥ L   → L = 0+1 = 1   window="bca"  max=3
L=1 R=4  'b'   'b' last at 1 ≥ L   → L = 1+1 = 2   window="cab"  max=3
L=2 R=5  'c'   'c' last at 2 ≥ L   → L = 2+1 = 3   window="abc"  max=3
L=3 R=6  'b'   'b' last at 4 ≥ L   → L = 4+1 = 5   window="cb"   max=3
L=5 R=7  'b'   'b' last at 6 ≥ L   → L = 6+1 = 7   window="b"    max=3

Answer 3  ("abc")
```

```python
def longest_unique_substr(s):
    """
    Time O(n), Space O(alphabet)
    """
    last_seen = {}          # char → index of its most recent occurrence
    L = 0
    best = 0
    for R, ch in enumerate(s):
        if ch in last_seen and last_seen[ch] >= L:
            L = last_seen[ch] + 1     # jump L past the duplicate
        last_seen[ch] = R
        best = max(best, R - L + 1)
    return best

print(longest_unique_substr("abcabcbb"))  # 3
print(longest_unique_substr("bbbbb"))     # 1
print(longest_unique_substr("pwwkew"))    # 3  ("wke")
```

### Variable-Size Window — At Most k Distinct Characters

```python
def longest_k_distinct(s, k):
    """
    Expand R freely.  When distinct-char count exceeds k, shrink L
    until it drops back to k.

    Time O(n), Space O(k)
    """
    from collections import defaultdict
    freq = defaultdict(int)
    L, best = 0, 0
    for R in range(len(s)):
        freq[s[R]] += 1
        while len(freq) > k:            # too many distinct
            freq[s[L]] -= 1
            if freq[s[L]] == 0:
                del freq[s[L]]          # remove key so len() is accurate
            L += 1
        best = max(best, R - L + 1)
    return best

print(longest_k_distinct("ecebaabc", 3))  # 7
```

---

## Prefix Sums

### Why Pre-Compute?

Without prefix sums, summing `arr[L..R]` requires iterating L→R every time — O(n) per query. With a prefix array built once in O(n), every subsequent range query is O(1).

```
arr    =  [ 1,  2,  3,  4,  5 ]
index       0   1   2   3   4

prefix =  [ 0,  1,  3,  6, 10, 15 ]
index       0   1   2   3   4   5

prefix[i] = arr[0] + arr[1] + … + arr[i−1]

Range sum  arr[L..R] = prefix[R+1] − prefix[L]

Example   arr[1..3] = prefix[4] − prefix[1] = 10 − 1 = 9
Verify    2 + 3 + 4 = 9  ✓
```

```python
def build_prefix(arr):
    p = [0] * (len(arr) + 1)
    for i in range(len(arr)):
        p[i+1] = p[i] + arr[i]
    return p

def range_sum(prefix, L, R):
    return prefix[R+1] - prefix[L]

arr = [1, 2, 3, 4, 5]
p   = build_prefix(arr)
print(p)                    # [0, 1, 3, 6, 10, 15]
print(range_sum(p, 1, 3))   # 9
print(range_sum(p, 0, 4))   # 15
```

### Advanced Use — Count Subarrays with Sum == k

```python
def count_subarrays_sum_k(arr, k):
    """
    Key insight: if prefix[j] − prefix[i] == k then arr[i..j−1] sums to k.
    Rearranged: we need prefix[i] == prefix[j] − k.
    Keep a running hash-map of how many times each prefix sum has appeared.

    arr = [1, 1, 1], k = 2
    ────────────────────────────────────
    running=0  need 0−2=−2  map={0:1}       count=0
    running=1  need 1−2=−1  map={0:1,1:1}   count=0
    running=2  need 2−2= 0  found 1 time!   count=1
    running=3  need 3−2= 1  found 1 time!   count=2
    ────────────────────────────────────
    Answer 2   (subarrays [1,1] starting at index 0 and 1)

    Time O(n), Space O(n)
    """
    seen = {0: 1}          # prefix_sum → count
    running = 0
    count   = 0
    for x in arr:
        running += x
        count  += seen.get(running - k, 0)
        seen[running] = seen.get(running, 0) + 1
    return count

print(count_subarrays_sum_k([1, 1, 1], 2))         # 2
print(count_subarrays_sum_k([1, 2, 3, 0, 1, 2], 3))  # 4
```

---

## Kadane's Algorithm

### The Problem
Find the **contiguous subarray** with the largest sum (array may contain negatives).

### The One-Line Intuition
At every index ask: *"Does extending the previous subarray help, or should I start fresh here?"*

```
arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

idx  val   extend vs start      cur    best   action
 0   −2    —                    −2     −2
 1    1    −2+1=−1  vs  1       1      1      restart
 2   −3     1−3=−2  vs −3      −2      1
 3    4    −2+4= 2  vs  4       4      4      restart
 4   −1     4−1= 3  vs −1       3      4
 5    2     3+2= 5  vs  2       5      5
 6    1     5+1= 6  vs  1       6      6      ← best
 7   −5     6−5= 1  vs −5       1      6
 8    4     1+4= 5  vs  4       5      6

Answer 6   subarray [4, −1, 2, 1]
```

```python
def max_subarray(arr):
    """  Time O(n), Space O(1)  """
    best = cur = arr[0]
    for x in arr[1:]:
        cur  = max(x, cur + x)
        best = max(best, cur)
    return best

print(max_subarray([-2,1,-3,4,-1,2,1,-5,4]))  # 6
print(max_subarray([-1,-2,-3]))                # -1
print(max_subarray([5,-3,5]))                  # 7
```

### Variant — Also Return the Actual Subarray

```python
def max_subarray_full(arr):
    best = cur = arr[0]
    best_start = best_end = cur_start = 0

    for i in range(1, len(arr)):
        if arr[i] > cur + arr[i]:       # restart
            cur        = arr[i]
            cur_start  = i
        else:                           # extend
            cur       += arr[i]

        if cur > best:
            best       = cur
            best_start = cur_start
            best_end   = i

    return best, arr[best_start:best_end+1]

print(max_subarray_full([-2,1,-3,4,-1,2,1,-5,4]))  # (6, [4,-1,2,1])
```

---

## Practice Problems

### Problem 1 — Container With Most Water (Medium)

**Statement:** Given heights, find two lines that trap the most water with the x-axis.

```
heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]
                                         8           8
Best pair: index 1 and 8                 |           |
  width  = 8 − 1 = 7                     |     …     |   7
  height = min(8, 7) = 7                 |           |   |
  water  = 7 × 7 = 49                    …           …   …
```

```python
def max_area(heights):
    """
    Why move the shorter pointer?
    Water = min(h[L], h[R]) × (R−L).  The shorter line caps the height.
    Moving the taller line inward can only shrink width with no height gain.
    Moving the shorter line is the only chance of a taller line appearing.

    Time O(n), Space O(1)
    """
    L, R   = 0, len(heights) - 1
    best   = 0
    while L < R:
        best = max(best, min(heights[L], heights[R]) * (R - L))
        if heights[L] < heights[R]:
            L += 1
        else:
            R -= 1
    return best

print(max_area([1,8,6,2,5,4,8,3,7]))  # 49
```

### Problem 2 — Minimum Window Substring (Hard)

**Statement:** Find the smallest window in `s` containing every character of `t`.

```python
def min_window(s, t):
    """
    Variable sliding window.
    Expand R until window is valid (contains all of t).
    Then shrink L as far as possible while still valid.
    Record the smallest valid window seen.

    s = "ADOBECODEBANC"  t = "ABC"

    Expand → "ADOBEC"  valid!  record len 6
    Shrink → "DOBEC"   missing A → stop
    Expand → … → "CODEBA"  valid! record len 6  (tie)
    Shrink → "ODEBA"   missing C → stop
    Expand → … → "EBANC"  valid!  record len 5
    Shrink → "BANC"    still valid! record len 4  ← best
    Shrink → "ANC"     missing B → stop
    End.  Answer "BANC"

    Time O(n), Space O(|t|)
    """
    from collections import Counter
    if not s or not t:
        return ""

    need     = Counter(t)
    required = len(need)        # unique chars we must satisfy
    have     = 0                # unique chars currently satisfied
    window   = {}

    L    = 0
    best = (float('inf'), 0, 0)  # (length, L, R)

    for R in range(len(s)):
        c = s[R]
        window[c] = window.get(c, 0) + 1
        if c in need and window[c] == need[c]:
            have += 1

        while have == required:                 # window is valid → shrink
            if R - L + 1 < best[0]:
                best = (R - L + 1, L, R)
            lc = s[L]
            window[lc] -= 1
            if lc in need and window[lc] < need[lc]:
                have -= 1
            L += 1

    return s[best[1]:best[2]+1] if best[0] != float('inf') else ""

print(min_window("ADOBECODEBANC", "ABC"))  # "BANC"
print(min_window("a", "aa"))               # ""
```

### Problem 3 — Longest Repeating Character Replacement (Medium)

**Statement:** Replace at most k characters to make the longest all-same-letter substring.

```python
def char_replacement(s, k):
    """
    Window is valid when:
      (window length) − (count of most frequent char in window) ≤ k

    If invalid, shrink from L.

    s = "AABABBA", k = 1
    Window "AABA"  most_freq A=3  replacements = 4−3 = 1 ≤ 1  ✓  len=4
    Window "AABAB" most_freq A=3  replacements = 5−3 = 2 > 1  ✗  shrink

    Answer 4

    Time O(n), Space O(1)  — at most 26 letters
    """
    from collections import defaultdict
    freq     = defaultdict(int)
    L        = 0
    max_freq = 0          # highest single-char count seen in window
    best     = 0

    for R in range(len(s)):
        freq[s[R]] += 1
        max_freq = max(max_freq, freq[s[R]])

        if (R - L + 1) - max_freq > k:   # invalid → shrink
            freq[s[L]] -= 1
            L += 1

        best = max(best, R - L + 1)
    return best

print(char_replacement("AABABBA", 1))  # 4
print(char_replacement("ABAB", 2))     # 4
```

### Problem 4 — Move Zeroes (Easy)

```python
def move_zeroes(nums):
    """
    Keep relative order of non-zero elements, push zeroes to end.

    [0, 1, 0, 3, 12]
    S=0 F=0  0   skip
    S=0 F=1  1   nums[0]=1  S=1  → [1,1,0,3,12]
    S=1 F=2  0   skip
    S=1 F=3  3   nums[1]=3  S=2  → [1,3,0,3,12]
    S=2 F=4  12  nums[2]=12 S=3  → [1,3,12,3,12]
    Fill [S:] with 0         →   [1,3,12,0,0]

    Time O(n), Space O(1)
    """
    S = 0
    for F in range(len(nums)):
        if nums[F] != 0:
            nums[S] = nums[F]
            S += 1
    for i in range(S, len(nums)):
        nums[i] = 0

nums = [0, 1, 0, 3, 12]
move_zeroes(nums)
print(nums)  # [1, 3, 12, 0, 0]
```

### Problem 5 — Merge Intervals (Medium)

```python
def merge_intervals(intervals):
    """
    Sort by start.  Walk through: if the current interval's start ≤
    previous end → they overlap → extend previous end if needed.
    Otherwise start a new merged interval.

    [[1,3],[2,6],[8,10],[15,18]]
    merged = [[1,3]]
    [2,6]   2 ≤ 3  → extend to [1,6]
    [8,10]  8 > 6  → push, start [8,10]
    [15,18] 15>10  → push, start [15,18]
    Result  [[1,6],[8,10],[15,18]]

    Time O(n log n), Space O(n)
    """
    if not intervals:
        return []
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for s, e in intervals[1:]:
        if s <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], e)
        else:
            merged.append([s, e])
    return merged

print(merge_intervals([[1,3],[2,6],[8,10],[15,18]]))  # [[1,6],[8,10],[15,18]]
print(merge_intervals([[1,4],[4,5]]))                  # [[1,5]]
```

---

## Mini-Project: Text Processing Toolkit

Every method maps directly to a technique from this chapter.

```python
from collections import Counter, defaultdict

class TextProcessor:
    def __init__(self, text=""):
        self.text = text

    def load(self, text):
        self.text = text
        return self

    # ── Word Frequency (hash map) ─────────────────────
    def word_freq(self):
        """Time O(n)"""
        return Counter(self.text.lower().split())

    def top_k(self, k):
        """Time O(n + k log n)  — heap inside most_common"""
        return self.word_freq().most_common(k)

    # ── N-Grams (fixed sliding window) ────────────────
    def char_ngrams(self, n):
        """Character n-grams.  Time O(m)"""
        return [self.text[i:i+n] for i in range(len(self.text) - n + 1)]

    def word_ngrams(self, n):
        """Word n-grams.  Time O(w)"""
        words = self.text.split()
        return [' '.join(words[i:i+n]) for i in range(len(words) - n + 1)]

    # ── Pattern Search ────────────────────────────────
    def find_all(self, pattern):
        """All start-indices of pattern.  Time O(n·m) worst case"""
        hits, start = [], 0
        while True:
            idx = self.text.find(pattern, start)
            if idx == -1: break
            hits.append(idx)
            start = idx + 1
        return hits

    # ── Palindrome Substrings (expand around centre) ──
    def palindromes(self, min_len=3):
        """
        For every index, expand outward while chars match.
        Check both odd-length and even-length centres.
        Time O(n²)
        """
        def expand(L, R):
            while L >= 0 and R < len(self.text) and self.text[L] == self.text[R]:
                L -= 1; R += 1
            return self.text[L+1:R]

        found = set()
        for i in range(len(self.text)):
            for p in (expand(i, i), expand(i, i+1)):   # odd, even
                if len(p) >= min_len:
                    found.add(p)
        return sorted(found, key=len, reverse=True)

    # ── Caesar Cipher (character arithmetic) ──────────
    def caesar(self, shift):
        """Time O(n)"""
        out = []
        for c in self.text:
            if c.isalpha():
                base = ord('A') if c.isupper() else ord('a')
                out.append(chr((ord(c) - base + shift) % 26 + base))
            else:
                out.append(c)
        return ''.join(out)

    # ── Run-Length Encoding (linear scan) ─────────────
    def rle(self):
        """
        "aaabbc" → "a3b2c1"
        Time O(n)
        """
        if not self.text: return ""
        parts, count = [], 1
        for i in range(1, len(self.text)):
            if self.text[i] == self.text[i-1]:
                count += 1
            else:
                parts.append(self.text[i-1] + str(count))
                count = 1
        parts.append(self.text[-1] + str(count))
        return ''.join(parts)


# ─── Demo ─────────────────────────────────────────────
if __name__ == "__main__":
    tp = TextProcessor("the quick brown fox jumps over the lazy dog the fox")
    print("Freq  :", tp.word_freq())
    print("Top 3 :", tp.top_k(3))
    print("Bigrams:", tp.word_ngrams(2))
    print("Find 'the':", tp.find_all("the"))

    tp2 = TextProcessor("abacabadabacaba")
    print("Palindromes:", tp2.palindromes(3))

    tp3 = TextProcessor("Hello World")
    enc = tp3.caesar(3)
    print(f"Encrypt: {tp3.text} → {enc}")
    print(f"Decrypt: {enc} → {TextProcessor(enc).caesar(23)}")

    tp4 = TextProcessor("aaabbcccc")
    print(f"RLE: {tp4.text} → {tp4.rle()}")
```

### Extension Ideas
1. **Spell checker** — edit distance (Chapter 11 preview).
2. **Word wrap** at a given width — greedy / DP.
3. **Trie autocomplete** — Chapter 8 preview.
4. **Benchmark** each method on large corpora and plot time vs input size.

---

## ML Relevance & Summary

### Where These Techniques Appear in ML
| Technique | ML use-case |
|---|---|
| String split / join | Tokenisation for NLP |
| Sliding window (fixed) | Feature extraction from time-series sensor data |
| Sliding window (variable) | Context windows in language models |
| Prefix sum | Cumulative reward in reinforcement learning |
| Two-pointer merge | Combining sorted sparse feature vectors |
| Kadane's | Detecting anomaly segments in signals |
| Frequency counting | Bag-of-words / TF-IDF features |
| RLE | Compressing sparse label sequences |

### Patterns Cheat-Sheet
| Pattern | Trigger phrase | Time | Space |
|---|---|---|---|
| Two Pointers (opposite) | "pair in sorted array" | O(n) | O(1) |
| Two Pointers (fast/slow) | "filter / compact in place" | O(n) | O(1) |
| Sliding Window (fixed) | "every subarray of size k" | O(n) | O(1) |
| Sliding Window (variable) | "longest/shortest satisfying …" | O(n) | O(n) |
| Prefix Sum | "range sum queries" | O(1)/query | O(n) |
| Kadane's | "max contiguous subarray" | O(n) | O(1) |

### Common Pitfalls
- **2-D array trap** — `[[0]*c]*r` shares rows. Use comprehension.
- **String concat in loop** — always collect into a list, then `join`.
- **Forgetting `k % n`** when rotating.
- **Not skipping duplicates** in three-sum → duplicate triplets.
- **Off-by-one** in prefix-sum indexing (`prefix[R+1] − prefix[L]`).

---

**Next → Chapter 4: Stacks & Queues**