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
| 2 | [Graphs I — traversal & connected components](graphs-1.html) | adjacency list, `visited`, BFS/DFS, components, islands | ⭐ core |
| 3 | [Topological sort](topological-sort.html) | dependency ordering on a DAG, cycle detection (Kahn + DFS) | ⭐ core |
| 4 | [Shortest path (Dijkstra)](shortest-path-dijkstra.html) | weighted routing with a min-heap frontier | core |
| 5 | [Grid BFS / DFS](grid-bfs-dfs.html) | matrix as an implicit graph: shortest path, islands, flood fill, multi-source | core |
| 6 | Geometry / polylines | points, polylines, distances, merge intervals | high |
| 7 | Binary search | sorted arrays + search-on-answer-space | foundation |
| 8 | Heap / top-k | `heapq`, k-largest, k-closest, merge-k | high |

*Order reflects the mapping/autonomy JD: lane/road-network **graph construction, connectivity, and
topology** are the interview-day core, so graphs lead.*

*Foundations (arrays/hashing, binary search) appear in almost every problem; the graph family is
the interview-day core when the domain is networks and topology.*

---

*These notes are written to be read on a phone in fragmented time, then drilled from a blank file.*
