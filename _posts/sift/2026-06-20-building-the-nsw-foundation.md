---
title: "Building the NSW foundation"
date: 2026-06-20
project: sift
---

### Starting below HNSW

The last Sift update ended with a plan to move beyond exhaustive vector search and build a custom HNSW index. I initially named the development branch `hnsw`, but that name got ahead of the implementation. What I have built so far is the single-layer graph search that HNSW grows from: a navigable small world, or NSW, graph.

That distinction matters. HNSW is hierarchical. It assigns nodes to different levels and uses the sparse upper layers to move quickly across the index before searching more carefully near the bottom. My current index has one layer, one entry point, and a graph of neighbouring vectors. It is not HNSW yet, but it contains several of the core operations that the hierarchical version will reuse.

The current structure is roughly:

```rust
struct Node {
    id: usize,
    embedding: Vec<f32>,
    neighbours: Vec<usize>,
}

struct HnswIndex {
    nodes: Vec<Node>,
    entry_point: Option<usize>,
    m: usize,
    ef: usize,
}
```

Each node owns an embedding and stores the IDs of connected nodes. The index keeps an entry point for graph traversal, `m` for the number of neighbours selected during insertion, and `ef` for the breadth of the candidate search.

The type is still called `HnswIndex` in the prototype, although `NswIndex` would describe its current behaviour more accurately.

### Inserting nodes into the graph

The first insertion is straightforward. The new node becomes node zero and the graph entry point:

```text
empty graph
-> insert node 0
-> entry point = node 0
```

Later insertions need to find a useful place in the existing graph. The index starts from the entry point, searches for nearby reachable nodes, takes the best `m` candidates, and connects the new node to them bidirectionally.

### Moving from greedy search to a search layer

My first traversal was greedy. It compared the query with the current node, moved to the most similar neighbour when that neighbour improved the score, and stopped at a local maximum.

```text
entry point
-> best immediate neighbour
-> best improving neighbour
-> stop when no neighbour is better
```

This is useful for understanding graph traversal, but it is easy for a greedy search to stop too early at a local maximum.<sup>*</sup> Stopping there can reduce the quality of the final result because the search does not explore alternative paths through the graph.

The current implementation now uses `search_layer()`, which keeps a wider set of possibilities open. It maintains two heaps:

```text
candidates: promising nodes that still need exploring
best_found: best ef nodes found so far
```

The candidate heap prioritises the highest similarity score, so the most promising unexplored node is visited first. The `best_found` heap uses the reverse ordering, keeping its worst retained result at the top. That makes it cheap to decide whether a newly visited neighbour deserves a place in the current result set.

The search begins by adding the entry point to both heaps and marking it as visited. It then repeatedly removes the best candidate, inspects its neighbours, calculates their cosine similarity to the query, and adds useful unvisited nodes. Once a neighbour improves on the worst retained result, it can enter the bounded `best_found` set.

The `ef` parameter controls how many strong candidates the search keeps:

```text
smaller ef -> less graph exploration, lower search cost
larger ef  -> more graph exploration, better chance of high recall
```

This is one of the important HNSW trade-offs. Approximate search is not simply "fast search"; it is a tunable balance between work and recall. Keeping exact cosine search in Sift means I will be able to measure that balance instead of guessing whether the approximate results look reasonable.

### What this is not yet

The current work is an algorithm prototype rather than a replacement for Sift's exact search. It is not connected to the ingest or query commands, it is not persisted, and it has not been evaluated on real code embeddings yet.

It also lacks the features that make HNSW hierarchical:

- random level assignment;
- multiple neighbour lists per node, one for each level;
- traversal from sparse upper layers to the dense base layer;
- bounded neighbour pruning;
- separate `ef_construction` and `ef_search` settings; and
- a public top-k search interface that returns indexed record IDs.

For now, exact search remains the production path and the correctness baseline. That is useful rather than temporary duplication: without exact top-k results, I cannot calculate recall for the approximate index.

### Next steps

The immediate next step is making the single-layer graph more complete. Existing nodes need their neighbour lists pruned when reverse connections push them beyond `m`, and construction breadth should be separated from query-time breadth.

After that, the index can become genuinely hierarchical. Nodes will receive randomized maximum levels, store neighbours per layer, and use greedy traversal through the upper layers before running the broader `search_layer()` operation at level zero.

<small><sup>*</sup> A lot of ANN literature describes this as a local minimum because it minimizes distance. I am maximizing cosine similarity instead: a higher similarity score is equivalent to a smaller cosine distance, so I use local maximum.</small>
