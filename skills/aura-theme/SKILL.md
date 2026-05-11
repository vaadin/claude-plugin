---
name: aura-theme
description: Generate and customize Vaadin Aura theme CSS configurations. Use when the user wants to create a new theme, adjust visual styles, customize a design system, pick colors, change typography, or configure the look and feel of a Vaadin application using the Aura theme. Triggers on requests like "create a theme", "make it look more modern", "use dark mode with emerald accents", "compact dense layout", or any visual design system configuration for Vaadin/Aura.
---

# Aura Theme Generator

Generate Vaadin Aura theme CSS files from free-form descriptions.

## Workflow

1. **Interpret the user's description** — identify the intended colors, mood, density, typography, layout, surface hierarchy, semantic brand colors, and any component-specific styling needs.
2. **Review relevant references** based on what was identified in step 1:
   - Always read [references/property-values.md](references/property-values.md) for valid property values and defaults.
   - Select and read the most closely matching reference CSS files as idiomatic examples. Choose based on the interpreted style:
     - [references/cozy-paper.css](references/cozy-paper.css) — dark mode, emerald accent, spacious, flat surfaces, mixed content scheme, side nav + button patterns
     - [references/fresh-mint.css](references/fresh-mint.css) — same base config as cozy-paper, clean mint variant
     - [references/monochrome.css](references/monochrome.css) — neutral monochrome, auto color scheme, opaque overlays, monospace font, button border patterns
     - [references/night-operator.css](references/night-operator.css) — dark operator theme, same nav/button patterns as cozy-paper
     - [references/preset-sunset-glass.css](references/preset-sunset-glass.css) — vibrant indigo, very rounded, heavy font weights, card elevation scoping
   - Read [references/components.md](references/components.md) only when component-specific styling (e.g., Grid) was identified in step 1.
3. **Map** each interpreted aspect to Aura properties using values from the references (always consider: surface level/opacity, app layout inset, font family, border radius; rarely: palette colors for semantic states; only when explicitly mentioned: component properties)
4. Locate the styles.css file (default: `/src/main/resources/META-INF/resources/styles.css`)
5. Name the file descriptively
6. Create the theme CSS file in the same directory as styles.css
7. Add an `@import` statement to styles.css to import the new theme file

## File Location and Integration

**Finding styles.css:**
- Default location: `/src/main/resources/META-INF/resources/styles.css`
- Search for this path first
- If not found, search for `**/styles.css` in the workspace

**Creating the theme file:**
- Create the new CSS file in the same directory where styles.css is located
- Use the descriptive filename determined in step 4

**Integrating the theme:**
- Add `@import "filename.css";` at the top of styles.css (where filename.css is the name of the generated theme file)
- If styles.css already has imports, add the new import before other content

## Interpreting Descriptions

Map user intent to Aura properties:

| User says | Property to set |
|---|---|
| dark mode, dark theme | `color-scheme: dark` |
| light mode | `color-scheme: light` |
| auto/system theme | `color-scheme: light dark` |
| mixed mode, dark nav | `color-scheme: dark` + `--aura-content-color-scheme: light` |
| color name as accent (e.g., "emerald", "sky") | `--aura-accent-color-light` + `--aura-accent-color-dark` |
| colorful, vibrant, saturated background | Use Accent background (oklch-derived from accent color) |
| warm/earthy background | Stone, Taupe, or Olive backgrounds |
| cool background | Slate, Mist, or Zinc backgrounds |
| neutral/clean | Neutral or White/Black backgrounds |
| muted, desaturated, soft colors | Reduce saturation/chroma across palette colors, user colors, and accents |
| bright, vivid, vibrant colors | Increase saturation/vibrance across palette colors, user colors, and accents |
| specific brand color for errors, warnings, success | Set corresponding palette color (e.g., `--aura-red`, `--aura-green`) |
| warmer/cooler neutral tones | Adjust `--aura-neutral-light` and `--aura-neutral-dark` |
| compact, dense | `--aura-base-size: 12` |
| spacious, roomy | `--aura-base-size: 20` |
| sharp, angular, square | `--aura-base-radius: 0` |
| slightly rounded | `--aura-base-radius: 3` (default, often omitted) |
| rounded, soft | `--aura-base-radius: 4` |
| very rounded, pill-shaped | `--aura-base-radius: 7` (use sparingly) |
| high contrast, accessible | `--aura-contrast-level: 2` |
| subtle, soft contrast | `--aura-contrast-level: 0.25` |
| flat, minimal, low surface | `--aura-surface-level: -0.5` |
| elevated, layered, card-like | `--aura-surface-level: 2` |
| semi-transparent surfaces | `--aura-surface-opacity: 0.5` (default) |
| opaque surfaces | `--aura-surface-opacity: 1` |
| small text | `--aura-base-font-size: 13` |
| large text | `--aura-base-font-size: 16` |
| monospace, code-like | Geist Mono, JetBrains Mono, or Atkinson Hyperlegible Mono |
| modern, clean font | Inter, Geist, Public Sans, or Manrope |
| accessible font | Atkinson Hyperlegible Next |
| opaque overlays, solid dialogs | `--aura-overlay-surface-opacity: 1` |
| maximized layout, edge-to-edge, no inset, full screen | `--aura-app-layout-inset: 0px` |
| default layout inset | `--aura-app-layout-inset: 1.5vmin` (default, omit) |
| allow content area to follow system light/dark | `--aura-content-color-scheme: light dark` |
| keep notifications always dark | `--aura-notification-color-scheme: dark` |
| notifications follow system | `--aura-notification-color-scheme: light dark` |

**Component-specific styling (Navigation — separate selectors, very common):**

The side nav patterns below appear in most quality themes. Apply them proactively for any polished app theme:
| Pattern | CSS |
|---|---|
| Accent icons on inactive nav items | `vaadin-side-nav-item:not([current]) > vaadin-icon { color: var(--aura-accent-text-color); }` |
| Current nav item: container background, no border | `vaadin-side-nav-item[current]::part(content) { --vaadin-side-nav-item-background: var(--vaadin-background-container); --vaadin-side-nav-item-border-color: transparent; }` |
| Nav section labels: uppercase, small, letter-spaced | `vaadin-side-nav::part(label) { text-transform: uppercase; font-size: var(--aura-font-size-xs); color: var(--vaadin-text-color); letter-spacing: 0.03em; }` |

**Component-specific styling (Buttons — separate selector, very common):**

This pattern is used in most quality themes to make secondary/default buttons feel neutral rather than accented:
```css
:is(vaadin-button, vaadin-menu-bar-button, vaadin-upload-button):not([theme~="primary"], [theme~="tertiary"]) {
  --aura-accent-color-light: var(--aura-neutral);
  --aura-accent-color-dark: var(--aura-neutral);
  --vaadin-button-background: var(--vaadin-background-container);
  --vaadin-button-border-color: transparent;
  --vaadin-button-shadow: none;
}
```
For monochrome/bordered button style (no fill, just border):
```css
html {
  --vaadin-button-border-color: var(--vaadin-border-color);
  --vaadin-button-shadow: none;
}
:is(vaadin-button, vaadin-menu-bar-button, vaadin-upload-button):not([theme~="primary"]) {
  --vaadin-button-background: var(--vaadin-background-container);
}
```

**Component-specific styling (Font weights — separate html block):**

For fonts with multiple weights (e.g., Manrope), adjust the weight scale for a heavier, more polished feel:
```css
html {
  --aura-font-weight-regular: 500;
  --aura-font-weight-medium: 600;
  --aura-font-weight-semibold: 700;
  --vaadin-button-font-weight: 600;
  --vaadin-input-field-value-font-weight: 500;
  --vaadin-checkbox-font-weight: 600;
  --vaadin-radio-button-font-weight: 600;
  --vaadin-message-name-font-weight: 600;
  --vaadin-grid-header-font-weight: 600;
}
```

**Component-specific styling (Cards/Widgets — separate selector):**

When a theme uses low global surface level but needs elevated cards:
```css
vaadin-card,
vaadin-dashboard-widget {
  --aura-surface-level: 4;
}
```

**Component-specific styling (Input Fields — applied on html):**
| User says | Property to set |
|---|---|
| customize input field appearance, borders, or background | Input field properties (`--vaadin-input-field-background`, `--vaadin-input-field-border-*`) |
| secondary accent for focus indicators | Focus ring color (`--vaadin-focus-ring-color`) |

**Component-specific styling (User Colors — applied on html):**
| User says | Property to set |
|---|---|
| customize avatar colors or chart palette | User color properties (`--vaadin-user-color-0` through `--vaadin-user-color-9`) |
| branded color palette for data visualization | Define all 10 user colors to match brand guidelines |

**Component-specific styling (Grid — separate selector):**
| User says | Property to set |
|---|---|
| customize grid borders, background, or corners | Grid container properties (`--vaadin-grid-background`, `--vaadin-grid-border-*`, `--vaadin-grid-border-radius`) |
| add/adjust row or column borders in grid | Grid structure properties (`--vaadin-grid-row-border-width`, `--vaadin-grid-column-border-width`) |
| customize grid header appearance | Grid header properties (`--vaadin-grid-header-font-size`, `--vaadin-grid-header-font-weight`, `--vaadin-grid-header-text-color`) |
| set grid row backgrounds or stripes | Grid row properties (`--vaadin-grid-row-background-color`, `--vaadin-grid-row-odd-background-color`) |

**Important considerations:**
- **Border radius:** Be conservative. Small components (buttons, inputs) use the same value. `--aura-base-radius: 7` is very rounded — only use when explicitly requested.
- **Colorful backgrounds:** When the user wants a vibrant/colorful UI, use the Accent background option with oklch formulas (see reference) instead of named backgrounds.
- **Color palette adjustment:** Adjust palette colors (`--aura-red`, `--aura-green`, etc.) to match the overall style (muted, bright, brand-specific). Keep color hues consistent (red stays red-ish, green stays green-ish) but adjust tone/saturation. Text colors are automatically derived.
- **User colors adjustment:** Adjust user color palette (`--vaadin-user-color-*`) to match the overall style for consistent visual appearance in avatars and charts.
- **Component styling:** Include side nav and button patterns proactively for polished app themes — they appear in the majority of quality reference themes. Grid properties only when explicitly mentioned.
- **Surface level/opacity:** Consider visual hierarchy needs. `--aura-surface-level: -0.5` (flat) is common in modern themes. Pair with card elevation scoping when needed.
- **Content color scheme:** `--aura-content-color-scheme: light dark` allows the content area to follow the system preference, commonly combined with a fixed dark or light `color-scheme` for the nav.
- **App layout inset:** Always use a unit (`0px` not `0`). This is required for CSS `calc()` functions.
- **Fonts:** Use only the curated fonts from the reference. Each has a pre-defined import URL.

## Output Rules

1. **Only include non-default properties.** Omit anything that matches the default value.
2. **Always pair light and dark accent colors** from the same hue family.
3. **Always pair light and dark background colors** — either by name (Zinc/Zinc) or using the Accent option with matching oklch formulas.
4. **Include `@import` for fonts** using the exact URL from the curated font list when a custom font family is set.
5. **Core Aura properties go on the `html` selector.** Additional component token overrides that apply globally may use a second `html {}` block.
6. **Include `color-scheme`** on `html` when not using the default (light).
7. **Use preset values** from the reference for sizes, radii, contrast, surface levels, and surface opacity. Custom hex colors are only allowed when the user explicitly requests a specific color not in the presets.
8. **App layout inset must include a unit** (e.g., `0px` not `0`).
9. **Include side nav and button component patterns** for any general app theme unless the user says the app has no navigation or custom button styles. These patterns come directly from the reference themes.
10. **Scope `--aura-surface-level` to `vaadin-card, vaadin-dashboard-widget`** when the global surface level is flat (`-0.5`) but cards need to stand out.

## File Naming

Name the output CSS file descriptively based on its configuration. Patterns:

- `{accent}-{background}.css` — e.g., `emerald-zinc.css`, `sky-neutral.css`
- `{accent}-{density}-{font}.css` — when density or font is distinctive
- Use lowercase, hyphen-separated names

## CSS File Structure

```css
@import url('https://fonts.googleapis.com/css2?family=FONT_NAME:wght@RANGE&display=swap');

/* Core Aura configuration */
html {
  color-scheme: VALUE;
  --aura-accent-color-dark: VALUE;
  --aura-accent-color-light: VALUE;
  --aura-app-layout-inset: VALUE;
  --aura-background-color-dark: VALUE;
  --aura-background-color-light: VALUE;
  --aura-base-font-size: VALUE;
  --aura-base-radius: VALUE;
  --aura-base-size: VALUE;
  --aura-content-color-scheme: VALUE;
  --aura-contrast-level: VALUE;
  --aura-font-family: VALUE;
  --aura-notification-color-scheme: VALUE;
  --aura-overlay-surface-opacity: VALUE;
  --aura-surface-level: VALUE;
  --aura-surface-opacity: VALUE;
  /* palette colors only when needed */
  --aura-blue: VALUE;
  --aura-green: VALUE;
  --aura-neutral-dark: VALUE;
  --aura-neutral-light: VALUE;
  --aura-orange: VALUE;
  --aura-purple: VALUE;
  --aura-red: VALUE;
  --aura-yellow: VALUE;
}

/* Side nav styling (include for most app themes) */
vaadin-side-nav-item:not([current]) > vaadin-icon {
  color: var(--aura-accent-text-color);
}

vaadin-side-nav-item[current]::part(content) {
  --vaadin-side-nav-item-background: var(--vaadin-background-container);
  --vaadin-side-nav-item-border-color: transparent;
}

vaadin-side-nav::part(label) {
  text-transform: uppercase;
  font-size: var(--aura-font-size-xs);
  color: var(--vaadin-text-color);
  letter-spacing: 0.03em;
}

/* Button styling — neutral secondary/default buttons (include for most app themes) */
:is(vaadin-button, vaadin-menu-bar-button, vaadin-upload-button):not([theme~="primary"], [theme~="tertiary"]) {
  --aura-accent-color-light: var(--aura-neutral);
  --aura-accent-color-dark: var(--aura-neutral);
  --vaadin-button-background: var(--vaadin-background-container);
  --vaadin-button-border-color: transparent;
  --vaadin-button-shadow: none;
}

/* Card/widget elevation (include when global surface-level is flat) */
vaadin-card,
vaadin-dashboard-widget {
  --aura-surface-level: 4;
}

/* Font weights (include for variable-weight fonts like Manrope) */
html {
  --aura-font-weight-regular: VALUE;
  --aura-font-weight-medium: VALUE;
  --aura-font-weight-semibold: VALUE;
  --vaadin-button-font-weight: VALUE;
  --vaadin-input-field-value-font-weight: VALUE;
  --vaadin-checkbox-font-weight: VALUE;
  --vaadin-radio-button-font-weight: VALUE;
  --vaadin-message-name-font-weight: VALUE;
  --vaadin-grid-header-font-weight: VALUE;
}

/* Input fields (only when explicitly requested) */
html {
  --vaadin-focus-ring-color: VALUE;
  --vaadin-input-field-background: VALUE;
  --vaadin-input-field-border-color: VALUE;
  --vaadin-input-field-border-radius: VALUE;
  --vaadin-input-field-border-width: VALUE;
}

/* User colors for avatars/charts (only when explicitly requested) */
html {
  --vaadin-user-color-0: VALUE;
  /* ... through --vaadin-user-color-9 */
}

/* Grid (only when explicitly requested) */
vaadin-grid {
  --vaadin-grid-background: VALUE;
  --vaadin-grid-border-color: VALUE;
  --vaadin-grid-border-radius: VALUE;
  --vaadin-grid-border-width: VALUE;
  --vaadin-grid-column-border-width: VALUE;
  --vaadin-grid-header-font-size: VALUE;
  --vaadin-grid-header-font-weight: VALUE;
  --vaadin-grid-header-text-color: VALUE;
  --vaadin-grid-row-background-color: VALUE;
  --vaadin-grid-row-border-width: VALUE;
  --vaadin-grid-row-odd-background-color: VALUE;
}
```

Omit the `@import` line if no custom font. Omit any property that uses its default value. Sort properties alphabetically within each block. Include only the sections relevant to the generated theme.

## Examples

Refer to the reference CSS files for full, complete, idiomatic examples — they are the gold standard:

- **Dark mode, emerald accent, spacious, accessible font, flat surfaces, mixed content scheme, nav + neutral buttons:** [references/cozy-paper.css](references/cozy-paper.css) and [references/night-operator.css](references/night-operator.css)
- **Neutral monochrome, auto color scheme, opaque overlays, monospace font, bordered buttons:** [references/monochrome.css](references/monochrome.css)
- **Vibrant indigo, very rounded, heavy font weights, card elevation, system color scheme:** [references/preset-sunset-glass.css](references/preset-sunset-glass.css)

**User prompt:** "Vibrant and colorful with yellow accents"

```css
html {
  --aura-accent-color-dark: #FACC15;
  --aura-accent-color-light: #efb100;
  --aura-background-color-dark: oklch(from var(--aura-accent-color-dark) 0.18 calc(c * 0.3) h);
  --aura-background-color-light: oklch(from var(--aura-accent-color-light) 0.9 calc(c * 0.3) h);
  --aura-contrast-level: 2;
}
```

**User prompt:** "Use our brand colors: #FF0000 for errors, #00AA00 for success"

```css
html {
  --aura-green: #00AA00;
  --aura-red: #FF0000;
}
```

**User prompt:** "Make the grid have rounded corners, no outer border, and add subtle column dividers"

```css
vaadin-grid {
  --vaadin-grid-border-radius: var(--vaadin-radius-m);
  --vaadin-grid-border-width: 0;
  --vaadin-grid-column-border-width: 1px;
}
```

**User prompt:** "Make input fields have a light background with a subtle border and use purple for focus indicators"

```css
html {
  --vaadin-focus-ring-color: #9810fa;
  --vaadin-input-field-background: var(--vaadin-background-container);
  --vaadin-input-field-border-color: var(--vaadin-border-color-subtle);
  --vaadin-input-field-border-width: 1px;
}
```

**User prompt:** "Use our brand color palette for avatars and charts"

```css
html {
  --vaadin-user-color-0: #2E7D32;
  --vaadin-user-color-1: #1565C0;
  --vaadin-user-color-2: #C62828;
  --vaadin-user-color-3: #F57C00;
  --vaadin-user-color-4: #7B1FA2;
  --vaadin-user-color-5: #00838F;
  --vaadin-user-color-6: #558B2F;
  --vaadin-user-color-7: #D32F2F;
  --vaadin-user-color-8: #1976D2;
  --vaadin-user-color-9: #E64A19;
}
```
