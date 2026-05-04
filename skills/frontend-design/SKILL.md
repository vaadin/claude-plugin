---
name: frontend-design
description: >
  Guide Claude on creating visually distinctive, polished Vaadin 25 interfaces that go beyond
  default theme styling. This skill should be used when the user asks to "make it look good",
  "improve the design", "style the view", "make it visually appealing",
  "add polish", "design a UI", "create a beautiful interface", or when building a new view
  where visual quality matters. Also trigger when the user wants to add animations, visual
  effects, or build polished component compositions in a Vaadin application.
version: 0.1.0
---

# Creating Distinctive Vaadin Interfaces

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_styling`, `get_component_java_api`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

This skill guides creation of distinctive, polished Vaadin interfaces that go beyond default theme styling. The goal is production-grade code with genuine attention to aesthetic detail — not generic defaults.

## Design Thinking

Before writing code, understand the context and commit to a clear aesthetic direction:

- **Purpose**: What problem does this interface solve? Who uses it? An admin dashboard has different design needs than a customer-facing portal.
- **Tone**: Pick a direction: clean and professional, warm and approachable, bold and data-dense, light and airy, dark and focused, editorial, playful, or refined. The direction should serve the users and the content.
- **Constraints**: Vaadin component library, Vaadin theme system (Aura or Lumo), server-side rendering model, accessibility requirements.
- **Differentiation**: What makes this interface feel crafted rather than default? What detail will users notice?

**CRITICAL**: Choose a clear direction and execute it consistently. Every color, spacing, and typography choice should reinforce the same aesthetic. A cohesive simple design always beats an inconsistent elaborate one.

Then implement working code (Java + CSS) that is:
- Production-grade and functional
- Visually cohesive with a clear aesthetic point of view
- Built on Vaadin's component library and theme system
- Refined in spacing, color, typography, and visual hierarchy

## The Vaadin Theme System

All visual customization starts with your chosen theme — **Aura** or **Lumo**. Both provide:

1. **CSS custom properties** — the foundation for colors, typography, spacing, sizing, borders, shadows. Override these to change the entire application's look.
2. **Component theme variants** — pre-built visual variations (e.g., primary, tertiary, compact). Use these before writing custom CSS.
3. **Custom CSS** — for anything the theme doesn't cover. Use `@StyleSheet` with view-scoped CSS files.

Lumo additionally provides:

4. **Utility classes** — Tailwind-like classes (`LumoUtility.*`) for layout, spacing, colors. Fast to apply from Java. Not available with Aura.

Always work top-down: change a theme property before writing component-specific CSS, and use a theme variant before writing custom styles.

For full details on theme selection, loading, and all design tokens, see the **theming** skill.

## Typography

Typography sets the tone of the entire interface.

**Customizing the font family (works with both themes):**

```css
/* In styles.css */
@font-face {
    font-family: 'Your Font';
    src: url('./fonts/your-font.woff2') format('woff2');
    font-display: swap;
}

html {
    /* Aura */
    --aura-font-family: 'Your Font', sans-serif;
    /* Lumo */
    --lumo-font-family: 'Your Font', sans-serif;
}
```

**Tuning the type scale:**

Aura — adjust one value and the entire scale is computed:

```css
html {
    --aura-base-font-size: 15;  /* unitless number, represents px */
}
```

Lumo — override individual tokens:

```css
html {
    --lumo-font-size-xxxl: 2.5rem;
    --lumo-font-size-xxl: 1.75rem;
    --lumo-font-size-xl: 1.375rem;
    --lumo-font-size-l: 1.125rem;
    --lumo-font-size-m: 1rem;
    --lumo-font-size-s: 0.875rem;
    --lumo-font-size-xs: 0.8125rem;
    --lumo-font-size-xxs: 0.75rem;
}
```

**Creating typographic hierarchy from Java (Lumo utility classes):**

```java
// Lumo theme only — requires Lumo.UTILITY_STYLESHEET
H2 title = new H2("Dashboard");
title.addClassNames(
    LumoUtility.FontSize.XXLARGE,
    LumoUtility.FontWeight.BOLD,
    LumoUtility.TextColor.HEADER
);

Span subtitle = new Span("Weekly performance overview");
subtitle.addClassNames(
    LumoUtility.FontSize.MEDIUM,
    LumoUtility.TextColor.SECONDARY
);
```

For Aura, use CSS classes or inline styles instead of `LumoUtility`.

**Key principle**: Establish a clear hierarchy — one dominant heading style, one body style, one secondary/caption style. Use font size, weight, and color together to differentiate levels. Avoid using more than 3-4 distinct text styles in a single view.

## Color and Theming

Color is the fastest way to give a Vaadin app a distinctive identity.

**Aura — customizing the accent and palette:**

```css
html {
    --aura-accent-color-light: hsl(220, 80%, 50%);
    --aura-accent-color-dark: hsl(220, 85%, 65%);
    --aura-background-color-light: hsl(220, 20%, 98%);
    --aura-background-color-dark: hsl(220, 20%, 12%);
    --aura-blue: hsl(220, 80%, 50%);
    --aura-green: hsl(150, 60%, 40%);
    --aura-red: hsl(0, 75%, 55%);
}
```

**Lumo — overriding the color palette:**

```css
html {
    /* Primary color — used for buttons, links, focus rings */
    --lumo-primary-color: hsl(220, 80%, 50%);
    --lumo-primary-color-50pct: hsla(220, 80%, 50%, 0.5);
    --lumo-primary-color-10pct: hsla(220, 80%, 50%, 0.1);
    --lumo-primary-text-color: hsl(220, 80%, 45%);
    --lumo-primary-contrast-color: #fff;

    /* Surface colors — backgrounds, cards, dialogs */
    --lumo-base-color: hsl(220, 20%, 98%);

    /* Error, success, warning */
    --lumo-error-color: hsl(0, 75%, 55%);
    --lumo-success-color: hsl(150, 60%, 40%);
    --lumo-warning-color: hsl(40, 95%, 50%);
}
```

**Dark mode:**

Both themes support the `@ColorScheme` annotation. For custom dark mode colors, see the theming skill for theme-specific selectors (`[theme~="dark"]` for Lumo, `-light`/`-dark` suffixed properties for Aura).

**Accent and semantic colors from Java (Lumo utility classes):**

```java
// Lumo theme only
badge.addClassNames(
    LumoUtility.Background.PRIMARY,
    LumoUtility.TextColor.PRIMARY_CONTRAST
);

warningCard.addClassNames(
    LumoUtility.Background.WARNING_10PCT,
    LumoUtility.TextColor.WARNING
);
```

**Key principle**: Commit to a cohesive palette. Pick one strong primary/accent color and use the theme's variant system (Lumo's opacity variants, Aura's computed variants) for secondary uses. A dominant primary with restrained accent colors outperforms an evenly distributed rainbow.

## Spacing and Density

Consistent spacing creates visual rhythm and professionalism.

**Aura — adjust the base size:**

```css
html {
    --aura-base-size: 18;     /* unitless, range 12–24 */
    --aura-base-radius: 8;    /* unitless, range 0–10 */
}
```

**Lumo — override individual spacing tokens:**

```css
html {
    --lumo-space-xs: 0.25rem;
    --lumo-space-s: 0.5rem;
    --lumo-space-m: 1rem;
    --lumo-space-l: 1.5rem;
    --lumo-space-xl: 2.5rem;
}
```

**Applying spacing from Java (Lumo utility classes):**

```java
// Lumo theme only
card.addClassNames(
    LumoUtility.Padding.LARGE,
    LumoUtility.Gap.MEDIUM
);

section.addClassNames(
    LumoUtility.Padding.Horizontal.LARGE,
    LumoUtility.Padding.Vertical.XLARGE
);
```

**Compact/dense variants:**

```java
// Cross-theme variants — work in both Aura and Lumo
grid.addThemeVariants(GridVariant.NO_BORDER);
textField.addThemeVariants(TextFieldVariant.SMALL);
button.addThemeVariants(ButtonVariant.SMALL);

// Lumo-only: compact grid
grid.addThemeVariants(GridVariant.LUMO_COMPACT);
```

**Key principle**: Pick a density and stick with it. Data-dense dashboards should be consistently compact. Spacious marketing-style views should be consistently airy. Mixing densities looks unintentional.

## Shadows, Borders, and Elevation

Create depth and visual hierarchy through elevation.

**Aura — surface level system:**

```css
.elevated-card {
    background: var(--aura-surface-color);
    --aura-surface-level: 2;
}

.sunken-area {
    background: var(--aura-surface-color);
    --aura-surface-level: -1;
}
```

**Lumo — explicit shadow tokens:**

```css
html {
    --lumo-box-shadow-xs: 0 1px 2px 0 rgba(0,0,0,0.05);
    --lumo-box-shadow-s: 0 2px 4px -1px rgba(0,0,0,0.1);
    --lumo-box-shadow-m: 0 4px 8px -2px rgba(0,0,0,0.1);
    --lumo-box-shadow-l: 0 8px 16px -4px rgba(0,0,0,0.15);
    --lumo-box-shadow-xl: 0 16px 32px -8px rgba(0,0,0,0.2);
}
```

```java
// Lumo theme only
card.addClassNames(
    LumoUtility.BoxShadow.SMALL,
    LumoUtility.BorderRadius.MEDIUM
);
```

**Border radius customization:**

```css
/* Lumo */
html {
    --lumo-border-radius-s: 4px;
    --lumo-border-radius-m: 8px;
    --lumo-border-radius-l: 16px;
}

/* Aura — adjust the base radius instead */
html {
    --aura-base-radius: 8;  /* computes all radius values */
}
```

**Subtle borders for separation (Lumo utility classes):**

```java
// Lumo theme only
section.addClassNames(
    LumoUtility.Border.BOTTOM,
    LumoUtility.BorderColor.CONTRAST_10
);
```

**Key principle**: Use elevation consistently to communicate hierarchy. Cards float above the surface, dialogs float above cards. Don't put heavy shadows on flat elements or flat shadows on floating elements.

## Motion and Animation

Vaadin's server-side model means animations should be CSS-driven. Keep them subtle and purposeful.

**View transitions with CSS:**

```css
/* Fade-in for view content */
.fade-in {
    animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(8px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Staggered reveal for list items */
.stagger-item {
    animation: fadeIn 0.3s ease-out backwards;
}

.stagger-item:nth-child(1) { animation-delay: 0.05s; }
.stagger-item:nth-child(2) { animation-delay: 0.1s; }
.stagger-item:nth-child(3) { animation-delay: 0.15s; }
.stagger-item:nth-child(4) { animation-delay: 0.2s; }
.stagger-item:nth-child(5) { animation-delay: 0.25s; }
```

```java
content.addClassName("fade-in");

// Staggered card reveal
for (Component card : cards) {
    card.addClassName("stagger-item");
}
```

**Hover and interaction effects:**

```css
.interactive-card {
    transition: box-shadow 0.2s ease, transform 0.2s ease;
}

.interactive-card:hover {
    box-shadow: 0 4px 8px -2px rgba(0,0,0,0.1);  /* or use theme token */
    transform: translateY(-2px);
}
```

**Key principle**: One or two well-crafted animations (like a staggered page-load reveal) create more impact than scattered micro-interactions everywhere. Prefer CSS transitions on hover/focus over complex keyframe animations. Keep durations under 400ms.

## Building Polished Components

### Styled cards (Lumo utility classes)

```java
// Lumo theme only — uses LumoUtility classes
public static Div createCard(String title, String value, String description) {
    Div card = new Div();
    card.addClassNames(
        LumoUtility.Background.BASE,
        LumoUtility.BorderRadius.MEDIUM,
        LumoUtility.BoxShadow.SMALL,
        LumoUtility.Padding.LARGE,
        LumoUtility.Display.FLEX,
        LumoUtility.FlexDirection.COLUMN,
        LumoUtility.Gap.SMALL
    );

    Span titleSpan = new Span(title);
    titleSpan.addClassNames(
        LumoUtility.FontSize.SMALL,
        LumoUtility.TextColor.SECONDARY,
        LumoUtility.FontWeight.MEDIUM
    );

    Span valueSpan = new Span(value);
    valueSpan.addClassNames(
        LumoUtility.FontSize.XXLARGE,
        LumoUtility.FontWeight.BOLD,
        LumoUtility.TextColor.HEADER
    );

    Span descSpan = new Span(description);
    descSpan.addClassNames(
        LumoUtility.FontSize.SMALL,
        LumoUtility.TextColor.TERTIARY
    );

    card.add(titleSpan, valueSpan, descSpan);
    return card;
}
```

For Aura, use CSS classes with Aura's surface system instead of `LumoUtility`:

```css
.metric-card {
    background: var(--aura-surface-color);
    --aura-surface-level: 2;
    border-radius: var(--vaadin-border-radius-m);
    padding: var(--vaadin-padding);
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}
```

### Status badges

Badge is a preview feature in Vaadin 25.1 — enable it with the `badgeComponent` feature flag. Available variants: `SUCCESS`, `WARNING`, `ERROR`, `FILLED`, `ICON_ONLY`, `NUMBER_ONLY`.

```java
// Badge component (Vaadin 25.1+ preview — requires badgeComponent feature flag)
Badge pending = new Badge("Pending");

Badge confirmed = new Badge("Confirmed");
confirmed.addThemeVariants(BadgeVariant.SUCCESS);

Badge warning = new Badge("Warning");
warning.addThemeVariants(BadgeVariant.WARNING);

Badge denied = new Badge("Denied");
denied.addThemeVariants(BadgeVariant.ERROR);

// With icon
Badge iconBadge = new Badge("Confirmed", VaadinIcon.CHECK.create());
iconBadge.addThemeVariants(BadgeVariant.SUCCESS);

// With number
Badge counter = new Badge("Inbox", 12);
counter.addThemeVariants(BadgeVariant.FILLED);
```

For Vaadin versions before 25.1, badges can be created with `Span` elements using the `theme="badge"` attribute, but this approach is deprecated.

### Data-dense dashboard layout (Lumo utility classes)

```java
// Lumo theme only — uses LumoUtility classes
VerticalLayout dashboard = new VerticalLayout();
dashboard.setPadding(true);
dashboard.setSpacing(false);
dashboard.addClassNames(
    LumoUtility.Gap.LARGE,
    LumoUtility.Background.CONTRAST_5
);

// Metric cards grid — uses CSS Grid for responsive layout
Div metrics = new Div(
    createCard("Revenue", "$48,200", "+12% from last month"),
    createCard("Users", "1,420", "+5% from last month"),
    createCard("Orders", "384", "+8% from last month")
);
metrics.addClassName("metrics-grid");
// Companion CSS:
// .metrics-grid {
//     display: grid;
//     grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
//     gap: var(--lumo-space-m);
//     width: 100%;
// }

dashboard.add(createSectionHeader("Overview"), metrics);
```

## Styling Vaadin Components with CSS

When theme properties and theme variants aren't enough, use CSS.

**Shadow DOM and `::part()` selectors** — Vaadin components use shadow DOM. Style internal parts with `::part()`:

```css
/* Style Grid header cells */
vaadin-grid::part(header-cell) {
    background-color: var(--lumo-contrast-5pct);  /* Lumo */
    /* or for Aura: --aura-surface-level: 0; */
    font-weight: 600;
}

/* Style Dialog overlay */
vaadin-dialog-overlay::part(overlay) {
    border-radius: 16px;
}

/* Style TextField input */
vaadin-text-field::part(input-field) {
    border-radius: 8px;
}
```

**Component-scoped styles** — apply CSS to a specific view:

```java
@StyleSheet("styles/views/dashboard-view.css")
@Route("dashboard")
public class DashboardView extends VerticalLayout {
    // ...
}
```

**Key principle**: Always prefer theme custom properties over hardcoded values in CSS. Use your active theme's tokens for colors, spacing, and sizing. This keeps styles consistent with the theme and makes dark mode, density changes, and future redesigns trivial.

## Common Anti-Patterns

1. **Hardcoded colors and sizes** — always use your theme's custom properties. Hardcoded values break dark mode and make theming impossible.
2. **Overriding component internals instead of theme properties** — if a color looks wrong, override the theme color token, not the individual component's CSS. One property change should update the entire app.
3. **Too many visual styles** — pick 2-3 card styles, 2-3 text hierarchies, one primary action style. Consistency creates professionalism.
4. **Ignoring theme variants** — check `ButtonVariant`, `GridVariant`, `TextFieldVariant`, etc. before writing custom CSS. The variant you need probably exists.
5. **Heavy animations on server-rendered updates** — Vaadin rerenders from the server. Complex entrance animations on every server push look janky. Use animations for initial page loads and user-initiated transitions, not for every data update.
6. **Flat visual hierarchy** — if everything looks the same, nothing stands out. Use size, weight, color, and elevation to guide the eye to what matters.

## Best Practices

1. **Start with theme properties** — customize your theme's core properties first. This alone can transform the look.
2. **Use component theme variants** — primary, tertiary, compact, badge themes. Use what's built in.
3. **Maintain a consistent color strategy** — one primary/accent, one or two additional colors, semantic colors for status.
4. **Design with elevation** — cards, dialogs, and menus should feel layered. Use your theme's elevation system consistently.
5. **Add motion sparingly** — CSS transitions on hover, a page-load fade-in, staggered card reveals. Keep it under 400ms.
6. **Test in dark mode** — if you customize any colors, verify both light and dark themes. Using theme properties makes this automatic.

## Detailed Reference

For component `::part()` selectors, CSS animation recipes, and color palette recipes, see `references/design-patterns.md`. For complete theme token tables and variant comparisons, see the **theming** skill's `references/theming-patterns.md`.
