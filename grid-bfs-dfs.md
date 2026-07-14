---
title: Grid BFS / DFS
---

# Grid BFS / DFS

An R×C grid **is a graph** — every cell `(r, c)` is a node, and its neighbors are the adjacent cells.
For a mapping/autonomy role this is the **BEV / occupancy-grid** pattern: flood-filling regions,
counting connected areas, shortest path across a costmap, spreading a value outward. The traversal is
the same BFS/DFS you already know; only two things change:

1. **Neighbors are computed, not stored** — no adjacency list; you generate them with a move-set.
2. **The `visited` key is the coordinate tuple `(r, c)`** (hashable — a list is not).

---

## 0. The grid toolkit

```python
R, C = len(grid), len(grid[0])          # dimensions (assume non-empty; guard if not)

# 4-directional move-set (up/down/left/right). Use the 8-dir set if diagonals count.
DIRS = [(-1, 0), (1, 0), (0, -1), (0, 1)]
DIRS8 = DIRS + [(-1,-1), (-1,1), (1,-1), (1,1)]

for dr, dc in DIRS:
    nr, nc = r + dr, c + dc
    if 0 <= nr < R and 0 <= nc < C:     # BOUNDS CHECK FIRST -- the #1 grid bug
        ...                             # (nr, nc) is a valid neighbor
```

**The bounds check `0 <= nr < R and 0 <= nc < C` must come *before* you touch `grid[nr][nc]`.**
Reading an out-of-range cell is the single most common grid mistake. In Python a negative index
silently wraps (`grid[-1]` is the last row, not an error), so an unchecked `nr = -1` gives *wrong
answers* rather than a crash — even nastier than an exception. Always guard first.

---

## 1. Grid BFS — shortest path on a grid

Because every step costs 1, **shortest path on a grid is BFS, not Dijkstra.** Track distance
alongside each cell.

> **Problem (Shortest Path in Binary Matrix):** In an R×C 0/1 grid, a *clear path* goes from
> `(0,0)` to `(R-1,C-1)` stepping only on `0` cells, moving **8-directionally**. Return the number of
> cells in the shortest clear path, or `-1` if none.

```python
from collections import deque

def shortest_path_binary(grid):
    n = len(grid)
    if grid[0][0] != 0 or grid[n-1][n-1] != 0:      # start or end blocked
        return -1
    DIRS8 = [(-1,0),(1,0),(0,-1),(0,1),(-1,-1),(-1,1),(1,-1),(1,1)]

    queue = deque([(0, 0, 1)])          # (row, col, path_length_so_far)
    visited = {(0, 0)}                  # mark on ENQUEUE (same rule as graph BFS)
    while queue:
        r, c, d = queue.popleft()
        if (r, c) == (n-1, n-1):
            return d                    # first arrival = shortest (BFS on unweighted grid)
        for dr, dc in DIRS8:
            nr, nc = r + dr, c + dc
            if (0 <= nr < n and 0 <= nc < n           # in bounds
                    and grid[nr][nc] == 0             # walkable
                    and (nr, nc) not in visited):
                visited.add((nr, nc))                 # mark on enqueue -> no duplicate cells
                queue.append((nr, nc, d + 1))
    return -1
```

Same BFS skeleton as the graph page: `deque`, mark-visited-on-enqueue, first-arrival-is-shortest.
The only grid-specific parts are the `DIRS` neighbor generation and the bounds check.

---

## 2. Grid DFS / flood fill — connected regions

Counting regions on a grid is connected-components with computed neighbors. **Number of Islands**
(covered on the Graphs I page) is the canonical example. Here's the flood-fill twist plus a common
**in-place** memory trick.

> **Problem (Flood Fill):** Given an `image` grid, a start pixel `(sr, sc)`, and a `newColor`, repaint
> every pixel connected to the start (4-directionally) that shares the start's original color.

```python
def flood_fill(image, sr, sc, newColor):
    R, C = len(image), len(image[0])
    old = image[sr][sc]
    if old == newColor:                 # guard: same color -> nothing to do (avoids infinite loop)
        return image
    DIRS = [(-1,0),(1,0),(0,-1),(0,1)]

    stack = [(sr, sc)]
    while stack:
        r, c = stack.pop()
        image[r][c] = newColor          # "visit" = repaint. The grid itself records visited.
        for dr, dc in DIRS:
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and image[nr][nc] == old:
                stack.append((nr, nc))
    return image
```

**In-place visited:** instead of a separate `visited` set, we mutate the grid — a repainted cell no
longer equals `old`, so it's never revisited. This saves O(R·C) memory and is a common grid idiom
(also used in Number of Islands by sinking `'1'`→`'0'`). The trade-off is you **destroy the input**;
only do it when that's acceptable, otherwise keep a `visited` set. The `old == newColor` guard is
essential — without it, a repaint-to-same-color never changes cells and the DFS loops forever.

---

## 3. Multi-source BFS — spread from many starts at once

A powerful grid pattern: seed the queue with **all** sources before starting. BFS then expands every
source simultaneously in lockstep, so each cell gets the distance to its *nearest* source for free.

> **Problem (Rotting Oranges):** Grid cells are `0` (empty), `1` (fresh orange), `2` (rotten). Each
> minute, a rotten orange rots its 4-directional fresh neighbors. Return the minutes until no fresh
> orange remains, or `-1` if impossible.

```python
from collections import deque

def oranges_rotting(grid):
    R, C = len(grid), len(grid[0])
    queue = deque()
    fresh = 0
    for r in range(R):                  # seed ALL initially-rotten cells as sources
        for c in range(C):
            if grid[r][c] == 2:
                queue.append((r, c, 0))  # (row, col, minute)
            elif grid[r][c] == 1:
                fresh += 1

    DIRS = [(-1,0),(1,0),(0,-1),(0,1)]
    minutes = 0
    while queue:
        r, c, t = queue.popleft()
        minutes = max(minutes, t)
        for dr, dc in DIRS:
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] == 1:
                grid[nr][nc] = 2         # rot it now (mark visited in-place, on enqueue)
                fresh -= 1
                queue.append((nr, nc, t + 1))

    return minutes if fresh == 0 else -1  # leftover fresh oranges = unreachable
```

The key move is **seeding the queue with every source up front** — then a single BFS computes the
nearest-source distance for all cells at once. This is the go-to for "how long to fill/reach
everything from multiple starts" and "distance to nearest X" grid questions.

---

## Complexity

For an R×C grid, BFS/DFS visits each cell once with O(1) neighbor work → **O(R·C) time**, **O(R·C)
space** (queue/stack + visited, or O(1) extra if you mark in-place). Optimal — you must look at every
cell at least once.

---

## One-screen summary

```
grid = graph   cell (r,c) is a node; neighbors are adjacent cells (computed, not stored)
moves          DIRS = [(-1,0),(1,0),(0,-1),(0,1)]   (+ 4 diagonals for 8-dir)
bounds FIRST   if 0 <= nr < R and 0 <= nc < C:  then touch grid[nr][nc]   (neg index wraps!)
visited key    the tuple (r, c)  in a set  (or mark in-place by mutating the grid)
shortest path  grid steps cost 1 -> BFS (deque), track (r, c, dist); first arrival = answer
regions        DFS/BFS flood-fill; count each new unvisited start (islands / provinces)
in-place mark  mutate the cell so it != its old value -> no visited set (destroys input)
multi-source   seed the queue with ALL sources up front -> nearest-source distance for free
complexity     O(R * C)
```

Read once, then drill from a blank file. Every grid problem is graph BFS/DFS with two extras:
compute neighbors via `DIRS`, and bounds-check before you index.
