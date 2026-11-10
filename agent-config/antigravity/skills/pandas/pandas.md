# pandas Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: pandas is an open-source Python data analysis and manipulation library providing high-performance, flexible data structures (`DataFrame` and `Series`) for structured tabular data processing, I/O operations, cleaning, transformation, and aggregation.
- **Core Use Cases**:
  - In-memory structured data wrangling, cleaning, indexing, filtering, and joining.
  - Ingestion and export of data across formats (CSV, Parquet, Excel, SQL, JSON, Feather).
  - Time-series manipulation, resampling, rolling window aggregations, and pivot operations.
- **Runtime / Environment Requirements**:
  - Python 3.10+ (for pandas 3.0+ / 3.1+).
  - Relies on NumPy, Apache Arrow (for PyArrow-backed engines/dtypes), and openpyxl/pyarrow backend engines.

---

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Vectorize Operations**: Use native vector operations (e.g., `df["a"] + df["b"]` or `.str`, `.dt` accessors) instead of iterating over rows using Python `for` loops or `.apply()` with custom Python functions.
- **Use `.loc` and `.iloc` for Explicit Indexing**: Always access and assign DataFrame subsets using `.loc[row_indexer, col_indexer]` (label-based) or `.iloc[row_indexer, col_indexer]` (positional) to avoid implicit view/copy ambiguity.
- **Specify PyArrow / Explicit Dtypes**: Prefer PyArrow backend string and numeric dtypes (`dtype="string[pyarrow]"` or PyArrow engine in `read_csv`) for significantly faster memory allocation and vectorized operations.
- **Chain Methods Cleanly**: Use method chaining (`df.query(...).assign(...).groupby(...)`) with lambda expressions for readable data pipelines.
- **Optimize File I/O Engine**: Use `engine="pyarrow"` when reading CSV files or export to column-oriented formats like Parquet (`df.to_parquet()`) for high-throughput I/O.

### DON'T:
- **Do Not Iteratively Append Rows in a Loop**: Avoid calling `df.append()` (deprecated/removed) or repeatedly concatenating single rows via `pd.concat()`; collect dicts or lists in memory and instantiate the `DataFrame` once.
- **Do Not Rely on `SettingWithCopyWarning` Silence**: Avoid chained assignments like `df[df['col'] > 0]['val'] = 5`; always use `.loc` directly: `df.loc[df['col'] > 0, 'val'] = 5`.
- **Do Not Use `.apply()` for Simple Math or String Functions**: Do not use `df['col'].apply(lambda x: x.lower())`; use vectorized series accessors such as `df['col'].str.lower()`.
- **Do Not Ignore Memory Management on Large Files**: Avoid loading massive datasets without specifying `usecols`, `chunksize`, or optimized categorical/integer data types.

---

## 3. Core API Signatures & Cheatsheet

### Critical Core Functions & Methods
- **`pd.read_csv(filepath_or_buffer, usecols=None, dtype=None, engine=None, ...)`**: Ingests CSV data into a DataFrame.
- **`df.loc[row_selector, col_selector]`**: Label-based indexing and assignment.
- **`df.groupby(by, as_index=True).agg(func)`**: Groups rows and applies aggregate functions.
- **`df.merge(right, how='inner', on=None, left_on=None, right_on=None)`**: Performs SQL-style database joins.
- **`df.assign(**kwargs)`**: Creates or overrides columns in a method-chain pipeline.

### Idiomatic Code Snippets

#### 1. Vectorized Data Transformation Pipeline with Method Chaining
```python
import pandas as pd
import numpy as np

# Load data with optimized types
df = pd.DataFrame({
    "department": ["Engineering", "Engineering", "Sales", "Sales", "HR"],
    "salary": [120000, 140000, 90000, 110000, 85000],
    "bonus_pct": [0.15, 0.20, 0.25, 0.10, 0.05],
    "hire_date": ["2021-03-15", "2019-06-01", "2022-01-10", "2020-11-05", "2023-02-18"]
})

# Vectorized transformation pipeline
processed_df = (
    df.assign(
        hire_date=lambda d: pd.to_datetime(d["hire_date"]),
        total_compensation=lambda d: d["salary"] * (1 + d["bonus_pct"]),
        department=lambda d: d["department"].astype("category")
    )
    .query("total_compensation > 90000")
    .sort_values(by="total_compensation", ascending=False)
)
```

#### 2. Advanced GroupBy Aggregation and Merging
```python
import pandas as pd

# 1. Perform multi-column aggregate stats
dept_stats = (
    processed_df.groupby("department", observed=True)
    .agg(
        avg_salary=("salary", "mean"),
        max_comp=("total_compensation", "max"),
        employee_count=("salary", "count")
    )
    .reset_index()
)

# 2. Merge aggregates back onto original data
final_df = processed_df.merge(dept_stats, on="department", how="left")
```

---

## 4. Error Handling & Edge Cases

### Common Exceptions & Handling
- **`KeyError`**: Raised when attempting to access a non-existent column name or index label. Ensure index alignment or use `.get()` / check `if col in df.columns`.
- **`ParserError`**: Raised during `read_csv` when file formatting is malformed (e.g., mismatched delimiters or line fields). Handle by setting `on_bad_lines='skip'` or specifying explicit `sep`.
- **`ValueError: Cannot set a frame with an non-aligned panel`**: Raised when assigning a Series with a non-matching index to a DataFrame column. Align indexes using `.reset_index(drop=True)` or convert to NumPy values (`.values`).

### Known Limitations & Gotchas
- **In-Place Operations (`inplace=True`)**: Operations using `inplace=True` are generally discouraged in modern pandas because they rarely improve performance or memory usage and break method chaining pipelines.
- **Copy vs View Behavior (PyArrow/Copy-on-Write)**: In modern pandas (pandas 3.0+ defaults to Copy-on-Write), mutating a slice derived from another DataFrame without `.loc` will not modify the parent object and may raise errors. Always use `.copy()` explicitly if you intend to modify a subset independently.
