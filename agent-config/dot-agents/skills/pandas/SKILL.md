---
name: pandas
description: High-performance tabular data manipulation, vectorized data transformations, multi-format I/O, aggregation, and time-series processing using pandas. Use when writing data cleaning pipelines, DataFrame wrangling, grouping, or exporting Parquet/CSV data.
---

# Pandas Engineering Skill

This skill governs data wrangling, in-memory processing, and tabular data pipelines using pandas[cite: 1]. To optimize context window and memory footprint, load the detailed steering rules strictly when executing deep data processing tasks[cite: 1].

**Reference Document:** `pandas.md`[cite: 1]

---

## 1. Scope & Core Capabilities

- **Vectorized Operations:** Native vector calculations and string/datetime accessors (`.str`, `.dt`) avoiding iterative loops[cite: 1].
- **Data Indexing & Slicing:** Explicit label-based (`.loc`) and positional (`.iloc`) operations adhering to Copy-on-Write semantics[cite: 1].
- **High-Throughput I/O:** Reading and writing Parquet, CSV, Excel, and Arrow formats with memory-efficient engines (`engine="pyarrow"`, `usecols`, `chunksize`)[cite: 1].
- **Aggregations & Merging:** Relational joins (`merge`), concat pipelines, and multi-metric `groupby().agg()` calls[cite: 1].

---

## 2. Core Implementation Directives

1. **Vectorize Everything:** Strictly avoid row iterations (`for` loops or custom `.apply()` calls for basic operations)[cite: 1]. Rely on vectorized series methods and method chaining[cite: 1].
2. **Avoid Chained Assignment:** Never use ambiguous chained assignments like `df[df['col'] > 0]['val'] = x`[cite: 1]; always assign explicitly via `df.loc[..., 'val'] = x`[cite: 1].
3. **No Incremental Growth:** Do not concatenate single rows sequentially in loops[cite: 1]; collect structures in memory and instantiate the `DataFrame` once[cite: 1].
