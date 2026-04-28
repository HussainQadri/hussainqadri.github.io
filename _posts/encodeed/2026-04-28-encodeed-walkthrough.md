---
title: "EncodeEd: Project Walkthrough"
date: 2026-04-28
project: encodeed
hide_date: true
---

EncodeEd is a Python desktop application I built for my A-level Computer Science project. It is a learning tool for exploring how lossless compression algorithms work, with a focus on showing the steps behind each method rather than just returning a compressed output.

The app supports several compression algorithms, including Run-Length Encoding, Huffman Coding, Shannon-Fano Coding, LZW, Arithmetic Coding, and LZ77. Each algorithm can compress user-provided text and show a breakdown of what is happening internally.

### Walkthrough

The user starts by selecting a compression algorithm from the left-hand panel. They can either type text directly into the input box or load a text file into the application.

After pressing **Compress**, the app displays the compressed output and a step-by-step explanation of how the selected algorithm processed the input. For example, Huffman Coding shows the generated codebook, while dictionary-based methods such as LZW show how repeated patterns are represented more compactly.

Some algorithms also include visual output. Huffman Coding can display the generated Huffman tree, and Shannon-Fano can show how symbols are split into groups. The app also runs timing tests over increasing input sizes and plots a runtime graph, giving a rough indication of how the algorithm scales. This of course, isn't representative of the actual **theoretical** runtime complexity, its purely empirical. There are so many overheads, unrelated to the algorithm itself that can dominate the algorithm's time taken. Empirical runtimes will differ between machines, the graph is useful to show that the time taken does scale with input size but doesn't necessarily match the theoretical complexity.

### What it demonstrates

The project covers a range of compression ideas:

- Frequency-based encoding with Huffman Coding and Shannon-Fano
- Repeated character compression with Run-Length Encoding
- Dictionary-based compression with LZW
- Sliding-window compression with LZ77
- Probability interval encoding with Arithmetic Coding

The main aim was to make these algorithms easier to understand by connecting the final compressed result to the decisions made during the process.

### Implementation

The project is written in Python with a PyQt5 interface. The algorithms are implemented directly rather than relying on compression libraries, because the purpose of the project was to understand and demonstrate the logic behind them.

### Reflection

As an A-level project, EncodeEd was mainly about learning how to turn theory into a working application. The most useful part was implementing the algorithms from scratch and then building explanations around them so that the output was not just a black box.

The project is not designed to be a production compression tool. It is an educational visualiser, and that is where it works best.
