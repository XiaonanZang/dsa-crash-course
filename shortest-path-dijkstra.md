---
title: Shortest Path — Dijkstra
---

# Shortest Path — Dijkstra

BFS finds the shortest path when every edge counts the same (fewest hops). But a real **road/lane
network has weights** — segment lengths, travel times, costs. "What's the shortest *distance* (not
fewest hops) from A to B on a weighted lane graph?" is a **Dijkstra** question, and it's the direct
routing analog for a mapping/autonomy role.

The one-line mental model, consistent with the BFS/DFS pages:

> **Dijkstra = BFS, but the container is a min-heap ordered by distance instead of a plain queue.**
> BFS pops the *nearest-in-hops* node; Dijkstra pops the *nearest-in-total-weight* node.

**Requirement:** edge weights must be **non-negative** (distances/times always are). Negative weights
break Dijkstra — that needs Bellman-Ford, which is out of scope here.

---

## 0. Weighted graph + the heap

Store weights in the adjacency list as `(neighbor, weight)` tuples, and use `heapq` (Python's
min-heap) as the frontier.

```python
from collections import defaultdict
import heapq

# weighted, directed adjacency list  u -> (v, w)
graph = defaultdict(list)
for u, v, w in edges:            # edge u -> v with weight w
    graph[u].append((v, w))
    # graph[v].append((u, w))   # add this line too if the graph is UNDIRECTED
```

**heapq refresher** (min-heap only):

```python
heap = []
heapq.heappush(heap, (dist, node))   # push a (distance, node) tuple
d, node = heapq.heappop(heap)        # pop the SMALLEST -> tuples compare by first element (dist)
```

A heap of tuples sorts by the first element, so pushing `(dist, node)` makes the heap always hand
back the **smallest-distance** node next. That ordering is the whole engine of Dijkstra.

---

## 1. The algorithm

Keep a `dist` map (best known distance to each node) and a min-heap frontier. Repeatedly pop the
closest unfinalized node and **relax** its edges: if going *through* it reaches a neighbor more
cheaply, update that neighbor and push it.

```python
import heapq
from collections import defaultdict

def dijkstra(n, edges, start):
    graph = defaultdict(list)
    for u, v, w in edges:              # directed; mirror for undirected
        graph[u].append((v, w))

    dist = {start: 0}                  # best distance found so far, per node
    heap = [(0, start)]                # (distance, node)

    while heap:
        d, node = heapq.heappop(heap)  # the closest unfinalized node
        if d > dist.get(node, float('inf')):
            continue                   # STALE entry -> a better path was already processed; skip
        for nxt, w in graph[node]:
            nd = d + w                 # distance to nxt going through node
            if nd < dist.get(nxt, float('inf')):   # found a shorter route to nxt
                dist[nxt] = nd
                heapq.heappush(heap, (nd, nxt))     # push the improved distance

    return dist                        # dist[x] = shortest distance start -> x (missing = unreachable)
```

**Two ideas make this correct and efficient:**

1. **The first time you *pop* a node, its distance is final.** Because the heap always returns the
   smallest distance and weights are non-negative, nothing popped later can improve it. That's the
   core Dijkstra guarantee.
2. **Lazy deletion (the stale-skip).** `heapq` has no "decrease-key," so instead of updating an entry
   in place, you just push a new, smaller `(nd, nxt)` and leave the old one. When an outdated entry
   surfaces, `d > dist[node]` catches it and you `continue`. This is simpler than a decrease-key heap
   and is the standard Python idiom — **don't forget the stale check**, or you'll reprocess nodes.

---

## 2. Worked example — Network Delay Time

> **Problem:** A network of `n` nodes labeled `1..n`. `times[i] = (u, v, w)` is a directed edge:
> a signal from `u` reaches `v` after `w` time. Send a signal from node `k`; return the time for
> **all** nodes to receive it, or `-1` if some node is unreachable. Example: `n=4`, `k=2`,
> `times=[(2,1,1),(2,3,1),(3,4,1)]` → `2` (the farthest node, 4, is reached at time 2).

"Time for all nodes to receive" = the **maximum** shortest-distance from `k`. Run Dijkstra, then take
the max over all `n` nodes (and return `-1` if any node was never reached).

```python
import heapq
from collections import defaultdict

def network_delay_time(times, n, k):
    graph = defaultdict(list)
    for u, v, w in times:
        graph[u].append((v, w))

    dist = {k: 0}
    heap = [(0, k)]
    while heap:
        d, node = heapq.heappop(heap)
        if d > dist.get(node, float('inf')):
            continue
        for nxt, w in graph[node]:
            nd = d + w
            if nd < dist.get(nxt, float('inf')):
                dist[nxt] = nd
                heapq.heappush(heap, (nd, nxt))

    if len(dist) < n:                  # some node never got a distance -> unreachable
        return -1
    return max(dist.values())          # slowest node = when everyone has the signal
```

The Dijkstra core is **identical** to the template — only the post-processing (`max`, reachability
check) is problem-specific. That's the pattern: learn the engine once, wrap it per problem.

---

## 3. Dijkstra vs BFS — when to use which

| | BFS | Dijkstra |
|---|---|---|
| edge weights | all equal (unweighted) | non-negative weights |
| frontier | `deque` (FIFO) | min-heap (`heapq`), ordered by distance |
| finds | fewest **edges** | least **total weight** |
| cost | O(V + E) | O(E log V) |

If a problem says "shortest path" and the edges are **unweighted (or all weight 1)**, use BFS — it's
simpler and faster. Reach for Dijkstra only when edges carry **different non-negative weights**. On a
grid where every move costs 1, that's still BFS, not Dijkstra.

---

## Complexity

**O(E log V)** time — each edge can push one heap entry, and heap ops are `log`. **O(V + E)** space
(graph + dist + heap). For interview-sized graphs this is the optimal general-purpose weighted
shortest-path algorithm.

---

## One-screen summary

```
use when       weighted edges, all weights NON-NEGATIVE (else Bellman-Ford)
model          Dijkstra = BFS with a MIN-HEAP frontier ordered by distance
represent      graph[u].append((v, w))   # weighted adjacency list
heap           heapq.heappush(heap, (dist, node)); heappop -> smallest dist (tuples sort by [0])
dist map       dist = {start: 0};  relax:  if d + w < dist.get(nxt, inf): update + push
stale skip     if d > dist.get(node, inf): continue    # lazy deletion, DON'T forget it
finalized      first POP of a node = its final shortest distance
unreachable    node absent from dist  (or dist == inf)
complexity     O(E log V)
BFS vs this    equal weights -> BFS (O(V+E)); different weights -> Dijkstra
```

Read once, then drill from a blank file. It's BFS with a heap, a `dist` map, and one stale-check line.
