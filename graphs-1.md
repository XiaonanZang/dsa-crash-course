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
  neighbors are the 4 (or 8) adjacent cells. You don't build an adjacency list — the neighbors are
  computed on the fly with a small move-set (see the Number of Islands example below, and the Grid
  page). Same BFS/DFS applies.

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

### Concrete trace — why marking on *pop* breaks

Take a **hub** node `4` reachable from three predecessors `1, 2, 3`, all reachable from source `0`:

```python
graph = {0: [1, 2, 3], 1: [0, 4], 2: [0, 4], 3: [0, 4], 4: [1, 2, 3]}
```

**BUGGY version — `visited.add(node)` happens on POP, so the enqueue test is just
`if nxt not in visited`:**

```
pop 0  -> mark {0}          ; look at 1,2,3 (none visited) -> enqueue 1,2,3   queue=[1,2,3]
pop 1  -> mark {0,1}        ; look at 0(visited), 4(NOT visited*) -> enqueue 4  queue=[2,3,4]
pop 2  -> mark {0,1,2}      ; look at 0(visited), 4(STILL not visited*) -> enqueue 4  queue=[3,4,4]
pop 3  -> mark {0,1,2,3}    ; look at 0(visited), 4(STILL not visited*) -> enqueue 4  queue=[4,4,4]
pop 4  -> mark {..,4}       ; process 4        queue=[4,4]
pop 4  -> already visited   ; wasted work      queue=[4]
pop 4  -> already visited   ; wasted work      queue=[]
```

`*` **This is the bug.** Node `4` is sitting *in the queue* but not yet *in `visited`* (visited only
updates on pop). So when `2` and `3` look at `4`, it still reads as "undiscovered" and they enqueue
it **again** — once per predecessor. `4` ends up in the queue **3 times**.

**CORRECT version — `visited.add(nxt)` on ENQUEUE (the code above):**

```
pop 0  -> look at 1,2,3 -> mark+enqueue all      visited={0,1,2,3}  queue=[1,2,3]
pop 1  -> look at 4 (not visited) -> mark+enqueue 4   visited={0,1,2,3,4}  queue=[2,3,4]
pop 2  -> look at 4 -> ALREADY visited -> skip    queue=[3,4]
pop 3  -> look at 4 -> ALREADY visited -> skip    queue=[4]
pop 4  -> process                                  queue=[]
```

The instant node `1` enqueues `4`, it marks `4` visited — so `2` and `3` see it's already discovered
and skip it. `4` is enqueued **exactly once**.

**Why it matters:** a node gets re-enqueued once per predecessor, so on a dense graph that's up to
**O(E) duplicate entries** — wasted time and memory, and without the "already visited" guard on pop
it degenerates into reprocessing. The rule in one line: **a node is discovered the moment it enters
the queue, so mark it there.**

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

## 2. DFS — depth-first search (iterative stack)

DFS dives as deep as possible before backtracking. It's the tool for **connectivity** and (with an
ordering tweak) cycle detection. We use the **iterative** form with an explicit stack — no recursion,
no recursion-limit worries, and it uses the **exact same discipline as BFS**: mark visited when you
**add** to the container.

```python
def dfs_iter(graph, start):
    visited = {start}                 # mark the start; then mark each node WHEN PUSHED
    stack = [start]
    order = []
    while stack:
        node = stack.pop()            # pop = top of stack -> LIFO (this is what makes it DFS)
        order.append(node)
        for nxt in graph[node]:
            if nxt not in visited:
                visited.add(nxt)      # mark on PUSH -> a shared child is claimed once, no duplicates
                stack.append(nxt)
    return order
```

Compare it line-for-line with `bfs()` above — they are **identical except for two things**:

| | BFS | DFS |
|---|---|---|
| container | `deque` | `list` (stack) |
| take from | `popleft()` (front, FIFO) | `pop()` (top, LIFO) |
| mark visited | on **enqueue** | on **push** |

So the single rule to memorize is: **mark visited the moment you add a node to the container.** Swap
the queue for a stack and you've turned BFS into DFS. Nothing else changes.

### Why `visited` exists at all (and when to mark)

`visited` is needed *only* because a node can be reached **more than once** — a **shared child**
(multiple parents) or a **cycle**. Take those away and you don't need it:

- **Tree** → every node has exactly one parent, no cycles → **no `visited` needed.** A queue (BFS) or
  stack (DFS) alone traverses it correctly.
- **Graph** → nodes converge and loop → without `visited`, shared children get re-expanded and cycles
  loop forever. `visited` is exactly what upgrades tree traversal into *graph* traversal.

And you mark **when you add** (enqueue/push) so the *first* path to reach a shared child claims it
immediately — every later parent then sees "already visited" and skips it, so nothing enters the
container twice.

> **Note on recursion:** DFS is often written recursively (`visited.add(node)` as the first line of a
> `dfs(node)` function). It's shorter, but risks Python's ~1000-deep recursion limit and buries the
> stack. For interviews focused on modeling + traversal, the iterative form above is the safe default
> — reach for recursion only if a problem is much cleaner with it.

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
DFS  (stack)   list;  pop;     mark visited ON PUSH   (iterative, no recursion needed)
BFS vs DFS     SAME code; mark-visited-on-ADD for both; only the container differs (popleft=BFS, pop=DFS)
why visited    only needed for shared children / cycles; a tree needs none
components     for node in range(n): if unvisited -> count += 1; flood-fill the group
path exists?   one traversal from A; was B visited?
visited key    a coordinate goes in the set as a TUPLE (r, c), never a list
complexity     O(V + E)  /  grid O(R*C)
```

Read once, then drill from a blank file. The whole family — components, islands, reachability — is
one skeleton: *loop nodes, flood-fill each unvisited one.*
