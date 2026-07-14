---
title: Stack
---

# Stack

A stack is a LIFO list — `append` to push, `pop` to take the top. In Python a plain **`list` is your
stack** (`append` / `pop` / `st[-1]`), no import needed. Two interview uses: **matching/nesting**
(parentheses, valid sequences) and the **monotonic stack** (next-greater / next-smaller in one pass).

```python
st = []
st.append(x)     # push
st.pop()         # pop the top (LIFO)
st[-1]           # peek the top without removing
not st           # True if empty
```

---

## 1. Matching / nesting — Valid Parentheses

> **Problem:** given a string of `()[]{}`, return True if every bracket is closed by the correct type
> in the correct order. Example: `"([])"` → True, `"([)]"` → False.

```python
def is_valid(s):
    pairs = {')': '(', ']': '[', '}': '{'}   # closer -> its matching opener
    st = []
    for c in s:
        if c in pairs:                       # a closing bracket
            if not st or st.pop() != pairs[c]:   # nothing to match, or wrong type
                return False
        else:                                # an opening bracket -> push it
            st.append(c)
    return not st                            # valid only if nothing is left unmatched
```

The stack captures **nesting order**: the most recent unclosed opener must be the first to close
(LIFO). Two failure modes: a closer with an empty stack (nothing to match), or a closer whose top
opener is the wrong type. Leftover openers at the end (`st` non-empty) also means invalid.

---

## 2. Monotonic stack — next greater element

A **monotonic stack** keeps its contents sorted (increasing or decreasing) by popping violators as you
go. It answers "for each element, what's the next larger/smaller one?" in a single O(n) pass instead
of O(n²).

> **Daily Temperatures:** for each day, how many days until a warmer temperature? (0 if none.)
> `[73,74,75,71,69,72,76,73]` → `[1,1,4,2,1,1,0,0]`.

```python
def daily_temperatures(temps):
    res = [0] * len(temps)
    st = []                              # holds INDICES, kept decreasing by temperature
    for i, t in enumerate(temps):
        while st and temps[st[-1]] < t:  # current temp resolves everything colder on the stack
            j = st.pop()
            res[j] = i - j               # distance from day j to this warmer day i
        st.append(i)
    return res
```

The stack holds indices of days still "waiting" for a warmer day, in decreasing temperature. When a
warmer temp arrives, it resolves all the colder ones on top at once (each is popped exactly once → the
whole thing is O(n) despite the inner `while`). Store **indices**, not values, when you need distances
or positions.

**The pattern generalizes:** "next greater / next smaller / previous greater" and problems like
*largest rectangle in histogram* and *trapping rain water* all ride on a monotonic stack. The tell is
"for each element, find the nearest bigger/smaller one."

---

## Complexity

Both patterns are **O(n)** time (each element is pushed and popped at most once), **O(n)** space for
the stack. The monotonic stack's inner `while` doesn't make it O(n²) — amortized, each index is
popped only once.

---

## One-screen summary

```
stack = list   push=append ; pop=pop() ; peek=st[-1] ; empty = not st
matching       parentheses: push openers; on a closer, st.pop() must equal the matching opener;
               valid iff no mismatch AND stack empty at the end
monotonic      keep stack sorted; pop violators as you scan; each index pushed/popped once -> O(n)
  next-greater store INDICES; while st and cond(top, cur): j=st.pop(); res[j]=... ; st.append(i)
tell           "nearest larger/smaller element", histogram, rain water -> monotonic stack
complexity     O(n) time, O(n) space
```

Read once, then drill from a blank file. A plain `list` is the stack; **matching** uses it for nesting
order, and the **monotonic stack** resolves "next greater/smaller" in one linear pass.
