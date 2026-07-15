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

## 0. The list — where the data starts

Before any hashing, you need fluency with the `list` — it is Python's dynamic array (`std::vector`),
the type the input arrives in and the one you build most answers into. It also doubles as your
**stack** (`append` / `pop`).

### Creating / initializing

```python
v = []                             # empty
v = [1, 2, 3]                      # literal
v = [0] * n                        # n zeros -> [0,0,0,...]  (init a result/count array)
v = list(range(n))                 # [0, 1, ..., n-1]
v = list(range(1, n + 1))          # [1, 2, ..., n]
grid = [[0] * C for _ in range(R)] # R x C of zeros -- the CORRECT 2D init
```

⚠️ **The 2D trap.** `[[0]*C]*R` makes all R rows *one shared list object*, so an edit to any cell
appears at that column in **every** row:

```python
grid = [[0]*3]*2          # looks like [[0,0,0],[0,0,0]] -- but both rows are ONE list
grid[1][1] = 5
print(grid)               # [[0, 5, 0], [0, 5, 0]]   <- row 0 changed too!
```

The fix is the comprehension form, which runs `[0]*C` **fresh for each row** so the rows are
independent:

```python
grid = [[0]*C for _ in range(R)]   # R independent rows -- always use this for a grid
```

Rule of thumb: `*` is safe only on the **innermost line, for a row of scalars** (`[0]*C`). Every
layer *above* that must be a comprehension.

### 3D and beyond

Same rule, one comprehension per dimension, built inside-out:

```python
row1D  = [0]*H                                          # H
grid2D = [[0]*C for _ in range(R)]                      # R x C     -> grid[r][c]
cube3D = [[[0]*H for _ in range(C)] for _ in range(R)]  # R x C x H -> cube[r][c][h]
```

This is safe because only the innermost `[0]*H` uses `*`; the two outer layers are comprehensions,
so nothing is shared. (`[[[0]*H]*C]*R` would alias *both* the middle and outer levels — never do that.)

### When NOT to allocate a dense grid

By 3D, pause and ask *do I really need a dense array?* If the space is huge but mostly empty, a
`dict` keyed by a tuple is cleaner and cheaper — you allocate only the cells you touch:

```python
seen = {}                          # sparse "grid"
seen[(r, c, h)] = 1                # tuple key -- no R*C*H allocation
if (r, c, h) in seen: ...          # O(1) lookup

visited = set()                    # when you only need presence, not a value
visited.add((r, c))
```

Dense 3D is right for real DP (state = position plus a couple of small parameters); a tuple-keyed
`dict`/`set` is right when the grid is large and sparse. "Does this dimension earn its allocation?"

### Access & slicing

```python
v[0]      v[-1]        # first, last  (negative index counts from the end)
v[i:j]                 # sublist from i up to (not including) j
v[:k]     v[k:]        # first k / everything from index k on
v[::-1]               # reversed copy  (palindrome check, reverse without a loop)
v[::2]                # every other element
```

`v[-1]` for "last element" is the idiom you reach for constantly (it is also your stack `top`).

### Mutating

```python
v.append(x)    # add to end            O(1)   -- also stack push
v.pop()        # remove & return last  O(1)   -- stack pop
v.pop(0)       # remove FIRST          O(n)   <-- avoid in loops; use deque for a queue
v.insert(i, x) # insert at index i     O(n)
v[i] = x       # overwrite in place
v.sort()       # sort in place, ascending
v.reverse()    # reverse in place
```

### Iterating — the three idioms that cover ~90% of loops

```python
for x in v:                  # value only
    ...
for i, x in enumerate(v):    # index AND value -- no C-style for loop needed
    ...
for a, b in zip(A, B):       # two lists in lockstep
    ...
for i in range(len(v)):      # raw index -- only when you need v[i] and v[i+1], etc.
    ...
```

### Built-ins, grouped by what they RETURN

The important distinction is not "does it help me loop" but "what type comes back," because that
decides whether you can use the result directly or must wrap it in `list()`.

```python
# 1. return a SCALAR (one value)
len(v)   sum(v)   min(v)   max(v)      # also any(v), all(v) -> bool

# 2. return a NEW LIST (already materialized -- use, index, print directly)
sorted(v)                              # v unchanged; sorted CANNOT be lazy (must see all first)

# 3. return a LAZY ITERATOR (great for looping; wrap in list() only if you need a real list)
reversed(v)    enumerate(v)    zip(A, B)    map(f, v)    filter(f, v)    range(n)
```

So `reversed` belongs with `enumerate` and `zip` (group 3), **not** with `len/sum/sorted`. All of
group 3 stream elements one at a time:

```python
for x in reversed(v): ...          # no list() needed -- looping consumes the iterator
w = list(reversed(v))              # list() only when you need a standalone list to keep/index
```

⚠️ Iterators are **single-use** — consume one twice and the second time is empty:

```python
z = zip(A, B)
list(z)   # [(a0,b0), ...]
list(z)   # []   <- already exhausted; list() it once and reuse the list
```

### Comprehensions — build a list from another in one line

```python
squares = [x * x for x in v]                  # map: transform every element
evens   = [x for x in v if x % 2 == 0]        # filter: keep some elements
pos_idx = [i for i, x in enumerate(v) if x > 0]  # transform + filter together
flat    = [x for row in grid for x in row]    # flatten a 2D grid
```

Comprehensions replace the "empty list, loop, append" boilerplate — but only reach for one when it
actually **transforms or filters**. If you just want the raw pairs, don't wrap enumerate in a
comprehension that rebuilds the same tuple — call the built-in directly:

```python
[(i, x) for i, x in enumerate(v)]   ==   list(enumerate(v))   # the comprehension does nothing; use the right side
```

### One-screen list cheat

```
init          v=[];  [0]*n;  [[0]*C for _ in range(R)];  list(range(n))
first / last  v[0] / v[-1]
slice         v[i:j]  v[:k]  v[k:]  v[::-1] (reverse)
grow / shrink append / pop  (O(1));  pop(0) / insert  (O(n) -- avoid)
loop          for x in v | for i,x in enumerate(v) | for a,b in zip(A,B)
builtins      len  sum  min  max  sorted  reversed
comprehension [f(x) for x in v if cond]
```

With the list under your fingers, the hash containers below are what turn an O(n²) scan into O(n).

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

### dict — the full operation set

```python
d = {}                      # empty
d = {"a": 1, "b": 2}        # literal
d = dict(a=1, b=2)          # same thing via keyword args

# --- write / read ---
d[key] = val                # insert or overwrite
d[key]                      # read -> val   (raises KeyError if key missing!)
d.get(key)                  # read -> val, or None if missing (no error)
d.get(key, 0)               # read -> val, or 0 if missing  (the safe counting idiom)
d.setdefault(key, [])       # get key, inserting [] first if it was missing

# --- membership / size ---
key in d                    # True/False, O(1)  (checks KEYS, not values)
len(d)                      # number of entries

# --- delete ---
del d[key]                  # remove an entry (KeyError if missing)
d.pop(key, None)            # remove & return val, or None if missing (safe)

# --- update / merge ---
d.update({"c": 3})          # add/overwrite from another dict
```

### The three iteration views (`.keys()` / `.values()` / `.items()`)

A dict stores key→val entries; how you loop it decides what you get back:

```python
d = {"a": 1, "b": 2, "c": 3}

for k in d:            # iterating a dict directly gives KEYS
    ...                #   k = "a", "b", "c"
for k in d.keys():     # same as above, explicit
    ...
for v in d.values():   # just the VALUES
    ...                #   v = 1, 2, 3
for k, v in d.items(): # (key, val) TUPLES  <- unpack both at once
    print(k, v)        #   "a" 1  /  "b" 2  /  "c" 3
```

`.items()` is the one you reach for most: it hands each entry back as a `(key, val)` tuple, and
`for k, v in d.items()` unpacks it in place. Turning a dict into a list of pairs is just:

```python
list(d.items())        # [("a", 1), ("b", 2), ("c", 3)]
list(d.keys())         # ["a", "b", "c"]
list(d.values())       # [1, 2, 3]
```

Common uses: sort entries by value with `sorted(d.items(), key=lambda kv: kv[1])`; find the key with
the max value with `max(d, key=d.get)`.

⚠️ Do **not** add or delete keys *while* looping `d.items()` — it raises "dictionary changed size
during iteration." If you must, loop over a snapshot: `for k, v in list(d.items()):`.

### set — the full operation set

A `set` stores **unique, unordered, hashable** elements with O(1) add / membership / remove. It is a
`dict` with keys but no values.

```python
s = set()                   # empty  (NOTE: {} is an empty DICT, not a set)
s = {1, 2, 3}               # literal
s = set([1, 2, 2, 3])       # from a list -> {1, 2, 3}  (dedups automatically)

# --- add / remove ---
s.add(x)                    # insert; returns None; duplicate = silent no-op (no error)
s.discard(x)                # remove if present; does NOTHING if missing (safe)
s.remove(x)                 # remove; raises KeyError if x is missing
s.pop()                     # remove & return an ARBITRARY element (unordered)

# --- query ---
x in s                      # O(1) membership
len(s)                      # size
```

**Elements must be hashable (immutable):** numbers, strings, tuples are OK; a `list` is not.
Use a tuple for a coordinate: `visited.add((r, c))`.

Because `add()` gives no signal on a duplicate, test first when you need to detect one:

```python
if x in seen: ...           # ask before adding
seen.add(x)
# or by size:
before = len(s); s.add(x); is_new = len(s) > before
```

**Set algebra** (great for "common / missing / distinct" questions):

```python
a | b      # union         -- in a OR b
a & b      # intersection  -- in a AND b   (e.g. common elements of two arrays)
a - b      # difference    -- in a but NOT b
a ^ b      # symmetric diff -- in exactly one of them
list(set(nums))   # dedup a list in one line
```

---

## 1. Two Sum — the "seen so far" map

> **Problem:** Given an integer array `nums` and an integer `target`, return the **indices** of the
> two numbers that add up to `target`. Exactly one solution exists, and you may not use the same
> element twice. Example: `nums=[2,7,11,15], target=9` → `[0, 1]` (because `2 + 7 = 9`).

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

> **Problem:** Given an integer array `nums`, return `True` if any value appears **at least twice**,
> and `False` if every element is distinct. Example: `[1,2,3,1]` → `True`; `[1,2,3,4]` → `False`.

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

> **Problem:** Given two strings `s` and `t`, return `True` if `t` is an **anagram** of `s` (same
> characters with the same counts, reordered). Example: `s="anagram", t="nagaram"` → `True`;
> `s="rat", t="car"` → `False`.

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

> **Problem:** Given a list of strings `strs`, group the ones that are anagrams of each other.
> Return a list of groups (order does not matter). Example: `["eat","tea","tan","ate","nat","bat"]`
> → `[["eat","tea","ate"], ["tan","nat"], ["bat"]]`.

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

- **Why `''.join(sorted(w))` and not just `sorted(w)`?** `sorted(w)` returns a **list**, and lists
  are **unhashable** — they cannot be dict keys or set elements (`groups[['a','e','t']]` → `TypeError:
  unhashable type: 'list'`). A key must be immutable/hashable, so wrap the sorted characters into a
  hashable type first. Two valid keys:
  ```python
  key = ''.join(sorted(w))    # -> "aet"          (string -- hashable)  ✔
  key = tuple(sorted(w))      # -> ('a','e','t')  (tuple  -- hashable)  ✔ also fine
  ```
  All anagrams collapse to the same signature (`"eat"`, `"tea"`, `"ate"` → `"aet"`). Same reason a
  coordinate goes into a set as a `(r, c)` **tuple**, never a `[r, c]` list.
- With `defaultdict(list)`, `groups[key].append(...)` works even for a brand-new key.
- A faster key (avoids sorting) is a 26-length count tuple: `key = tuple(count of each letter)`.

---

## 5. Top K Frequent — Counter meets the heap

> **Problem:** Given an integer array `nums` and an integer `k`, return the `k` **most frequent**
> elements (any order). Example: `nums=[1,1,1,2,2,3], k=2` → `[1, 2]`.

Teaches the **two-step shape** of "top k" problems — (1) **count** with a plain map, (2) **rank**
by frequency — and the **bucket** trick that makes the ranking O(n) instead of O(n log k).

**Primary version — plain `dict` + bucket, O(n).** Nothing but a map, a list of buckets, and two
loops. This is the one to be able to write cold:

```python
def top_k_frequent(nums, k):
    # 1. count: plain map  value -> frequency
    count = {}
    for x in nums:
        count[x] = count.get(x, 0) + 1              # get(x, 0) = current count or zero

    # 2. bucket by frequency: index = frequency (a frequency can be at most n)
    buckets = [[] for _ in range(len(nums) + 1)]
    for val, freq in count.items():
        buckets[freq].append(val)                   # file each value under its frequency

    # 3. walk high frequency -> low, take the first k
    result = []
    for freq in range(len(buckets) - 1, 0, -1):
        for val in buckets[freq]:
            result.append(val)
            if len(result) == k:
                return result
    return result
```

Why it is O(n): counting is O(n), bucketing places each value in one pass, and walking the buckets
is O(n) — no sort, no heap, so no `log` factor. The trick works because the sort key (frequency) is
a bounded integer ≤ n, so you can *index* by it instead of *comparing*.

**Shorthand version — `Counter.most_common(k)`, O(n log k).** Same idea, but `most_common` uses a
heap internally, adding the `log k`. One line; perfectly acceptable in an interview — reach for the
bucket version only when asked "can you do strictly O(n)?":

```python
from collections import Counter

def top_k_frequent_short(nums, k):
    return [val for val, cnt in Counter(nums).most_common(k)]
    # Counter(nums) == the plain count loop above; most_common(k) does the ranking (heap).
```

---

## 6. Product of Array Except Self — the prefix/suffix trick

> **Problem:** Given an integer array `nums`, return an array `res` where `res[i]` equals the
> **product of every element except `nums[i]`** — computed **without division** and in O(n).
> Example: `nums=[1,2,3,4]` → `[24, 12, 8, 6]`.

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

## 7. Sparse Matrix — dict-of-keys (construct / add / multiply)

> **Problem:** A large matrix is **mostly zeros**. Store it efficiently, then support **addition** and
> **multiplication** of two such matrices. Return results in the same compact form.

The payoff of Section 0's tuple-keyed dict: represent an `m×n` grid that's 99% zeros as a
`{(row, col): value}` map storing **only the nonzeros** — memory becomes O(nonzeros), not O(m·n). It's
a pure data-structure problem: the storage choice cascades into how clean add and multiply are.
Sparse matrices are everywhere in this domain — an **adjacency matrix is a sparse matrix**, and so are
ML feature matrices.

**Construct — the map:**
```python
def to_sparse(matrix):
    data = {}                                  # {(r, c): value} -- nonzeros only
    for r, row in enumerate(matrix):
        for c, val in enumerate(row):
            if val != 0:
                data[(r, c)] = val
    return data
```

**Add — iterate the UNION of nonzero cells:**
```python
def add(A, B):                                 # A, B same dimensions
    result = {}
    for key in A.keys() | B.keys():            # only cells nonzero in AT LEAST one matrix
        s = A.get(key, 0) + B.get(key, 0)
        if s != 0:                             # a sum can CANCEL to zero -> don't store it
            result[key] = s
    return result
```
The subtle correctness point most people miss: **drop cells that sum to zero**, or your "sparse" matrix
silently fills with explicit zeros.

**Multiply — align A's columns to B's rows (the crux):**
```python
from collections import defaultdict

def multiply(A, B):                            # A is m×k, B is k×n
    B_by_row = defaultdict(list)               # index B by its row: {x: [(j, val), ...]}
    for (x, j), v in B.items():
        B_by_row[x].append((j, v))

    result = defaultdict(int)
    for (i, x), a in A.items():                # each nonzero A[i][x]
        for (j, b) in B_by_row[x]:             # only B-entries in the MATCHING row x
            result[(i, j)] += a * b            # accumulate the sum-of-products
    return {k: v for k, v in result.items() if v != 0}
```
`A[i][x] * B[x][j]` is nonzero only when both exist **and share the index `x`** (A's column = B's row).
Grouping B by row `x` turns "find all B-entries that pair with this A-entry" into an O(1) dict lookup, so
you only ever multiply pairs that actually contribute — never touching a zero. Same drop-the-zeros
discipline on the way out. This is the `scipy.sparse` idea (dict-of-keys / CSR) in miniature.

**Class form — bundle state with the ops:**

The three functions share the same `data` map, so the natural next step is a class. State (`data` + shape)
travels with the object, add/multiply return a properly-shaped `SparseMatrix`, and the drop-zeros rule now
lives in the `set` mutator too.

```python
from collections import defaultdict

class SparseMatrix:
    def __init__(self, matrix=None, rows=0, cols=0):
        self.data = {}                       # {(r, c): value} -- nonzeros only
        if matrix is not None:               # build from a dense 2D list
            self.rows = len(matrix)
            self.cols = len(matrix[0]) if matrix else 0
            for r, row in enumerate(matrix):
                for c, val in enumerate(row):
                    if val != 0:
                        self.data[(r, c)] = val
        else:                                # or start empty with known shape
            self.rows, self.cols = rows, cols

    def get(self, r, c):
        return self.data.get((r, c), 0)      # missing key == 0

    def set(self, r, c, val):
        if val != 0:
            self.data[(r, c)] = val
        else:
            self.data.pop((r, c), None)      # setting to 0 must REMOVE, not store a 0

    def add(self, other):                    # same dimensions
        out = SparseMatrix(rows=self.rows, cols=self.cols)
        for key in self.data.keys() | other.data.keys():   # union of nonzero cells
            s = self.get(*key) + other.get(*key)
            if s != 0:                       # a sum can CANCEL to zero -> don't store it
                out.data[key] = s
        return out

    def multiply(self, other):               # self is m×k, other is k×n
        out = SparseMatrix(rows=self.rows, cols=other.cols)
        B_by_row = defaultdict(list)         # index other by its row: {x: [(j, val), ...]}
        for (x, j), v in other.data.items():
            B_by_row[x].append((j, v))
        acc = defaultdict(int)
        for (i, x), a in self.data.items():  # each nonzero A[i][x]
            for (j, b) in B_by_row[x]:       # only B-entries in the MATCHING row x
                acc[(i, j)] += a * b
        out.data = {k: v for k, v in acc.items() if v != 0}
        return out

    def to_dense(self):                      # back to dense for printing / tests
        grid = [[0] * self.cols for _ in range(self.rows)]
        for (r, c), v in self.data.items():
            grid[r][c] = v
        return grid
```

Two things the class buys you: `set(r, c, 0)` must **`pop`** the key (not store a zero — same drop-zeros
discipline, now on the mutator), and `self.get(*key)` **star-unpacks** the `(r, c)` tuple into two args
(`self.get(key)` would pass the tuple whole and break). Keep add/multiply **pure** (return a new matrix,
mutate nothing); offer an in-place variant only as the follow-up optimization.

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
sparse matrix        {(r,c): v} nonzeros; add=union+drop-zeros; mul=index B by row  # dict-of-keys
reverse loop         for i in range(n-1, -1, -1):
index + value        for i, x in enumerate(v):
```

Read once, then drill from a blank file. The algorithms are simple; the goal is that the Python
idioms (`enumerate`, `Counter`, `defaultdict`, `.get(k, 0)`) come out of your fingers without thinking.
