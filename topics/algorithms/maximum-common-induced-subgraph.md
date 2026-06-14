---
title: Maximum Common Induced Subgraph via the Modular Product
topic: algorithms
tags: [graphs, graph-theory, subgraph-matching, clique, modular-product, Bron-Kerbosch]
---

# Maximum common induced subgraph via the modular product

## The problem

Given two graphs $G$ and $H$, find the largest subgraph that appears in both — the **maximum common induced subgraph** (MCIS).
"Induced" means the match must preserve both edges *and* non-edges: two vertices that are connected in $G$ must be connected in $H$, and two vertices that are not connected in $G$ must not be connected in $H$.

This problem is NP-hard (see [P versus NP](p-versus-np.md) for the complexity theory background).
But it reduces cleanly to another well-studied NP-hard problem — maximum clique — via the modular product graph.


## The modular product

The [modular product](graph-products.md#modular-product-g-diamond-h) $G \diamond H$ has vertex set $V(G) \times V(H)$, with two vertices $`(u_1, v_1)`$ and $`(u_2, v_2)`$ adjacent when their adjacency relations agree in both factors (XNOR): both pairs adjacent, or both pairs non-adjacent. Vertices sharing a component are never adjacent.


## Why cliques correspond to common subgraphs

A clique in $G \diamond H$ is a set of product vertices $\{(u_1, v_1), \ldots, (u_k, v_k)\}$ where every pair is adjacent.
By the XNOR predicate, this means:

$$\text{adj}_G(u_i, u_j) = \text{adj}_H(v_i, v_j) \quad \text{for all } i \neq j$$

The $u$'s define a subset of $V(G)$.
The $v$'s define a subset of $V(H)$.
The clique condition guarantees that these two subsets have identical adjacency structure — wherever two $u$'s are connected in $G$, the corresponding $v$'s are connected in $H$, and wherever two $u$'s are not connected, the corresponding $v$'s are also not connected.

That is precisely a common induced subgraph of size $k$, with the clique vertices defining the vertex correspondence between the two factors.

The reduction is bijective: every common induced subgraph of size $k$ corresponds to a clique of size $k$ in $G \diamond H$, and vice versa.
Therefore, the **maximum** common induced subgraph corresponds to the **maximum** clique.


## Finding the maximum clique: Bron–Kerbosch

The standard algorithm for enumerating maximal cliques is **Bron–Kerbosch** (1973).
It maintains three sets:

- $R$ — the current clique (vertices confirmed to belong)
- $P$ — candidate vertices that are adjacent to every vertex in $R$ (could extend the clique)
- $X$ — already-processed vertices adjacent to every vertex in $R$ (used to avoid reporting duplicate cliques)

```python
def bron_kerbosch(R, P, X, graph):
    if not P and not X:
        report R as a maximal clique
        return
    for v in list(P):
        neighbors = graph.neighbors(v)
        bron_kerbosch(R | {v}, P & neighbors, X & neighbors, graph)
        P.remove(v)
        X.add(v)
```

The initial call is `bron_kerbosch(R=∅, P=V, X=∅)`.

The algorithm's correctness rests on one invariant: every vertex in $P$ is adjacent to every vertex in $R$.
This holds trivially at the start ($R = \emptyset$) and is maintained by the intersection `P & neighbors` — only vertices adjacent to the newly added $v$ *and* to everything already in $R$ survive into the recursive call.
Since `&` is set intersection, $P$ can only shrink, which is structurally necessary: a clique requires all-pairs adjacency, so the candidate pool must narrow as the clique grows.

The $X$ set prevents duplicate reporting.
If $P = \emptyset$ but $X \neq \emptyset$, the current $R$ could be extended by a vertex that was already explored in an earlier branch — so $R$ is not maximal in unexplored territory, and is pruned.


## Practical considerations

### Product graph density

The XNOR predicate fires on non-adjacency as well as adjacency.
For sparse input graphs, most vertex pairs are non-adjacent, so the "both non-adjacent" clause produces many edges.
The modular product of two sparse graphs can therefore be dense, making the clique search expensive.

### Seed ordering for bounded execution

When operating under a time budget, the order in which seeds are explored determines the quality of the best clique found before timeout.
Effective heuristics include sorting seed vertices by descending degree in $G \diamond H$ (high-degree vertices participate in more potential cliques) and using a greedy graph coloring to compute upper bounds on clique size per branch — branches whose coloring bound does not exceed the best clique found so far can be pruned entirely.

Since the product vertices are pairs $(u, v)$, factor-level information can guide seed ordering without constructing the full product graph.
A simple heuristic is to prioritize $`(u, v)`$ pairs where $`\deg_G(u) \cdot \deg_H(v)`$ is large, as these have the most opportunities for XNOR matches.

### Complexity

The worst-case number of maximal cliques in a graph on $N$ vertices is $3^{N/3}$ (the Moon–Moser bound), and Bron–Kerbosch with pivoting matches this bound.
For the modular product, $N = n \cdot m$, so the worst case is $3^{nm/3}$ — doubly exponential in the factor sizes.
In practice, real graphs have far fewer maximal cliques, and pivot selection plus degeneracy ordering reduce the branching dramatically.


## Summary

The pipeline is:

1. Given graphs $G$ and $H$, construct the modular product $G \diamond H$ (vertex set $V(G) \times V(H)$, XNOR adjacency predicate).
2. Find the maximum clique in $G \diamond H$ using Bron–Kerbosch (or any clique solver).
3. The clique vertices $`(u_1, v_1), \ldots, (u_k, v_k)`$ define the maximum common induced subgraph: $`\{u_1, \ldots, u_k\}`$ in $G$ corresponds to $`\{v_1, \ldots, v_k\}`$ in $H$ with identical adjacency.

The reduction is exact, and it leverages decades of optimized clique-finding machinery to solve a problem that would otherwise require specialized subgraph-matching algorithms.
