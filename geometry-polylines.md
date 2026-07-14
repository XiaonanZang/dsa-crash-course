---
title: Geometry / Polylines
---

# Geometry / Polylines

HD-map elements — lanes, road boundaries, markings — are **points and polylines**. So the geometry
that shows up in a mapping interview is: measure **distances**, find the **closest point on a
segment**, score two polylines against each other (**Chamfer distance** — the standard
vectorized-map accuracy metric), and reason about **intervals/overlaps**. None of it needs heavy math;
it's careful indexing plus a couple of formulas.

Everything is plain Python — **no numpy** in a coding round (see the Arrays & Hashing note on why).

---

## 0. Representing points and polylines

```python
p = (x, y)                       # a point is a tuple  (hashable -> can go in a set/dict)
polyline = [(x0, y0), (x1, y1), (x2, y2), ...]   # an ordered list of points
```

A point is a **tuple** (immutable, hashable — same reasoning as grid coordinates). A polyline is just
a list of points in order; consecutive points define its segments.

---

## 1. Distance — and the squared-distance trick

```python
import math

def dist(a, b):                          # Euclidean distance
    return math.hypot(a[0]-b[0], a[1]-b[1])   # hypot = sqrt(dx*dx + dy*dy), avoids overflow

def dist2(a, b):                         # SQUARED distance -- no sqrt
    dx, dy = a[0]-b[0], a[1]-b[1]
    return dx*dx + dy*dy

def manhattan(a, b):                     # L1 (grid) distance
    return abs(a[0]-b[0]) + abs(a[1]-b[1])
```

**The squared-distance trick:** when you only need to *compare* distances (nearest point, "is this
closer?"), compare **`dist2`** and skip the `sqrt`. `sqrt` is monotonic, so it never changes which is
smaller — computing it just wastes time and adds floating-point error. Only take the real `sqrt` at
the very end if you need an actual length. This matters in nearest-neighbor loops (like Chamfer below).

---

## 2. Closest point on a segment (point-to-segment distance)

The core polyline primitive: how far is a point `P` from a segment `AB`? Project `P` onto the line,
**clamp** the projection to the segment's endpoints, and measure to that clamped point.

```python
def point_seg_dist(p, a, b):
    ax, ay = a; bx, by = b; px, py = p
    abx, aby = bx-ax, by-ay          # segment vector A->B
    apx, apy = px-ax, py-ay          # A->P
    ab2 = abx*abx + aby*aby          # |AB|^2
    if ab2 == 0:                     # A == B: the "segment" is a single point
        return math.hypot(apx, apy)
    t = (apx*abx + apy*aby) / ab2    # projection parameter: where P lands along AB
    t = max(0.0, min(1.0, t))        # CLAMP to [0,1] -> stay on the segment, not the infinite line
    cx, cy = ax + t*abx, ay + t*aby  # closest point on the segment
    return math.hypot(px-cx, py-cy)
```

**The clamp is the whole trick.** `t` is how far along `AB` the perpendicular foot lands: `t=0` is `A`,
`t=1` is `B`, `0<t<1` is between them. Without clamping you'd measure to the *infinite line*; clamping
to `[0,1]` keeps the closest point on the actual segment (so if the foot falls beyond an endpoint, you
correctly measure to that endpoint). Distance from a point to a whole **polyline** = the min of
`point_seg_dist` over all its segments.

---

## 3. Chamfer distance — scoring two polylines (the map-eval metric)

This is the domain payoff. To evaluate a **predicted** map polyline against the **ground-truth** one
(exactly what vectorized-map models like MapTR are scored on), you sample points along each and
compute **Chamfer distance**: for every point in one set, find its nearest neighbor in the other, and
average — **both directions**.

```python
def chamfer(A, B):                       # A, B are lists of points (sampled along polylines)
    def one_way(P, Q):                   # avg nearest-neighbor distance from P to Q
        total = 0.0
        for p in P:
            best = min(dist2(p, q) for q in Q)   # squared-dist trick inside the hot loop
            total += math.sqrt(best)             # sqrt once, only for the winner
        return total / len(P)
    return one_way(A, B) + one_way(B, A) # symmetric: both directions
```

**Why both directions?** A one-way term alone can be fooled: a prediction that covers only *half* the
ground truth still scores well going pred→GT (every predicted point is near a GT point), but scores
badly GT→pred (half the GT points have no nearby prediction). Summing both directions penalizes both
*wrong* points and *missing* ones. The naive version is **O(|A|·|B|)**; that's fine for interview
sizes, and the talking point is "sample the polylines, then bidirectional nearest-neighbor."

---

## 4. Intervals — merge overlapping ranges

Intervals (segments on a number line: time ranges, 1-D spans) are the other classic in this bucket.
The archetype is **Merge Intervals**, and the key move is: **sort by start, then sweep once.**

> **Problem:** Given intervals like `[[1,3],[2,6],[8,10],[15,18]]`, merge all overlapping ones →
> `[[1,6],[8,10],[15,18]]`.

```python
def merge_intervals(intervals):
    intervals.sort(key=lambda iv: iv[0])     # sort by START -- the enabling step
    merged = []
    for start, end in intervals:
        if merged and start <= merged[-1][1]:    # overlaps the last kept interval?
            merged[-1][1] = max(merged[-1][1], end)   # extend it (take the farther end)
        else:
            merged.append([start, end])          # no overlap -> start a new interval
    return merged
```

**Why sorting by start works:** once sorted, any interval that overlaps the current group must start
*before the group's current end*, so a single left-to-right pass catches every overlap — you only ever
compare against the **last** merged interval. The overlap test is `start <= merged[-1][1]`, and you
extend with `max(...)` because the next interval might end *earlier* than the current group (nested).

**Overlap primitive** (worth memorizing): two intervals `[a,b]` and `[c,d]` overlap iff
`a <= d and c <= b`. This underlies meeting-rooms, interval-insert, and interval-intersection problems.

---

## Complexity

- distance / point-to-segment: **O(1)**; point-to-polyline: **O(segments)**.
- Chamfer (naive): **O(|A|·|B|)**.
- merge intervals: **O(n log n)** (dominated by the sort), **O(n)** extra.

---

## One-screen summary

```
point / polyline   p = (x, y) tuple ; polyline = [p0, p1, ...] ordered points
distance           math.hypot(dx, dy) ; SQUARED dist2 = dx*dx+dy*dy for COMPARISONS (skip sqrt)
pt -> segment      project t = dot(AP,AB)/|AB|^2 ; CLAMP t to [0,1] ; measure to A + t*AB
pt -> polyline     min of pt->segment over all segments
chamfer            bidirectional avg nearest-neighbor: one_way(A,B)+one_way(B,A)  (map-eval metric)
merge intervals    sort by START ; if start <= last_end: last_end = max(last_end,end) else append
overlap test       [a,b] & [c,d] overlap  iff  a <= d and c <= b
complexity         dist O(1) ; chamfer O(|A||B|) ; merge O(n log n)
```

Read once, then drill from a blank file. The two ideas that carry most problems: **clamp the
projection to `[0,1]`** for point-to-segment, and **sort by start** for intervals.
