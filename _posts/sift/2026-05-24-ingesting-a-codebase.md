---
title: "Ingesting a codebase"
date: 2026-05-24
project: sift
---

### Separating ingestion from search

The first Sift prototype proved the retrieval loop over one file: parse functions, embed them, embed a query, and rank the results by cosine similarity. It also exposed the main architectural problem. If every search command parses and embeds the source again, then search gets slower for work that should only have happened once.

I have now split the CLI into two commands:

```text
sift ingest <directory>
sift search "<query>"
```

`ingest` handles source-code work. It scans supported files in a directory, parses them with Tree-sitter, extracts functions, creates embeddings, and writes a persistent index. `search` no longer needs a source path. It loads the saved vectors, embeds only the new query, compares that query against the stored embeddings, and prints the closest functions.

The separation is:

```text
ingest:
directory -> functions -> embeddings -> stored index

search:
query -> query embedding -> stored index -> ranked results
```

### What is stored

The indexed unit is still a function. Tree-sitter extracts the whole function source for retrieval, while also keeping the header for compact output. I have added file location metadata so a search result points back to usable code rather than only printing a function name.

Each indexed function contains:

```rust
struct IndexedFunction {
    path: String,
    header: String,
    source: String,
    line_number: usize,
    embedding: Vec<f32>,
}
```

The line number comes from the start position of the captured Tree-sitter function node. When search returns a match, Sift can now show where it lives:

```text
src/index.rs:47 pub fn load_index()
```

For now, the records are written to:

```text
.sift/index.json
```

JSON is useful at this point because I can open the index and check that the parser, metadata, and embeddings all line up. It is not intended to be the final storage format, as ingesting larger codebases will produce a lot of large embedding vectors. This can take up a lot of space.

### Current search

Search is currently exhaustive. It loads all indexed functions, calculates cosine similarity between the query vector and every stored function vector, sorts the scores from highest to lowest, and prints the top results.

```text
query embedding
-> compare against every stored function embedding
-> sort by cosine similarity
-> print top matches
```

This is deliberately simple and useful as a baseline. It gives an exact answer according to the current embedding model and cosine metric. That matters because the next stage is not only making search faster, but being able to measure whether a faster method still returns the right neighbours.

### Scaling beyond exact search

An exhaustive scan will not be enough for very large codebases. If an index contains hundreds of thousands or millions of code records, loading JSON and comparing every vector for every query becomes the wrong shape of solution.

The direction I want to explore is implementing an HNSW index myself. HNSW is an approximate nearest-neighbour algorithm that builds a layered graph of nearby vectors. Instead of checking every stored vector, a query traverses promising regions of the graph and returns likely nearest matches much faster.

The existing exact search will stay in the project as ground truth. A custom HNSW implementation only means something if I can compare it against exhaustive search and report:

```text
exact search latency
HNSW search latency
index construction time
index size
recall@5 and recall@10
```

This also means JSON is temporary. It is fine for inspectable records while the pipeline is changing. A persistent ANN index will eventually need compact binary storage for vectors and graph edges, with metadata such as paths, line numbers, headers, and source stored separately or in a small local database.
