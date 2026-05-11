# Aura Theme Property Reference

## Table of Contents
- [Color Scheme](#color-scheme)
- [Accent Color](#accent-color)
- [Color Palette](#color-palette)
- [Background Color](#background-color)
- [Contrast](#contrast)
- [Surface Level](#surface-level)
- [Surface Opacity](#surface-opacity)
- [Overlay Opacity](#overlay-opacity)
- [Border Radius](#border-radius)
- [Density / Base Size](#density--base-size)
- [Font Size](#font-size)
- [Font Family](#font-family)
- [App Layout Inset](#app-layout-inset)

**Component-specific styling:** See [components.md](components.md)

---

## Color Scheme

Set via the native `color-scheme` CSS property on `html`.

| Value | Meaning |
|---|---|
| `light` | Light mode only |
| `dark` | Dark mode only |
| `light dark` | Follow OS preference (Auto) |

For mixed mode (dark nav, light content):
```css
html {
  color-scheme: dark;
  --aura-content-color-scheme: light;
}
```

Additional properties:
- `--aura-content-color-scheme` — Color scheme for App Layout content area. Values: `light`, `dark`, `light dark`.
- `--aura-notification-color-scheme` — Color scheme for notifications. Values: `light`, `dark`, `light dark`.

Only set these when they differ from the global `color-scheme`.

---

## Accent Color

Two properties — one for light scheme, one for dark scheme.

### Light Accent Colors (for light backgrounds — need sufficient contrast)

| Name | Hex |
|---|---|
| Default | `#3266e4` |
| Neutral | `#222222` |
| Red | `#e7000b` |
| Orange | `#ca3500` |
| Amber | `#e17100` |
| Yellow | `#efb100` |
| Lime | `#497d00` |
| Green | `#008236` |
| Emerald | `#009966` |
| Teal | `#009689` |
| Cyan | `#0092b8` |
| Sky | `#0084d1` |
| Blue | `#155dfc` |
| Indigo | `#4f39f6` |
| Violet | `#7f22fe` |
| Purple | `#9810fa` |
| Fuchsia | `#a800b7` |
| Pink | `#c6005c` |
| Rose | `#ec003f` |

### Dark Accent Colors (for dark backgrounds — lighter/more vibrant)

| Name | Hex |
|---|---|
| Default | `#3266e4` |
| Neutral | `#eeeeee` |
| Red | `#F87171` |
| Orange | `#FB923C` |
| Amber | `#FBBF24` |
| Yellow | `#FACC15` |
| Lime | `#A3E635` |
| Green | `#4ADE80` |
| Emerald | `#34D399` |
| Teal | `#2DD4BF` |
| Cyan | `#22D3EE` |
| Sky | `#38BDF8` |
| Blue | `#60A5FA` |
| Indigo | `#818CF8` |
| Violet | `#A78BFA` |
| Purple | `#C084FC` |
| Fuchsia | `#E879F9` |
| Pink | `#F472B6` |
| Rose | `#FB7185` |

Light and dark accent colors are **always paired by hue** (e.g., Emerald light `#009966` with Emerald dark `#34D399`). The dark variant is a lighter/more vibrant version of the same hue to provide contrast on dark backgrounds.

Custom hex values are allowed when the user explicitly requests a specific brand color.

---

## Color Palette

Aura provides a customizable color palette consisting of neutral (grayscale) and saturated colors. These colors are used throughout the UI for semantic purposes and can be overridden to match brand colors.

### Neutral Color (Grayscale)

The neutral color forms the basis of text and border colors.

- `--aura-neutral` — Adapts to the color scheme (read-only, use specific variants below)
- `--aura-neutral-light` — Dark gray for light mode (default: dark gray ~#222)
- `--aura-neutral-dark` — Off-white for dark mode (default: off-white ~#eee)

### Saturated Colors

These palette colors are used for semantic purposes (success, error, warning, info) and visual accents throughout the UI.

**Properties:**
- `--aura-red`
- `--aura-orange`
- `--aura-yellow`
- `--aura-green`
- `--aura-blue`
- `--aura-purple`

**Default values** (approximate):
- Red: `#e7000b`
- Orange: `#ca3500`
- Yellow: `#efb100`
- Green: `#008236`
- Blue: `#155dfc`
- Purple: `#9810fa`

### Text Colors

Each saturated palette color has a corresponding text color property with enhanced contrast for use on backgrounds.

**Properties:**
- `--aura-red-text`
- `--aura-orange-text`
- `--aura-yellow-text`
- `--aura-green-text`
- `--aura-blue-text`
- `--aura-purple-text`

These text colors are computed to have sufficient contrast against the background and are read-only (derived from the saturated colors).

**When to customize:**
- Adjust saturated colors to match the overall style (muted, bright, brand-specific). Keep color hues consistent (red stays red-ish, green stays green-ish) but adjust tone and saturation.
- Customize neutral colors when adjusting the overall grayscale tone.
- Text colors are automatically derived — do not override them directly.

---

## Background Color

### Light Background Colors

| Name | Hex |
|---|---|
| Default | `#F4F5F7` |
| White | `#ffffff` |
| Slate | `#f1f5f9` |
| Gray | `#e5e7eb` |
| Zinc | `#f4f4f5` |
| Neutral | `#f5f5f5` |
| Stone | `#f5f5f4` |
| Taupe | `#f3f1f1` |
| Mauve | `#f3f1f3` |
| Mist | `#f1f3f3` |
| Olive | `#f4f4f0` |
| Accent | `oklch(from var(--aura-accent-color-light) 0.9 calc(c * 0.3) h)` |

### Dark Background Colors

| Name | Hex |
|---|---|
| Default | `#151922` |
| Black | `#000000` |
| Slate | `#131822` |
| Gray | `#15181f` |
| Zinc | `#18181b` |
| Neutral | `#171717` |
| Stone | `#1c1917` |
| Taupe | `#1d1816` |
| Mauve | `#1d161e` |
| Mist | `#161b1d` |
| Olive | `#1d1d16` |
| Accent | `oklch(from var(--aura-accent-color-dark) 0.18 calc(c * 0.3) h)` |

Light and dark backgrounds are **always paired by name** (e.g., Zinc light with Zinc dark).

**Accent Background:**
The "Accent" background option creates a colorful background tinted with the accent color using oklch color functions. This produces a vibrant, saturated look where the entire UI is infused with the accent hue.

- Light: `oklch(from var(--aura-accent-color-light) 0.9 calc(c * 0.3) h)` — lightness 0.9, chroma × 0.3
- Dark: `oklch(from var(--aura-accent-color-dark) 0.18 calc(c * 0.3) h)` — lightness 0.18, chroma × 0.3

Use this when the user requests a **colorful, vibrant, or saturated** visual theme.

---

## Contrast

Property: `--aura-contrast-level`

| Label | Value |
|---|---|
| Low | `0.25` |
| Mid (default) | `1` |
| High | `2` |

Affects computed text and border color contrast. Use these exact values.

---

## Surface Level

Property: `--aura-surface-level`

| Label | Value |
|---|---|
| Low | `-0.5` |
| Mid (default) | `1` |
| High | `2` |

Controls surface "elevation" of built-in components. Surface colors create visual hierarchy — lighter colors imply more elevation (closer to the user). In light mode, levels 3–4 result in white. In dark mode, level 8+ may cause contrast issues.

Use these exact values.

---

## Surface Opacity

Property: `--aura-surface-opacity`

| Value | Description |
|---|---|
| `0.5` | Default — semi-transparent surfaces that can layer |
| `1` | Opaque surfaces |

Transparency allows nesting the same surface color to create more sense of elevation. Only set when choosing opaque surfaces.

---

## Overlay Opacity

Property: `--aura-overlay-surface-opacity`

| Label | Value |
|---|---|
| Translucent (default) | `0.85` |
| Opaque | `1` |

Only set when choosing Opaque. Translucent is the default, so omit it.

---

## Border Radius

Property: `--aura-base-radius`

| Shape | Value |
|---|---|
| Square | `-1` |
| Slightly rounded | `0` |
| Default | `3` |
| Rounded | `4` |
| Very rounded / Pill | `7` |

Unitless number. Use these exact values.

**Important:** The same border radius applies to all components, including small ones like buttons and input fields. Be conservative with rounding — `7` makes small components fully rounded and should only be used when explicitly requested.

---

## Density / Base Size

Property: `--aura-base-size`

| Label | Value |
|---|---|
| S (compact) | `12` |
| M (default) | `16` |
| L (spacious) | `20` |

Unitless number. Controls gap and padding. Use these exact values.

---

## Font Size

Property: `--aura-base-font-size`

| Label | Value |
|---|---|
| XS | `13` |
| S (default) | `14` |
| M | `15` |
| L | `16` |

Unitless number representing base font size in px. Use these exact values.

---

## Font Family

Property: `--aura-font-family`

Format: `'Font Name', var(--aura-font-family-system)`

Always include `var(--aura-font-family-system)` as fallback. When using a font from the list below, include the corresponding `@import` rule at the top of the CSS file.

**Curated Fonts:**

| Font Name | Import URL |
|---|---|
| Inter | `https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap` |
| Roboto | `https://fonts.googleapis.com/css2?family=Roboto:wght@100..900&display=swap` |
| Public Sans | `https://fonts.googleapis.com/css2?family=Public+Sans:wght@100..900&display=swap` |
| Geist | `https://fonts.googleapis.com/css2?family=Geist:wght@100..900&display=swap` |
| Manrope | `https://fonts.googleapis.com/css2?family=Manrope:wght@200..800&display=swap` |
| Atkinson Hyperlegible Next | `https://fonts.googleapis.com/css2?family=Atkinson+Hyperlegible+Next:wght@400;500;600;700&display=swap` |
| Geist Mono | `https://fonts.googleapis.com/css2?family=Geist+Mono:wght@100..900&display=swap` |
| JetBrains Mono | `https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@100..800&display=swap` |
| Atkinson Hyperlegible Mono | `https://fonts.googleapis.com/css2?family=Atkinson+Hyperlegible+Mono:wght@400;500;600;700&display=swap` |

**Prefer fonts from this curated list.**

Example:
```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');

html {
  --aura-font-family: 'Inter', var(--aura-font-family-system);
}
```

When no font is specified, omit the property entirely (defaults to Instrument Sans).

---

## App Layout Inset

Property: `--aura-app-layout-inset`

| Label | Value |
|---|---|
| Off | `0px` |
| On (default) | `1.5vmin` |

**Must include a unit**, even for zero (`0px` not `0`).
