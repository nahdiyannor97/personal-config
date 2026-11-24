#### Plotly Dash Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Low-code reactive Python web framework for analytical data apps, interactive visualizations, multi-page dashboards, and MCP-enabled AI tool apps (Dash MCP).
* **Architecture:** Flask/Werkzeug (or ASGI/WSGI server backends) + React frontend + Plotly.js graphics + reactive callback DAG.
* **Runtime & Environment Requirements:** Python 3.9+; Dash 3.x / Dash 4.x (`pip install "dash>=4"` for redesigned WCAG 2.2 Dash Core Components and React 18/19 runtime). Optional `orjson` for accelerated callback JSON serialization, `dash-ag-grid` for large tabular datasets, Celery/Diskcache for background callbacks.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Import all core primitives from the unified top-level namespace: `from dash import Dash, dcc, html, Input, Output, State, callback, ctx, Patch, no_update, MATCH, ALL`.
  * Instantiate the application with `Dash(__name__)` so static assets (`assets/` directory) resolve deterministically across deployment environments, and expose `server = app.server` for WSGI production runners (e.g., Gunicorn).
  * Start the local dev server with `app.run(debug=True)`.
  * Keep callbacks pure and stateless: share user/session state across callbacks using `dcc.Store` (memory/session/local) or server-side caches (Redis), never global Python variables.
  * Use `Patch()` for partial property updates (e.g., appending traces or updating figure layout properties) instead of re-serializing entire figures or large dicts.
  * Use `ctx.triggered_id` to inspect which input triggered a multi-input callback.
  * Use `background=True` callbacks with a `DiskcacheManager` or `CeleryManager` for long-running computations, and `dash-ag-grid` (`dag.AgGrid`) over `dash_table.DataTable` for large production tables.
  * Use `clientside_callback` for lightweight, zero-latency browser-side UI updates.
* **DON'T:**
  * Do not call `app.run_server(...)` (removed in Dash 3.0+; use `app.run(...)`).
  * Do not use legacy standalone imports (`import dash_core_components`, `import dash_html_components`, `import dash_table`).
  * Do not use `@app.long_callback` (removed in Dash 3.0+; use `@callback(..., background=True)`).
  * Do not mutate global variables or in-memory DataFrames inside callbacks; concurrent users/workers will experience race conditions and state corruption.
  * Do not register multiple callbacks targeting the exact same `(component_id, property)` output without explicitly setting `allow_duplicate=True` AND `prevent_initial_call=True` (or `"initial_duplicate"`).

##### 3. Core API Signatures & Cheatsheet
* `Dash(name: str, *, use_pages: bool = False, external_stylesheets: list[str | dict] | None = None, suppress_callback_exceptions: bool = False, background_callback_manager: BaseBackgroundCallbackManager | None = None)`
* `@callback(*args, prevent_initial_call: bool | Literal["initial_duplicate"] = False, running: list[tuple[Output, Any, Any]] | None = None, background: bool = False, on_error: Callable[[Exception], Any] | None = None)`
* `Output(component_id: str | dict, component_property: str, allow_duplicate: bool = False)`
* `Input(component_id: str | dict, component_property: str)`
* `State(component_id: str | dict, component_property: str)`
* `Patch()` -> supports item/attribute assignment, `.append()`, `.extend()`, `.clear()`, `del` for partial prop updates.
* `dash.register_page(module: str, *, path: str | None = None, name: str | None = None)` (used with `dash.page_container` when `use_pages=True`).

```python
from dash import Dash, Input, Output, Patch, State, callback, ctx, dcc, html, no_update
from dash.exceptions import PreventUpdate
import plotly.express as px

app = Dash(__name__)
server = app.server

df = px.data.gapminder()

app.layout = html.Div([
    dcc.Dropdown(
        id="country-select",
        options=sorted(df["country"].unique()),
        value="Indonesia",
        clearable=False,
    ),
    html.Button("Highlight Peak", id="highlight-btn", n_clicks=0),
    dcc.Graph(id="gdp-graph"),
])

@callback(
    Output("gdp-graph", "figure"),
    Input("country-select", "value"),
)
def update_graph(country: str):
    if not country:
        raise PreventUpdate
    filtered = df[df["country"] == country]
    return px.line(filtered, x="year", y="gdpPercap", title=f"{country} GDP per Capita")

@callback(
    Output("gdp-graph", "figure", allow_duplicate=True),
    Input("highlight-btn", "n_clicks"),
    prevent_initial_call=True,
)
def update_title_color(n_clicks: int):
    if ctx.triggered_id != "highlight-btn" or not n_clicks:
        return no_update
    patched_figure = Patch()
    patched_figure["layout"]["title"]["font"] = {"color": "crimson"}
    return patched_figure

if __name__ == "__main__":
    app.run(debug=True)
```

```python
from dash import ALL, Input, Output, callback, ctx, html

@callback(
    Output("summary-out", "children"),
    Input({"type": "filter-input", "index": ALL}, "value"),
)
def summarize_dynamic_inputs(values: list[str | None]) -> str:
    triggered = ctx.triggered_id
    active = [v for v in values if v]
    return f"Triggered by {triggered}; active count: {len(active)}"
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:**
  * Raise `dash.exceptions.PreventUpdate` to abort updating *all* outputs of a callback cleanly without raising an error in the UI.
  * Return `dash.no_update` for individual outputs in a multi-output callback when only a subset of outputs should change.
  * Use `on_error=handler_fn` in `@callback(...)` or global `Dash(..., on_error=global_handler)` (Callback Error Handlers) to intercept callback exceptions and call `set_props(component_id, props_dict)` to display user-friendly error notifications.
* **Edge Cases & Gotchas:**
  * **Dynamic Layouts:** Components added dynamically via callbacks do not exist in the initial layout; set `suppress_callback_exceptions=True` or use `app.validation_layout` to avoid initial callback validation errors.
  * **Multi-Output Return Shape:** When multiple `Output`s are passed as a list/tuple, the callback must return a tuple/list of matching length; a single `Output` wrapped in a 1-element list requires returning a 1-element list/tuple.
  * **JSON Serialization:** All callback inputs/outputs and `dcc.Store` data must be JSON-serializable; serialize DataFrames as records (`df.to_dict("records")`) or store server-side with a cache key in `dcc.Store`.

---
