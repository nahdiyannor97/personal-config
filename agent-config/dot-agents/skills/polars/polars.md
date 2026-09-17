# Polars Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: Polars is an extremely fast, multi-threaded DataFrame library written in Rust with Python bindings. It leverages Apache Arrow memory format and query optimization techniques (such as predicate pushdown, projection pushdown, and expression simplification) to perform high-performance data processing.
- **Core Use Cases**:
  - High-speed data manipulation, filtering, aggregation, and joins on large structured datasets.
  - Out-of-core and memory-efficient streaming data processing via `LazyFrame`.
  - Fast I/O operations with Parquet, CSV, IPC (Feather), Delta Lake, JSON, and relational databases.
  - Type-safe, vectorized data expressions without Python GIL bottlenecks.
- **Runtime & Environment Requirements**:
  - **Python 3.8+**.
  - Powered by Rust core engine with Apache Arrow memory allocation.

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Prefer Lazy Execution (`LazyFrame`)**: Use `polars.scan_parquet()`, `polars.scan_csv()`, or `df.lazy()` over eager reading (`polars.read_*`). This allows Polars to build and optimize execution plans, push down filters/projections, and drastically reduce memory usage prior to calling `.collect()`.
- **Use Polars Expressions (`polars.col()`, `polars.lit()`)**: Perform data operations inside `.select()`, `.with_columns()`, `.filter()`, and `.group_by().agg()` using native Polars expressions instead of Python loops or lambdas.
- **Leverage Expression Parallelism**: Place independent transformations in a list or pass as arguments inside `.select()` or `.with_columns()`. Polars runs independent expressions concurrently across available thread pools.
- **Use Window Functions (`.over()`)**: Calculate group-level metrics alongside original rows using `.over("group_column")` rather than manually performing `group_by()` and merging results back.
- **Explicitly Cast Dtypes**: Use `.cast(pl.DataType)` when handling mixed data types or enforcing strict schema conformance across datasets.

### DON'T:
- **Do Not Use `.apply()` / `map_elements()` Unnecessarily**: Avoid passing custom Python functions into `.map_elements()` or `.map_batches()` unless native expressions cannot achieve the task. Python function calls bypass Rust vectorization and reintroduce GIL locks.
- **Do Not Iterate Over Rows**: Never use `for row in df.iter_rows()` or convert to dictionaries for processing.
- **Do Not Use Slicing Syntax for Columns**: Avoid `df['col']` or `df[['col1', 'col2']]` inside transformation chains. Use `df.select(pl.col('col'))` or `df.with_columns(...)`.
- **Do Not Mutate Data In-Place**: Polars DataFrames and LazyFrames are immutable transformation pipelines that return new objects.
- **Avoid Unnecessary Eager Conversions**: Do not call `.to_pandas()` or `.to_numpy()` in the middle of a processing pipeline unless interfacing directly with an external library.

## 3. Core API Signatures & Cheatsheet

### Critical Core Methods & Functions
- `polars.scan_parquet(source, ...)` / `polars.scan_csv(source, ...)` -> `LazyFrame`: Lazy scanners for Parquet and CSV files.
- `polars.col(name: str | List[str] | DataType)` -> `Expr`: Refers to columns in an expression chain.
- `polars.lit(value: Any)` -> `Expr`: Creates a literal expression.
- `LazyFrame.collect(streaming: bool = False)` -> `DataFrame`: Evaluates and executes the lazy query plan into memory.
- `DataFrame.select(*exprs: IntoExpr)` -> `DataFrame`: Selects or computes columns.
- `DataFrame.with_columns(*exprs: IntoExpr)` -> `DataFrame`: Adds or overwrites columns.
- `DataFrame.filter(predicate: Expr)` -> `DataFrame`: Filters rows based on a boolean expression.
- `DataFrame.group_by(*keys: str | Expr).agg(*exprs: Expr)` -> `DataFrame`: Groups data and computes aggregations.
- `DataFrame.join(other: DataFrame | LazyFrame, on: str | Expr | List, how: str = 'inner')` -> `DataFrame`: Joins two data structures.

### Idiomatic Code Snippets

#### 1. Lazy Pipeline with Filtering, Transformation & Aggregation
```python
import polars as pl

# 1. Build optimized lazy execution query
query = (
    pl.scan_parquet("data/*.parquet")
    .filter(pl.col("status") == "ACTIVE")
    .filter(pl.col("amount") > 100.0)
    .with_columns(
        (pl.col("amount") * 1.1).alias("amount_with_tax"),
        pl.col("timestamp").dt.date().alias("transaction_date")
    )
    .group_by("category", "transaction_date")
    .agg(
        pl.col("amount_with_tax").sum().alias("total_revenue"),
        pl.col("transaction_id").count().alias("transaction_count"),
        pl.col("amount").mean().alias("avg_amount")
    )
    .sort("total_revenue", descending=True)
)

# 2. Execute plan with optimal Rust query engine
df: pl.DataFrame = query.collect()
```

#### 2. Window Calculations with `.over()`
```python
import polars as pl

df = pl.DataFrame({
    "department": ["HR", "HR", "IT", "IT", "IT"],
    "employee": ["Alice", "Bob", "Charlie", "David", "Eve"],
    "salary": [60000, 80000, 90000, 110000, 95000]
})

# Perform group-level window aggregations without explicit joins
result = df.with_columns(
    pl.col("salary").mean().over("department").alias("dept_avg_salary"),
    (pl.col("salary") - pl.col("salary").mean().over("department")).alias("salary_diff_from_dept_avg"),
    pl.col("salary").rank(descending=True).over("department").alias("dept_salary_rank")
)
```

#### 3. Joins & Data Unpivoting
```python
import polars as pl

df1 = pl.LazyFrame({"id": [1, 2, 3], "val1": ["A", "B", "C"]})
df2 = pl.LazyFrame({"id": [1, 2, 4], "val2": [10, 20, 40]})

# Join lazy frames and unpivot (melt) columns
unpivoted_df = (
    df1.join(df2, on="id", how="left")
    .unpivot(
        on=["val1", "val2"],
        index=["id"],
        variable_name="metric",
        value_name="metric_value"
    )
    .collect()
)
```

## 4. Error Handling & Edge Cases

### Exception & Error Handling Patterns
- **`polars.exceptions.PolarsError`**: Base exception class for all Polars errors.
- **`polars.exceptions.ColumnNotFoundError`**: Raised when referencing a non-existent column name inside an expression.
- **`polars.exceptions.ComputeError`**: Raised during invalid runtime operations (e.g., dividing string columns or type mismatches). Catch explicitly when executing dynamic queries:
  ```python
  import polars as pl
  from polars.exceptions import ComputeError, ColumnNotFoundError

  try:
      df = pl.read_csv("data.csv")
      result = df.select(pl.col("non_existent_column"))
  except ColumnNotFoundError as e:
      print(f"Column selection failed: {e}")
  except ComputeError as e:
      print(f"Computation error during execution: {e}")
  ```
- **`polars.exceptions.SchemaError`**: Raised when column schemas do not align during `concat()` or `join()` calls.
- **`polars.exceptions.PolarsInefficientMapWarning`**: Emitted when `.map_elements()` is called for operations that can be written using native Polars expressions.

### Limitations & Common Gotchas
- **Categorical Mismatches Across Datasets**: When joining or concatenating multiple LazyFrames containing Categorical types, wrap operations in `with pl.StringCache():` to maintain a global string dictionary and avoid `StringCacheMismatchError`.
- **Null vs. NaN Disambiguation**: Polars distinguishes between `null` (missing data, native Apache Arrow) and `NaN` (floating-point Not-a-Number). Operations like `.drop_nulls()` will not drop `NaN` values; use `.drop_nans()` or `.fill_nan()` explicitly.
- **Strict Cast Conversions**: `.cast()` defaults to `strict=True`, raising `ComputeError` if any value cannot be converted. Pass `strict=False` to turn unparseable values into `null`.