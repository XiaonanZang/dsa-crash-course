---
title: Graphs I — Traversal & Connected Components
---

# Graphs I — Traversal & Connected Components

This is the interview-day core for a mapping/autonomy role. A **lane or road network is a graph**:
intersections and lane segments are **nodes**, their connections are **edges**. "Can the vehicle
reach B from A?", "how many disconnected sub-networks are in this map?", "which lanes belong to the
same junction?" are all **graph traversal + connected-components** questions. Master representation,
BFS, DFS, and component counting and you cover the bulk of the likely coding round.

Everything here is built on the Arrays & Hashing substrate: an adjacency list is a
`defaultdict(list)`, and `visited` is a `set`.

---

## 0. Representing a graph in Python

There is no built-in graph type — you build one from a `dict`. The **adjacency list** is the default
representation for almost every interview problem (sparse graphs, O(V + E) space).

```python
from collections import defaultdict

# Build an adjacency list from an edge list
edges = [(0, 1), (0, 2), (1, 2), (3, 4)]
graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)      # undirected: add BOTH directions. Directed: this line only for u->v
# graph = {0:[1,2], 1:[0,2], 2:[0,1], 3:[4], 4:[3]}
```

Key choices to state out loud in an interview:

- **Undirected vs directed:** undirected → add the edge both ways (`graph[u].append(v)` *and*
  `graph[v].append(u)`). Directed (a one-way lane) → only `graph[u].append(v)`.
- **Weighted** (distances/costs, needed for Dijkstra later): store `(neighbor, weight)` tuples:
  ```python
  graph[u].append((v, w))
  ```
- **Grid as an implicit graph:** an R×C grid *is* a graph where each cell `(r, c)` is a node and its
  neighbors are the 4 (or 8) adjacent cells. You don't build an adjacency list — you compute
  neighbors on the fly (covered in the Grid page). Same BFS/DFS applies.

```python
# the 4-neighbor move set for grids (up/down/left/right)
DIRS = [(-1, 0), (1, 0), (0, -1), (0, 1)]
for dr, dc in DIRS:
    nr, nc = r + dr, c + dc
    if 0 <= nr < R and 0 <= nc < C:      # bounds check is the #1 grid gotcha
        ...  # (nr, nc) is a neighbor
```

---

## 1. BFS — breadth-first search (queue)

BFS explores level by level from a source. It's the tool for **shortest path in an unweighted
graph** (fewest hops) and for simple reachability. The engine is a **`deque`** (O(1) at both ends —
never a list, whose `pop(0)` is O(n)) plus a `visited` set.

```python
from collections import deque

def bfs(graph, start):
    visited = {start}                 # mark as visited WHEN enqueued, not when dequeued
    queue = deque([start])
    order = []
    while queue:
        node = queue.popleft()        # popleft = front of the queue -> FIFO
        order.append(node)
        for nxt in graph[node]:
            if nxt not in visited:
                visited.add(nxt)      # mark on enqueue -> prevents the same node queued twice
                queue.append(nxt)
    return order
```

**The one gotcha that causes duplicate/​infinite work:** mark a node visited **when you add it to the
queue**, not when you pop it. If you wait until pop, the same node can be enqueued many times before
it's ever processed.

**BFS gives shortest hop-distance for free** — track a distance alongside:

```python
def shortest_hops(graph, start, target):
    visited = {start}
    queue = deque([(start, 0)])       # (node, distance)
    while queue:
        node, dist = queue.popleft()
        if node == target:
            return dist               # first time we reach target = fewest edges
        for nxt in graph[node]:
            if nxt not in visited:
                visited.add(nxt)
                queue.append((nxt, dist + 1))
    return -1                         # unreachable
```

Because BFS expands in rings of increasing distance, the **first** time it reaches a node is along a
shortest (fewest-edge) path. (This only holds for *unweighted* graphs — weighted needs Dijkstra.)

---

## 2. DFS — depth-first search (recursion or explicit stack)

DFS dives as deep as possible before backtracking. It's the tool for **connectivity, cycle
detection, and topological sort** (next page). Two forms:

**Recursive** (clean, but watch Python's ~1000 recursion limit on deep graphs):

```python
def dfs(graph, node, visited):
    visited.add(node)
    for nxt in graph[node]:
        if nxt not in visited:
            dfs(graph, nxt, visited)
```

**Iterative** with an explicit stack (a list) — safe for deep graphs, mirrors what you'd do in C++
to control stack depth:

```python
def dfs_iter(graph, start):
    visited = set()
    stack = [start]
    while stack:
        node = stack.pop()            # pop = top of stack -> LIFO (this is what makes it DFS)
        if node in visited:
            continue
        visited.add(node)             # mark on POP for the iterative form
        for nxt in graph[node]:
            if nxt not in visited:
                stack.append(nxt)
    return visited
```

**BFS vs DFS in one line:** same code shape, the *only* structural difference is the container —
**queue (`popleft`) = BFS**, **stack (`pop`) = DFS**. BFS finds shortest unweighted paths; DFS is
lighter for "is it connected / is there a cycle / order the nodes."

---

## 3. Connected components — the money pattern

> **Problem:** Given `n` nodes labeled `0..n-1` and a list of undirected `edges`, count the number
> of **connected components** (groups of nodes reachable from each other). Example: `n=5`,
> `edges=[[0,1],[1,2],[3,4]]` → `2` (the group `{0,1,2}` and the group `{3,4}`).

This is the direct analog of "how many disconnected road networks are in this map?" The template:
**loop every node; each time you find an unvisited one, that's a new component — flood-fill it so its
whole group gets marked, then move on.**

```python
from collections import defaultdict, deque

def count_components(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    visited = set()
    components = 0
    for node in range(n):             # every node, so isolated nodes count too
        if node not in visited:
            components += 1           # found a new component
            # flood-fill this whole component (BFS shown; DFS works identically)
            queue = deque([node])
            visited.add(node)
            while queue:
                cur = queue.popleft()
                for nxt in graph[cur]:
                    if nxt not in visited:
                        visited.add(nxt)
                        queue.append(nxt)
    return components
```

**Why loop over *all* `n` nodes and not just the edges?** A node with no edges (an isolated
intersection) is still its own component. Iterating `range(n)` catches those; iterating only the edge
list would miss them.

**Same skeleton solves the whole family:**
- *Does a path exist from A to B?* → one BFS/DFS from A, check if B was visited.
- *Number of Islands (grid)* → same loop, but "unvisited land cell" starts a new component and you
  flood-fill its `1`s (Grid page).
- *Size of the largest component* → track the count of nodes marked in each flood-fill, keep the max.

---

## 4. Worked example — Number of Islands (grid = graph)

> **Problem:** Given an R×C grid of `'1'` (land) and `'0'` (water), count the **islands** (groups of
> `'1'`s connected 4-directionally). Example: a grid with two separated blobs of land → `2`.

This is connected-components on an **implicit** grid graph — the exact pattern above, with neighbors
computed instead of stored. It's a very common stand-in for occupancy/BEV-grid questions.

```python
from collections import deque

def num_islands(grid):
    if not grid:
        return 0
    R, C = len(grid), len(grid[0])
    DIRS = [(-1, 0), (1, 0), (0, -1), (0, 1)]
    visited = set()
    islands = 0

    for r in range(R):
        for c in range(C):
            if grid[r][c] == '1' and (r, c) not in visited:
                islands += 1                       # new island
                queue = deque([(r, c)])            # flood-fill it
                visited.add((r, c))
                while queue:
                    cr, cc = queue.popleft()
                    for dr, dc in DIRS:
                        nr, nc = cr + dr, cc + dc
                        if (0 <= nr < R and 0 <= nc < C      # in bounds
                                and grid[nr][nc] == '1'      # is land
                                and (nr, nc) not in visited):
                            visited.add((nr, nc))
                            queue.append((nr, nc))
    return islands
```

Note the coordinate `(r, c)` is stored in the `visited` **set as a tuple** — the exact "tuples are
hashable, lists are not" point from Arrays & Hashing. And the bounds check `0 <= nr < R` before
touching `grid[nr][nc]` is the single most common grid bug — always guard first.

---

## Complexity

For adjacency-list BFS/DFS over V nodes and E edges: **O(V + E) time** (each node and edge visited
once), **O(V) space** (visited set + queue/stack). For an R×C grid: **O(R·C)** — every cell visited
once. This is optimal; you cannot do better than looking at each node/edge once.

---

## One-screen summary

```
represent      graph = defaultdict(list); graph[u].append(v)  (+append(v,u) if undirected)
grid neighbors DIRS = [(-1,0),(1,0),(0,-1),(0,1)]; bounds-check 0<=nr<R and 0<=nc<C FIRST
BFS  (queue)   deque; popleft; mark visited ON ENQUEUE; gives shortest UNWEIGHTED path
DFS  (stack)   list; pop; recursion (raise recursionlimit) or explicit stack for deep graphs
BFS vs DFS     same code, only the container differs: popleft=BFS, pop=DFS
components     for node in range(n): if unvisited -> count += 1; flood-fill the group
path exists?   one traversal from A; was B visited?
visited key    a coordinate goes in the set as a TUPLE (r, c), never a list
complexity     O(V + E)  /  grid O(R*C)
```

Read once, then drill from a blank file. The whole family — components, islands, reachability — is
one skeleton: *loop nodes, flood-fill each unvisited one.*
