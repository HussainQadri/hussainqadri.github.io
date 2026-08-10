---
title: "From NSW to HNSW"
date: 2026-08-10
project: sift
---

### A complete search path

The last Sift update ended with a single-layer NSW graph. It could insert vectors and search through neighbouring nodes, but it was still an isolated algorithm prototype: it was not hierarchical, persisted, connected to the CLI, or tested on real codebases.

That has changed. Sift now builds a custom hierarchical navigable small world index during ingestion, writes it to `.sift-index/hnsw.bin`, reloads it for queries, and uses it as the default search path. Exact cosine search is still available with `--exact`, both as a useful fallback and as the ground truth for measuring HNSW recall.

The graph now gives each node a randomized maximum layer. Search descends greedily through the sparse upper layers, then performs a wider search at layer zero. Insertion follows the same route before connecting a new node to nearby candidates.

### Fixing the graph

Getting the structure in place did not automatically produce a good index. An early benchmark on ripgrep reached only `0.66` average recall@10. Increasing the search breadth barely helped, which showed that the problem was in graph construction rather than query-time exploration.

The graph was becoming fragmented because it selected only the closest neighbours and removed useful bridge edges while pruning. I changed construction to select a more diverse set of neighbours, allow up to `2 * M` connections at layer zero, and preserve incoming connections when a node prunes its own list.

After those changes, recall@10 on ripgrep rose to roughly `0.99–1.0`. The strongest larger experiment so far used 768-dimensional quantized Jina embeddings to index 23,734 functions from rust-analyzer and produced:

```text
Average recall@10:    1.000
HNSW median search:   4.113 ms
Exact median search: 18.808 ms
Search speedup:       4.57x
```

This measures how closely HNSW reproduces exact vector search. It does not yet prove that the embedding model returned the function a person wanted, but it does show that the custom graph can search a real index quickly without losing the exact top results in that query set.

### Making ingestion faster

The rest of the ingestion pipeline has also changed. Sift now walks repositories recursively while respecting normal ignore rules, parses supported files in parallel, and indexes Java methods alongside Rust, Python, and C++ functions. It uses the quantized Snowflake Arctic Embed XS model, producing 384-dimensional embeddings from at most 256 tokens of each function.

Embedding generation was still responsible for about 90% of ingestion time, while a single model session used only part of the CPU. Sift now runs eight independent model sessions and controls the number of inference threads used by each one. On my eight-core i7-9700K, the median full ingest time for rust-analyzer changed from:

```text
Serial embedding:   170.78 s
Parallel embedding: 123.46 s
Time saved:          47.32 s
```

The parallel and serial runs produced identical function metadata and embedding vectors. The speedup costs memory—peak usage increased from about 430 MiB to 741 MiB—but it proves that ingestion can use the available CPU more effectively.

Sift also now reports discovered and repeated function bodies, rejects invalid or empty ingests without replacing an existing index, prints syntax-highlighted results, and runs CI on Linux, macOS, and Windows.

### The next target

Two minutes is still too slow for a first ingest. Incremental indexing will help later runs, but it does not solve the first experience of pointing Sift at a large repository.

The next stage is therefore speed again, beginning with CPU inference so the normal install remains useful without a GPU. A roughly ten-second first ingest for a repository the size of rust-analyzer would be an incredible result. That is an aggressive target—the current parallel approach is already beginning to hit memory-bandwidth limits—so reaching it will require more than adding threads. The next experiments will focus on the model and inference path while measuring retrieval quality against the exact-search baseline.
