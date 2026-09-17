\---

name: polars

description: High-performance data manipulation and query optimization standards using Polars. Use when building, refactoring, or optimizing data processing pipelines, ETL workflows, and dataframe transformations in Python.

\---



\# Polars Data Engineering \& Transformation Skill



When this skill is activated, you must enforce the Rust-backed performance optimizations, memory safety principles, and execution guidelines defined for this project.



\## Execution Instruction



Before designing, writing, or refactoring any data processing pipelines:



1\. Use the `view\_file` tool to inspect the full steering rules in `polars.md`.

2\. Prioritize `LazyFrame` pipelines (`scan\_parquet`, `scan\_csv`, `.lazy()`) to ensure predicate/projection pushdown runs prior to `.collect()`.

3\. Enforce native Polars expressions (`pl.col()`, `pl.lit()`, `.over()`) across all `.select()` and `.with\_columns()` chains—strictly avoid Python-level row iteration (`iter\_rows()`) and unvectorized `.map\_elements()` functions.

4\. Validate schema transformations explicitly with `.cast()`, isolate Categorical types using `pl.StringCache()` where joins/concats occur, and properly disambiguate `null` versus `NaN` handling.

