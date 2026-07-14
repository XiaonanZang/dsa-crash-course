---
title: Heap / Top-K
---

# Heap / Top-K

A heap (priority queue) gives you the **min (or max) in O(1)** and push/pop in **O(log n)**. You've
already used it as the frontier in Dijkstra; here it's the tool for **top-k**, **k-closest**, and
**merge-k** problems. Python's `heapq` is a **min-heap** over a plain list.

```python
import heapq
heap = []
heapq.heappush(heap, x)        # push,  O(log n)
heapq.heappop(heap)            # pop the SMALLEST, O(log n)
heap[0]                        # peek min without popping, O(1)
heapq.heapify(lst)             # turn a list into a heap IN PLACE, O(n)
heapq.nlargest(k, it)          # top-k largest ; heapq.nsmallest(k, it)
```

**Max-heap trick:** Python only has a min-heap, so **push negatives** for a max-heap:
`heapq.heappush(h, -x)`, then `-heapq.heappop(h)`. For tuples, the heap sorts by the first element:
`heappush(h, (priority, item))`.

---

## 1. Top-K — a min-heap of size k

The core idea: to keep the **k largest** items, maintain a **min-heap of size k**. The smallest of
your current top-k sits at the root; if a new item beats it, swap.

> **Kth Largest Element in an Array:** return the k-th largest value.

```python
import heapq

def find_kth_largest(nums, k):
    heap = []                            # min-heap holding the k largest seen so far
    for x in nums:
        heapq.heappush(heap, x)
        if len(heap) > k:                # too big -> drop the smallest of the top-k
            heapq.heappop(heap)
    return heap[0]                       # root = smallest of the k largest = kth largest
```

Why a *min*-heap for *largest*? Because you want O(1) access to the **weakest** member of your top-k
so you can evict it. This runs in **O(n log k)** and O(k) space — better than sorting (O(n log n)) when
k ≪ n. (`heapq.nlargest(k, nums)[-1]` is the one-liner equivalent.)

---

## 2. K Closest — heap with a distance key

> **K Closest Points to Origin:** return the k points nearest to (0,0). (Directly the "k nearest
> lanes / neighbors" flavor in a mapping domain.)

```python
import heapq

def k_closest(points, k):
    heap = []                            # max-heap of size k, keyed by NEGATIVE squared distance
    for x, y in points:
        d = x*x + y*y                    # squared distance -- no sqrt needed for comparison
        heapq.heappush(heap, (-d, x, y)) # negate -> min-heap behaves as a max-heap
        if len(heap) > k:
            heapq.heappop(heap)          # evict the farthest (largest real distance)
    return [(x, y) for _, x, y in heap]
```

Two reused ideas: the **squared-distance trick** (skip `sqrt` when only comparing, from Geometry), and
the **negate-for-max-heap** trick so the size-k heap evicts the *farthest* point. O(n log k).

---

## 3. Merge K sorted lists — heap of the k heads

> **Problem:** merge `k` sorted lists into one sorted list.

```python
import heapq

def merge_k(lists):
    heap = []
    for i, lst in enumerate(lists):      # seed with the head of each list
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))   # (value, which-list, index-in-list)
    out = []
    while heap:
        val, i, j = heapq.heappop(heap)  # smallest current head across all lists
        out.append(val)
        if j + 1 < len(lists[i]):        # push the next element from that same list
            heapq.heappush(heap, (lists[i][j+1], i, j+1))
    return out
```

The heap always holds **one candidate per list** (the current head), so popping the min gives the next
overall element. The tuple carries `(value, list-index, position)` so you know where to pull the
replacement from. **O(N log k)** where N = total elements, k = number of lists — the `log k` is the
heap of heads.

---

## Complexity

- top-k / k-closest: **O(n log k)** time, **O(k)** space — beats sorting when k ≪ n.
- merge-k: **O(N log k)**.
- `heapify`: **O(n)**; each push/pop: **O(log n)**; peek `heap[0]`: **O(1)**.

---

## One-screen summary

```
heapq          min-heap over a list: heappush / heappop(smallest) / heap[0] peek / heapify O(n)
max-heap       push -x, read -heappop  (Python has min-heap only)
tuples         heappush(h, (priority, item)) -> sorts by first element
top-k largest  MIN-heap of size k: push, if len>k: heappop; root = kth largest   O(n log k)
k closest      max-heap of size k keyed by -squared_distance; evict farthest      O(n log k)
merge k        heap of the k current heads (value, list_i, pos); pop min, push next  O(N log k)
one-liners     heapq.nlargest(k, it) / nsmallest(k, it)
complexity     push/pop O(log n), peek O(1), top-k O(n log k)
```

Read once, then drill from a blank file. The recurring move is a **size-k heap**: min-heap to keep the
largest, max-heap (negate) to keep the closest — root is always the one to evict.
