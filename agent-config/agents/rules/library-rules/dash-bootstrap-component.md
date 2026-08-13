# Dash Bootstrap Components Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: *dash-bootstrap-components* (`dbc`) is a comprehensive component library designed for use with Plotly Dash, enabling developers to build highly responsive, Bootstrap-styled web application layouts in Python [2].
- **Core Use Cases**:
  - Constructing responsive grid layouts and robust user interfaces inside Plotly Dash [3].
  - Applying thematic styling via Bootstrap v5 and Bootswatch themes to Dash components [2, 3].
- **Runtime or Environment Requirements**:
  - Python environment with Plotly Dash installed [2].
  - Installation supported via PyPI (`pip install dash-bootstrap-components`) or Anaconda (`conda install -c conda-forge dash-bootstrap-components`) [1, 2].
  - A Bootstrap v5 compatible stylesheet must be linked to the application for proper component rendering [1, 2].

---

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Link a Compatible Stylesheet**: Always explicitly link a Bootstrap v5 compatible stylesheet when instantiating the Dash app [2]. The library does not ship with built-in CSS to preserve stylesheet design flexibility [2].
- **Use the Themes Submodule**: Utilize CDN links provided inside the `dbc.themes` submodule (e.g., `dbc.themes.BOOTSTRAP` or other Bootswatch themes) to quickly link standard stylesheets via JSDelivr [3].
- **Incorporate into App Layout**: Design layouts hierarchically, embedding the bootstrap components inside a top-level container (such as `dbc.Container`) within the standard `app.layout` [3].

### DON'T:
- **Do Not Omit CSS Stylesheets**: Never assume that simply importing the python package will apply the styling. Without an explicitly linked stylesheet, components will render without structure or Bootstrap styles [2].
- **Do Not Mix Bootstrap Versions**: Do not pair *dash-bootstrap-components* version 2 with outdated Bootstrap v4 stylesheets, as version 2 is explicitly built to target Bootstrap v5 [1, 2].

---

## 3. Core API Signatures & Cheatsheet

### Key API Patterns
- **App Instantiation with Themes**:
  ```python
  import dash
  import dash_bootstrap_components as dbc
  app = dash.Dash(external_stylesheets=[dbc.themes.BOOTSTRAP])
  ```
- **dbc.Container**:
  - Serves as the responsive wrapper for app components [3].
  - Common attributes: `children`, `className` (e.g., `"p-5"` for padding) [3].
- **dbc.Alert**:
  - Contextual feedback component [3].
  - Common attributes: `children`, `color` (e.g., `"success"`, `"info"`, `"warning"`, `"danger"`) [3].

### Idiomatic Code Snippets

#### 1. Minimal Application Layout
```python
import dash
import dash_bootstrap_components as dbc

# Initialize App with a Bootstrap CDN theme
app = dash.Dash(external_stylesheets=[dbc.themes.BOOTSTRAP])

# Define the layout utilizing a Container and responsive Alert
app.layout = dbc.Container(
    dbc.Alert("Hello Bootstrap!", color="success"),
    className="p-5",
)

if __name__ == "__main__":
    app.run()
```

#### 2. Layout Structure with Nested Components
```python
import dash
import dash_bootstrap_components as dbc

# Application initialized with the responsive BOOTSTRAP theme
app = dash.Dash(external_stylesheets=[dbc.themes.BOOTSTRAP])

# Complex modular layout structure
app.layout = dbc.Container(
    [
        dbc.Alert("Welcome to your Dash Bootstrap application!", color="info"),
        dbc.Container(
            "This nested container houses your dashboard controls and graphs.",
            className="mt-3 p-3 border rounded"
        )
    ],
    className="p-4",
)
```

---

## 4. Error Handling & Edge Cases

### Exception & Error Handling Patterns
- **Property Validation Errors**: Plotly Dash performs type-checking on properties. Supplying non-compatible arguments (such as passing an invalid list instead of a string to component keyword arguments like `color`) will trigger standard Dash property validation errors.

### Known Limitations & Gotchas
- **Missing Styling**: The most common gotcha is unstyled, broken-looking pages. This is always caused by a failure to configure the `external_stylesheets` parameter upon `dash.Dash` instantiation [2].
- **Version Compatibility**: Version 2 introduces breaking changes compared to earlier versions [1]. Verify stylesheet version alignment in the project's changelog if upgrading legacy v1 dashboards [1].
