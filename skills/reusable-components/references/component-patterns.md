# Reusable Component Patterns Reference

## Decision Matrix: Which Base to Use?

| Scenario | Base class | Why |
|----------|-----------|-----|
| Extracting a view section into its own component | `Composite<T>` | Hides internals, clean API |
| Pre-configured variant of existing component | Extend the component | You want the full parent API |
| Combining multiple components into one | `Composite<T>` | Hides root API, clean encapsulation |
| Field that works with Binder | `AbstractField<C, V>` | Handles value change boilerplate |
| Container that accepts children | `Component` + `HasComponents` | Provides standard add/remove API |
| Layout-like component with named slots | `Composite<T>` with slot methods | Explicit API over generic add() |

## Composite<T> Template

```java
public class MyComponent extends Composite<VerticalLayout> {

    // Declare internal components as fields
    private final H3 title = new H3();
    private final Span description = new Span();

    public MyComponent() {
        // Configure the root layout
        getContent().setPadding(true);
        getContent().setSpacing(false);
        getContent().add(title, description);
    }

    // Public API — what users of this component can do
    public void setTitle(String text) {
        title.setText(text);
    }

    public void setDescription(String text) {
        description.setText(text);
    }
}
```

Key points:
- `getContent()` returns the root component (protected, not public)
- Internal components are private fields
- Public methods define the component's contract

## AbstractField<C, V> Template

```java
public class RangeSlider extends AbstractField<RangeSlider, Range> {

    public RangeSlider() {
        super(Range.empty());  // default/empty value
        // Build the UI elements
    }

    @Override
    protected void setPresentationValue(Range value) {
        // Update the visual representation
        // Called by setValue() after validation
    }

    // Optional: customize equality check
    @Override
    protected boolean valueEquals(Range value1, Range value2) {
        return Objects.equals(value1, value2);
    }
}
```

## Custom Event Template

```java
// Define the event
public class SelectionEvent extends ComponentEvent<MyComponent> {

    private final String selectedItem;

    public SelectionEvent(MyComponent source, boolean fromClient, String selectedItem) {
        super(source, fromClient);
        this.selectedItem = selectedItem;
    }

    public String getSelectedItem() {
        return selectedItem;
    }
}

// In the component — fire and expose listener registration
public class MyComponent extends Composite<Div> {

    public Registration addSelectionListener(
            ComponentEventListener<SelectionEvent> listener) {
        return addListener(SelectionEvent.class, listener);
    }

    private void onItemClicked(String item) {
        fireEvent(new SelectionEvent(this, false, item));
    }
}
```

## Signal Binding Pattern

```java
// Child component accepts a signal and binds reactively
public class EmployeeDetail extends Composite<VerticalLayout> {
    public EmployeeDetail(ValueSignal<Employee> employee) {
        Span nameLabel = new Span();
        nameLabel.bindText(employee.map(Employee::getName));
        
        Span emailLabel = new Span();
        emailLabel.bindText(employee.map(Employee::getEmail));
        
        getContent().add(nameLabel, emailLabel);
    }
}

// Parent creates signal, children bind to it
ValueSignal<Employee> selected = new ValueSignal<>(null);
EmployeeDetail detail = new EmployeeDetail(selected);
// Updating the signal updates all bound children:
selected.set(employee);
```

## Callback Pattern

```java
// One-off component with callback actions
public class ActionFooter extends Composite<HorizontalLayout> {
    public ActionFooter(Runnable onSave, Runnable onCancel) {
        getContent().add(
            new Button("Save", e -> onSave.run()),
            new Button("Cancel", e -> onCancel.run())
        );
    }
}

// Usage in view:
ActionFooter footer = new ActionFooter(
    () -> service.save(entity),
    () -> UI.getCurrent().navigate(ListView.class)
);
```

## Builder / Fluent API Pattern

For components with many optional configuration properties:

```java
public class DataCard extends Composite<VerticalLayout> {

    public DataCard withTitle(String title) {
        // add title component
        return this;
    }

    public DataCard withIcon(VaadinIcon icon) {
        // add icon
        return this;
    }

    public DataCard withValue(String value) {
        // add value display
        return this;
    }
}

// Usage:
new DataCard()
    .withTitle("Revenue")
    .withIcon(VaadinIcon.DOLLAR)
    .withValue("$1.2M");
```

## Advanced: Field and Container Interfaces

### HasValue<E, V> — for field-like components

Implement when your component represents an editable value that should work with `Binder`. This is the most important interface for form integration.

Requirements: define `setValue()`, `getValue()`, value change events, empty value, read-only mode, and required indicator.

Extend `AbstractField<C, V>` for a base implementation that handles most of the boilerplate.

```java
public class StarRating extends AbstractField<StarRating, Integer> {

    public StarRating() {
        super(0); // default/empty value
        // build UI: 5 clickable star icons
    }

    @Override
    protected void setPresentationValue(Integer value) {
        // update the star icons to reflect the value
    }
}
```

### HasComponents — for container components

Implement when your component can accept arbitrary child components. Provides `add()`, `remove()`, `removeAll()`.

```java
@Tag("div")
public class CardGroup extends Component implements HasComponents {
    // add() and remove() are provided by the interface
}
```

Only implement this when arbitrary children make sense. If your component has specific slots, use explicit methods instead:

```java
public void setHeader(Component header) { ... }
public void setBody(Component body) { ... }
```

### HasStyle — for style customization

Automatically available on components that extend `Component`. Provides `addClassName()`, `getStyle()`, etc. Consider whether to delegate or restrict style access in your Composite.

## Lifecycle Pattern: Event Bus Subscription

```java
public class NotificationBell extends Composite<Div> {

    private Registration busRegistration;

    @Override
    protected void onAttach(AttachEvent event) {
        var eventBus = event.getSession().getAttribute(AppEventBus.class);
        busRegistration = eventBus.subscribe(NotificationEvent.class, this::onNotification);
    }

    @Override
    protected void onDetach(DetachEvent event) {
        if (busRegistration != null) {
            busRegistration.remove();
            busRegistration = null;
        }
    }

    private void onNotification(NotificationEvent event) {
        // Update badge count, etc.
    }
}
```

## Anti-patterns

### Leaking internals

```java
// BAD: exposes the internal layout
public class UserCard extends Composite<HorizontalLayout> {
    public HorizontalLayout getLayout() {
        return getContent(); // Don't expose this!
    }
}

// GOOD: expose intent-based methods
public class UserCard extends Composite<HorizontalLayout> {
    public void setCompact(boolean compact) {
        getContent().setSpacing(!compact);
        getContent().setPadding(!compact);
    }
}
```

### Over-extending

```java
// BAD: extending VerticalLayout just to group components
public class LoginForm extends VerticalLayout {
    // Users can call add(), remove(), setSpacing(), etc.
    // which may break the form's layout
}

// GOOD: use Composite to control the API
public class LoginForm extends Composite<VerticalLayout> {
    // Only your public methods are accessible
}
```

### God components

```java
// BAD: one component doing everything
public class UserManagementPanel extends Composite<Div> {
    // user list + user form + role editor + permission matrix + audit log
    // This should be 5+ separate components
}
```

Split into focused components and compose them at the view level.

### Premature extraction

```java
// BAD: extracting a trivial 3-line setup into its own class
public class SaveButton extends Composite<Button> {
    public SaveButton() {
        getContent().setText("Save");
        getContent().addThemeVariants(ButtonVariant.LUMO_PRIMARY);
    }
}

// GOOD: just configure inline — no component needed
var saveButton = new Button("Save");
saveButton.addThemeVariants(ButtonVariant.LUMO_PRIMARY);
```

Don't extract until there's a real reason: reuse, encapsulated state, or growing complexity.
