---
title: Topological Sort
---

# Topological Sort

Topological sort answers: **"given a set of things with dependencies, is there a valid order to do
them in, and what is it?"** For a mapping/autonomy role this is the tool for **lane/intersection
dependency ordering** and, just as importantly, **cycle detection** — "is this lane-connectivity
graph consistent (a DAG), or does it contain a contradictory cycle?" It builds directly on the BFS/DFS
traversal from Graphs I.

It applies to one specific graph type: a **DAG — Directed Acyclic Graph**. Directed (dependencies
have a direction: A must come before B) and acyclic (no cycles — a cycle means no valid order exists).

```
edge  u -> v   means "u must come before v"
A topological order is any linear ordering where every edge points FORWARD.
If the graph has a cycle, NO topological order exists.
```

There are two standard algorithms. Know **Kahn's (BFS + in-degree)** cold — it's the most common and
gives cycle detection for free. Know the **DFS** version as the alternative.

---

## Key concept — in-degree

The **in-degree** of a node = how many edges point *into* it = how many prerequisites it has. A node
with in-degree 0 has no unmet prerequisites, so it can go **first**. That single idea drives Kahn's
algorithm.

```python
# in-degree of every node, from a directed edge list  u -> v
from collections import defaultdict, deque

graph = defaultdict(list)     # u -> list of nodes that depend on u
indeg = defaultdict(int)      # node -> number of prerequisites
for u, v in edges:            # edge u -> v : u before v
    graph[u].append(v)
    indeg[v] += 1             # v has one more prerequisite
```

---

## 1. Kahn's algorithm (BFS + in-degree) — the one to know cold

Repeatedly take a node with **in-degree 0** (no remaining prerequisites), append it to the order, and
"remove" it by decrementing its neighbors' in-degrees. When a neighbor's in-degree hits 0, it's now
free — enqueue it.

```python
from collections import defaultdict, deque

def topo_sort(n, edges):
    graph = defaultdict(list)
    indeg = [0] * n                       # nodes labeled 0..n-1
    for u, v in edges:                    # u must come before v
        graph[u].append(v)
        indeg[v] += 1

    queue = deque([i for i in range(n) if indeg[i] == 0])   # all zero-prereq nodes start
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for nxt in graph[node]:
            indeg[nxt] -= 1               # "remove" node -> nxt loses a prerequisite
            if indeg[nxt] == 0:           # nxt now has no prerequisites left
                queue.append(nxt)

    # CYCLE CHECK: if we couldn't place every node, a cycle blocked some.
    return order if len(order) == n else []   # [] = no valid order (cycle exists)
```

**The cycle check is the elegant part:** if the graph is a DAG, every node eventually reaches
in-degree 0 and lands in `order`, so `len(order) == n`. If there's a cycle, the nodes in that cycle
can *never* reach in-degree 0 (they forever wait on each other), so they never get enqueued and
`len(order) < n`. **A short order = a cycle.** You get cycle detection with no extra code.

### If nodes aren't labeled `0..n-1` — use a dict

`indeg = [0] * n` is a shortcut that works **only because the nodes are integers `0..n-1`** (the
label *is* the index). It quietly does double duty: it counts in-degrees **and** `range(n)` enumerates
every node, including the roots that have no incoming edges. For arbitrary labels (strings, sparse
ids) switch to a `defaultdict(int)` — but then you must supply the **full node set** separately:

```python
from collections import defaultdict, deque

def topo_sort(nodes, edges):          # nodes = the complete set of labels
    graph = defaultdict(list)
    indeg = defaultdict(int)
    for u, v in edges:                # edge u -> v
        graph[u].append(v)
        indeg[v] += 1

    # seed with EVERY node whose in-degree is 0 -- scan all nodes, not indeg's keys:
    # a source-only root never appears as a `v`, so it's NOT a key in indeg
    queue = deque([x for x in nodes if indeg[x] == 0])
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for nxt in graph[node]:
            indeg[nxt] -= 1
            if indeg[nxt] == 0:
                queue.append(nxt)

    return order if len(order) == len(nodes) else []   # cycle check now vs len(nodes)
```

Two watch-outs: the cycle check becomes **`len(order) == len(nodes)`** (no integer `n` anymore), and
`indeg[x] == 0` on a `defaultdict(int)` *inserts* `x` as a side effect — use `indeg.get(x, 0)` if you
want to avoid that. So: **integer nodes `0..n-1` → `[0]*n`; arbitrary labels → `defaultdict(int)` plus
a full node set.** The list is just the special case where the labels are the indices.

---

## 2. Worked example — Course Schedule II

> **Problem:** `numCourses` courses labeled `0..numCourses-1`. `prerequisites[i] = [a, b]` means you
> must take `b` before `a`. Return **an order** you can take all courses in, or `[]` if impossible
> (a cycle). Example: `numCourses=4`, `prereqs=[[1,0],[2,0],[3,1],[3,2]]` → `[0,1,2,3]` (or `[0,2,1,3]`).

This is a direct application of Kahn's. The only care point is edge direction: `[a, b]` means
`b -> a` (b before a).

```python
from collections import defaultdict, deque

def find_order(numCourses, prerequisites):
    graph = defaultdict(list)
    indeg = [0] * numCourses
    for a, b in prerequisites:            # [a, b] : b BEFORE a  -> edge b -> a
        graph[b].append(a)
        indeg[a] += 1

    queue = deque([c for c in range(numCourses) if indeg[c] == 0])
    order = []
    while queue:
        course = queue.popleft()
        order.append(course)
        for nxt in graph[course]:
            indeg[nxt] -= 1
            if indeg[nxt] == 0:
                queue.append(nxt)

    return order if len(order) == numCourses else []
```

**The sibling problem, "Course Schedule I"** ("can you finish all courses?") is the *same code*
returning a bool: `return len(order) == numCourses`. Whether a valid order exists **is** the cycle
question.

---

## 3. The DFS version (post-order + reverse)

The alternative: DFS from each node, and record a node **after** all its descendants are done
(post-order). Reverse that post-order and you have a topological order. Cycle detection needs an
explicit "currently on the recursion stack" marker.

```python
def topo_sort_dfs(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)

    WHITE, GRAY, BLACK = 0, 1, 2          # unvisited / in-progress / done
    color = [WHITE] * n
    order = []
    has_cycle = False

    def dfs(node):
        nonlocal has_cycle
        color[node] = GRAY                # GRAY = on the current DFS path
        for nxt in graph[node]:
            if color[nxt] == GRAY:        # back-edge to a node on our path = CYCLE
                has_cycle = True
            elif color[nxt] == WHITE:
                dfs(nxt)
        color[node] = BLACK               # done -> record in post-order
        order.append(node)

    for i in range(n):
        if color[i] == WHITE:
            dfs(i)

    return order[::-1] if not has_cycle else []   # reverse post-order = topo order
```

**The GRAY/BLACK distinction is the cycle test:** GRAY means "on the path I'm currently exploring."
If DFS hits a GRAY node, we've looped back onto our own path — a cycle. A BLACK node is just a
finished branch (fine to re-encounter, not a cycle). This three-color scheme is the standard way to
detect a cycle in a **directed** graph.

**Which to use:** Kahn's (BFS) is usually the cleaner interview answer — iterative (no recursion-limit
worry) and the cycle check is just `len(order) == n`. Reach for DFS if you're already doing a DFS
pass or the problem is phrased around ordering-after-completion.

---

## Complexity

Both algorithms are **O(V + E)** time (each node and edge processed once) and **O(V + E)** space
(graph + in-degree/queue or recursion stack). Optimal — you must look at every dependency at least
once.

---

## One-screen summary

```
applies to     a DAG (directed + acyclic). A cycle => NO valid order exists.
edge u->v      "u must come before v"
in-degree      indeg[v] = # of prerequisites of v; indeg 0 => can go now

KAHN (BFS, know this cold):
  build graph + indeg; queue all indeg==0 nodes
  pop -> append to order -> for each nbr: indeg[nbr]-=1; if 0: enqueue
  cycle check: len(order) == n ? valid : cycle exists

DFS (alternative):
  post-order (append after recursing children), then REVERSE
  cycle detect: WHITE/GRAY/BLACK; hitting a GRAY node = cycle

course schedule  I ("can finish?") = bool len(order)==n ;  II ("give order") = order itself
complexity     O(V + E)
```

Read once, then drill from a blank file. The whole pattern is: *keep taking a no-prerequisite node,
remove it, repeat — and if you get stuck before placing everyone, that's your cycle.*
