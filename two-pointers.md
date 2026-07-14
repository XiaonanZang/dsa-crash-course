---
title: Two Pointers
---

# Two Pointers

Two indices walking through a sequence, used to turn an O(n²) nested scan into a single **O(n)** pass.
Two flavors: **opposite ends** (start + end moving toward each other, usually on a *sorted* array) and
**same direction** (a slow and a fast pointer). Built on the Arrays & Hashing substrate.

---

## 1. Opposite ends — converging pointers

Two pointers at the two ends move inward, each step eliminating one candidate so a nested O(n²) scan
collapses to O(n). But *what* tells you which pointer to move splits into **two different engines** —
and confusing them is a real trap:

- **Engine A — sortedness gives monotonicity.** The array is **sorted**, so a comparison to a target
  is monotonic: `sum too small → the only way up is lo += 1`. Break the sort and this logic collapses.
  (Two Sum II below.)
- **Engine B — a greedy elimination invariant.** The values are **arbitrary / unsorted**. What lets you
  move a pointer is a proof that one side is already maximized and safe to retire, not any ordering.
  (Container With Most Water below is *not* sorted.)

Diagnostic: **ask "is the array sorted?"** Sorted → target-comparison logic (Engine A). Unsorted but
opposite-ends still works → look for a greedy invariant like a min-height bottleneck (Engine B).

> **Two Sum II (sorted input):** given a **sorted** array and a target, return the indices of the two
> numbers that sum to target.

```python
def two_sum_sorted(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        s = nums[lo] + nums[hi]
        if s == target:
            return [lo, hi]
        elif s < target:
            lo += 1              # sum too small -> need a bigger number -> move left pointer up
        else:
            hi -= 1              # sum too big  -> need a smaller number -> move right pointer down
    return []
```

The sorted order is the whole engine: if the sum is too small, the *only* way to grow it is `lo += 1`;
too big, `hi -= 1`. Each step eliminates one candidate, so it's O(n) instead of the O(n²) hash-free
scan. (Unsorted? Use the hash-map Two Sum from Arrays & Hashing instead.)

> **Valid Palindrome:** check if a string reads the same forwards and backwards (letters/digits only).

```python
def is_palindrome(s):
    lo, hi = 0, len(s) - 1
    while lo < hi:
        while lo < hi and not s[lo].isalnum(): lo += 1     # skip non-alphanumeric
        while lo < hi and not s[hi].isalnum(): hi -= 1
        if s[lo].lower() != s[hi].lower():
            return False
        lo += 1; hi -= 1
    return True
```

> **Container With Most Water:** heights array; pick two lines forming the largest water container.

```python
def max_area(height):
    lo, hi = 0, len(height) - 1
    best = 0
    while lo < hi:
        best = max(best, min(height[lo], height[hi]) * (hi - lo))   # area = shorter side * width
        if height[lo] < height[hi]:      # move the SHORTER line inward -- moving the taller can't help
            lo += 1
        else:
            hi -= 1
    return best
```

The insight: area is limited by the **shorter** line, so moving the taller inward only shrinks width
with no upside — always advance the shorter side. Note this array is **not sorted** (Engine B): the
decision rule comes from a greedy invariant, not from ordering.

**Why retiring the shorter line is provably safe (the part that looks like a leap).** Two forces fight:
width **only ever shrinks** as you move inward, and height is **capped by the shorter line** (water
spills over the short one). So the only way to beat the current area is to raise the *minimum*. Say
`height[lo] < height[hi]`, so `lo` is the bottleneck. The container you just measured pairs `lo` with
`hi`, the **farthest** partner `lo` will ever have. Any other partner `j` (with `lo < j < hi`) gives:

- smaller width: `j - lo < hi - lo`, and
- height still capped by `min(height[lo], height[j]) ≤ height[lo]`.

So `area(lo, j) ≤ height[lo]·(j - lo) < height[lo]·(hi - lo) = area(lo, hi)`. **Every** remaining
container using `lo` is strictly worse — there is nothing left to gain from `lo`, so drop it
(`lo += 1`) and never look back. Moving the *taller* line instead would shrink width while the min
stays capped by the untouched short line: area can only stay equal or fall. That's why "advance the
shorter side" isn't a heuristic, it's the only move that can't discard the optimum.

Trace `height = [1,8,6,2,5,4,8,3,7]`: `(1,7)→8`, move lo; `(8,7)→49`, move hi; `(8,3)→18`, move hi;
`(8,8)→40`, … best stays **49** = answer. Each step permanently eliminates the shorter line, so O(n).

---

## 2. Same direction — slow / fast

Both pointers move forward but play **different roles**: `fast` is the **reader** (scans every element,
never stops — it's just the `for` loop), and `slow` is the **writer / boundary** (marks the end of the
finalized region, advances only when you *commit* a value). The array splits into three zones:

```
[ 0 .. slow ]         finalized prefix (the answer, growing)
[ slow+1 .. fast-1 ]  scanned garbage / duplicates already skipped
[ fast .. end ]       not looked at yet
```

> **Remove Duplicates from Sorted Array (in place):** keep one of each value, return the new length.

```python
def remove_duplicates(nums):
    if not nums:
        return 0
    slow = 0                        # slow = last unique position written
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:
            slow += 1
            nums[slow] = nums[fast]  # overwrite the next slot with the new unique value
    return slow + 1                  # count = index + 1
```

`slow` walks the "clean" prefix; `fast` scans ahead for the next distinct value. One pass, O(1) extra
space.

**Why this NEEDS a sorted array.** The rule `if nums[fast] != nums[slow]: commit` only catches
duplicates that are **adjacent in value**. Sorting is what guarantees every copy of a value sits in one
contiguous run (`[1,1,1,2,2,3]`), so comparing against the single last-committed value catches them all.
If the array were **unsorted** (`[4,1,7,2,9,4]`), the two 4s are non-adjacent: by the time `fast` reaches
the second 4, `nums[slow]` is some other value, the `!=` passes, and the duplicate survives. Slow/fast is
a **contiguous-run collapser**, not a general dedup — it only earns its O(1) space because sorting
pre-groups the duplicates. For unsorted dedup you need a `seen = set()` (O(n) space) from Arrays & Hashing.

### Slow / fast on a linked list — Floyd's cycle detection

The same slow/fast idea detects a cycle in a linked list in **O(1) space** (no `visited` set):

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next          # tortoise: 1 step
        fast = fast.next.next     # hare: 2 steps
        if slow is fast:          # collided inside the loop -> cycle
            return True
    return False                  # fast fell off the end -> no cycle
```

**Why they must meet if there's a cycle.** No cycle → `fast` reaches `None` first (the `while` guard
catches it). Cycle → neither can leave the loop, and once both are inside, `fast` gains **exactly 1
step per move** on `slow`, so the gap between them counts down `… 3, 2, 1, 0` and **cannot skip over 0**.
When it hits 0 they're on the same node. (This is why it's specifically 2-vs-1: a bigger stride could
leap past the meeting point.) O(n) time, O(1) space — vs a `visited` set which is also O(n) time but
O(n) space. "Can you do it without extra memory?" is the interviewer fishing for exactly this.

**Follow-up — where does the cycle start?** After the collision, reset one pointer to `head` and move
**both** one step at a time; they meet at the cycle's entrance:

```python
    slow = head
    while slow is not fast:
        slow = slow.next
        fast = fast.next          # both move 1 now
    return slow                   # == cycle start
```

Why it lands on the entrance: with `L` = head→entrance distance and `C` = cycle length, at the meeting
point `fast` has walked a whole number of extra loops, which forces `L ≡ -k (mod C)`. So walking `L`
more steps from the meeting point reaches the entrance, and `L` is also head→entrance — the two
pointers converge there. (Linked List is otherwise an accepted gap for the mapping JD; this lives here
because it's the marquee slow/fast application.)

---

## Complexity

**O(n)** time (each pointer advances at most n steps), **O(1)** extra space. The win over a nested
loop is the whole point — recognize "sorted array + pair/target" or "in-place filter" and reach for
two pointers.

---

## One-screen summary

```
opposite ends   lo=0, hi=n-1; two engines for "which pointer moves":
  A sorted      monotonic: s<target -> lo+=1 ; s>target -> hi-=1   (needs SORTED)
  B greedy      container: move SHORTER side inward, area=min(h)*width  (NOT sorted)
  palindrome    compare s[lo] vs s[hi], skip non-alnum, move both inward
same direction  fast=reader (scans all); slow=writer (end of finalized prefix)
  in-place dedup slow=last-unique; if nums[fast]!=nums[slow]: slow+=1; nums[slow]=nums[fast]
                 (needs SORTED — only collapses ADJACENT dups; unsorted -> use a set)
  floyd cycle   slow+=1, fast+=2; meet => cycle; gap shrinks by 1/step, can't skip 0
complexity      O(n) time, O(1) space
trigger         "sorted + find a pair/target", greedy bottleneck, or "filter/dedup in place"
```

Read once, then drill from a blank file. The signal is a **sorted array with a pair/target**, or an
**in-place filter** — both collapse a nested loop into one linear pass.
