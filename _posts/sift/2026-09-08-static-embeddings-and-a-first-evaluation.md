---
title: "Static embeddings and a first evaluation"
date: 2026-09-08
project: sift
---

### Two loose threads

The last Sift update ended with a speed target and an unproven assumption. The speed target was a roughly ten-second first ingest of rust-analyzer, down from about two minutes. The assumption was that the embedding model surfaced the functions a person actually wanted. Recall against exact search proved the graph faithfully reproduced brute force, but if the model ranked the wrong functions highly, both searches would agree on the wrong answer.

Both threads have moved. Sift now runs on a static embedding model, and it has a relevance metric that does not involve the graph at all.

### Static embeddings

Sift previously embedded code with quantized Snowflake Arctic Embed XS running through FastEmbed. It now uses Potion Code 16M v2, a static embedding model from minishlab loaded through model2vec. Static means there is no transformer at inference time. Every token in the input maps directly to a learned vector, and the final embedding is those vectors combined. There is no attention, so encoding a function is closer to a lookup and an average than to a neural network forward pass.

I picked the 16M code model deliberately. minishlab's larger 32M Potion models exist, but they are general purpose rather than code tuned, so stepping up in size would have traded away the one property I cared about.

The switch also simplified the ingestion architecture. FastEmbed is gone, and with it the eight independent model sessions. A single shared model session now embeds batches of 1,024 sources in length order, while file discovery and Tree-sitter parsing still run in parallel. All vectors are 256-dimensional.

### What ingestion costs now

Measured today, cache-warm, second run of each:

```text
ripgrep       2,977 functions   10.52 s  ->  1.89 s
rust-analyzer 23,996 functions  123.46 s  ->  24.52 s
```

The August figures on the left are the parallel FastEmbed pipeline from the last post. ripgrep now ingests in under two seconds, about 5.6 times faster. rust-analyzer dropped from 123.46 s to 24.52 s, five times faster.

That misses the ten-second target, but the shape of the problem changed. Discovering and parsing all 23,996 rust-analyzer functions took 1.26 s of the 24.52 s, so embedding is no longer obviously the bottleneck. The rest of the time goes into inference and into writing the index files, and index.json still stores every embedding as JSON text. The process also peaked at about 1.4 cores, so the machine has headroom the old eight-session approach burned on redundant model copies.

### Judging relevance

Recall compares HNSW against exact search inside the same embedding space. It cannot tell you whether the space is any good. For that, Sift now has an `evaluate` command built on graded judgements.

A judgement file pairs a natural language query with the functions that should be retrieved, each graded from 1 (marginally relevant) to 3 (highly relevant). The command runs exhaustive search so the graph is completely out of the picture, scores each ranking with nDCG@k, and averages across queries. On nine judged queries against Sift's own codebase:

```text
Average nDCG@3:   0.942
Average nDCG@5:   0.972
Average nDCG@10:  0.972
```

This is the first number in the project that claims to measure what a person wanted rather than what the graph retrieved. The caveats are real: nine queries, judged by me, on a codebase I know intimately. A high score from the author's own judgements is a starting point, *not* a benchmark. This was to prove the nDCG pipeline, but the results shouldn't be taken seriously. In reality, with more realistic human labels nDCG will be a **LOT** lower. I believe this is my over optimisation, I thought that recall@k and ingestion speed was most important and so I used this model. Unfortunately, we are sacrificing something, and that something is semantic accuracy. This is my next hurdle and I am still trying to figure out what I can do here.

### Search speed at 256 dimensions

Benchmarking the ripgrep index with three queries at fifty runs each:

```text
Average recall@10:    1.0
HNSW median search:   0.617 ms
Exact median search:  0.777 ms
```

The honest observation is how small the gap has become. In August, with 768-dimensional Jina vectors over 23,734 functions, exact search was 4.57 times slower than HNSW. At 256 dimensions and roughly 3,000 records, brute force is nearly free: a full scan touches about 3 MB of vectors. Shrinking the dimension cut the cost of both search modes, and it cut exact search's cost the most.

HNSW still wins, and its advantage grows with corpus size, but the more interesting consequence goes the other way. With embeddings this cheap, exact search is a defensible default for small repositories, and the graph earns its complexity only at scale. That is a stronger position than "the graph is fast" on its own.

### Next

Storage is now the visible bottleneck. Every embedding is stored twice, once in index.json and once in hnsw.bin, and the JSON copy turns each 256-float vector into kilobytes of text. A compact binary representation would shrink the files and the write time with one change.

The ten-second rust-analyzer ingest still stands. It is no longer obviously an inference problem; it may be a serialization problem. And the judgement set needs to grow past my own codebase before any nDCG number deserves to be called a benchmark.
