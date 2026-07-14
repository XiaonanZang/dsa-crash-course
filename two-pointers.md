---
title: Two Pointers
---

# Two Pointers

Two indices walking through a sequence, used to turn an O(n²) nested scan into a single **O(n)** pass.
Two flavors: **opposite ends** (start + end moving toward each other, usually on a *sorted* array) and
**same direction** (a slow and a fast pointer). Built on the Arrays & Hashing substrate.

---

## 1. Opposite ends — converging pointers

Two pointers at the two ends move inward based on a comparison. The array is usually **sorted**, which
is what lets you decide *which* pointer to move.

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
with no upside — always advance the shorter side.

---

## 2. Same direction — slow / fast

Both pointers move forward; the slow one marks a write position or lags behind the fast one.

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
space. The fast/slow idea also powers linked-list cycle detection (see the Linked List material).

---

## Complexity

**O(n)** time (each pointer advances at most n steps), **O(1)** extra space. The win over a nested
loop is the whole point — recognize "sorted array + pair/target" or "in-place filter" and reach for
two pointers.

---

## One-screen summary

```
opposite ends   lo=0, hi=n-1; move based on comparison (needs SORTED array usually)
  sorted pair   s<target -> lo+=1 ;  s>target -> hi-=1
  container     move the SHORTER side inward (area = min(h)*width)
  palindrome    compare s[lo] vs s[hi], skip non-alnum, move both inward
same direction  slow marks write/lag position; fast scans ahead
  in-place dedup slow=last-unique; if nums[fast]!=nums[slow]: slow+=1; nums[slow]=nums[fast]
complexity      O(n) time, O(1) space
trigger         "sorted + find a pair/target" or "filter/dedup in place" -> two pointers
```

Read once, then drill from a blank file. The signal is a **sorted array with a pair/target**, or an
**in-place filter** — both collapse a nested loop into one linear pass.
