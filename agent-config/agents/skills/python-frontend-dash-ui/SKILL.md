---
name: dash-ui
description: Frontend and analytical dashboard engineering using Plotly Dash, Dash Mantine Components (DMC), and Dash Bootstrap Components (DBC). Use when building reactive dashboards, charting interfaces, interactive callbacks, pattern-matching updates, or responsive UI layouts.
---

# Dash UI & Dashboard Engineering Skill

This skill orchestrates Python-based frontend and analytical dashboard development. To maintain context efficiency and conserve token limits, **never load all documentation files at once**. Identify the specific layout library or reactivity pattern required and read only the corresponding module file using file reading tools.

---

## 1. Domain Detection & Module Routing

Inspect the task and load the appropriate `.md` module before generating or refactoring UI components:

| Framework / Component Focus | Module to Read | Key Responsibilities |
| :--- | :--- | :--- |
| **Core Dash, Callbacks & AG Grid** | `dash-plotly.md` | Reactive `@callback` definitions, Clientside Callbacks, Pattern-Matching (`MATCH`, `ALL`), `dcc.Graph`, `dash_ag_grid`, `dcc.Store` state management, and `PreventUpdate`. |
| **Modern Mantine UI & Forms (DMC)** | `dash-mantine-components.md` | `dmc.MantineProvider` setup, `AppShell` layouts, `Grid`/`SimpleGrid`, inputs (`DatePickerInput`, `MultiSelect`), DMC charts, and dark/light color schemes. |
| **Bootstrap v5 Layouts (DBC)** | `dash-bootstrap-component.md` | `dash-bootstrap-components`, `dbc.themes` CDN setup, `dbc.Container`, responsive grid columns, and contextual alerts. |

---

## 2. Integrated Dashboard Construction Pipeline

When building an end-to-end dashboard interface:

1. **Framework & Layout Selection:**
   * If building with Mantine components, read `dash-mantine-components.md` and wrap the root in `dmc.MantineProvider`.
   * If building with Bootstrap, read `dash-bootstrap-component.md` and ensure `external_stylesheets=[dbc.themes.BOOTSTRAP]` is passed to `dash.Dash`.
2. **Component & Visualization Placement:**
   * Read `dash-plotly.md` to define `dcc.Graph` elements, Plotly figures, or `dash_ag_grid.AgGrid` tables.
3. **Interactivity & Callbacks:**
   * Read `dash-plotly.md` to wire up `@callback` pipelines, handle state via `dcc.Store`, or apply `PreventUpdate` to guard against empty/null triggers.

---

## 3. Strict Development Directives

* **Stateless Callbacks:** Never modify global Python variables inside callbacks; pass state through `dcc.Store` or browser memory.
* **Style Provider Guard:** Always verify that the required provider (`dmc.MantineProvider`) or stylesheet (`dbc.themes`) is loaded at app initialization.
* **Performance Optimization:** Use Clientside Callbacks for fast UI manipulation and add `debounce=True` or `prevent_initial_call=True` where appropriate to minimize network roundtrips.
