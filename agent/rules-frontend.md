# Frontend & UI Specific Rules

## 1. Dash Framework & Component Library

* **Component Libraries (Hybrid Approach):** Utilize a combination of **Dash Bootstrap Components (DBC)** and **Dash Mantine Components (DMC)**. Choose the best component based on visual quality and functional fit for each specific use case.
* **Macro Layouts (Grid System):** Strictly use **Dash Bootstrap Components (`dbc.Container`, `dbc.Row`, `dbc.Col`)** for the overarching page structure and largest layout grids.
* **Micro Layouts (Inner Components):** Use **Dash Mantine Components (`dmc.Stack`, `dmc.Group`)** for arranging and spacing inner elements within the broader DBC layout blocks.
* **Static Assets:** Ensure all static assets (CSS stylesheets, images, fonts) are properly placed in and referenced from the root `assets/` directory.
