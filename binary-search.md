---
title: Binary Search
---

# Binary Search

Halve the search space each step → **O(log n)**. It applies to a **sorted array**, and, more
powerfully, to **search-on-answer-space** (binary search over a numeric answer with a monotonic
feasibility test). It's a universal building block — and it's where **off-by-one bugs** live, so the
value here is a template you trust rather than re-derive under pressure.

---

## 1. The canonical template (trust this one)

```python
def binary_search(nums, target):
    lo, hi = 0, len(nums) - 1        # INCLUSIVE range [lo, hi]
    while lo <= hi:                  # <= because the range includes hi
        mid = lo + (hi - lo) // 2    # avoids overflow (irrelevant in Python, but the right habit)
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1             # target is to the RIGHT -> discard mid and left
        else:
            hi = mid - 1             # target is to the LEFT  -> discard mid and right
    return -1                        # not found
```

**The three off-by-one decisions that must be consistent** (get these wrong and you infinite-loop or
skip elements):

1. **Range convention:** `[lo, hi]` inclusive → initialize `hi = len - 1` and loop `while lo <= hi`.
2. **Shrink past mid:** `lo = mid + 1` / `hi = mid - 1` — always move *past* `mid` (you already
   checked it). Writing `lo = mid` with an inclusive range is the classic infinite loop.
3. **`mid = lo + (hi - lo)//2`** biases low; fine for this template.

Pick one convention and use it every time — the bugs come from mixing `< / <=` with `mid / mid±1`.

---

## 2. Bounds — first / last occurrence (and `bisect`)

For duplicates, "find target" isn't enough — you want the **first** or **last** index. Instead of
returning on match, keep shrinking toward the boundary.

```python
def first_occurrence(nums, target):
    lo, hi = 0, len(nums) - 1
    ans = -1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            ans = mid                # record, but keep searching LEFT for an earlier one
            hi = mid - 1
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return ans
```

Python's **`bisect`** module does this for you and is worth knowing:

```python
import bisect
bisect.bisect_left(nums, x)    # leftmost index where x could be inserted (first >= x)
bisect.bisect_right(nums, x)   # rightmost such index (first > x)
# count of x in a sorted list:  bisect_right(nums, x) - bisect_left(nums, x)
```

In an interview, writing the loop shows you understand it; `bisect` is the fast path if allowed.

---

## 3. Binary search on the answer (the powerful one)

The high-value pattern: when the answer is a number and "is `X` feasible?" is **monotonic** (if `X`
works, everything larger works — or vice versa), binary-search the *answer*, not an array.

> **Koko Eating Bananas:** piles of bananas, `h` hours. At speed `k` bananas/hour Koko eats
> `ceil(pile/k)` hours per pile. Find the **minimum** `k` to finish within `h` hours.

```python
import math

def min_eating_speed(piles, h):
    def hours_at(k):                        # feasibility: hours needed at speed k
        return sum(math.ceil(p / k) for p in piles)

    lo, hi = 1, max(piles)                  # answer space: speeds 1 .. max pile
    while lo < hi:                          # searching for the boundary -> [lo, hi), lo==hi at end
        mid = lo + (hi - lo) // 2
        if hours_at(mid) <= h:              # mid works -> maybe smaller works too
            hi = mid                        # keep mid as a candidate (note: hi = mid, not mid-1)
        else:
            lo = mid + 1                    # mid too slow -> need faster
    return lo                               # smallest feasible speed
```

The trick: you're not searching an array, you're searching **the range of possible answers** `[1,
max(piles)]`, using a monotonic feasibility check (`faster speed → fewer hours`). Note this uses the
other common template — `while lo < hi` with `hi = mid` — which converges on the **boundary** (the
smallest feasible value) rather than an exact match. Tell for this pattern: "minimize/maximize X such
that a condition holds."

### Why `lo <= hi` here but `lo < hi` there (the part everyone trips on)

The `<=` vs `<` choice is **not free**. It is forced by whether your shrink step moves *past* `mid` or
*keeps* `mid`. Get this and every off-by-one bug disappears.

**The two templates answer different questions:**

- **Template A (`lo <= hi`, `mid ± 1`)** asks *"is this exact element present, yes or no?"* Every index
  is a candidate you **check and reject**. When `nums[mid]` is not the target, `mid` is done, so you
  discard it with `lo = mid + 1` or `hi = mid - 1`. The target may not exist; the range shrinks to
  empty and you return `-1`.
- **Template B (`lo < hi`, `hi = mid`)** asks *"where is the boundary between fails and works?"* The
  answer is **guaranteed to exist** in the range (you set `hi` to a value that always works, e.g.
  `max(piles)`). So when `mid` is feasible, `mid` **might itself be the answer**, and you cannot throw
  it away. You keep it with `hi = mid`.

**The loop condition is then forced by one question: does the shrink step always make progress?**

- `lo = mid + 1` / `hi = mid - 1` **always** move past `mid`, so progress is guaranteed and `lo <= hi`
  is safe. When `lo == hi` there is still exactly **one** unchecked element (`[lo, hi]` with `lo == hi`
  is one slot) and you *want* to check it. Using `<` here would **skip that last element**.
- `hi = mid` does **not** move past `mid`. Watch the range of two, `lo` and `hi = lo + 1`:
  `mid = lo + (hi - lo)//2 = lo`. If feasible, `hi = mid = lo`, now `lo == hi`, done.
  But **if you had written `lo <= hi`**, the loop continues with `lo == hi == mid`, feasible again,
  `hi = mid = lo`, nothing changes, **infinite loop**. So `hi = mid` **requires** `lo < hi`. The moment
  `lo == hi` the range has collapsed to one value, and because you never discarded the answer, that one
  value **is** the answer: stop and `return lo`.

| Shrink step | Moves past mid? | Loop condition | Why |
|---|---|---|---|
| `lo=mid+1` / `hi=mid-1` (reject mid) | yes, always | `lo <= hi` | must still check the final single element |
| `hi = mid` (keep mid) | no | `lo < hi` | `<=` would spin forever on the last element |

**Mental model:** Template A **rejects** `mid` every step and may find nothing. Template B **keeps**
`mid` as a live candidate and squeezes the window until it is one cell wide, and the answer always
survives. Reject → `mid ± 1` → `<=`. Keep → `hi = mid` → `<`. Mixing them (`hi = mid` with `<=`, or
`mid - 1` with `<`) is exactly how you infinite-loop or skip the answer, which is why you trust **one**
of each template and never blend.

---

## Complexity

**O(log n)** for array search; for answer-space search it's **O(log(range) · cost-of-check)** — here
`O(log(max pile) · n)`. The `log` factor is the whole reason to reach for it.

---

## One-screen summary

```
sorted-array template   lo,hi = 0,n-1 ; while lo<=hi ; mid=lo+(hi-lo)//2
  found                 nums[mid]==target -> return mid
  go right / left       <target: lo=mid+1 ;  >target: hi=mid-1     (always move PAST mid)
first/last occurrence   on match, record + keep shrinking toward the side you want
bisect                  bisect_left (first >=x) / bisect_right (first >x) ; count = right-left
answer-space template   while lo<hi ; if feasible(mid): hi=mid else lo=mid+1 ; return lo
  use when              "min/max X such that <monotonic condition>"
complexity              O(log n) ; answer-space O(log(range) * check)
pitfalls                mixing <=/< with mid/mid±1 -> infinite loop or skipped element
```

Read once, then drill from a blank file. Trust **one** array template (`lo<=hi`, `mid±1`) and **one**
answer-space template (`lo<hi`, `hi=mid`) — the bugs all come from mixing them.
