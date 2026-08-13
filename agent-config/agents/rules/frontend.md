# Frontend & UI Specific Rules

## 1. Dash Framework & Component Library

* **Component Libraries (Hybrid Approach):** Utilize a combination of **Dash Bootstrap Components (DBC)** and **Dash Mantine Components (DMC)**. Choose the best component based on visual quality and functional fit for each specific use case.
* **Macro Layouts (Grid System):** Strictly use **Dash Bootstrap Components (`dbc.Container`, `dbc.Row`, `dbc.Col`)** for the overarching page structure and largest layout grids.
* **Micro Layouts (Inner Components):** Use **Dash Mantine Components (`dmc.Stack`, `dmc.Group`)** for arranging and spacing inner elements within the broader DBC layout blocks.
* **Static Assets:** Ensure all static assets (CSS stylesheets, images, fonts) are properly placed in and referenced from the root `assets/` directory.

## 2. Dash Callbacks & State Management

* **Explicit Keyword Arguments:** Always explicitly declare `component_id` and `component_property` inside `Output`, `Input`, and `State` objects.
* **Clientside Callbacks:** Always use `clientside_callback` combined with `ClientsideFunction(namespace="...", function_name="...")` for browser-side interactions.

    ```python
    clientside_callback(
        ClientsideFunction(
            namespace="...",
            function_name="...",
        ),
        Output(
            component_id="...",
            component_property="...",
        ),
        Input(
            component_id="...",
            component_property="...",
        ),
        State(
            component_id="...",
            component_property="...",
        )
    )
    ```

* **Non Clientside Callbacks:** Always use `@callback` decorator for non clientside callback on python function.

    ```python
    @callback(
        output=dict(
            output_name=Output(
                component_id="...",
                component_property="..."
            )
        ),
        inputs=dict(
            input_name=Input(
                component_id="...",
                component_property="..."
            )
        ),
        state=dict(
            state_name=State(
                component_id="...",
                component_property="..."
            )
        )
    )
    def function_name(input_name: type) -> return_type:
        """Docstring explanation here."""
        return return_value
    ```
