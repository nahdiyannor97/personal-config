# Dash Mantine Components Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: Dash Mantine Components (DMC) is a high-performance library that wraps the React Mantine library for use in Plotly Dash applications [4]. It contains over 100 customizable, highly accessible, and feature-rich visual components with unified styling, automated dark/light mode support, and a robust built-in notifications system [4, 5].
- **Core Use Cases**:
  - Rapidly constructing responsive dashboard layouts (using `AppShell`, `Grid`, `SimpleGrid`, `Group`, and `Stack`) [5].
  - Creating rich user forms with powerful inputs (such as `DatePickerInput`, `NumberInput`, `MultiSelect`, `TextInput`, and `SegmentedControl`) [1, 2, 3].
  - Visualizing multi-dimensional data with native charting tools (e.g., `AreaChart`, `BarChart`, `DonutChart`, `LineChart`) [3].
- **Runtime & Environment Requirements**:
  - **Python**: 3.8+ (for modern Dash applications).
  - **Library Version**: Dash Mantine Components `2.8.0` (based on React Mantine `8.3.18`) [5].
  - Compatible with any standard WSGI/ASGI Python server deploying Plotly Dash (e.g., Gunicorn, Uvicorn).

---

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Wrap Layouts with MantineProvider**: Always wrap your root layout inside `dmc.MantineProvider` to properly distribute CSS variables, active themes, and color schemes across child components [1, 6].
- **Use Native Layout Components**: Always use responsive containers such as `AppShell`, `Grid`, `SimpleGrid`, `Group`, and `Stack` to design fast and clean layouts [5].
- **Structure AppShell Properly**: Structure complex layouts using sub-layout elements such as `AppShellHeader`, `AppShellNavbar`, `AppShellAside`, `AppShellFooter`, and `AppShellSection` inside your parent `dmc.AppShell` container [9].
- **Configure Dates Nationally or Globally**: Use `dmc.DatesProvider` to wrap and globally configure settings (locale, first day of the week) for date components like `DatePickerInput` and `DateInput` [3, 8].
- **Apply Theme Switch Dynamically**: Leverage built-in theme toggle components (e.g., `dmc.ColorSchemeToggle`) to toggle light/dark modes without manually rebuilding style sheets [2, 5].
- **Debounce Inefficient Callbacks**: Use the `debounce` prop in compatible inputs (like search bars or text inputs) to delay callback executions and reduce network overhead [4, 9].
- **Inject Custom Iconography**: Use `Dash Iconify` (`dash-iconify`) to seamlessly introduce inline vector icons into buttons, alerts, and inputs [4].

### DON'T:
- **Do Not Mix Obsolete Notification Syntax**: Do not use older v0.12 notifications methods; instead, follow the unified Migration Guide to implement the modern notifications system in V2 [4, 8].
- **Do Not Style Elements with Hardcoded CSS for Responsive Views**: Avoid writing hardcoded media queries or raw inline styling when built-in `Style Props` and grid systems (`SimpleGrid`, `Grid`) can handle responsive layouts natively [1, 5, 6].
- **Do Not Skip State Persistence**: Avoid managing standard form-reset memory manually in python callbacks when you can declare the standard `persistence` props to maintain states across page loads [4, 9].

---

## 3. Core API Signatures & Cheatsheet

### Critical Core Components & Classes
- **`dmc.MantineProvider(children, theme, forceColorScheme, ...)`**: The root provider establishing the styling context [1, 6].
- **`dmc.AppShell(children, header, navbar, padding, ...)`**: Handles global dashboard grid positioning (sidebar, header, content) [1, 9].
- **`dmc.DatePickerInput(label, description, value, type, ...)`**: High-quality component supporting single, range, or multi-date picker outputs [3, 5, 8, 9].
- **`dmc.BarChart(data, dataKey, series, ...)`**: Renders production-ready responsive charts using unified theme presets [3].

### Idiomatic Code Snippets

#### 1. Basic Setup & Global Provider Wrapper
```python
import dash
import dash_mantine_components as dmc
from dash import html

app = dash.Dash(__name__)

# The entire layout must be enclosed in dmc.MantineProvider to render properly
app.layout = dmc.MantineProvider(
    children=dmc.Container(
        children=[
            dmc.Title("Dash Mantine Components V2", order=1),
            dmc.Text("Consistent styling out-of-the-box!", size="lg"),
            dmc.Button("Click Me", color="blue", size="md"),
        ],
        size="md",
        py="xl",
    ),
    forceColorScheme="light"  # Force or toggle dark/light mode automatically
)

if __name__ == "__main__":
    app.run_server(debug=True)
```

#### 2. Advanced Responsive Grid & DatePicker Layout
```python
import dash_mantine_components as dmc
from datetime import datetime

# Build complex responsive grids with custom spacing
grid_layout = dmc.Grid(
    children=[
        dmc.GridCol(
            dmc.TextInput(
                label="Product Name", 
                placeholder="Enter item...",
                required=True
            ),
            span={"base": 12, "md": 6} # 100% width on mobile, 50% on medium screens
        ),
        dmc.GridCol(
            dmc.DatePickerInput(
                id="date-picker",
                label="Select Expiry Date",
                placeholder="Pick date",
                value=datetime.now().date(),
                clearable=True
            ),
            span={"base": 12, "md": 6}
        ),
    ],
    gutter="lg"
)
```

#### 3. Data Visualization with BarChart
```python
import dash_mantine_components as dmc

data = [
    {"month": "January", "Smartphones": 1200, "Laptops": 900},
    {"month": "February", "Smartphones": 1900, "Laptops": 1200},
]

chart_component = dmc.BarChart(
    h=300,
    data=data,
    dataKey="month",
    series=[
        {"name": "Smartphones", "color": "blue.6"},
        {"name": "Laptops", "color": "teal.6"}
    ]
)
```

---

## 4. Error Handling & Edge Cases

### Exception & Error Handling Patterns
- **Input Error States**: Leverage built-in UI error boundaries on forms. Components (like `TextInput`, `NumberInput`, `PasswordInput`) feature an `error` prop. If validation fails in a callback, return a descriptive string to the `error` property to display a red validation message instantly without raising python exceptions [5, 8].
- **Loading State Preemption**: Use `dmc.LoadingOverlay` or `Skeleton` components inside callbacks to shield charts and tables while heavy processes are running, avoiding unresponsive UI locks [2, 4, 8, 9].

### Known Limitations & Gotchas
- **Dates Range Format Issues**: When configuring `DatePickerInput` with `type="range"`, ensure your handling code validates both dates, as the list returned may temporarily contain `None` values (e.g., `[start_date, None]`) during the user's selection process [9].
- **Icon Rendering Overrides**: DMC relies heavily on `Dash Iconify` for vector rendering. Raw strings passed into button icons will not render correctly; always wrap the icon string inside a `dash_iconify.DashIconify(icon="...")` element [4].
