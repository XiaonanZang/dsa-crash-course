---
title: Sliding Window
---

# Sliding Window

A sliding window is a two-pointer technique for **contiguous** subarrays/substrings. A `right` pointer
grows the window; a `left` pointer shrinks it when a constraint is violated. It turns "check every
subarray" (O(n²)) into a single **O(n)** pass. Two forms: **variable-size** (grow/shrink to satisfy a
condition) and **fixed-size** (window of length k slides across).

---

## 1. Variable window — the core template

Grow `right` one step at a time; whenever the window becomes **invalid**, shrink from `left` until
it's valid again. Record the answer each step.

```python
def longest_valid_window(s):
    left = 0
    state = {}                       # whatever tracks validity (counts, a set, a sum, ...)
    best = 0
    for right in range(len(s)):
        # 1. include s[right] in the window
        add(state, s[right])
        # 2. while the window is INVALID, shrink from the left
        while is_invalid(state):
            remove(state, s[left])
            left += 1
        # 3. window is now valid -> record
        best = max(best, right - left + 1)
    return best
```

The three beats — **add right, shrink while invalid, record** — are every variable-window problem. The
only thing that changes is what "state" and "invalid" mean.

> **Longest Substring Without Repeating Characters:** longest substring with all-distinct chars.

```python
def length_of_longest_substring(s):
    seen = set()                     # chars currently in the window
    left = 0
    best = 0
    for right in range(len(s)):
        while s[right] in seen:      # invalid: duplicate -> shrink until the dup is gone
            seen.remove(s[left])
            left += 1
        seen.add(s[right])
        best = max(best, right - left + 1)
    return best
```

Here "invalid" = the new char is already in the window; shrinking from the left drops chars until the
duplicate leaves. Each char is added once and removed once, so it's O(n), not O(n²).

---

## 2. Fixed window — size k slides across

When the window size is a constant `k`, slide it: add the entering element, remove the leaving one.

> **Maximum Sum Subarray of Size k:**

```python
def max_sum_k(nums, k):
    window = sum(nums[:k])           # first window
    best = window
    for right in range(k, len(nums)):
        window += nums[right] - nums[right - k]   # add entering, subtract leaving
        best = max(best, window)
    return best
```

`nums[right] - nums[right-k]` is the key line: instead of re-summing k elements each step (O(n·k)),
you update the running sum in O(1) — the whole point of the sliding window.

---

## 3. Window with a target condition (shrink to minimize)

Some problems want the **shortest** valid window — then you shrink *while still valid* and record
inside the shrink loop.

> **Minimum Size Subarray Sum:** shortest contiguous subarray with sum ≥ target (positive nums).

```python
def min_subarray_len(target, nums):
    left = 0
    total = 0
    best = float('inf')
    for right in range(len(nums)):
        total += nums[right]
        while total >= target:           # valid -> try to shrink for a shorter answer
            best = min(best, right - left + 1)
            total -= nums[left]
            left += 1
    return best if best != float('inf') else 0
```

Note the difference from problem 1: for a **longest** window you shrink *while invalid* and record
after; for a **shortest** window you shrink *while valid* and record inside the shrink. Knowing which
is the one thing to get right.

---

## Complexity

**O(n)** — `right` advances n times and `left` advances at most n times total (each element enters and
leaves the window once). **O(k)** space for the window state.

---

## One-screen summary

```
use when       CONTIGUOUS subarray/substring with a constraint
variable window for right in ...: add(right); while invalid: remove(left); left+=1; record
  longest      shrink while INVALID, record AFTER the while
  shortest     shrink while VALID,   record INSIDE the while
fixed window k  window += nums[right] - nums[right-k]   (O(1) update, don't re-sum)
window length  right - left + 1
complexity     O(n) time, O(k) space
```

Read once, then drill from a blank file. The template is **add-right / shrink-while-bad / record** —
and the only decision is longest (shrink while invalid) vs shortest (shrink while valid).
