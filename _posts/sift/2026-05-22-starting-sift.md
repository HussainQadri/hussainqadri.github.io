---
title: "Starting Sift"
date: 2026-05-22
project: sift
---

### The idea

I have started building Sift, a small Rust CLI for semantic code search.

The first target is deliberately narrow. Given one source file and a natural-language query, I want to find the functions that are most related to that query. Normal text search works well when I know the exact identifier or phrase I am looking for. It is less useful when I remember the purpose of a function but not its name.

The current command shape is:

```text
sift search <path> "<query>"
```

At this stage, `search` does all the work in one run. It parses the file, extracts functions, creates embeddings, embeds the query, compares vectors, and prints ranked results. Persistence and indexing whole folders can come later once this smaller pipeline behaves sensibly.

### Extracting functions

The first part is deciding what the searchable unit should be. For now, that unit is a function.

I am using Tree-sitter to parse source files. The project currently has language specs for Rust, Python, and C++, each with a query that captures a function node and its body node. The full function source is useful for search because it gives the embedding model more context than the signature alone.

The output should still stay compact. Printing an entire function for every result is too noisy, so extraction keeps two pieces of text together:

```text
header: the function signature to print
source: the full function text to embed
```

For example, for a Python function the result can display:

```python
def compute_yin(signal, sample_rate, W=None, threshold=0.15, freq_max=600, freq_min=200):
```

while the embedding is created from the whole function body as well.

Keeping those together matters. I initially had separate paths for extracting headers and extracting full functions. That makes it too easy to lose which header belongs to which embedded function. The extractor now produces one record per Tree-sitter match with both the printable header and the function source.

### Embeddings and ranking

The embedding layer uses `fastembed` for now. Function source text becomes vectors, and the query gets converted into another vector with the same model. Ranking is currently a direct cosine similarity comparison between the query vector and every function vector.

The first version is simple:

1. Extract functions from one file.
2. Embed each full function.
3. Embed the query string.
4. Calculate cosine similarity.
5. Sort scores from highest to lowest.
6. Print the matching function headers.

This is enough to test the main idea against real code. Searching the Rabab tuner YIN implementation for:

```text
cumulative mean normalised difference function
```

does find the relevant area, but it also shows where pure embedding search is not sharp enough yet. The function is named `cmndf`, while the query uses the long form of the acronym. A related function named `difference_function` can rank highly because those words are directly present in the identifier.

That is useful feedback. Code search is not only about semantic similarity; identifiers carry a lot of meaning too.

### Next steps

The next step is improving ranking before making the project larger.

I want to keep testing full-function embeddings, but add code-aware signals around them. The immediate candidates are identifier matching, acronym handling, and printing scores while I compare results. That should make it clearer when an embedding score is doing useful work and when a smaller exact signal should help it.

After that, the search pipeline needs a real ingest step. Re-embedding every function on each search is fine for a prototype over one file, but not for a tool that should search a codebase. The longer path is to ingest files, store function metadata and vectors, and let `search` query that stored index.
