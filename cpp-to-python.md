---
title: C++ → Python cheat sheet (DS&A)
---

# C++ → Python cheat sheet for DS&A

If your DS&A muscle memory is C++ (`vector`, `unordered_map`, raw `for` loops, recursion), the
algorithms transfer directly. What changes is syntax and a handful of idioms. This page is the
bridge: your C++ construct on the left, the Python you write instead on the right.

---

## 1. Containers

| Need | C++ | Python |
|---|---|---|
| Dynamic array | `vector<int> v;` | `v = []` |
| 2D array | `vector<vector<int>> g(R, vector<int>(C, 0));` | `g = [[0]*C for _ in range(R)]` |
| Hash map | `unordered_map<int,int> m;` | `m = {}` or `m = defaultdict(int)` |
| Hash set | `unordered_set<int> s;` | `s = set()` |
| Stack | `stack<int> st;` (`push`/`pop`/`top`) | `st = []` (`append` / `pop` / `st[-1]`) |
| Queue | `queue<int> q;` (`push`/`pop`/`front`) | `from collections import deque; q = deque()` (`append` / `popleft`) |
| Min-heap | `priority_queue<int,…,greater<int>>` | `import heapq; heap = []` |
| Pair | `pair<int,int> p;` | `p = (a, b)` (tuple) |
| Ordered map | `map<int,int>` | plain `dict` (keeps insertion order); use `sorted()` when you need key order |

**The single biggest gotcha:** a Python `list` is your `vector`, and it doubles as your **stack**
(`append`/`pop`). But do **not** use a list as a queue — `list.pop(0)` is O(n). Use
`collections.deque` with `popleft()` for O(1) queue behavior. This matters in BFS.

---

## 2. The 2D-array trap

```python
# WRONG — every row is the SAME list object (aliasing bug)
grid = [[0] * C] * R          # editing grid[0][0] changes all rows

# RIGHT — a fresh row each time
grid = [[0] * C for _ in range(R)]
```
This is the most common Python bug for people coming from C++, where `vector<vector<int>>` never
aliases. Always use the comprehension form for a 2D grid.

---

## 3. Loops

| C++ | Python |
|---|---|
| `for (int i = 0; i < n; i++)` | `for i in range(n):` |
| `for (int i = n-1; i >= 0; i--)` | `for i in range(n-1, -1, -1):` |
| `for (auto x : v)` | `for x in v:` |
| index **and** value | `for i, x in enumerate(v):` |
| two lists together | `for a, b in zip(A, B):` |
| `while (cond)` | `while cond:` |
| `do { … } while (cond);` | no `do-while`: use `while True: … if not cond: break` |

Prefer `for x in v` and `enumerate` over C-style index loops. You reach for a raw `range(n)` index
only when you actually need the index (e.g., comparing `v[i]` and `v[i+1]`).

---

## 4. Hash maps: `dict` vs `defaultdict` vs `Counter`

```python
from collections import defaultdict, Counter

# frequency count — three ways, pick Counter
freq = {}
for x in v: freq[x] = freq.get(x, 0) + 1     # plain dict, .get(key, default)

freq = defaultdict(int)
for x in v: freq[x] += 1                       # auto-zero on first touch

freq = Counter(v)                              # one line; also freq.most_common(k)

# adjacency list (graphs) — defaultdict(list) is the idiom
graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)                         # undirected
```
- `d.get(k, default)` reads without inserting (C++ `count()` then `[]`).
- `defaultdict(int)` for counting, `defaultdict(list)` for adjacency lists.
- `Counter` for frequency; `Counter(v).most_common(k)` gives the top-k directly.

---

## 5. Heap = `heapq` (min-heap only)

```python
import heapq
heap = []
heapq.heappush(heap, x)        # push
smallest = heapq.heappop(heap) # pop the MINIMUM
heap[0]                        # peek min without popping

# MAX-heap: negate on the way in and out
heapq.heappush(heap, -x)
largest = -heapq.heappop(heap)

# heap of tuples sorts by the first element (then second, …)
heapq.heappush(heap, (dist, node))   # Dijkstra: (distance, node)
```
Python only has a **min-heap**. For a max-heap, push negatives. For "top-k largest," a min-heap of
size k is the standard trick (`heapq.nlargest(k, v)` also works).

---

## 6. Sorting with a comparator

```python
v.sort()                                 # in place, ascending
v.sort(reverse=True)                     # descending
w = sorted(v)                            # returns a new list

# sort by a key (C++ comparator → Python key function)
pts.sort(key=lambda p: p[0])             # by first coordinate
pts.sort(key=lambda p: (p[0], -p[1]))    # by x asc, then y desc
words.sort(key=len)                      # by length
```
In C++ you write a comparator returning bool; in Python you give a **key function** that maps each
element to a sort value. Tuples sort lexicographically, so `key=lambda p:(a,b)` sorts by `a` then `b`.

---

## 7. Strings (the immutability gotcha)

```python
s = "hello"
s[0]                     # 'h'  — indexing is fine
# s[0] = 'H'             # ERROR — strings are IMMUTABLE (unlike C++ std::string)

# to modify: convert to list, edit, join back
chars = list(s)
chars[0] = 'H'
s = ''.join(chars)

# build a string in a loop — join a list, do NOT += in a loop (O(n^2))
parts = []
for x in items: parts.append(str(x))
result = ''.join(parts)
```
Python strings are immutable, so in-place edits and repeated `+=` are traps. Collect into a list
and `''.join(...)` once.

---

## 8. Sentinels, swap, ternary, misc

| Need | C++ | Python |
|---|---|---|
| Infinity | `INT_MAX` | `float('inf')` (and `float('-inf')`) |
| Swap | `swap(a, b);` | `a, b = b, a` |
| Ternary | `cond ? a : b` | `a if cond else b` |
| Integer division | `a / b` (ints) | `a // b` (`/` gives a float in Python) |
| Char to code | `(int)c` | `ord(c)` / `chr(n)` |
| Min/max of two | `min(a,b)` | `min(a, b)` (same; also `min(list)`) |

---

## 9. Recursion

Recursion is identical in spirit, with one gotcha: Python's default recursion limit is ~1000.

```python
import sys
sys.setrecursionlimit(10**6)   # raise it for deep DFS on large inputs

def dfs(node, visited):
    visited.add(node)
    for nxt in graph[node]:
        if nxt not in visited:
            dfs(nxt, visited)
```
For very deep graphs, prefer an **iterative** DFS with an explicit stack (a list) to avoid hitting
the limit — same as you would reason about stack depth in C++.

---

## 10. The idiom slips that bite under a clock

These aren't algorithm mistakes, they're the Python-idiom slips that surface when you write cold with
your hands cold. They cost real minutes chasing a `TypeError` when the approach was already right.
Read them, then watch for them the moment you drill.

**Call vs subscript — methods need `()`, not `[]`.**
```python
q.popleft()          # RIGHT — it's a method call
q.popleft            # wrong — this is the method OBJECT, you never called it
lst.append(x)        # RIGHT
lst.append[x]        # wrong — TypeError: 'builtin_function_or_method' is not subscriptable
```
`[]` means "index into"; `()` means "call". A method is called. When in doubt: does this *do* something? Then it needs `()`.

**Container init vs append — `deque()`/`list()`/`extend()` ITERATE their argument; `append()` takes one element.**
```python
deque([(r, c)])      # RIGHT — a list holding one tuple -> deque has one coordinate
deque((r, c))        # wrong — iterates the tuple -> deque holds two ints r and c
q.append((nr, nc))   # RIGHT — the tuple goes in whole, as one element
q.append([(nr, nc)]) # wrong — a list-wrapping-a-tuple goes in; next popleft can't unpack it
```
Rule: **if a call iterates its arg (`deque`, `list`, `extend`, `set`), wrap your item so iteration yields exactly it. If it takes one element (`append`, `push`), pass it bare.**

**Iterate values vs indices — `for x in arr` gives VALUES, not positions.**
```python
for i in range(len(arr)):        # RIGHT when you need the index i
    if arr[i] == 0: q.append(i)
for i, v in enumerate(arr):      # RIGHT — index AND value
    if v == 0: q.append(i)
for x in arr:                    # gives the VALUE x; appending x is NOT the node id
    if x == 0: q.append(x)       # bug: seeds the value 0, not the position
```
When you're seeding a queue/visited with **node ids** (in-degree-0 nodes, grid cells), you need the **index**. Reach for `enumerate` or `range(len(...))`, not a bare `for x in arr`.

**Char grids hold strings, not ints.**
```python
if grid[r][c] == "1":     # RIGHT — LeetCode grids are often chars '1'/'0'
if grid[r][c] == 1:       # wrong — silently always False; your whole scan finds nothing
```

**Bounds are inclusive-zero — use `>= 0`, not `> 0`.**
```python
if 0 <= nr < R and 0 <= nc < C:    # RIGHT — row 0 / col 0 are valid
if nr > 0 and nc > 0:              # wrong — drops the entire first row and first column
```

*(These are the exact slips that recur across cold drills. The algorithm is rarely the problem; this
list is. One read-through before you drill turns 4-bug runs into 1-bug runs.)*

---

## 11. The one-screen summary

```
vector           -> list                      []           append / pop / [-1]
2D vector        -> [[0]*C for _ in range(R)]  (never [[0]*C]*R)
unordered_map    -> dict / defaultdict(int)    d.get(k, default)
unordered_set    -> set()                      x in s  is O(1)
stack            -> list                       append / pop / st[-1]
queue (BFS)      -> collections.deque          append / popleft   (NOT list.pop(0))
priority_queue   -> heapq (min-heap)           push -x for max-heap
pair             -> tuple (a, b)
sort(cmp)        -> sorted(key=lambda ...)      tuples sort lexicographically
INT_MAX          -> float('inf')
string edit      -> list(s) ... ''.join(chars) (strings are immutable)
```

Keep this open while you drill. The algorithms are the ones you already know; this page is just the
translation layer.
