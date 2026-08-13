# Dash Plotly Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: Dash is an open-source low-code Python framework developed by Plotly for rapidly building interactive, analytical web applications [1, 19]. It integrates Plotly.js for interactive visualizations, React.js for UI rendering, and Flask for backend routing and middleware execution.
- **Core Use Cases**:
  - Analytical dashboards, data visualization platforms, and reporting systems [1].
  - Interactive grid manipulation, styling, and server-side filtering using Dash AG Grid [4, 5] and Dash DataTable [7].
  - Scientific visualizers such as biological network graphs (Dash Bio) [8] and 3D graphics (Dash VTK) [9].
- **Runtime & Environment Requirements**:
  - Python 3.x backend environment.
  - Integration support with Jupyter notebooks (via `jupyter-dash` or workspace integrations) [17], Databricks Notebooks [13], and production ASGI/WSGI servers like Gunicorn.

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Structure App around Layout and Callbacks**: Construct the interface using hierarchical layouts (`html.Div`, `dcc.Graph`) and define interactions explicitly through declarative `@callback` (or `@app.callback`) decorators [1, 20].
- **Use the Correct Component Library**: Use **Dash AG Grid** for robust, highly interactive, high-performance tabular layouts [4, 5], **Dash Core Components (dcc)** for form inputs and interactive elements [3, 20], and **Dash HTML Components** for raw DOM tag definitions [3, 21].
- **Optimize with Clientside Callbacks**: Write clientside callbacks in JavaScript to execute simple UI updates or fast mathematical calculations directly on the client browser, minimizing network roundtrips [1, 20].
- **Define Pattern-Matching Callbacks**: Use pattern-matching callback signatures (`ALL`, `MATCH`, `ALLSMALLER`) to dynamically bind callbacks to components generated on-the-fly [1, 20].
- **Leverage Background Callbacks**: Offload long-running operations or database queries to background callbacks using Celery or disk caching to prevent freezing the server's main thread [1, 16, 20].
- **Adopt Flexible Callback Signatures**: Use type annotations and flexible parameter bindings (like `Input`, `Output`, `State`) directly in your callback functions to clean up multi-input structures [1, 20].

### DON'T:
- **Do Not Modify Global Variables Inside Callbacks**: Callbacks must be stateless and must never modify global state variable instances. Modifying globals causes race conditions and data corruption when multiple users access the app concurrently.
- **Do Not Use Deprecated Libraries**: Do not rely on deprecated component libraries such as **Dash User Analytics** or **Chatbot Builder** [12]. Use modern equivalents or integration patterns.
- **Do Not Return Full Datasets Over Callbacks**: Avoid sending large pandas DataFrames directly through callback outputs. Use **Dash Store (`dcc.Store`)** or background caching to store and transfer lightweight JSON-serialized data [1, 3, 20].
- **Do Not Duplicate Callback Outputs**: Avoid declaring multiple callbacks that target the same `Output` property of a component, unless specifically using duplicate output configurations supported in newer Dash versions [1, 20].

## 3. Core API Signatures & Cheatsheet

### Critical Core Interfaces
- **App Initializer**:
  ```python
  import dash
  app = dash.Dash(__name__, external_stylesheets=[...])
  ```
- **Callback Decorator**:
  ```python
  from dash import callback, Input, Output, State
  # @callback registers dynamic property updates based on user inputs
  ```
- **Clientside Callback**:
  ```python
  from dash import clientside_callback, ClientsideFunction
  # Executes raw JS logic inside the client browser session
  ```
- **Graph Component**:
  ```python
  from dash import dcc
  # dcc.Graph(id="graph-id", figure=fig) renders responsive Plotly.js charts
  ```

### Idiomatic Code Snippets

#### 1. A Minimal Interactive Dash App
```python
from dash import Dash, dcc, html, Input, Output, callback
import plotly.express as px

app = Dash(__name__)

# Hierarchical layouts defined cleanly using component trees
app.layout = html.Div([
    html.H1("Dynamic Plotly Chart"),
    dcc.Dropdown(
        id="dropdown-selection",
        options=[{"label": x, "value": x} for x in ["Gold", "Silver", "Bronze"]],
        value="Gold"
    ),
    dcc.Graph(id="indicator-graph")
])

@callback(
    Output("indicator-graph", "figure"),
    Input("dropdown-selection", "value")
)
def update_graph(selected_value):
    # Stateful filter calculations over clean inputs
    fig = px.bar(x=["A", "B", "C"], y=[10, 15, 8], title=f"Visualizing {selected_value}")
    return fig

if __name__ == "__main__":
    app.run(debug=True)
```

#### 2. Advanced Pattern-Matching Callbacks
```python
from dash import Dash, html, dcc, Input, Output, State, MATCH, ALL, callback

app = Dash(__name__)

app.layout = html.Div([
    html.Button("Add Filter", id="add-filter-btn", n_clicks=0),
    html.Div(id="filters-container", children=[]),
    html.Div(id="output-container")
])

@callback(
    Output("filters-container", "children"),
    Input("add-filter-btn", "n_clicks"),
    State("filters-container", "children")
)
def add_filter(n_clicks, current_filters):
    # Generates a dynamic dropdown matching a specific schema pattern
    new_filter = dcc.Dropdown(
        id={"type": "dynamic-filter", "index": n_clicks},
        options=[{"label": f"Option {i}", "value": i} for i in range(1, 4)]
    )
    current_filters.append(new_filter)
    return current_filters

@callback(
    Output("output-container", "children"),
    Input({"type": "dynamic-filter", "index": ALL}, "value")
)
def aggregate_filters(values):
    # Intercepts outputs from ALL dynamically matched filters
    selected = [val for val in values if val is not None]
    return f"Selected values: {selected}"
```

#### 3. High-Performance Tabular Display using Dash AG Grid
```python
from dash import Dash, html
import dash_ag_grid as dag
import pandas as pd

df = pd.DataFrame({
    "make": ["Toyota", "Ford", "Porsche"],
    "model": ["Corolla", "F-150", "Boxster"],
    "price": [25000, 45000, 72000]
})

app = Dash(__name__)

app.layout = html.Div([
    dag.AgGrid(
        id="ag-grid-table",
        rowData=df.to_dict("records"),
        columnDefs=[{"field": col} for col in df.columns],
        defaultColDef={"resizable": True, "sortable": True, "filter": True},
        columnSize="sizeToFit",
    )
])
```

## 4. Error Handling & Edge Cases

### Exception & Error Handling Patterns
- **PreventUpdate Exception**: To halt a callback dynamically under specific conditions without raising an unhandled stack trace or updating target layouts, raise `PreventUpdate`:
  ```python
  from dash.exceptions import PreventUpdate
  
  @callback(
      Output("output-div", "children"),
      Input("btn", "n_clicks")
  )
  def handle_click(n_clicks):
      if n_clicks is None or n_clicks == 0:
          raise PreventUpdate
      return f"Button clicked {n_clicks} times"
  ```
- **Callback Error Handlers**: Custom global error handlers can be registered to intercept uncaught backend or validation exceptions, preventing application crashes [1, 20].

### Known Limitations & Common Gotchas
- **Initial App Callback Trigger**: By default, Dash triggers all callbacks during application startup to establish structural consistency [1, 20]. Pass `prevent_initial_call=True` to the `@callback` decorator properties to disable this behavior.
- **Multithreading Thread-Safety**: Global variables must be treated as **read-only**. Any state changes must happen inside browser memory elements (`dcc.Store`) or external, thread-safe session backends (Redis caching) [1, 16, 20].
- **Circular Callback Dependencies**: Declaring a loop where Callback A outputs to Component B, and Callback B outputs back to Component A will raise a runtime circular dependency error. Design architectures as unidirectional pipelines.
