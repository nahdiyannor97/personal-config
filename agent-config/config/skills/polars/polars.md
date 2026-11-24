#### Polars Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Rust-backed, multi-threaded, Apache Arrow columnar DataFrame and query engine for high-throughput, out-of-core ETL and analytics.
* **Runtime & Environment Requirements:** Polars **2.x** (Python 3.9+, Rust crate `polars`, R, NodeJS). Zero mandatory external runtime dependencies; optional extras: `polars[pyarrow]`, `polars[database]`, `polars[gpu]`.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Prefer the **Lazy API** (`pl.scan_parquet`, `pl.scan_csv`, `df.lazy()`) over eager execution so the query optimizer applies predicate pushdown, projection pushdown, and common subplan elimination before `.collect()` or `.sink_parquet()`.
  * Assume **streaming execution is the default** for `LazyFrame.collect()` in Polars 2.0+: whenever output row order matters after `join`, `group_by`, or `unpivot`, explicitly call `.sort(...)` or pass `maintain_order="left"` (for joins) / `maintain_order=True` (for `group_by`).
  * Compose pure column expressions (`pl.col(...)`, `pl.lit(...)`, `pl.when(...).then(...).otherwise(...)`, window `.over(...)`) inside contexts (`select`, `with_columns`, `filter`, `group_by().agg()`).
  * Explicitly cast operands to matching dtypes before calling `.is_in(...)` (Polars 2.0 rejects lossy implicit casts).
  * Align column names and dtypes explicitly before calling `pl.concat(...)`.
  * Use `unpivot()` instead of deprecated `melt()`, and `bin_intervals()` / `bin_quantiles()` / `bin_ranks()` instead of deprecated `cut()` / `qcut()`.
* **DON'T:**
  * Do not assume row ordering is preserved across `join`, `group_by`, or `unpivot` in Polars 2.x; the default streaming engine intentionally reorders rows for throughput.
  * Do not combine `polars.selectors` (`cs.*`) with `pl.col(...)` using `&`, `|`, or `^` expecting column selection; in Polars 2.0+ this falls through to element-wise bitwise operations on compatible integer columns.
  * Do not use Python UDFs (`map_elements`) or Python `for`-loops over rows when native Polars expressions or Rust expression plugins exist (UDFs break parallelization and SIMD).
  * Do not assume `explode()` on an empty list produces a `null` row; in Polars 2.0+, exploding an empty list produces **zero rows**.
  * Do not rely on `hash()` values being stable across Polars versions or default headerless CSV column names being `column_1` (Polars 2.0 uses `column_0`).

##### 3. Core API Signatures & Cheatsheet
* `pl.scan_parquet(source: str | Path, **kwargs) -> LazyFrame` | `pl.scan_csv(source: str | Path, *, schema: SchemaDict | None = None, **kwargs) -> LazyFrame`
* `LazyFrame.filter(*predicates: IntoExpr) -> LazyFrame`
* `LazyFrame.select(*exprs: IntoExpr) -> LazyFrame` | `LazyFrame.with_columns(*exprs: IntoExpr) -> LazyFrame`
* `LazyFrame.group_by(*by: IntoExpr, maintain_order: bool = False)` -> `.agg(*aggs: IntoExpr) -> LazyFrame`
* `LazyFrame.join(other: LazyFrame, *, on: str | Expr | Sequence | None = None, how: JoinStrategy = "inner", maintain_order: MaintainOrderJoin = "none") -> LazyFrame`
* `LazyFrame.collect(*, engine: Literal["auto", "in-memory", "streaming", "gpu"] = "auto") -> DataFrame`
* `LazyFrame.sink_parquet(path: str | Path, **kwargs) -> None`
* `LazyFrame.explain(optimized: bool = True) -> str`

```python
import polars as pl
import polars.selectors as cs

lf = (
    pl.scan_parquet("data/transactions/*.parquet")
    .filter(pl.col("amount") > 0)
    .with_columns(
        net_amount=(pl.col("amount") - pl.col("discount").fill_null(0.0)),
        region_share=(pl.col("amount") / pl.col("amount").sum().over("region")),
    )
    .group_by("region", "currency")
    .agg(
        total_net=pl.col("net_amount").sum(),
        tx_count=pl.len(),
        avg_share=pl.col("region_share").mean(),
    )
    .sort("total_net", descending=True)
)

# Inspect optimized query plan and execute with default streaming engine
plan = lf.explain()
result: pl.DataFrame = lf.collect()
```

```python
import polars as pl

left = pl.LazyFrame({"id": [1, 2, 3], "val": [10, 20, 30]})
right = pl.LazyFrame({"id": [2, 3], "label": ["b", "c"]})

# Explicit order preservation under Polars 2.0 streaming engine
joined = (
    left.join(right, on="id", how="left", maintain_order="left")
    .filter(pl.col("id").cast(pl.Int64).is_in([1, 2]))
    .collect()
)
```

```rust
use polars::prelude::*;

fn run_query() -> PolarsResult<DataFrame> {
    let q = LazyCsvReader::new(PlRefPath::new("docs/assets/data/iris.csv"))
        .with_has_header(true)
        .finish()?
        .filter(col("sepal_length").gt(lit(5)))
        .group_by(vec![col("species")])
        .agg([col("*").sum()]);

    q.collect()
}
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:** Catch `polars.exceptions.PolarsError` and specific subclasses: `ColumnNotFoundError`, `SchemaError`, `ComputeError`, `InvalidOperationError`, `DuplicateError`.
* **Edge Cases & Gotchas:**
  * **Duplicate Output Names:** Expressions like `pl.datetime(...)` inherit the name of their leftmost input argument in Polars 2.0+; always chain `.alias("explicit_name")` inside `with_columns()` to prevent accidental column overwrites.
  * **Schema Alignment on `scan_csv`:** In Polars 2.0+, passing `schema={...}` matches columns by **name**, not positional index.
  * **Arithmetic Upcasting:** Adding signed integer + `UInt64` produces `Int128` (exact) rather than `Float64`; cast explicitly if downstream consumers do not support `Int128`.
  * **Force In-Memory Engine:** If migrating legacy order-dependent pipelines temporarily, override per-query via `lf.collect(engine="in-memory")` or process-wide via `POLARS_ENGINE_AFFINITY=in-memory`.

---
