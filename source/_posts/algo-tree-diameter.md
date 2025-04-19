---
title: Diameter of a Tree
cover: /img/wow.png
mathjax: true
categories: Algorithm
tags: 
    - Tree
    - Graph
    - Diameter
    - BFS
abbrlink: 283833
date: 2023-02-17 00:00:00
updated: 2023-02-17 00:00:00
---

Tree is special kind of graph. It is more structured and often exhibits amazing properties. In this post, we are gonna discuss diameter of the tree.

## Problem Description
A **tree** is defined as an unweighted undirected graph.

$$     
T = (V, E)
$$

The **diameter** of the tree is defined as the longest distance between all pairs of nodes.
$$
\text{diameter}(T)=max_{(u, v)\in E} \text{ }d(u, v)
$$

Note that distance between any pair of node is **unique**. The goal is to find the diameter inside a tree.

For example, in the following tree, the diameter = 3 (Path: 1, 2, 4, 5)

![](/img/algo/tree.png)

## Algorithm
This problem can be solved by running two times of **BFS**.

```python=
let s be an arbitrary vertex
perform BFS on s and let u be the furthest vertex from s
perform BFS on u and let v be the furthest vertex from u
return d(u, v)
```

## Proof
Let `u` and `v` be the endpoints of diameter.

If we perform BFS from `u`, we are bound to find `v` as the furthest vertex. (otherwise, `(u,v)` won't be the diameter, as we can find a longer path than `(u, v)`)

So it suffices to prove that **in the first BFS, we will find one endpoint of the diameter**.

Let `w` be the furthest node from s (found in the first BFS). We will prove by **contradiction**, thus assume `w != u` and `w != v`.

Consider the trees in the following cases:
* `s` on the diameter
* `s` NOT on the diameter
    + path of `s` to `w` intersects with the diameter
    + path of `s` to `w` does NOT intersect with the diameter

### Case 1: `s` on the diameter

![](/img/algo/case1.png)

WLOG, let `d(s, v) >= d(s, u)`. Since `w` is furthest from `s`, then
```
d(u, w) = d(u, s) + d(s, w) > d(u, s) + d(s, v) = d(u, v)
```

A contradiction occurs, as path from `u` to `w` is longer than `u` to `v`. Therefore, `w` must be `u` or `v` in this case.

We can conclude an important lemma from case 1.


{% note green Lemma %}
**Lemma**

If we perform BFS on any node which is on the diamter, we will find an endpoint of the diameter as the furthest node from the starting node.

![](/img/algo/case1.1.png)

{% endnote %}

### Case 2: `s` NOT on the diameter

#### Case 2-1: `s` to `w` intersects with the diameter

Let `i` be the intersection node.

![](/img/algo/case2.1.png)

Since `i` is on the diameter, then by lemma, the node furthest from `i` must be `u` or `v`. 

Thus the furthest node from `s` must be `u` or `v`.

#### Case 2-2: `s` to `w` does NOT intersect with the diameter

Let `i` and `j` be the nodes on `s->w` and `u->v`, respectively, and they connects both path.

![](/img/algo/case2.21.png)

WLOG, assume `d(j, v) > d(j, u)`.
Since `w` is farthest from `s`, then `d(i, w) > d(i, v)`.

![](/img/algo/case2.22.png)


Furthermore, we can deduce that `d(w, j) > d(j, v)`.

![](/img/algo/case2.23.png)

However, `j` is on the diameter, by lemma, the furthest node from `j` must be `v`. Thus a contradiction occurs.

## Excercise: Find the sum of distances of all nodes to their farthest nodes

Given an unweighted tree, for each node, compute the farthest distance starting from it. Sum the distances for all nodes.

For example, in the following tree, we have the farthest distance
- node 1: 3
- node 2: 2
- node 3: 3
- node 4: 2
- node 5: 3

So the answer would be `3 + 2 + 3 + 2 + 3 = 13`

![](/img/algo/tree.png)

{% hideToggle Answer %}

A naive way to do this is to perform BFS from each node, which results in `O(n^2)` time complexity.

From the [the proof section](#proof), we know that the farthest node of any node will be one of the two endpoints of the diameter. So we can perform BFS only from these two endpoints and collect their distance to any node in the tree. It will take 3 passes of BFS to do that
- 1st BFS: find one endpoint `u` of the diameter 
- 2nd BFS: find another endpoint `v` of the diameter, and also compute the distances of `u` to any other node
- 3rd BFS: find the distance of `v` to any other node

In summary the optimized algorithm will take `O(n)` time.

{% endhideToggle %}