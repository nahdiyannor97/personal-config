#### Dash Bootstrap Components (dbc) Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Bootstrap 5 component library for Plotly Dash providing responsive 12-column grid layouts, navigation bars, cards, modals, forms, alerts, spinners, and Bootswatch themes.
* **Runtime & Environment Requirements:** `dash-bootstrap-components>=2.0.0` (`pip install dash-bootstrap-components` or `conda install -c conda-forge dash-bootstrap-components`); strictly requires `dash>=3.0.4` (React 18+ / Dash 3+ & 4+). Requires a Bootstrap 5 stylesheet via `external_stylesheets=[dbc.themes.BOOTSTRAP]` (or any Bootswatch theme in `dbc.themes`).

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Register a Bootstrap 5 theme stylesheet explicitly in `Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])` (or include `dbc.icons.BOOTSTRAP` / `dbc.icons.FONT_AWESOME` for icons).
  * Build responsive page structures using `dbc.Container` -> `dbc.Row` -> `dbc.Col` with responsive breakpoint props (`xs`, `sm`, `md`, `lg`, `xl`, `xxl`, or `width`).
  * Use `class_name` (snake_case) consistently for Bootstrap 5 utility classes (e.g., `class_name="mb-3 d-flex justify-content-between"`).
  * In `dbc.Modal` (v2+), use `class_name` to style the modal root container (matching `style`) and `dialog_class_name` to style the inner modal dialog (matching `dialog_style`).
  * Use `dash.ctx.triggered_id` to detect button/component triggers in callbacks.
  * Use `dbc.Spinner(..., target_components={...})` for granular loading indicators targeting specific component properties.
* **DON'T:**
  * Do not install `dash-bootstrap-components<2.0` alongside `dash>=3.0` (version mismatch breaks React rendering).
  * Do not reference any `*_timestamp` properties (e.g., `n_clicks_timestamp`); all `_timestamp` props were removed in v2.0.0.
  * Do not use removed v1 props: `Modal.modal_class_name`, `DropdownMenu.right`, `DropdownMenu.addon_type`, `Table.dark`, `Navbar.light`, `NavbarSimple.light`, `Carousel.ride`, `Popover.inner_class_name`.
  * Do not omit `external_stylesheets` unless self-hosting Bootstrap 5 CSS inside the `assets/` folder; `dbc` does not bundle Bootstrap CSS automatically.

##### 3. Core API Signatures & Cheatsheet
* `dbc.Container(children=None, *, fluid: bool | str = False, class_name: str | None = None)`
* `dbc.Row(children=None, *, align: str | None = None, justify: str | None = None, class_name: str | None = None)`
* `dbc.Col(children=None, *, width: int | str | dict | None = None, xs/sm/md/lg/xl/xxl: int | str | dict | None = None)`
* `dbc.Modal(children=None, *, id: str | None = None, is_open: bool = False, size: Literal["sm", "lg", "xl"] | None = None, centered: bool = False, class_name: str | None = None, dialog_class_name: str | None = None)`
* `dbc.Spinner(children=None, *, color: str | None = None, type: Literal["border", "grow"] = "border", target_components: dict[str, str | list[str]] | None = None, display: Literal["auto", "show", "hide"] = "auto")`
* Themes & Icons: `dbc.themes.BOOTSTRAP`, `dbc.themes.FLATLY`, `dbc.themes.DARKLY`, `dbc.icons.BOOTSTRAP`.

```python
from dash import Dash, Input, Output, State, callback, html
import dash_bootstrap_components as dbc

app = Dash(__name__, external_stylesheets=[dbc.themes.FLATLY, dbc.icons.BOOTSTRAP])

app.layout = dbc.Container([
    dbc.Row([
        dbc.Col(html.H2("Analytics Console"), md=8),
        dbc.Col(
            dbc.Button([html.I(className="bi bi-gear me-2"), "Settings"], id="open-modal", color="primary"),
            md=4,
            class_name="text-md-end",
        ),
    ], align="center", class_name="my-3"),
    dbc.Row([
        dbc.Col(dbc.Card([dbc.CardHeader("KPI"), dbc.CardBody("99.95% Uptime")]), lg=4, md=6, class_name="mb-3"),
        dbc.Col(dbc.Card([dbc.CardHeader("Latency"), dbc.CardBody("42 ms")]), lg=4, md=6, class_name="mb-3"),
    ]),
    dbc.Modal([
        dbc.ModalHeader(dbc.ModalTitle("Configuration")),
        dbc.ModalBody("Adjust runtime parameters."),
        dbc.ModalFooter(dbc.Button("Close", id="close-modal", color="secondary")),
    ], id="settings-modal", is_open=False, centered=True, dialog_class_name="modal-lg"),
], fluid=True)

@callback(
    Output("settings-modal", "is_open"),
    Input("open-modal", "n_clicks"),
    Input("close-modal", "n_clicks"),
    State("settings-modal", "is_open"),
    prevent_initial_call=True,
)
def toggle_modal(open_clicks, close_clicks, is_open):
    return not is_open
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:** Standard Dash callback exception handling (`PreventUpdate`, `no_update`, `dbc.Alert(..., color="danger", is_open=True)` for user-facing validation feedback).
* **Edge Cases & Gotchas:**
  * **Modal Styling Regression (v1 -> v2):** In v2.0+, setting `class_name` on `dbc.Modal` targets the outer `.modal` wrapper, NOT `.modal-dialog`. Move dialog-level classes (such as custom widths) to `dialog_class_name`.
  * **Input Debounce:** `dbc.Input` defaults to firing on every keystroke (`debounce=False`); set `debounce=True` (blur/Enter) or `debounce=300` (ms) to prevent flooding server callbacks.
  * **Dark Mode:** Bootstrap 5.3 dark mode can be toggled dynamically via `data-bs-theme="dark"` on the root HTML element without swapping stylesheets.

---
