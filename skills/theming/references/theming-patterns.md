# Theming Patterns Quick Reference

## Aura Core Properties

| Property | Purpose | Default | Range/Notes |
|----------|---------|---------|-------------|
| `--aura-font-family` | Application font | Instrument Sans | Any font stack |
| `--aura-base-font-size` | Base font size (unitless, px) | 16 | Computes xs–xl scale |
| `--aura-base-line-height` | Base line height (unitless) | 1.5 | Relative to font size |
| `--aura-base-size` | Controls gap/padding (unitless) | 16 | Range 12–24 |
| `--aura-base-radius` | Controls border radius (unitless) | 6 | Range 0–10 |
| `--aura-font-smoothing` | Font smoothing | antialiased | Set `auto` to disable |

## Aura Color Palette

| Property | Purpose | Light/Dark variants |
|----------|---------|-------------------|
| `--aura-neutral` | Neutral/gray tones | — |
| `--aura-red` | Error, destructive | — |
| `--aura-orange` | Accent | — |
| `--aura-yellow` | Warning | — |
| `--aura-green` | Success | — |
| `--aura-blue` | Info, links | — |
| `--aura-purple` | Accent | — |
| `--aura-accent-color` | Primary action color | `-light` / `-dark` |
| `--aura-background-color` | Page background | `-light` / `-dark` |

### Aura Surface System

| Property | Purpose | Default |
|----------|---------|---------|
| `--aura-surface-color` | Computed surface bg (read-only) | — |
| `--aura-surface-level` | Elevation level | 1 |
| `--aura-surface-opacity` | Surface transparency | 0.5 |

### Aura Text Properties

| Property | Purpose |
|----------|---------|
| `--aura-text-color` | Default text color |
| `--aura-text-color-secondary` | Secondary/muted text |

## Lumo Color Tokens

### Primary
| Token | Usage |
|-------|-------|
| `--lumo-primary-color` | Buttons, links, active indicators |
| `--lumo-primary-color-50pct` | Hover backgrounds, selections |
| `--lumo-primary-color-10pct` | Subtle backgrounds, highlights |
| `--lumo-primary-text-color` | Primary-colored text |
| `--lumo-primary-contrast-color` | Text on primary background |

### Semantic
| Token | Usage |
|-------|-------|
| `--lumo-error-color` / `-50pct` / `-10pct` / `-text-color` | Errors, destructive actions |
| `--lumo-success-color` / `-50pct` / `-10pct` / `-text-color` | Success states |
| `--lumo-warning-color` / `-50pct` / `-10pct` / `-text-color` | Warnings |

### Surface & Contrast
| Token | Usage |
|-------|-------|
| `--lumo-base-color` | Page/app background |
| `--lumo-contrast-5pct` | Subtle backgrounds, zebra striping |
| `--lumo-contrast-10pct` | Borders, dividers |
| `--lumo-contrast-20pct` | Stronger borders |
| `--lumo-contrast-50pct` | Disabled text |
| `--lumo-contrast-60pct` | Secondary text |
| `--lumo-contrast-70pct` | Icons |
| `--lumo-contrast-80pct` | Subheadings |
| `--lumo-contrast-90pct` | Body text |
| `--lumo-contrast` | Headings, emphasis |

### Text Colors (Lumo Utility Classes)
| Class | Maps to |
|-------|---------|
| `LumoUtility.TextColor.HEADER` | `--lumo-header-text-color` |
| `LumoUtility.TextColor.BODY` | `--lumo-body-text-color` |
| `LumoUtility.TextColor.SECONDARY` | `--lumo-secondary-text-color` |
| `LumoUtility.TextColor.TERTIARY` | `--lumo-tertiary-text-color` |
| `LumoUtility.TextColor.DISABLED` | `--lumo-disabled-text-color` |
| `LumoUtility.TextColor.PRIMARY` | `--lumo-primary-text-color` |
| `LumoUtility.TextColor.ERROR` | `--lumo-error-text-color` |
| `LumoUtility.TextColor.SUCCESS` | `--lumo-success-text-color` |
| `LumoUtility.TextColor.WARNING` | `--lumo-warning-text-color` |

## Lumo Typography Tokens

| Token | Default | Usage |
|-------|---------|-------|
| `--lumo-font-family` | System stack | Base font |
| `--lumo-font-size-xxxl` | 2rem | Hero headings |
| `--lumo-font-size-xxl` | 1.5rem | Page titles |
| `--lumo-font-size-xl` | 1.25rem | Section headings |
| `--lumo-font-size-l` | 1.125rem | Subheadings |
| `--lumo-font-size-m` | 1rem | Body text |
| `--lumo-font-size-s` | 0.875rem | Small text, labels |
| `--lumo-font-size-xs` | 0.8125rem | Captions |
| `--lumo-font-size-xxs` | 0.75rem | Micro text |
| `--lumo-line-height-m` | 1.625 | Body line height |
| `--lumo-line-height-s` | 1.375 | Compact line height |
| `--lumo-line-height-xs` | 1.25 | Dense line height |

## Lumo Spacing Tokens

| Token | Default | Utility Class |
|-------|---------|---------------|
| `--lumo-space-xs` | 0.25rem | `LumoUtility.Padding.XSMALL` / `Gap.XSMALL` |
| `--lumo-space-s` | 0.5rem | `LumoUtility.Padding.SMALL` / `Gap.SMALL` |
| `--lumo-space-m` | 1rem | `LumoUtility.Padding.MEDIUM` / `Gap.MEDIUM` |
| `--lumo-space-l` | 1.5rem | `LumoUtility.Padding.LARGE` / `Gap.LARGE` |
| `--lumo-space-xl` | 2.5rem | `LumoUtility.Padding.XLARGE` / `Gap.XLARGE` |

## Lumo Shadow Tokens

| Token | Utility Class | Usage |
|-------|---------------|-------|
| `--lumo-box-shadow-xs` | `BoxShadow.XSMALL` | Subtle depth |
| `--lumo-box-shadow-s` | `BoxShadow.SMALL` | Cards, panels |
| `--lumo-box-shadow-m` | `BoxShadow.MEDIUM` | Hover states, dropdowns |
| `--lumo-box-shadow-l` | `BoxShadow.LARGE` | Dialogs, popovers |
| `--lumo-box-shadow-xl` | `BoxShadow.XLARGE` | Modal overlays |

## Lumo Border Radius Tokens

| Token | Default | Utility Class |
|-------|---------|---------------|
| `--lumo-border-radius-s` | 0.25em | `BorderRadius.SMALL` |
| `--lumo-border-radius-m` | 0.5em | `BorderRadius.MEDIUM` |
| `--lumo-border-radius-l` | 1em | `BorderRadius.LARGE` |

## Component Theme Variants Comparison

### Button
| Variant | Aura | Lumo |
|---------|------|------|
| Primary | `AURA_PRIMARY` | `LUMO_PRIMARY` |
| Tertiary | `AURA_TERTIARY` | `LUMO_TERTIARY` |
| Tertiary inline | — | `LUMO_TERTIARY_INLINE` |
| Small | — | `LUMO_SMALL` |
| Large | — | `LUMO_LARGE` |
| Icon | — | `LUMO_ICON` |
| Error | `AURA_ERROR` | `LUMO_ERROR` |
| Success | `AURA_SUCCESS` | `LUMO_SUCCESS` |
| Warning | `AURA_WARNING` | `LUMO_WARNING` |
| Contrast | — | `LUMO_CONTRAST` |

### Grid
| Variant | Aura | Lumo |
|---------|------|------|
| No border | `AURA_NO_BORDER` | `LUMO_NO_BORDER` |
| No row borders | `AURA_NO_ROW_BORDERS` | `LUMO_NO_ROW_BORDERS` |
| Compact | `AURA_COMPACT` | `LUMO_COMPACT` |
| Column borders | `AURA_COLUMN_BORDERS` | `LUMO_COLUMN_BORDERS` |
| Wrap cell content | `AURA_WRAP_CELL_CONTENT` | `LUMO_WRAP_CELL_CONTENT` |

### TextField / TextArea
| Variant | Aura | Lumo |
|---------|------|------|
| Small | — | `LUMO_SMALL` |
| Helper above | — | `LUMO_HELPER_ABOVE_FIELD` |
| Align center | — | `LUMO_ALIGN_CENTER` |
| Align right | — | `LUMO_ALIGN_RIGHT` |

### Badge Component (Vaadin 25.1+)

Badge is now a proper component (preview feature, requires `badgeComponent` feature flag):

| Variant | Effect |
|---------|--------|
| _(default)_ | Neutral badge |
| `BadgeVariant.SUCCESS` | Green/success |
| `BadgeVariant.ERROR` | Red/error |
| `BadgeVariant.WARNING` | Yellow/warning |
| `BadgeVariant.FILLED` | Filled background |
| `BadgeVariant.ICON_ONLY` | Icon without text (text still needed for accessibility) |
| `BadgeVariant.NUMBER_ONLY` | Number display only |

Use `badge.addThemeVariants(BadgeVariant.SUCCESS)`. Combine variants: `badge.addThemeVariants(BadgeVariant.SUCCESS, BadgeVariant.FILLED)`.

> **Deprecated:** The theme-attribute approach (`theme="badge success"` on `Span`) from earlier Vaadin versions is deprecated. Use the `Badge` component instead.

## Theme Setup Recipes

### Aura App Setup

```java
@StyleSheet(Aura.STYLESHEET)
@StyleSheet("styles.css")
@ColorScheme(ColorScheme.Value.LIGHT_DARK) // follow OS preference
public class Application implements AppShellConfigurator {
}
```

```css
/* styles.css */
html {
    --aura-font-family: 'Inter', sans-serif;
    --aura-base-font-size: 15;
    --aura-base-size: 16;
    --aura-base-radius: 8;
    --aura-accent-color-light: #2563eb;
    --aura-accent-color-dark: #60a5fa;
}
```

### Lumo App Setup

```java
@StyleSheet(Lumo.STYLESHEET)
@StyleSheet(Lumo.UTILITY_STYLESHEET)
@StyleSheet("styles.css")
public class Application implements AppShellConfigurator {
}
```

```css
/* styles.css */
html {
    --lumo-font-family: 'Inter', sans-serif;
    --lumo-primary-color: hsl(220, 80%, 50%);
    --lumo-primary-color-50pct: hsla(220, 80%, 50%, 0.5);
    --lumo-primary-color-10pct: hsla(220, 80%, 50%, 0.1);
    --lumo-primary-text-color: hsl(220, 80%, 45%);
    --lumo-border-radius-m: 8px;
}

[theme~="dark"] {
    --lumo-base-color: hsl(220, 20%, 12%);
    --lumo-primary-color: hsl(220, 85%, 65%);
}
```

### Dark Mode Toggle (Both Themes)

```java
// Works with both Aura and Lumo
@ColorScheme(ColorScheme.Value.LIGHT_DARK)
public class Application implements AppShellConfigurator {
}
```

### Custom Palette — Aura (Professional Blue)

```css
html {
    --aura-accent-color-light: hsl(215, 75%, 50%);
    --aura-accent-color-dark: hsl(215, 75%, 65%);
    --aura-blue: hsl(215, 75%, 50%);
}
```

### Custom Palette — Lumo (Professional Blue)

```css
html {
    --lumo-primary-color: hsl(215, 75%, 50%);
    --lumo-primary-color-50pct: hsla(215, 75%, 50%, 0.5);
    --lumo-primary-color-10pct: hsla(215, 75%, 50%, 0.1);
    --lumo-primary-text-color: hsl(215, 75%, 45%);
}
```

## Lumo Utility Class Categories

| Category | Examples |
|----------|---------|
| Background | `Background.BASE`, `Background.PRIMARY`, `Background.CONTRAST_5` |
| Border | `Border.ALL`, `Border.BOTTOM`, `Border.TOP` |
| Border Color | `BorderColor.CONTRAST_10`, `BorderColor.PRIMARY` |
| Border Radius | `BorderRadius.SMALL`, `BorderRadius.MEDIUM`, `BorderRadius.LARGE` |
| Box Shadow | `BoxShadow.SMALL`, `BoxShadow.MEDIUM`, `BoxShadow.LARGE` |
| Display | `Display.FLEX`, `Display.GRID`, `Display.HIDDEN` |
| Flex Direction | `FlexDirection.ROW`, `FlexDirection.COLUMN` |
| Font Size | `FontSize.SMALL`, `FontSize.MEDIUM`, `FontSize.XXLARGE` |
| Font Weight | `FontWeight.BOLD`, `FontWeight.SEMIBOLD`, `FontWeight.NORMAL` |
| Gap | `Gap.XSMALL`, `Gap.SMALL`, `Gap.MEDIUM`, `Gap.LARGE` |
| Padding | `Padding.SMALL`, `Padding.MEDIUM`, `Padding.LARGE` |
| Text Color | `TextColor.HEADER`, `TextColor.SECONDARY`, `TextColor.PRIMARY` |

All categories support responsive breakpoint variants: `*.Breakpoint.Small.*`, `*.Breakpoint.Medium.*`, etc.
