#### Dash Mantine Components (DMC) Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Comprehensive React UI component library for Plotly Dash built on **Mantine v9** (`mantine==9.7.4`), offering 100+ accessible components (`AppShell`, `Grid`, `SimpleGrid`, `Splitter`, `Select`/`MultiSelect`, `DatePicker`, `Notifications`, `RichTextEditor`, Styles API, Style Props, and native light/dark color schemes).
* **Runtime & Environment Requirements:** `dash-mantine-components==3.0.0` strictly requires **`dash>=4.5`** and **React 19** (`19.2.4`). (Use DMC v2.x for Dash 2/3 with Mantine v8.)

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Wrap the entire `app.layout` inside a single root `dmc.MantineProvider(...)` (configuring `theme`, `forceColorScheme`, or `defaultColorScheme`).
  * Use compound components for `dmc.AppShell`: `dmc.AppShellHeader`, `dmc.AppShellNavbar`, `dmc.AppShellMain`, `dmc.AppShellAside`, `dmc.AppShellFooter` (supports `mode="static"` and `layout="alt"` in v3).
  * Use `gap` (and `rowGap`/`columnGap`) on `dmc.Grid`, `dmc.Group`, `dmc.Stack`, `dmc.Flex`, and `dmc.SimpleGrid`.
  * Leverage Style Props (`p`, `m`, `w`, `h`, `bg`, `c`, `fz`, `fw`, `bd`, `bdrs`) and the `styles` / `classNames` Styles API for fine-grained theming instead of ad-hoc inline CSS overrides.
  * Mount `dmc.NotificationContainer()` (or `dmc.NotificationProvider`) once at app root when triggering programmatic notifications.
  * Pass primitive `str`, `int`, or `bool` values directly to `Select`, `MultiSelect`, `SegmentedControl`, and `CheckboxGroup` (supported natively in v3.0.0).
  * Use `dmc.add_figure_templates()` (or Plotly figure templates from DMC) to synchronize Plotly graphs with Mantine light/dark themes.
* **DON'T:**
  * Do not render DMC components outside of `dmc.MantineProvider`; theming, CSS variables, and overlays will fail to render properly.
  * Do not use removed/renamed pre-v3 props or components:
    * Do not use `gutter` on `dmc.Grid` (renamed to `gap` in v3.0.0).
    * Do not use `dmc.TypographyStylesProvider` (renamed to `dmc.Typography` in v3.0.0).
    * Do not use `initialState` on `dmc.Spoiler` (renamed to `defaultExpanded`).
    * Do not use `positionDependencies` on `Popover`/`Tooltip` (removed in v3.0.0).
    * Do not use `width`/`height`/`caption` on `dmc.Image` (use `w`, `h`, and `fallbackSrc`).
    * Do not use `dash._dash_renderer._set_react_version(...)` on Dash 3+/4+ (handled automatically).

##### 3. Core API Signatures & Cheatsheet
* `dmc.MantineProvider(children=None, *, theme: dict | None = None, forceColorScheme: Literal["light", "dark"] | None = None, defaultColorScheme: Literal["light", "dark", "auto"] = "light")`
* `dmc.AppShell(children=None, *, header: dict | None = None, navbar: dict | None = None, aside: dict | None = None, footer: dict | None = None, padding: str | int | dict = "md", mode: Literal["fixed", "static"] = "fixed", layout: Literal["default", "alt"] = "default")`
* `dmc.Grid(children=None, *, gap: str | int | dict = "md", rowGap: str | int | None = None, columnGap: str | int | None = None, columns: int = 12)` + `dmc.GridCol(children=None, *, span: int | str | dict = 12, align: str | None = None)`
* `dmc.SimpleGrid(children=None, *, cols: int | dict | None = None, minColWidth: str | int | None = None, spacing: str | int = "md")`
* `dmc.Select(id=None, *, label: str | None = None, data: list[str | dict] = [], value: str | int | bool | None = None, searchable: bool = False, clearable: bool = False, error: str | bool | None = None)`
* `dmc.Splitter(children=None, *, orientation: Literal["horizontal", "vertical"] = "horizontal")`

```python
from dash import Dash, Input, Output, callback
import dash_mantine_components as dmc

app = Dash(__name__)

app.layout = dmc.MantineProvider(
    defaultColorScheme="auto",
    theme={"primaryColor": "indigo", "defaultRadius": "md"},
    children=dmc.AppShell(
        header={"height": 60},
        navbar={"width": 260, "breakpoint": "sm"},
        padding="md",
        children=[
            dmc.AppShellHeader(
                dmc.Group(
                    [dmc.Title("DMC v3 Console", order=3), dmc.ColorSchemeToggle()],
                    justify="space-between",
                    h="100%",
                    px="md",
                )
            ),
            dmc.AppShellNavbar(
                dmc.Stack([
                    dmc.Select(
                        id="env-select",
                        label="Environment",
                        data=[
                            {"value": "prod", "label": "Production"},
                            {"value": "staging", "label": "Staging"},
                        ],
                        value="prod",
                        allowDeselect=False,
                    ),
                    dmc.NumberInput(id="limit-input", label="Row Limit", value=100, min=10, max=5000),
                ], gap="sm"),
                p="md",
            ),
            dmc.AppShellMain(
                dmc.Grid(
                    gap="md",
                    children=[
                        dmc.GridCol(dmc.Card(id="status-card", withBorder=True, p="lg"), span={"base": 12, "md": 6}),
                        dmc.GridCol(dmc.Card("Metrics", withBorder=True, p="lg"), span={"base": 12, "md": 6}),
                    ],
                )
            ),
        ],
    ),
)

@callback(
    Output("status-card", "children"),
    Input("env-select", "value"),
    Input("limit-input", "value"),
)
def render_status(env: str, limit: int | None):
    return dmc.Text(f"Active environment: {env} (limit={limit})", fw=500)
```

##### 4. Error Handling & Edge Cases
* **Official Exception & Validation Pattern:** Surface input validation errors directly on form controls by returning a string message to the component's `error` prop (or `False`/`None` when valid), and use `dmc.Alert` / `dmc.Notification` for action-level errors.
* **Edge Cases & Gotchas:**
  * **Dash Version Pinning:** DMC 3.0.0 strictly enforces `dash>=4.5` (React 19). Mixing DMC 3 with Dash 2/3 causes runtime React hook/reconciler crashes.
  * **Notifications Hover Behavior (v3):** Hovering over any notification pauses the auto-close timer for *all* visible notifications by default; set `pauseResetOnHover="notification"` to pause only the hovered item.
  * **SimpleGrid `minColWidth`:** When `minColWidth` is provided on `dmc.SimpleGrid`, the `cols` prop is ignored.
  * **CSS Attribute Casing (`mod`):** The `mod` prop automatically converts camelCase keys to kebab-case `data-*` attributes in v3.

---
