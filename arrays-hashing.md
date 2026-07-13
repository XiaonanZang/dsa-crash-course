---
title: Arrays & Hashing
---

# Arrays & Hashing

The foundation pattern. Almost every medium problem uses a hash map or set somewhere, and the
graph family (adjacency lists, `visited`) is built directly on top of these. Master three
containers and the syntax for driving them, and you cover most easy/medium array problems.

The core idea is always the same: **trade space for time.** A nested loop that scans for a match
is O(n²). Put what you have seen into a `dict` or `set` and each lookup becomes O(1), so the whole
scan drops to O(n).

---

## The three containers

| Python | Role | C++ analog | Ops you actually use |
|---|---|---|---|
| `dict` | hash map | `unordered_map` | `d[k]`, `d.get(k, default)`, `k in d`, `d.items()` |
| `set` | hash set | `unordered_set` | `s.add(x)`, `x in s`, `s.discard(x)` |
| `collections.Counter` | dict that counts | — | `Counter(v)`, `c[k]`, `c.most_common(k)` |
| `collections.defaultdict` | dict with a default | — | `defaultdict(int)`, `defaultdict(list)` |

```python
from collections import Counter, defaultdict

d = {}                      # empty hash map
d.get(k, 0)                 # read with a default, without inserting k
k in d                      # O(1) membership

s = set()                   # empty hash set
s.add(x); x in s            # O(1) add + membership

Counter(v)                  # {value: count} in one line; compares with ==
defaultdict(int)            # missing key -> 0   (counting)
defaultdict(list)           # missing key -> []  (bucketing / adjacency lists)
```

---

## 1. Two Sum — the "seen so far" map

The archetype for the whole pattern. Teaches the **one-pass dict**: store each value as you go so a
future element can look back and find its complement in O(1).

```python
def two_sum(nums, target):
    seen = {}                        # value -> index seen so far
    for i, x in enumerate(nums):     # enumerate = index AND value together
        need = target - x
        if need in seen:             # O(1) lookup instead of a second loop
            return [seen[need], i]
        seen[x] = i                  # record x only AFTER checking, avoids using it twice
    return []
```

- `enumerate(nums)` yields `(i, x)` pairs — this is the Python replacement for `for (int i…)`.
- Brute force is O(n²) nested loops; the dict makes it a single O(n) pass.
- Order matters: check `need in seen` **before** inserting `x`, so an element cannot pair with itself.

---

## 2. Contains Duplicate — the set

Teaches **set for membership and dedup**.

```python
def contains_duplicate(nums):
    seen = set()
    for x in nums:
        if x in seen:
            return True
        seen.add(x)
    return False

# Pythonic one-liner: a set drops duplicates, so a length change means a dup existed
def contains_duplicate_short(nums):
    return len(set(nums)) != len(nums)
```

- `set(nums)` dedups an entire list in one call.
- `x in seen` is O(1); the same check on a `list` would be O(n) — never scan a list for membership.

---

## 3. Valid Anagram — Counter compared directly

Teaches `Counter` for frequency and the fact that **two Counters compare with `==`**. This is the
single biggest time-saver for rusty candidates: no manual counting loops.

```python
from collections import Counter

def is_anagram(s, t):
    return Counter(s) == Counter(t)      # two frequency maps, one comparison
```

If they forbid `Counter` and want it "by hand," the pattern is a single frequency dict:

```python
def is_anagram_manual(s, t):
    if len(s) != len(t):
        return False
    count = {}
    for c in s:
        count[c] = count.get(c, 0) + 1   # .get(c, 0) = "current count or zero"
    for c in t:
        if count.get(c, 0) == 0:         # c missing or already used up
            return False
        count[c] -= 1
    return True
```

---

## 4. Group Anagrams — defaultdict with a computed key

Teaches `defaultdict(list)` to **bucket items by a derived key** with no existence check.

```python
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)           # a missing key auto-creates an empty list
    for w in strs:
        key = ''.join(sorted(w))         # anagram signature: "eat" -> "aet"
        groups[key].append(w)            # no "if key not in groups" needed
    return list(groups.values())
```

- `sorted(w)` returns a list of characters; `''.join(...)` glues them into a hashable string key.
- With `defaultdict(list)`, `groups[key].append(...)` works even for a brand-new key.
- A faster key (avoids sorting) is a 26-length count tuple: `key = tuple(count of each letter)`.

---

## 5. Top K Frequent — Counter meets the heap

Teaches `Counter.most_common(k)`, a one-liner that hides a heap, plus the bucket-sort alternative.

```python
from collections import Counter

def top_k_frequent(nums, k):
    return [val for val, cnt in Counter(nums).most_common(k)]
    # most_common(k) returns the top-k (value, count) pairs, highest count first;
    # the comprehension unpacks each pair and keeps just the value.
```

O(n log k) bucket-sort version — worth knowing since it is the "can you do better" follow-up:

```python
def top_k_frequent_bucket(nums, k):
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]   # index = frequency
    for val, freq in count.items():
        buckets[freq].append(val)                  # place value at its frequency
    result = []
    for freq in range(len(buckets) - 1, 0, -1):    # walk high freq -> low
        for val in buckets[freq]:
            result.append(val)
            if len(result) == k:
                return result
    return result
```

---

## 6. Product of Array Except Self — the prefix/suffix trick

Teaches an **array-only** technique (no division allowed): build the answer from a left pass and a
right pass. Shows up because it is the same prefix-accumulation idea used later in intervals.

```python
def product_except_self(nums):
    n = len(nums)
    res = [1] * n
    prefix = 1
    for i in range(n):               # res[i] = product of everything to the LEFT
        res[i] = prefix
        prefix *= nums[i]
    suffix = 1
    for i in range(n - 1, -1, -1):   # multiply in the product of everything to the RIGHT
        res[i] *= suffix
        suffix *= nums[i]
    return res
```

- `range(n - 1, -1, -1)` is the Python reverse loop (`for (i = n-1; i >= 0; i--)`).
- Two O(n) passes, O(1) extra space beyond the output. No hash map here — a reminder that "arrays"
  problems are not always hashing problems.

---

## Complexity at a glance

| Operation | `list` | `dict` / `set` |
|---|---|---|
| membership (`x in …`) | O(n) | **O(1)** |
| index access | O(1) | — |
| insert / append | O(1) amortized | O(1) |

The whole pattern is: if you catch yourself writing `x in some_list` inside a loop, that inner
scan is O(n) — move those elements into a `set` or `dict` and the lookup becomes O(1).

---

## One-screen summary

```
seen-so-far map      seen = {}; if need in seen: ...; seen[x] = i     # Two Sum
dedup / membership   s = set(); x in s; s.add(x)                      # Contains Duplicate
frequency map        Counter(v); Counter(a) == Counter(b)            # Valid Anagram
count by hand        d[c] = d.get(c, 0) + 1
bucket by key        g = defaultdict(list); g[key].append(x)         # Group Anagrams
top-k                Counter(v).most_common(k)                       # Top K Frequent
prefix/suffix pass   left pass then right pass, O(1) space           # Product Except Self
reverse loop         for i in range(n-1, -1, -1):
index + value        for i, x in enumerate(v):
```

Read once, then drill from a blank file. The algorithms are simple; the goal is that the Python
idioms (`enumerate`, `Counter`, `defaultdict`, `.get(k, 0)`) come out of your fingers without thinking.
