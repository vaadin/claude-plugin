---
name: theming
description: >
  Guide Claude on Vaadin 25 theming with both Aura and Lumo themes. This skill should be used
  when the user asks to "choose a theme", "set up Aura", "set up Lumo", "customize the theme",
  "change colors", "enable dark mode", "customize design tokens", "use theme variants",
  "use utility classes", "change the color scheme", or needs help with theme-specific CSS
  custom properties, component theme variants, or building a custom theme in Vaadin 25.
version: 0.1.0
---

# Vaadin 25 Theming: Aura & Lumo

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_styling`, `get_component_java_api`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

Vaadin 25 ships two fully supported themes — **Aura** and **Lumo**. They differ in visual style, customization approach, and available features. This skill covers both.

## Choosing a Theme

| Factor | Aura | Lumo |
|--------|------|------|
| Visual style | Modern, uses transparencies and gradients | Classic, clean and flat |
| Customization model | Few core properties, computed variations | Many individual tokens, granular control |
| Color system | 7-color palette, auto-computed variants, dynamic accent color | Semantic color scales with explicit opacity variants |
| Font sizing | Single `--aura-base-font-size` and individual `--aura-font-size-*`tokens | Individual `--lumo-font-size-*` tokens |
| Spacing/sizing | Single `--aura-base-size` and individual `--vaadin-padding-*` and `--vaadin-gap-*` properties | Individual `--lumo-space-*` tokens |
| Border radius | Single `--aura-base-radius` and individual `--vaadin-radius-*` tokens | Individual `--lumo-border-radius-*` tokens |
| Utility classes | None | `LumoUtility.*` (Tailwind-like) |
| Dark mode | `color-scheme: light dark` (CSS native) | `color-scheme: light dark` (CSS native) |
| Elevation | Surface level system (`--aura-surface-level`) and shadow tokens (`--aura-shadow-*`) | Explicit shadow tokens (`--lumo-box-shadow-*`) |

**Choose Aura** when you want a modern look with minimal configuration — change a few core properties and the entire app adapts. Good for rapid prototyping and apps where broad visual consistency matters more than pixel-level control. You can still have fine-grained control over individual design tokens, if needed. Good for data-heavy enterprise apps.

**Choose Lumo** when you need fine-grained control over individual design tokens, want to use utility classes from Java, or need the compact/dense theme preset. Good for data-heavy enterprise apps and when you want Tailwind-like styling from Java.

**Pick one theme and stick with it.** Mixing theme tokens (e.g., `--aura-*` properties with Lumo, or `LumoUtility` classes with Aura) produces broken or unpredictable results.

## Loading a Theme

Themes are loaded via `@StyleSheet` on your `AppShellConfigurator` class. The theme stylesheet must be loaded **before** any other stylesheets.

**Aura:**

```java
@StyleSheet(Aura.STYLESHEET)
@StyleSheet("styles.css")
public class Application implements AppShellConfigurator {
}
```

**Lumo:**

```java
@StyleSheet(Lumo.STYLESHEET)
@StyleSheet("styles.css")
public class Application implements AppShellConfigurator {
}
```

**Lumo with utility classes:**

```java
@StyleSheet(Lumo.STYLESHEET)
@StyleSheet(Lumo.UTILITY_STYLESHEET)
@StyleSheet("styles.css")
public class Application implements AppShellConfigurator {
}
```

There is no utility class stylesheet for Aura. `Lumo.UTILITY_STYLESHEET` only works with the Lumo theme.

## Color

### Aura Color System

Aura has a 7-color palette: **neutral**, **red**, **orange**, **yellow**, **green**, **blue**, **purple**. Each is a single value from which text, border, and surface variants are computed automatically. The neutral color is color-scheme dependent and is computed based on the `--aura-background-color-light` and `--aura-background-color-dark` tokens.

```css
html {
    /* Override palette colors */
    --aura-red: #ef4444;
    --aura-green: #22c55e;
    --aura-blue: #3b82f6;
    --aura-purple: #8b5cf6;
    --aura-orange: #f97316;
    --aura-yellow: #eab308;
}
```

Certain palette colors are used for the built-in component variants (e.g., buttons, badges, notifications). They work on any component that uses the accent color:

- `info` variant -> blue accent color
- `error` variant -> red accent color
- `success` variant -> green accent color
- `warning` variant -> yellow accent color

**Accent color** — the primary action color. Applied per-component or globally:

```css
html {
    /* Global accent color. You need to override it separately for ligth and dark color schemes */
    --aura-accent-color-light: #3b82f6;
    --aura-accent-color-dark: #3b82f6;
}

/* Or per-component */
vaadin-button {
    --aura-accent-color-light: #42C556;
    --aura-accent-color-dark: #42C556; 
}
```

**Background and surface colors:**

```css
html {
    --aura-background-color-light: #f8fafc;
    --aura-background-color-dark: #0f172a;
}
```

For more variations than the palette provides, use CSS `color-mix()` or relative color functions.

### Lumo Color System

Lumo uses semantic color scales with explicit opacity variants:

```css
html {
    /* Primary — buttons, links, active indicators */
    --lumo-primary-color: hsl(220, 80%, 50%);
    --lumo-primary-color-50pct: hsla(220, 80%, 50%, 0.5);
    --lumo-primary-color-10pct: hsla(220, 80%, 50%, 0.1);
    --lumo-primary-text-color: hsl(220, 80%, 45%);
    --lumo-primary-contrast-color: #fff;

    /* Semantic status colors */
    --lumo-error-color: hsl(0, 75%, 55%);
    --lumo-success-color: hsl(150, 60%, 40%);
    --lumo-warning-color: hsl(40, 95%, 50%);

    /* Surface colors */
    --lumo-base-color: hsl(220, 20%, 98%);
    --lumo-contrast-5pct: hsla(220, 20%, 0%, 0.05);
    --lumo-contrast-10pct: hsla(220, 20%, 0%, 0.1);
}
```

Each semantic color (primary, error, success, warning) has `*-color`, `*-color-50pct`, `*-color-10pct`, `*-text-color`, and `*-contrast-color` variants. The contrast scale (`--lumo-contrast-5pct` through `--lumo-contrast`) provides 10 levels for backgrounds, borders, and text.

## Dark Mode

Both themes use the shared `@ColorScheme` annotation for static configuration:

```java
@ColorScheme(ColorScheme.Value.DARK)
public class Application implements AppShellConfigurator {
}
```

Values: `LIGHT` (default), `DARK`, `LIGHT_DARK` (follows OS preference).

Both themes use the native CSS `color-scheme` property. The `LIGHT_DARK` value maps to `color-scheme: light dark`, which automatically follows the user's OS preference.

### Aura Dark Mode

Aura color properties have `-light` and `-dark` suffixed variants. If you customize colors and support both schemes, override both:

```css
html {
    --aura-background-color-light: #f8fafc;
    --aura-background-color-dark: #0f172a;
    --aura-accent-color-light: #3b82f6;
    --aura-accent-color-dark: #60a5fa;
}
```

### Lumo Dark Mode

With Lumo, use the native `light-dark()` function to support both color schemes when customizing the colors:

```css
html {
    --lumo-base-color: light-dark(hsl(220, 20%, 92%), hsl(220, 20%, 12%));
    --lumo-primary-color: light-dark(hsl(220, 85%, 55%), hsl(220, 85%, 65%));
    --lumo-body-text-color: light-dark(hsla(0, 0%, 0%, 0.9), hsla(0, 0%, 100%, 0.9));
}
```

## Typography

### Aura Typography

Aura computes a full type scale from a single base value:

```css
html {
    --aura-font-family: 'Your Font', sans-serif;
    --aura-base-font-size: 16;      /* unitless number, represents px, maps directly to `--aura-font-size-m` */
    --aura-base-line-height: 1.5;   /* unitless, relative to font size */
}
```

This computes `--aura-font-size-xs` through `--aura-font-size-xl` and corresponding line heights automatically. You can override the individual font-size tokens explicitly, if needed.

html {
    --aura-font-size-xs: 0.625rem;
}

Aura also supports dynamic font sizing on iOS/iPadOS.

### Lumo Typography

Lumo provides individual tokens for each size:

```css
html {
    --lumo-font-family: 'Your Font', sans-serif;
    --lumo-font-size-xxxl: 2rem;
    --lumo-font-size-xxl: 1.5rem;
    --lumo-font-size-xl: 1.25rem;
    --lumo-font-size-l: 1.125rem;
    --lumo-font-size-m: 1rem;
    --lumo-font-size-s: 0.875rem;
    --lumo-font-size-xs: 0.8125rem;
    --lumo-font-size-xxs: 0.75rem;
}
```

### Custom Font Loading (Both Themes)

```css
@font-face {
    font-family: 'Your Font';
    src: url('./fonts/your-font.woff2') format('woff2');
    font-display: swap;
}

html {
    /* Aura */
    --aura-font-family: 'Your Font', sans-serif;
    /* OR Lumo */
    --lumo-font-family: 'Your Font', sans-serif;
}
```

## Spacing, Sizing, and Border Radius

### Aura

Aura computes spacing and sizing from core properties:

```css
html {
    --aura-base-size: 16;     /* unitless, range 12–24, controls gap/padding, should ideally be divisible by 4 */
    --aura-base-radius: 6;    /* unitless, range 0–8, controls border radius */
}
```

These compute the shared `--vaadin-gap`, `--vaadin-padding`, and `--vaadin-radius` properties automatically. If needed, you can override those individual tokens explicitly.

Aura has built-in `theme="xsmall", `theme="small"`, `theme="large"` and `theme="xlarge"` variants that change the base-size and base-font-size accordingly. They work on all components and layouts. They are useful if you want a certain part of you UI to be more compact.

### Lumo

Lumo provides individual tokens:

```css
html {
    /* Spacing */
    --lumo-space-xs: 0.25rem;
    --lumo-space-s: 0.5rem;
    --lumo-space-m: 1rem;
    --lumo-space-l: 1.5rem;
    --lumo-space-xl: 2.5rem;

    /* Border radius */
    --lumo-border-radius-s: 4px;
    --lumo-border-radius-m: 8px;
    --lumo-border-radius-l: 16px;
}
```

**Lumo compact preset** — reduces component sizing globally. Load as an additional stylesheet. Useful for data-dense views.

**Lumo utility classes for spacing (Lumo only):**

```java
card.addClassNames(
    LumoUtility.Padding.LARGE,
    LumoUtility.Gap.MEDIUM
);
```

## Component Theme Variants

Both themes provide style variants via `addThemeVariants()`. Variants supported by both themes use unprefixed enum values (e.g., `ButtonVariant.PRIMARY`); `LUMO_*` prefixes apply only to variants that are exclusive to Lumo.

```java
// Cross-theme — works in both Aura and Lumo
button.addThemeVariants(ButtonVariant.PRIMARY);

// Lumo-only variants are prefixed
button.addThemeVariants(ButtonVariant.LUMO_TERTIARY_INLINE);
```

Check the component documentation for what each theme supports. Common variants:

| Variant | Aura | Lumo | Enum value |
|---------|------|------|------------|
| Button: primary | yes | yes | `ButtonVariant.PRIMARY` |
| Button: tertiary | yes | yes | `ButtonVariant.TERTIARY` |
| Button: small / large | yes | yes | `ButtonVariant.SMALL` / `LARGE` |
| Button: success | yes | yes | `ButtonVariant.SUCCESS` |
| Button: error | yes | yes | `ButtonVariant.ERROR` |
| Button: warning | yes | yes | `ButtonVariant.WARNING` |
| Button: tertiary-inline | — | yes | `ButtonVariant.LUMO_TERTIARY_INLINE` |
| Button: icon | — | yes | `ButtonVariant.LUMO_ICON` |
| Button: contrast | — | yes | `ButtonVariant.LUMO_CONTRAST` |
| Grid: no border | yes | yes | `GridVariant.NO_BORDER` |
| Grid: no row borders | yes | yes | `GridVariant.NO_ROW_BORDERS` |
| Grid: column borders | yes | yes | `GridVariant.COLUMN_BORDERS` |
| Grid: row stripes | yes | yes | `GridVariant.ROW_STRIPES` |
| Grid: compact | — | yes | `GridVariant.LUMO_COMPACT` |
| TextField: small | yes | yes | `TextFieldVariant.SMALL` |

**Aura accent colors for buttons** — Aura uses CSS classes instead of theme attributes for certain color variants:

```css
/* These classes are applied automatically by ButtonVariant.SUCCESS / ERROR / WARNING in Aura */
vaadin-button.aura-accent-green  { /* success */ }
vaadin-button.aura-accent-red    { /* error */ }
vaadin-button.aura-accent-yellow { /* warning */ }
```

## Utility Classes (Lumo Only)

`LumoUtility.*` provides Tailwind-like utility classes for layout, spacing, colors, typography, borders, shadows, and more. They require the Lumo theme and `Lumo.UTILITY_STYLESHEET`.

```java
card.addClassNames(
    LumoUtility.Background.BASE,
    LumoUtility.BorderRadius.MEDIUM,
    LumoUtility.BoxShadow.SMALL,
    LumoUtility.Padding.LARGE,
    LumoUtility.Display.FLEX,
    LumoUtility.FlexDirection.COLUMN,
    LumoUtility.Gap.SMALL
);
```

Categories: `Background`, `Border`, `BorderColor`, `BorderRadius`, `BoxShadow`, `Display`, `FlexDirection`, `FlexGrow`, `FlexShrink`, `FlexWrap`, `FontSize`, `FontWeight`, `Gap`, `Height`, `JustifyContent`, `Margin`, `Overflow`, `Padding`, `Position`, `TextAlignment`, `TextColor`, `Width`, and responsive breakpoint variants.

**Aura has no equivalent.** If using Aura, use CSS custom properties, inline styles, or custom CSS classes instead. You can also enable the experimental Tailwind support and use string-based class names.

## Shadows and Elevation

### Aura: Surface Level System

Aura uses a surface color system where elevation is controlled by `--aura-surface-level`. Higher levels are lighter in light mode (closer to white) and lighter in dark mode (closer to the user). You can combine the surface color with the shadow tokens.

```css
.card {
    background: var(--aura-surface-color);
    --aura-surface-level: 2;      /* default is 1 */
    --aura-surface-opacity: 0.5;  /* transparency, default 0.5 */
    box-shadow: var(--aura-shadow-s);
}
```

NOTE: you can only change the level and opacity on certain elements that match the list of selectors in the aura surface.css source file (https://github.com/vaadin/web-components/blob/71acbbfd677a9a3272fc79b4279d4b298f37640c/packages/aura/src/surface.css#L7-L41).

The surface color is computed from `--aura-background-color`, the level, and the opacity. Nesting elements with the same surface color creates a stacking effect due to transparency.

### Lumo: Explicit Shadow Tokens

Lumo provides individual shadow tokens:

```css
html {
    --lumo-box-shadow-xs: 0 1px 2px 0 rgba(0,0,0,0.05);
    --lumo-box-shadow-s: 0 2px 4px -1px rgba(0,0,0,0.1);
    --lumo-box-shadow-m: 0 4px 8px -2px rgba(0,0,0,0.1);
    --lumo-box-shadow-l: 0 8px 16px -4px rgba(0,0,0,0.15);
    --lumo-box-shadow-xl: 0 16px 32px -8px rgba(0,0,0,0.2);
}
```

Apply from Java with utility classes (Lumo only):

```java
card.addClassNames(LumoUtility.BoxShadow.SMALL);
```

## Component Style Properties

Both themes support shared `--vaadin-*` component style properties that work regardless of theme:

```css
/* Shared properties — work with any theme */
vaadin-horizontal-layout {
    --vaadin-horizontal-layout-gap: 16px;
}

vaadin-text-field {
    --vaadin-input-field-border-radius: 8px;
}
```

## Building a Custom Theme

Create a reusable theme CSS file that overrides the core properties of your chosen theme:

**Aura custom theme:**

```css
/* styles/my-theme.css */
html {
    --aura-font-family: 'Inter', sans-serif;
    --aura-base-font-size: 15;
    --aura-base-size: 16;
    --aura-base-radius: 8;
    --aura-accent-color-light: #2563eb;
    --aura-accent-color-dark: #60a5fa;
    --aura-background-color-light: #f8fafc;
    --aura-background-color-dark: #0f172a;
}
```

**Lumo custom theme:**

```css
/* styles/my-theme.css */
html {
    --lumo-font-family: 'Inter', sans-serif;
    --lumo-primary-color: light-dark(hsl(220, 80%, 50%), hsl(220, 85%, 65%));
    --lumo-primary-color-50pct: light-dark(hsla(220, 80%, 50%, 0.5), hsl(220, 85%, 65%, 0.5));
    --lumo-primary-color-10pct: light-dark(hsla(220, 80%, 50%, 0.1), hsl(220, 85%, 65%, 0.1));
    --lumo-primary-text-color: light-dark(hsl(220, 80%, 45%), hsl(220, 80%, 70%));
    --lumo-border-radius-m: 8px;
    --lumo-border-radius-l: 16px;
}
```

Load with `@StyleSheet`:

```java
@StyleSheet(Aura.STYLESHEET)   // or Lumo.STYLESHEET
@StyleSheet("styles/my-theme.css")
@StyleSheet("styles.css")
public class Application implements AppShellConfigurator {
}
```

## Best Practices

1. **Pick one theme and commit** — don't mix Aura and Lumo tokens or utility classes in the same app.
2. **Prefer theme properties over hardcoded values** — write `var(--lumo-space-m)` not `1rem`, or adjust `--aura-base-size` instead of hardcoding pixel values.
3. **Test both color schemes** — if you customize any colors, verify both light and dark mode. Using theme properties makes this mostly automatic.
4. **Use component theme variants before custom CSS** — check what `ButtonVariant`, `GridVariant`, etc. offer before writing custom styles.
5. **Load themes before other styles** — the `@StyleSheet` for the theme must come before application stylesheets.
6. **Use `::part()` for component internals** — Vaadin components use shadow DOM; `::part()` selectors are the supported way to style internal parts.

## Anti-Patterns

1. **Mixing theme tokens** — using `--aura-*` properties in a Lumo app or `--lumo-*` in an Aura app. These properties don't exist in the other theme and will have no effect.
2. **Using `LumoUtility` with Aura** — utility classes only work with the Lumo theme. With Aura, use custom CSS classes or inline styles.
3. **Hardcoding colors and sizes** — breaks dark mode and makes theme changes impossible. Always use the active theme's properties.
4. **Using `@CssImport`** — this annotation is discouraged in Vaadin 25. Use `@StyleSheet` instead.
5. **Loading theme after app styles** — theme stylesheets must come first in `@StyleSheet` ordering.

## Detailed Reference

For complete token tables, variant comparison charts, and theme setup recipes, see `references/theming-patterns.md`. For component `::part()` selectors and CSS animation recipes, see the frontend-design skill's `references/design-patterns.md`.
