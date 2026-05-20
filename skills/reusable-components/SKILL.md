---
name: reusable-components
description: >
  Guide Claude on structuring Vaadin 25 Flow views into focused, reusable components.
  This skill should be used when the user asks to "structure a view", "organize view code",
  "break down a complex view", "extract a component", "split a view into components",
  "simplify a large view", "create a reusable component", "use Composite",
  "compose components", or when a view is growing beyond ~200 lines,
  has multiple logical sections, or contains repeated UI patterns.
version: 0.2.0
---

# Structuring Views into Components in Vaadin 25

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## When to Extract Components

Not every view needs decomposition. Extract when you see these signals:

- **View exceeds ~200 lines** — hard to navigate and reason about
- **Cohesive groups** — a cluster of components that form a logical unit (e.g., a filter bar, a detail panel)
- **Repeated patterns** — the same group of components appears in multiple views
- **Isolated state** — a section manages its own state independently from the rest of the view

### Before and After

A monolithic `OrderView` with filters, a grid, and a detail panel all in one class:

```java
// BEFORE: everything in one 300-line view
public class OrderView extends Composite<VerticalLayout> {
    // 15 filter fields, grid setup, detail panel, all interleaved...
}

// AFTER: decomposed into focused components
public class OrderView extends Composite<VerticalLayout> {

    private final OrderFilterBar filterBar = new OrderFilterBar();
    private final Grid<Order> grid = new Grid<>(Order.class);
    private final OrderDetailPanel detailPanel = new OrderDetailPanel();

    public OrderView(OrderService service) {
        filterBar.addFilterChangeListener(e -> refreshGrid(service, e.getFilter()));
        grid.asSingleSelect().addValueChangeListener(e -> detailPanel.setOrder(e.getValue()));
        getContent().add(filterBar, grid, detailPanel);
    }
}
```

Each extracted component owns its own layout and state; the view wires them together.

## The Extraction Workflow

1. **Identify cohesive groups** — look for clusters of fields, buttons, or layouts that serve one purpose.
2. **Define the boundary** — what does the component own (internal state, layout) vs. what does it need from outside (data, configuration)?
3. **Choose the right base** — use `Composite<T>` by default. Extend an existing component only when you want its full API. Use `AbstractField<C, V>` when the component represents an editable value for Binder. See the next section for details.
4. **Extract and wire up** — move the component's fields and layout code into its own class. Pass required data through constructors or setters.
5. **Connect parent and child** — use method calls for parent→child communication, custom events for child→parent. See "Wiring Communication" below.

## Choosing the Right Base: Composite vs. Extend

When extracting a view section into its own component, you need to choose a base class. Lead with `Composite<T>` — it's the right choice in most cases.

### Composite<T> (recommended default)

Use when you want to **hide** the root component's API and expose only what you explicitly define. `Composite<T>` wraps a root component and makes `getContent()` protected, so users can only interact through your public methods.

```java
public class UserCard extends Composite<HorizontalLayout> {

    private final Avatar avatar = new Avatar();
    private final Span name = new Span();
    private final Span role = new Span();

    public UserCard() {
        VerticalLayout info = new VerticalLayout(name, role);
        info.setSpacing(false);
        info.setPadding(false);

        getContent().setAlignItems(FlexComponent.Alignment.CENTER);
        getContent().add(avatar, info);
    }

    public void setUser(String userName, String userRole, String imageUrl) {
        name.setText(userName);
        role.setText(userRole);
        avatar.setImage(imageUrl);
        avatar.setName(userName);
    }
}
```

Good for: compound components, encapsulated UI blocks, extracted view sections.

### Extend an existing component

Use when you want to **add** to an existing component's API. The parent class's full public API remains accessible.

```java
public class PrimaryButton extends Button {
    public PrimaryButton(String text) {
        super(text);
        addThemeVariants(ButtonVariant.PRIMARY);
    }
}
```

Good for: pre-configured variants, adding convenience methods, specializing behavior.

Risk: every public method on the parent is part of your API. Users can call anything on Button, which may break your component's invariants.

**Prefer Composite for new components.** It produces cleaner APIs and prevents accidental misuse. See the reference file for the full decision matrix.

## Defining the Component's API

### Principles

1. **Expose intent, not implementation** — public methods should describe what the component does, not how it's built. `setUser(name, role)` is better than exposing the internal `Span` and `Avatar`.

2. **Make invalid states unrepresentable** — if your component requires both a title and an icon, take them as constructor parameters rather than offering separate setters that can be called in any order.

3. **Follow Vaadin conventions** — users expect familiar patterns:
   - `setValue()` / `getValue()` for components with a value
   - `setLabel()` for field labels
   - `setEnabled()` / `setReadOnly()` for interaction states
   - `addXxxListener()` for events

4. **Use typed events for reusable components** — define custom `ComponentEvent` subclasses for components intended for reuse. For one-off components, lightweight callbacks (`Runnable`, `Consumer<T>`) are appropriate. Typed events integrate with Vaadin's event system and support `@DomEvent` for client-side events.

### Constructor Design

Provide a no-arg constructor for compatibility with Vaadin's declarative systems, then offer convenience constructors for common usage:

```java
public class StatusBadge extends Composite<Span> {

    public StatusBadge() {
        // default state
    }

    public StatusBadge(Status status) {
        setStatus(status);
    }

    public void setStatus(Status status) {
        getContent().setText(status.getLabel());
        getContent().getElement().getThemeList().clear();
        getContent().getElement().getThemeList().add("badge " + status.getTheme());
    }
}
```

## Wiring Communication Between Components

When a view is split into parent and child components, they need to communicate. Use the right pattern for the direction.

**Choosing a pattern:**
- **Setters** — simplest way to push data down. Use by default.
- **Signals** — reduce boilerplate when multiple components share the same state.
- **Events** — standard Vaadin pattern for child-to-parent communication. Use for reusable components.
- **Callbacks** — lighter than events. Use for one-off components.

You can mix patterns in the same view.

### Parent → Child: method calls

The parent holds a reference to the child and calls its public methods directly:

```java
// In the parent view
detailPanel.setOrder(selectedOrder);
filterBar.reset();
```

### Child → Parent: custom events

Children should not know about their parent. Fire a typed event and let the parent listen:

```java
public class OrderFilterBar extends Composite<HorizontalLayout> {

    public Registration addFilterChangeListener(
            ComponentEventListener<FilterChangeEvent> listener) {
        return addListener(FilterChangeEvent.class, listener);
    }

    private void onFilterChanged() {
        fireEvent(new FilterChangeEvent(this, false, buildFilter()));
    }

    public static class FilterChangeEvent extends ComponentEvent<OrderFilterBar> {
        private final OrderFilter filter;

        public FilterChangeEvent(OrderFilterBar source, boolean fromClient,
                OrderFilter filter) {
            super(source, fromClient);
            this.filter = filter;
        }

        public OrderFilter getFilter() {
            return filter;
        }
    }
}
```

### Reactive alternative: signals

When multiple child components depend on the same piece of state, signals reduce boilerplate. Instead of calling setters on each child, define a `ValueSignal` and let children bind to it:

```java
public class EmployeeDetail extends Composite<VerticalLayout> {
    private final Span nameLabel = new Span();
    private final Span emailLabel = new Span();

    public EmployeeDetail(ValueSignal<Employee> employee) {
        nameLabel.bindText(employee.map(Employee::getName));
        emailLabel.bindText(employee.map(Employee::getEmail));
        getContent().add(nameLabel, emailLabel);
    }
}
```

Use setters when updates are triggered by specific events and the flow is easy to follow. Use signals when multiple components share the same state and you want to avoid manual coordination. See the `signals` skill for details.

### Lightweight alternative: callbacks

For one-off components where defining a full event class is overkill, accept a `Runnable` or `Consumer<T>`:

```java
public class DetailFooter extends Composite<Div> {
    public DetailFooter(Runnable onEdit, Runnable onDelete) {
        Button editButton = new Button("Edit", e -> onEdit.run());
        Button deleteButton = new Button("Delete", e -> onDelete.run());
        getContent().add(editButton, deleteButton);
    }
}
```

Use events for reusable components (type-safe, `Registration`-based). Use callbacks for one-off components where simplicity matters.

### Wrapper with Slots

A common pattern for layout-style components with named areas:

```java
public class PageHeader extends Composite<HorizontalLayout> {

    private final Div titleSlot = new Div();
    private final Div actionsSlot = new Div();

    public PageHeader() {
        getContent().setWidthFull();
        getContent().setAlignItems(FlexComponent.Alignment.CENTER);
        getContent().setJustifyContentMode(FlexComponent.JustifyContentMode.BETWEEN);
        getContent().add(titleSlot, actionsSlot);
    }

    public void setTitle(String title) {
        titleSlot.removeAll();
        titleSlot.add(new H2(title));
    }

    public void setActions(Component... actions) {
        actionsSlot.removeAll();
        HorizontalLayout actionBar = new HorizontalLayout(actions);
        actionsSlot.add(actionBar);
    }
}
```

## Lifecycle Considerations

- **onAttach()** — called when the component is added to the UI. Use for initialization that requires the component to be in the DOM (e.g., accessing session data, subscribing to event buses).
- **onDetach()** — called before removal from the UI. Use for cleanup (e.g., unsubscribing from event buses, releasing resources).
- Always clean up in `onDetach()` what you set up in `onAttach()` to prevent memory leaks.

## Mixin Interfaces

Vaadin provides mixin interfaces that add standard functionality to your component. Implement only what your component needs:

- `HasSize` — `setWidth()`, `setHeight()`, and related sizing methods
- `HasComponents` — `add()`, `remove()`, and child component management
- `HasText` — `setText()` and `getText()`
- `HasEnabled` — `setEnabled()` for enabling and disabling
- `HasTheme` — theme variant support

```java
public class StatusBadge extends Composite<HorizontalLayout>
        implements HasSize {
    // Now callers can use setWidth(), setHeight(), etc.
}
```

## Best Practices

1. **Start with the view, extract when needed** — don't pre-optimize. Build the view first, then extract when signals appear (see "When to Extract Components").
2. **Prefer Composite over direct extension** — it gives you API control and prevents leaking internals.
3. **Keep components focused** — a component should do one thing well. If it has too many responsibilities, split it.
4. **Use typed events for child→parent communication** — `addXxxListener()` returning `Registration` is the Vaadin way.
5. **Don't expose internal components** — return data from getters, not the underlying Spans and Divs.
6. **Test extracted components in isolation** — reusable components should be testable with Vaadin's UI unit testing framework without needing a full application context.
