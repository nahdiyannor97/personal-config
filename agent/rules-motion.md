# Motion & UI Animation Rules (Fluke Motion Skill)

## 1. Core Motion Philosophy

* **Single System & Restraint:** All UI animations in a project MUST share a single timing clock and easing scale. Never add motion for the sake of decoration—motion must convey physics, state change, or focus.
* **Ponytail Alignment:** Prioritize native CSS transitions and transform/opacity properties over heavy JavaScript animation libraries. Do not add external motion dependencies if standard CSS or built-in UI component transition props (e.g., Mantine/Bootstrap transitions) can achieve the effect.

## 2. Standard Tokens (Timing & Easing)

### Durations
* **Micro-interactions (Hover, Click, Toggle):** `150ms` - `200ms`
* **Medium Transitions (Dropdown, Tooltip, Modal, Toast):** `250ms` - `350ms`
* **Macro Transitions (Page load, Layout shift, Accordion expansion):** `400ms` - `500ms`

### Easing Curves (Cubic-Bezier)
* **Emphasized / Natural (Standard):** `cubic-bezier(0.2, 0.0, 0.0, 1.0)` (Fast entrance, smooth deceleration)
* **Decelerate (Entrances / Dialog open):** `cubic-bezier(0.0, 0.0, 0.2, 1.0)`
* **Accelerate (Exits / Dialog close):** `cubic-bezier(0.4, 0.0, 1.0, 1.0)`
* **Spring / Elastic (Buttons, Badges):** `cubic-bezier(0.34, 1.56, 0.64, 1)`

## 3. Performance & Compositor Rules (60 FPS Budget)

* **Compositor-Only Rule:** ONLY animate GPU-accelerated properties: `transform` (translate, scale, rotate) and `opacity`.
* **Prohibited Animated Properties:** NEVER animate layout-triggering properties such as `width`, `height`, `margin`, `padding`, `top`, `left`, or `border-width`. Use CSS `transform: scale()` or CSS Grid fraction transitions instead.
* **Hardware Acceleration:** Apply `will-change: transform, opacity;` sparingly and only on actively animated elements. Remove it post-animation if managed via JavaScript.

## 4. Accessibility First (Mandatory)

* Every custom CSS animation or transition MUST include a `prefers-reduced-motion` media query fallback that completely disables motion or replaces it with a subtle cross-fade (`opacity` only).

    ```css
    @media (prefers-reduced-motion: reduce) {
      *, ::before, ::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
      }
    }
    ```

## 5. Dash & Frontend Integration Rules

* **Clientside Execution:** Complex interaction animations in Dash MUST be handled browser-side using clientside_callback or custom CSS in assets/ to eliminate server-roundtrip latency.
* **Mantine Component Transitions:** Prefer using dmc built-in transition props (e.g., transitionProps={{ duration: 200, transition: 'fade' }}) before writing custom clientside JS.
