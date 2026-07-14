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

The **Likelihood** column is the odds this shows up in *this* (mapping/autonomy) interview — it runs
top-to-bottom with the reading order. It is **not** a topic's general importance: binary search is a
universal building block, but it's low here because this JD is graph-heavy.

| # | Pattern | What it covers | Likelihood (this JD) |
|---|---|---|---|
| 0 | [C++ → Python cheat sheet](cpp-to-python.html) | Translate DS&A muscle memory from C++ to Python idioms | reference |
| 1 | [Arrays & Hashing](arrays-hashing.html) | `dict` / `set` / `Counter`, frequency, dedup, two-sum family | must-know · prereq |
| 2 | [Graphs I — traversal & connected components](graphs-1.html) | adjacency list, `visited`, BFS/DFS, components, islands | ⭐ must-know |
| 3 | [Topological sort](topological-sort.html) | dependency ordering on a DAG, cycle detection (Kahn + DFS) | ⭐ must-know |
| 4 | [Shortest path (Dijkstra)](shortest-path-dijkstra.html) | weighted routing with a min-heap frontier (+ A*) | likely |
| 5 | [Grid BFS / DFS](grid-bfs-dfs.html) | matrix as an implicit graph: shortest path, islands, flood fill, multi-source | likely |
| 6 | [Geometry / polylines](geometry-polylines.html) | points, polylines, point-to-segment, Chamfer (map-eval), merge intervals | likely |
| 7 | Binary search | sorted arrays + search-on-answer-space | review |
| 8 | Heap / top-k | `heapq`, k-largest, k-closest, merge-k | review |

*Why graphs lead: the JD centers on lane/road-network **graph construction, connectivity, and
topology**, so the graph family is the interview-day core. Arrays & Hashing is #1 because everything
above is built on it (adjacency list = `defaultdict(list)`, `visited` = `set`).*

---

*These notes are written to be read on a phone in fragmented time, then drilled from a blank file.*
