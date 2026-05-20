---
name: vaadin-layouts
description: >
  Guide Claude on using Vaadin 25 HorizontalLayout and VerticalLayout correctly.
  This skill should be used when the user asks to "create a layout", "arrange components",
  "align items", "fix layout sizing", "use HorizontalLayout", "use VerticalLayout",
  or needs help with spacing, padding, margins, flex-grow, flex-shrink, or alignment
  in Vaadin Flow views. Also trigger when debugging layout issues like components
  shrinking unexpectedly or overflowing their container, or when choosing between
  layout components (HorizontalLayout vs VerticalLayout vs FlexLayout vs AppLayout).
version: 0.1.0
---

# Vaadin Layouts: HorizontalLayout & VerticalLayout

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`, `get_component_styling`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## When NOT to Use HorizontalLayout / VerticalLayout

Before reaching for these layouts, consider whether a different component is a better fit:

**Use AppLayout instead** when building the application shell — the top-level structure with a navigation drawer, header bar, and main content area. AppLayout handles responsive drawer collapsing, hamburger menus on mobile, and navbar placement automatically. Building this with nested HorizontalLayout/VerticalLayout is a common anti-pattern that produces worse results with more code.

**Use Dashboard instead** when building a grid of cards or widgets. Dashboard provides automatic responsive reflow, optional user-configurable drag-and-drop rearrangement, and proper grid-based sizing. Nesting HorizontalLayouts with wrapping to simulate a card grid is fragile and doesn't give you the two-dimensional control that a grid-based layout provides.

**Use CSS Grid (via custom CSS, or Lumo utility classes if using the Lumo theme) instead** when you need a true two-dimensional grid layout — rows AND columns. HorizontalLayout and VerticalLayout are one-dimensional (flexbox). If you find yourself nesting multiple HorizontalLayouts inside a VerticalLayout to create a grid, CSS Grid is the right tool.

**Use FormLayout instead** when arranging form fields. FormLayout handles responsive column counts, label positioning, and field sizing automatically.

HorizontalLayout and VerticalLayout are the right choice for arranging components in a single row or column — toolbars, button groups, card contents, view sections, and similar one-dimensional arrangements.

## FlexLayout: When You Need More Control

FlexLayout provides the same flexbox capabilities as HorizontalLayout/VerticalLayout but with more control: you can change direction at runtime, reverse item order, and configure all flex properties explicitly. However, the Vaadin documentation recommends using HorizontalLayout and VerticalLayout over FlexLayout for most cases. They offer better default styling (gaps, padding) and a more intuitive API. FlexLayout exists primarily for backward compatibility and edge cases where you need to toggle between horizontal and vertical direction dynamically.

## Critical Default Behaviors

Understand these defaults before writing any layout code — they cause the most confusion.

**VerticalLayout defaults:**
- Width: 100% (fills parent)
- Height: undefined (hugs content)
- Spacing: ON
- Padding: ON

**HorizontalLayout defaults:**
- Width: undefined (hugs content)
- Height: undefined (hugs content)
- Spacing: ON
- Padding: OFF

These asymmetric defaults are the #1 source of layout surprises. When nesting a VerticalLayout inside a HorizontalLayout, the VerticalLayout will try to take 100% of the parent width. Explicitly set sizes when composing layouts.

## Alignment

Both layouts are CSS flexbox containers. The axis terminology maps directly:

**VerticalLayout** (main axis = vertical):
- `setJustifyContentMode()` — controls vertical distribution (START, CENTER, END, BETWEEN, AROUND, EVENLY)
- `setAlignItems()` — controls horizontal alignment (START, CENTER, END, STRETCH)

**HorizontalLayout** (main axis = horizontal):
- `setJustifyContentMode()` — controls horizontal distribution
- `setAlignItems()` — controls vertical alignment (STRETCH is the default here, unlike VerticalLayout)

Override alignment on individual children with `layout.setAlignSelf(Alignment.X, child)`.

HorizontalLayout also offers `addToStart()`, `addToMiddle()`, `addToEnd()` for grouped positioning. These cannot be combined with `setJustifyContentMode()`.

## Spacing, Padding, Margin

**Spacing** — gap between children. Both layouts have it ON by default. Control it with `setSpacing(boolean)`. Fine-tune using theme variants:

```java
layout.setSpacing(false);
layout.getThemeList().add("spacing-xs"); // xs, s, (default), l, xl
```

Or set a custom pixel value:

```java
layout.setSpacing(12, Unit.PIXELS);
```

**Padding** — space between layout border and content. VerticalLayout has it ON by default; HorizontalLayout has it OFF. Use `setPadding(boolean)`.

**Margin** — space outside the layout border. OFF by default on both. Use `setMargin(boolean)`.

## Expanding Items with Flex

Make a component fill remaining space:

```java
layout.setFlexGrow(1, expandingComponent);
```

Multiple expand ratios work proportionally: `setFlexGrow(2, a)` and `setFlexGrow(1, b)` gives `a` twice the space of `b`.

## Sizing with setWidthFull / setHeightFull (the shrinking trap)

By default in Vaadin 25, `setWidthFull()` / `setHeightFull()` / `setSizeFull()` set literal `width: 100%` / `height: 100%` on the component. Inside a flex layout that means "100% of the parent," not "the remaining space." When such a component sits next to a fixed-size sibling, the default `flex-shrink: 1` on every child lets *both* shrink to fit, so the fixed sibling can end up smaller than its specified size.

Two reliable fixes — use either or both:

```java
// Prefer expand()/setFlexGrow over setWidthFull
HorizontalLayout layout = new HorizontalLayout(fixedSizeComponent, fullSizeComponent);
fixedSizeComponent.setWidth("200px");
layout.setFlexGrow(1, fullSizeComponent);  // takes all remaining space
```

```java
// Or pin the fixed component so it cannot shrink
HorizontalLayout layout = new HorizontalLayout(fixedSizeComponent, fullSizeComponent);
fixedSizeComponent.setWidth("200px");
fullSizeComponent.setWidthFull();
layout.setFlexShrink(0, fixedSizeComponent);
```

`expand(component)` is shorthand for `setFlexGrow(1, component)`. With multiple growing children, ratios distribute proportionally: `setFlexGrow(2, a)` and `setFlexGrow(1, b)` gives `a` twice the space of `b`.

> **Experimental:** Vaadin 25 ships a `layoutComponentImprovements` feature flag (Flow only) that rewires `setWidthFull` / `setHeightFull` / `setSizeFull` to apply `flex: 1` instead of a literal percentage, which prevents fixed-size siblings from shrinking and also sets `min-size: 0` on nested layouts. It is **off by default** — do not assume the behavior unless the user has explicitly enabled the flag.

## The Overflow Trap

Components inside layouts can overflow when the content exceeds the layout's constrained size. This commonly causes unwanted scrollbars.

**Fix:** set minimum size to 0 on the overflowing component:

```java
overflowingComponent.setMinHeight("0");
// or
overflowingComponent.setMinWidth("0");
```

(The experimental `layoutComponentImprovements` flag also sets the min-size of nested layouts to 0 when `setSizeFull()` is used, but that is opt-in — by default you still set it manually.)

## Wrapping

HorizontalLayout supports wrapping to prevent overflow:

```java
layout.setWrap(true);
```

When combining wrapping with `addToStart`/`addToMiddle`/`addToEnd`, group items into sub-layouts for proper wrap behavior.

## Scrollable Layouts

When a layout needs to scroll, wrap it inside a `Scroller` rather than controlling overflow with CSS. Make sure the inner layout does **not** have an explicit height set — let it hug its content so the Scroller can determine when scrolling is needed. This makes it immediately clear from the code which parts of the UI scroll and which don't.

```java
// GOOD — explicit scrollable area, easy to understand
Scroller scroller = new Scroller();
scroller.setSizeFull();

VerticalLayout content = new VerticalLayout(); // height is undefined (default) — hugs content
content.add(/* many components */);

scroller.setContent(content);

// BAD — scrolling controlled via CSS, not obvious from the code
VerticalLayout content = new VerticalLayout();
content.setSizeFull();
content.getStyle().set("overflow", "auto");
content.add(/* many components */);
```

## Best Practices

1. **Explicitly disable padding/spacing when not needed** — nested layouts accumulate unwanted space fast. Use `setPadding(false)` and `setSpacing(false)` on inner layouts.
2. **Use the right layout component** — AppLayout for app shells, Dashboard for card grids, FormLayout for forms. Don't build everything with HorizontalLayout/VerticalLayout.
3. **Don't over-nest** — a single-child HorizontalLayout just for alignment is unnecessary. Use `setAlignSelf()` on the parent instead.
4. **Use Lumo utility classes for spacing fine-tuning** (Lumo theme only) — `LumoUtility.Gap`, `LumoUtility.Padding` provide more granular control via `addClassNames()`. For Aura, use custom CSS or `--vaadin-*-layout-gap` properties.
5. **Use custom CSS properties for global adjustments** — `--vaadin-horizontal-layout-gap`, `--vaadin-vertical-layout-gap`, `--vaadin-*-layout-padding`, `--vaadin-*-layout-margin`.

## Detailed Reference

For a quick-reference cheat sheet of defaults, alignment values, and troubleshooting patterns, see `references/layout-cheatsheet.md`.
