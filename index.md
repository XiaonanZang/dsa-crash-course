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

Two columns, two axes:
- **Likelihood** = odds this shows up in *this* (mapping/autonomy) interview. Runs top-to-bottom with
  the reading order. *Not* a topic's general importance — binary search is a universal building block
  but sits low here because the JD is graph-heavy.
- **Drill** = how much cold-drilling it's worth (writing it blind, then running it). Separate from
  likelihood: a topic can be lower-likelihood yet high-drill if it's bug-prone (binary search).
  Scale: **must-drill → drill → optional → skip-ish**.

| # | Pattern | What it covers | Likelihood | Drill |
|---|---|---|---|---|
| 0 | [C++ → Python cheat sheet](cpp-to-python.html) | Translate DS&A muscle memory from C++ to Python idioms | reference | — |
| 1 | [Arrays & Hashing](arrays-hashing.html) | `dict` / `set` / `Counter`, frequency, dedup, two-sum family | must-know · prereq | drill |
| 2 | [Graphs I — traversal & connected components](graphs-1.html) | adjacency list, `visited`, BFS/DFS, components, islands | ⭐ must-know | must-drill |
| 3 | [Topological sort](topological-sort.html) | dependency ordering on a DAG, cycle detection (Kahn + DFS) | ⭐ must-know | must-drill |
| 4 | [Shortest path (Dijkstra)](shortest-path-dijkstra.html) | weighted routing with a min-heap frontier (+ A*) | likely | drill |
| 5 | [Grid BFS / DFS](grid-bfs-dfs.html) | matrix as an implicit graph: shortest path, islands, flood fill, multi-source | likely | drill |
| 6 | [Geometry / polylines](geometry-polylines.html) | points, polylines, point-to-segment, Chamfer (map-eval), merge intervals | likely | drill |
| 7 | [Binary search](binary-search.html) | sorted arrays + search-on-answer-space | possible · foundation | must-drill |
| 8 | [Two Pointers](two-pointers.html) | converging / slow-fast on (usually sorted) arrays | possible | optional |
| 9 | [Sliding Window](sliding-window.html) | contiguous subarray/substring, grow/shrink window | possible | drill |
| 10 | [Stack](stack.html) | matching / nesting, monotonic stack (next-greater) | possible | optional |
| 11 | [Heap / top-k](heap.html) | `heapq`, top-k (size-k heap), k-closest, merge-k | review | skip-ish |

*Why graphs lead: the JD centers on lane/road-network **graph construction, connectivity, and
topology**, so the graph family is the interview-day core. Arrays & Hashing is #1 because everything
above is built on it (adjacency list = `defaultdict(list)`, `visited` = `set`).*

*Drill budget (2 days out): spend it on **JD-core (2–6) ≥ Binary Search ≥ Sliding Window** first;
Two Pointers / Stack / Heap are read-mostly. Binary search is only "possible" to appear but is
**must-drill** because off-by-one boundary bugs are the classic cold-hands failure.*

---

*These notes are written to be read on a phone in fragmented time, then drilled from a blank file.*
