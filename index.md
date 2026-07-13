---
title: Crash Course for DS&A
---

# Crash Course for DS&A

A fast, practical crash course in data structures and algorithms for coding interviews.
Each page pairs the intuition, the Python template, and a worked example with explanation
comments. Written to be read in short sittings and drilled from.

The order is **graph-first**: for interviews where the domain involves networks, connectivity,
and topology (maps, routing, dependencies), graph patterns carry the most weight, and they are
built on arrays and hashing, so those come first as the substrate.

## Roadmap

| # | Pattern | What it covers | Priority |
|---|---|---|---|
| 0 | [C++ → Python cheat sheet](cpp-to-python.html) | Translate DS&A muscle memory from C++ to Python idioms | start here |
| 1 | [Arrays & Hashing](arrays-hashing.html) | `dict` / `set` / `Counter`, frequency, dedup, two-sum family | foundation |
| 2 | Graphs I — BFS / DFS / components | adjacency list, `visited`, traversal, connected components | core |
| 3 | Grid BFS / DFS | matrix as an implicit graph: islands, flood fill | core |
| 4 | Graphs II — topological sort + Dijkstra | ordering with dependencies, shortest path with a heap | core |
| 5 | Geometry / intervals | points, polylines, distances, merge intervals | high |
| 6 | Binary search | sorted arrays + search-on-answer-space | foundation |
| 7 | Heap / top-k | `heapq`, k-largest, k-closest, merge-k | high |

*Foundations (arrays/hashing, binary search) appear in almost every problem; the graph family is
the interview-day core when the domain is networks and topology.*

---

*These notes are written to be read on a phone in fragmented time, then drilled from a blank file.*
