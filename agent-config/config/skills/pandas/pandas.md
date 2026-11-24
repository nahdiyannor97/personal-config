#### pandas Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** High-performance Python library for labeled 1D (`Series`) and 2D (`DataFrame`) tabular data manipulation, cleaning, time-series indexing, reshaping, and exploratory analysis.
* **Runtime & Environment Requirements:** pandas **3.1.x** (BSD-licensed); Python 3.11+ recommended; strongly recommended to install **`pyarrow`** alongside pandas to power the default Arrow-backed string dtype and fast Parquet/CSV I/O.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Write single-step assignments using `.loc[row_indexer, col_indexer] = value` or functional chaining (`.assign(...)`, `.pipe(...)`, `.query(...)`) under **Copy-on-Write (CoW)** semantics (the sole mode in pandas 3.x).
  * Check string columns using `pd.api.types.is_string_dtype(ser)` rather than comparing `ser.dtype == "object"` (pandas 3.x defaults all inferred text columns to the dedicated `str` dtype).
  * Vectorize arithmetic, boolean masks, and string/datetime operations via `.str.*`, `.dt.*`, and NumPy ufuncs.
  * Specify explicit `dtype`, `parse_dates`, and `usecols` during `read_csv` / `read_parquet` to minimize memory overhead.
  * Always pass `observed=True` when grouping by `Categorical` columns.
* **DON'T:**
  * Do not use chained assignment (`df["col"][mask] = val` or `df[mask]["col"] = val`); under pandas 3.x Copy-on-Write, chained assignment never mutates the parent `DataFrame` and raises `ChainedAssignmentError`.
  * Do not rely on `inplace=True` on views/subsets or expect `SettingWithCopyWarning` (removed in pandas 3.0).
  * Do not call defensive `.copy()` solely to silence `SettingWithCopyWarning`; indexing operations already behave as logical copies under CoW without physical memory duplication until mutated.
  * Do not iterate over rows with `df.iterrows()` for numeric or string transformations.
  * Do not use deprecated `DataFrame.append()`, `Series.swapaxes()`, or `df.groupby(..., axis=1)`.

##### 3. Core API Signatures & Cheatsheet
* `pd.read_csv(filepath_or_buffer, *, usecols: Sequence | None = None, dtype: dict | None = None, parse_dates: list | bool = False) -> DataFrame`
* `pd.read_parquet(path, *, columns: list[str] | None = None) -> DataFrame`
* `DataFrame.loc[row_indexer, col_indexer]` | `DataFrame.iloc[row_pos, col_pos]`
* `DataFrame.assign(**kwargs) -> DataFrame`
* `DataFrame.groupby(by, *, as_index: bool = True, observed: bool = True, dropna: bool = True)` -> `.agg(named_agg_dict)`
* `DataFrame.merge(right: DataFrame, *, how: Literal["left", "right", "outer", "inner", "cross"] = "inner", on: str | list[str] | None = None, validate: str | None = None) -> DataFrame`

```python
import pandas as pd

df = pd.DataFrame({
    "region": ["APAC", "APAC", "EMEA", "EMEA"],
    "revenue": [120.0, 250.0, 90.0, 310.0],
    "units": [4, 10, 3, 12],
})

# Pandas 3.x: 'region' automatically uses dedicated `str` dtype (PyArrow-backed if installed)
assert pd.api.types.is_string_dtype(df["region"])

# Copy-on-Write compliant single-step mutation
df.loc[df["units"] >= 10, "tier"] = "enterprise"
df["tier"] = df["tier"].fillna("standard")

# Method chaining with named aggregations
summary = (
    df.assign(unit_price=lambda d: d["revenue"] / d["units"])
    .groupby(["region", "tier"], as_index=False, observed=True)
    .agg(
        total_revenue=("revenue", "sum"),
        avg_unit_price=("unit_price", "mean"),
        order_count=("units", "size"),
    )
    .sort_values("total_revenue", ascending=False)
)
```

```python
import pandas as pd

orders = pd.DataFrame({"order_id": [1, 2], "cust_id": [10, 20], "amount": [50.0, 75.0]})
customers = pd.DataFrame({"cust_id": [10, 20], "name": ["Alice", "Bob"]})

# Enforce join cardinality deterministically with `validate`
enriched = orders.merge(customers, on="cust_id", how="left", validate="many_to_one")
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:**
  * Handle `KeyError` (missing column/index labels), `pd.errors.MergeError` (failed `validate=` cardinality check), `pd.errors.EmptyDataError` / `pd.errors.ParserError` (malformed I/O), and `pd.errors.ChainedAssignmentError` (invalid CoW chained assignment).
* **Edge Cases & Gotchas:**
  * **Missing Values in `str` vs Numeric Dtypes:** In pandas 3.x, `str` columns use `np.nan` / `pd.NA` consistently for missing strings; use `pd.isna(x)` rather than `x is None` or `x == np.nan`.
  * **Silent Fan-Out on Merges:** Always pass `validate="one_to_one"`, `"many_to_one"`, or `"one_to_many"` to `.merge()` to fail fast on unexpected duplicate keys.
  * **Datetime Timezones:** Comparing timezone-naive and timezone-aware `Timestamp` columns raises `TypeError`; normalize via `.dt.tz_localize("UTC")` or `.dt.tz_convert("UTC")`.

---
