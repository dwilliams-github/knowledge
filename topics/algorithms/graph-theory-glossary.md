---
title: Graph Theory Glossary
topic: algorithms
tags: [graphs, graph-theory, glossary]
---

# Graph theory glossary

Core graph-theoretic terms used across the algorithms section.

**Graph.** A structure $G = (V, E)$ consisting of a set of **vertices** (or nodes) $V$ and a set of **edges** (or links) $E$.
Each edge connects two vertices.

**Simple graph.** A graph with no self-loops (an edge from a vertex to itself) and no parallel edges (multiple edges between the same pair of vertices).

**Undirected graph.** A graph where edges have no direction; the edge $\{u, v\}$ is the same as $\{v, u\}$.

**Directed graph (digraph).** A graph where each edge is an ordered pair $(u, v)$, representing a one-way connection from $u$ to $v$.

**Weighted graph.** A graph where each edge carries a scalar value (its weight).

**Adjacent.** Two vertices $u$ and $v$ are adjacent if an edge exists between them. Written $u \sim v$, or equivalently $\{u, v\} \in E$.

**Adjacency relation.** The binary relation $\sim$ on $V$ defined by the edge set: $u \sim v \iff \{u, v\} \in E$. When multiple graphs are in play, subscripts disambiguate: $`u \sim_G v`$ means "adjacent in $G$."

**Degree.** The number of edges incident to a vertex. In a digraph, **in-degree** and **out-degree** count incoming and outgoing edges separately.

**Path.** A sequence of edges connecting distinct vertices.

**Cycle.** A path that returns to its starting vertex.

**Connected.** A graph is connected if every pair of vertices has a path between them.

**Clique.** A subset of vertices that are all mutually adjacent — every pair is connected by an edge. A clique of size $k$ is sometimes written $K_k$. The **maximum clique** is the largest clique in a graph; finding it is NP-hard.

**Bipartite.** A graph whose vertices can be partitioned into two sets such that every edge connects a vertex in one set to a vertex in the other.

**Adjacency matrix.** The $|V| \times |V|$ matrix $A$ where $`A_{ij} = 1`$ (or the edge weight) if vertices $i$ and $j$ are adjacent, and 0 otherwise.

**Factor.** In the context of graph products, one of the input graphs being multiplied — analogous to multiplicands in arithmetic.

**Component (of a product vertex).** One element of the pair $(u, v)$ in $V(G) \times V(H)$. The first component $u$ belongs to $V(G)$; the second component $v$ belongs to $V(H)$.
