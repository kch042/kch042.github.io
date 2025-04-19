---
title: Binary search from scratch
cover: /img/wow.png
mathjax: true
categories: Algorithm
tags: 
    - Binary Search
abbrlink: 143829
date: 2022-05-31 00:00:00
updated: 2022-06-02 00:00:00
---

Binary search is an useful algorithm to look up data when the input preserve some ordering pattern. The most common example is finding some number in a sorted array. The algorithm can be extended to solve problems such as finding the lower bound in a query range. Below sections demonstrates the algorithm and the proof of some properties for it to work.

## Intuition

Suppose we are given a **sorted** array, say 
$$A = [1, 1, 2, 4, 6, 9]$$
then we can see that all numbers in $A[0:3]$ are **smaller** than $4$, while all in $A[3:6]$ are **equal or greater than** $4$.

Thus, we can define a **predicate**, 
$$f(x) := (x >= 4)$$

Let $B$ be the array of $f(A)$, then
$$B = [0, 0, 0, 1, 1, 1]$$

As we can see, $B$ is a predicate array of contigous $0$ followed by contiguous $1$.

## Algorithm
The following algorithm will help us find the **smallest** index $i$ in $B$ for which the predicate $f(i)$ holds true.

That is, we want to find the **first** $1$ in $B$. (Keep this in mind !)

```python=
function find(n, f):
    i, j := 0, n
    while i < j:
        mid = (i+j)/2   # floor
        if f(mid):
            j = m       # preserve f(j) = 1
        else
            i = m+1     # preserve f(i-1) = 0
    return i            # Note i == j here
```

## Proof of Correctness
Let $n = len(A)$

Define $f(-1) = 0, f(n) = 1$

### Lemma 1
Invariant:
- $f(i-1) = 0$
- $f(j) = 1$

#### Proof by induction
- At first, $f(-1) = 0$ and $f(n) = 1$, true.
- Later on, if $f(mid) = 0$, then we set $i = mid+1$, preserving $f(i-1) = 0$.
- if $f(mid) = 1$, then we set $j = mid$, still $f(j) = 1$ is preserved.

### Lemma 2
When the loop in line 3 breaks, $i = j$ 
#### Proof
Since $i<j$ at first, we have $i \le\ mid < j$ (since we always take mid using floor).

Consider the last iteration, 
- If we update i, since $mid+1 \le\ j$, then $i$ must be $j$
- If we update j, since $i \le\ mid$, then $j$ must be $i$

### Theorem
By the property of sorted array, and Lemma 1, 2.
We can confirm that the returned index $i$ is the **smallest** index such that $f(i) = 1$


## Application
### Find the first element greater or equal to x
Let $f(v) = (v \geq x)$
```go=
func firstGreaterEqual(x int, nums []int) int {
    i, j := 0, len(nums)
    for i < j {
        m := int(uint(l+r)>>1)  // prevent overflow
        
        if nums[m] >= x {  // f(v) = (v >= x)
            j = m
        } else {
            i = m+1
        }
    }
    
    return i
}
```

### Find the largest element < x
```go=
func largestLess(x int, nums []int) int {
    // can you see why ?
    return firstGreaterEqual(x, nums)-1  
}
```

### Find the last x if duplicated x exist in A
Let $f(v) = (v > x)$
```go=
func lastDuplicate(x int, nums []int) int {
    i, j := 0, len(nums)
    for i < j {
        m := int(uint(i+j)>>1)
        
        if nums[m] > x {
            j = m
        } else {
            i = m+1
        }
    }
    return i - 1
}
```

### Find the first x if duplicated x exist in A
Let $f(v) = (v \geq x)$
```go=
func firstDuplicate(x int, nums []int) int {
    i, j := 0, len(nums)
    for i < j {
        m := int(uint(i+j)>>1)
    
        if nums[m] >= x {
            j = m
        } else {
            i = m+1
        }
    }
    
    return i
}
```