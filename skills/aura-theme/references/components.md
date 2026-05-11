# Component Styling Reference

Component-specific CSS custom properties allow fine-grained control over individual component appearance. Only use these when the user explicitly requests component-specific styling.

## Table of Contents
- [Input Fields](#input-fields)
- [User Colors](#user-colors)
- [Grid](#grid)

---

## Input Fields

Input field styling properties control the appearance of all input components (TextField, TextArea, ComboBox, DatePicker, etc.). These properties are applied at the root level and affect all input field components.

**Apply these properties on the `html` selector** (not component-specific selectors).

### Input Field Properties

- `--vaadin-input-field-background` — Background color of input fields
- `--vaadin-input-field-border-color` — Border color of input fields
- `--vaadin-input-field-border-radius` — Corner radius of input fields
- `--vaadin-input-field-border-width` — Border width of input fields
- `--vaadin-focus-ring-color` — Color of the focus ring (suitable for secondary accent color)

### When to Use

- **Background/border properties:** When customizing the visual style of all input fields
- **Border radius:** When matching input fields to the overall theme's border radius
- **Focus ring color:** When using a secondary accent color or customizing focus indicators

### Usage Note

These properties target all input components globally. Apply them in the `html {}` block alongside other theme properties, not in component-specific selectors.

---

## User Colors

User color properties define a palette of 10 colors used in Avatars (for color-coded user identification) and charts. These properties are applied at the root level and should be customized when brand colors or specific color palettes are required for data visualization.

**Apply these properties on the `html` selector** (not component-specific selectors).

### User Color Properties

- `--vaadin-user-color-0` — First user/chart color
- `--vaadin-user-color-1` — Second user/chart color
- `--vaadin-user-color-2` — Third user/chart color
- `--vaadin-user-color-3` — Fourth user/chart color
- `--vaadin-user-color-4` — Fifth user/chart color
- `--vaadin-user-color-5` — Sixth user/chart color
- `--vaadin-user-color-6` — Seventh user/chart color
- `--vaadin-user-color-7` — Eighth user/chart color
- `--vaadin-user-color-8` — Ninth user/chart color
- `--vaadin-user-color-9` — Tenth user/chart color

### When to Use

- **Avatar identification:** When customizing the color palette used for user avatars without images
- **Chart colors:** When defining a color palette for data visualizations
- **Style consistency:** Adjust these colors to match the overall theme style (muted, bright, brand-specific)

### Usage Note

These properties target Avatars and chart components globally. Adjust them to match the overall color style of the theme for visual consistency. Apply them in the `html {}` block alongside other theme properties.

---

## Grid

Grid styling properties control the appearance of data tables.

### Grid Container

Properties for the overall grid container appearance:

- `--vaadin-grid-background` — Background color of the entire grid
- `--vaadin-grid-border-width` — Width of the outer border
- `--vaadin-grid-border-color` — Color of the outer border
- `--vaadin-grid-border-radius` — Corner radius of the grid

### Grid Structure

Properties for internal grid structure and dividers:

- `--vaadin-grid-row-border-width` — Width of horizontal borders between rows
- `--vaadin-grid-column-border-width` — Width of vertical borders between columns

### Grid Header

Properties for grid header styling:

- `--vaadin-grid-header-font-size` — Font size of header text
- `--vaadin-grid-header-font-weight` — Font weight of header text
- `--vaadin-grid-header-text-color` — Text color of headers

### Grid Rows

Properties for row appearance:

- `--vaadin-grid-row-background-color` — Background color of all rows (default state)
- `--vaadin-grid-row-odd-background-color` — Background color of odd rows (for striping)

### When to Use

- **Grid container properties:** When customizing grid borders, background, or shape
- **Grid structure properties:** When adding/adjusting row or column borders
- **Grid header properties:** When customizing header appearance
- **Grid row properties:** When setting row backgrounds or stripes

### Properties Not Included

The following properties are excluded from the skill (per requirements):

- Hover states (`--vaadin-grid-row-hover-background-color`)
- Selected states (`--vaadin-grid-row-selected-background-color`)
- Cell-specific properties (`--vaadin-grid-cell-*`)
- Resize handle (`--vaadin-grid-column-resize-handle-color`)
