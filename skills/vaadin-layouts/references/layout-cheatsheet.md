# Layout Quick-Reference Cheatsheet

## Default Comparison

| Property        | VerticalLayout | HorizontalLayout |
|-----------------|----------------|------------------|
| Width           | 100% (fills)   | undefined (hugs) |
| Height          | undefined (hugs) | undefined (hugs) |
| Spacing         | ON             | ON               |
| Padding         | ON             | OFF              |
| Margin          | OFF            | OFF              |
| Align items     | START (left)   | STRETCH (vertical) |
| Justify content | START (top)    | START (left)     |

## Alignment Values

### JustifyContentMode (main axis distribution)

| Value     | Effect |
|-----------|--------|
| `START`   | Pack to start (default) |
| `CENTER`  | Center all items |
| `END`     | Pack to end |
| `BETWEEN` | Even space between, none at edges |
| `AROUND`  | Even space around, half at edges |
| `EVENLY`  | Perfectly even space everywhere |

### Alignment (cross axis)

| Value     | Effect |
|-----------|--------|
| `START`   | Align to start edge |
| `CENTER`  | Center on cross axis |
| `END`     | Align to end edge |
| `STRETCH` | Fill cross axis (if size undefined) |
| `BASELINE`| Align text baselines (HorizontalLayout only) |

## Spacing Theme Variants

To use a non-default spacing size:

```java
layout.setSpacing(false);                    // disable default
layout.getThemeList().add("spacing-xs");     // add variant
```

| Variant       | Size |
|--------------|------|
| `spacing-xs` | Extra small |
| `spacing-s`  | Small |
| `spacing`    | Medium (default when enabled) |
| `spacing-l`  | Large |
| `spacing-xl` | Extra large |

Or use custom pixel values:

```java
layout.setSpacing(8, Unit.PIXELS);
```

## CSS Custom Properties

| Property | Component |
|----------|-----------|
| `--vaadin-horizontal-layout-gap` | HorizontalLayout |
| `--vaadin-horizontal-layout-padding` | HorizontalLayout |
| `--vaadin-horizontal-layout-margin` | HorizontalLayout |
| `--vaadin-vertical-layout-gap` | VerticalLayout |
| `--vaadin-vertical-layout-padding` | VerticalLayout |
| `--vaadin-vertical-layout-margin` | VerticalLayout |

## Common Patterns

### Toolbar with left and right sections

```java
HorizontalLayout toolbar = new HorizontalLayout();
toolbar.setWidthFull();
toolbar.setPadding(true);
toolbar.setJustifyContentMode(FlexComponent.JustifyContentMode.BETWEEN);
toolbar.setAlignItems(FlexComponent.Alignment.CENTER);

toolbar.add(leftGroup, rightGroup);
```

### Centered card in full-height view

```java
VerticalLayout view = new VerticalLayout();
view.setSizeFull();
view.setJustifyContentMode(FlexComponent.JustifyContentMode.CENTER);
view.setAlignItems(FlexComponent.Alignment.CENTER);

view.add(card);
```

### Sidebar + content area

```java
HorizontalLayout shell = new HorizontalLayout();
shell.setSizeFull();
shell.setSpacing(false);
shell.setPadding(false);

VerticalLayout sidebar = new VerticalLayout();
sidebar.setWidth("250px");
shell.setFlexShrink(0, sidebar); // prevent sidebar from shrinking

VerticalLayout content = new VerticalLayout();
shell.setFlexGrow(1, content);   // content takes remaining space

shell.add(sidebar, content);
```

### Scrollable content area

```java
Scroller scroller = new Scroller();
scroller.setSizeFull();

VerticalLayout content = new VerticalLayout(); // no explicit height — hugs content
content.add(/* many components */);

scroller.setContent(content);
```

Wrap scrollable layouts in a `Scroller` rather than using CSS overflow. Don't set an explicit height on the inner layout — let it hug its content so the Scroller knows when to scroll.

## Choosing the Right Layout Component

| Need | Use | NOT |
|------|-----|-----|
| App shell with nav drawer + header + content | `AppLayout` | Nested HorizontalLayout/VerticalLayout |
| Grid of cards / dashboard widgets | `Dashboard` or CSS Grid | Wrapped HorizontalLayouts |
| Form field arrangement | `FormLayout` | Manual VerticalLayout with spacing |
| Single row of components | `HorizontalLayout` | — |
| Single column of components | `VerticalLayout` | — |
| Dynamic horizontal/vertical toggle | `FlexLayout` | Swapping between H/V Layout |

## Sizing Behavior (the shrinking trap)

By default, `setWidthFull()` / `setHeightFull()` / `setSizeFull()` set literal `width: 100%` / `height: 100%`. Inside a flex layout this means "100% of the parent," not "the remaining space," so a fixed-size sibling can shrink because every child has `flex-shrink: 1`.

Reliable fixes:
- `layout.setFlexGrow(1, fullSizeComponent)` (or `layout.expand(fullSizeComponent)`) — takes the remaining space without forcing 100%.
- `layout.setFlexShrink(0, fixedComponent)` — prevents the fixed sibling from shrinking below its specified size.

> **Experimental:** Enabling the `layoutComponentImprovements` feature flag (Flow only) rewires `setWidthFull` / `setHeightFull` / `setSizeFull` to apply `flex: 1` and set min-size to 0 on nested layouts. Off by default — don't assume it without confirming.

## Troubleshooting Decision Tree

**Problem: Component renders smaller than specified size**
1. Is the parent constraining it? → Check parent sizing chain up to the root
2. Is `setFlexShrink` set to a non-zero value? → Set `setFlexShrink(0, component)` to prevent shrinking

**Problem: Unwanted scrollbars / overflow**
1. Is it a nested layout? → Set `setMinHeight("0")` or `setMinWidth("0")` on the overflowing component
2. Is content just too wide? → Enable wrapping with `setWrap(true)` on HorizontalLayout

**Problem: Extra whitespace around content**
1. Are you nesting layouts? → Disable padding/spacing on inner layouts: `setPadding(false)`, `setSpacing(false)`
2. Is margin enabled? → Check `setMargin(false)`
