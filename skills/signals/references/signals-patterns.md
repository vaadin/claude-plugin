# Signals Patterns Reference

## Local Signal Types

| Type | Purpose | Key Methods |
|------|---------|-------------|
| `ValueSignal<T>` | Single value, single UI session | `get()`, `peek()`, `set(T)`, `update(fn)`, `replace(old, new)`, `modify(Consumer)`, `map(fn)`, `asReadonly()` |
| `ListSignal<T>` | Ordered list with per-entry reactivity | `insertFirst()`, `insertLast()`, `insertAt(index, val)`, `remove(entry)`, `clear()`, `moveTo(entry, index)`, `get()`/`peek()` returns `List<ValueSignal<T>>` |

Package: `com.vaadin.flow.signals.local`

## Shared Signal Types

| Type | Purpose | Key Methods |
|------|---------|-------------|
| `SharedValueSignal<T>` | Single value, shared across users | `get()`, `peek()`, `set(T)`, `update(fn)`, `replace(old, new)`, `asReadonly()` |
| `SharedNumberSignal` | Numeric with atomic arithmetic | `set(double)`, `incrementBy(double)`, `getAsInt()`, `peek()`, `get()` |
| `SharedListSignal<T>` | Ordered list, shared | `insertFirst()`, `insertLast()`, `insertAt(val, ListPosition)`, `remove()`, `moveTo(entry, ListPosition)` |
| `SharedMapSignal<T>` | Key-value map (String keys) | `put(key, val)`, `putIfAbsent(key, val)`, `remove(key)`, `peek()` returns `Map<String, SharedValueSignal<T>>` |
| `SharedNodeSignal` | Tree (value + list + map children) | `putChildWithValue(key, val)`, `insertChildWithValue(val, ListPosition)`, `asMap(Class)` |

Package: `com.vaadin.flow.signals.shared`

## Scope Comparison

| Scope | Declaration | Lifetime | Sharing |
|-------|-------------|----------|---------|
| View | Private instance field | New per navigation | Single view instance |
| Session | `@Component @VaadinSessionScope` bean | HTTP session | All tabs in same session |
| Application | `@Component` (singleton) bean | Application | All users |

## Effect and Binding Patterns

### Effect -- custom reactive logic

```java
Signal.effect(component, () -> {
    String first = firstName.get();
    String last = lastName.get();
    boolean hasName = !first.isEmpty() || !last.isEmpty();
    nameLabel.setText(first + " " + last);
    nameLabel.setVisible(hasName);
});
```

### Contextual effect -- detect initial run and background changes

```java
Signal.effect(component, ctx -> {
    String value = signal.get();
    span.setText(value);
    if (!ctx.isInitialRun() && ctx.isBackgroundChange()) {
        span.getElement().flashClass("highlight");
    }
});
```

### Component bindings -- direct, preferred over effects for simple cases

```java
// Text
label.bindText(signal);
label.bindText(counter.map(c -> "Count: " + c));
label.bindText(() -> firstName.get() + " " + lastName.get());

// Visibility
panel.bindVisible(showSignal);
panel.bindVisible(searchText.map(t -> !t.isEmpty()));

// Enabled state
button.bindEnabled(formValidSignal);

// Two-way form field
field.bindValue(signal, signal::set);

// Read-only, required indicator
field.bindReadOnly(lockedSignal);
field.bindRequiredIndicatorVisible(requiredSignal);

// Placeholder, helper text
field.bindPlaceholder(placeholderSignal);
field.bindHelperText(helperSignal);

// Size
panel.bindWidth(widthSignal);
panel.bindHeight(heightSignal);

// Dynamic children
container.bindChildren(listSignal, itemSignal -> createRow(itemSignal));

// Data components (Grid, ComboBox)
Signal.effect(grid, () -> grid.setItems(listSignal.getValues().toList()));

// Class names
panel.bindClassName("highlighted", boolSignal);
panel.bindClassNames(classListSignal);

// Theme names
grid.bindThemeName("compact", compactSignal);
grid.bindThemeNames(themeListSignal);

// Inline styles
div.getStyle().bind("background-color", colorSignal);

// Change callback
span.bindText(signal).onChange(ctx -> {
    if (ctx.isBackgroundChange()) {
        ctx.getElement().flashClass("highlight");
    }
});
```

### Element-level bindings

```java
element.bindText(signal);
element.bindAttribute("aria-label", labelSignal);
element.bindProperty("hidden", hiddenSignal, null);
element.bindProperty("hidden", hiddenSignal, hiddenSignal::set);
element.bindVisible(visibleSignal);
element.bindEnabled(enabledSignal);
element.getClassList().bind("active", isActiveSignal);
element.getClassList().bind(classListSignal);
element.getStyle().bind("color", colorSignal);
element.flashClass("highlight");
```

## Two-Way Binding Patterns

### Simple field binding

```java
TextField field = new TextField("Name");
field.bindValue(nameSignal, nameSignal::set);
```

### Record property binding (immutable -- use `updater`)

```java
record Todo(String text, boolean done) {
    Todo withDone(boolean done) { return new Todo(this.text, done); }
}

ValueSignal<Todo> todoSignal = new ValueSignal<>(new Todo("Task", false));

Checkbox checkbox = new Checkbox();
checkbox.bindValue(todoSignal.map(Todo::done), todoSignal.updater(Todo::withDone));
```

### Bean property binding (mutable -- use `modifier`)

```java
TextField nameField = new TextField("Name");
nameField.bindValue(
    userSignal.map(User::getName),
    userSignal.modifier(User::setName));
```

### Comparison

| Method | Use Case |
|--------|----------|
| `map(getter)` | Read-only transformation |
| `bindValue(signal.map(getter), signal.updater(wither))` | Two-way with immutable values (records) |
| `bindValue(signal.map(getter), signal.modifier(setter))` | Two-way with mutable beans |

## Complete Patterns

### Shared todo list (collaborative, multi-user)

```java
@Push
public class SharedTodoList extends VerticalLayout {
    record Todo(String text, boolean done) {
        Todo withDone(boolean done) { return new Todo(this.text, done); }
    }

    private final SharedListSignal<Todo> todos = new SharedListSignal<>(Todo.class);

    public SharedTodoList() {
        TextField input = new TextField("New todo");
        Button addBtn = new Button("Add", e -> {
            if (!input.getValue().isBlank()) {
                todos.insertLast(new Todo(input.getValue(), false));
                input.clear();
            }
        });

        VerticalLayout list = new VerticalLayout();
        list.setPadding(false);

        list.bindChildren(todos, todoSignal -> {
            HorizontalLayout row = new HorizontalLayout();
            row.setAlignItems(FlexComponent.Alignment.CENTER);

            Checkbox checkbox = new Checkbox();
            checkbox.bindValue(
                todoSignal.map(Todo::done),
                todoSignal.updater(Todo::withDone));

            Span text = new Span();
            text.bindText(todoSignal.map(Todo::text));
            text.getStyle().bind("text-decoration",
                todoSignal.map(t -> t.done() ? "line-through" : "none"));

            Button removeBtn = new Button("Delete",
                e -> todos.remove(todoSignal));

            row.add(checkbox, text, removeBtn);
            return row;
        });

        add(new HorizontalLayout(input, addBtn), list);
    }
}
```

### Local form with computed validation

```java
public class SignupForm extends VerticalLayout {
    private final ValueSignal<String> email = new ValueSignal<>("");
    private final ValueSignal<String> password = new ValueSignal<>("");
    private final Signal<Boolean> formValid = Signal.computed(() ->
        email.get().contains("@") && password.get().length() >= 8);

    public SignupForm() {
        TextField emailField = new TextField("Email");
        emailField.bindValue(email, email::set);

        PasswordField passwordField = new PasswordField("Password");
        passwordField.bindValue(password, password::set);

        Button submit = new Button("Sign Up");
        submit.bindEnabled(formValid);
        submit.addClickListener(e -> {
            // process signup with email.peek(), password.peek()
        });

        add(emailField, passwordField, submit);
    }
}
```

### Session-scoped preferences

```java
@Component
@VaadinSessionScope
public class UserPreferencesSignal {
    private final ValueSignal<String> theme = new ValueSignal<>("light");
    private final ValueSignal<Locale> locale = new ValueSignal<>(Locale.ENGLISH);

    public ValueSignal<String> getThemeSignal() { return theme; }
    public Signal<Locale> getLocaleSignal() { return locale.asReadonly(); }
    public void setLocale(Locale loc) { locale.set(loc); }
}
```

Usage in a view:

```java
@Route("settings")
public class SettingsView extends VerticalLayout {
    public SettingsView(UserPreferencesSignal prefs) {
        ComboBox<String> themeSelect = new ComboBox<>("Theme");
        themeSelect.setItems("light", "dark");
        themeSelect.bindValue(prefs.getThemeSignal(), prefs.getThemeSignal()::set);
    }
}
```

### Read-only encapsulation

```java
@Component
public class CounterService {
    private final SharedNumberSignal counter = new SharedNumberSignal();

    public Signal<Double> getCounter() {
        return counter.asReadonly();
    }

    public void increment() {
        counter.incrementBy(1);
    }
}
```

### Dynamic phone number fields with ListSignal

```java
public class PhoneNumbersEditor extends VerticalLayout {
    private final ListSignal<String> phoneNumbers = new ListSignal<>();

    public PhoneNumbersEditor() {
        VerticalLayout phoneList = new VerticalLayout();
        phoneList.setPadding(false);

        phoneList.bindChildren(phoneNumbers, phoneSignal -> {
            HorizontalLayout row = new HorizontalLayout();
            row.setAlignItems(FlexComponent.Alignment.CENTER);

            TextField phoneField = new TextField();
            phoneField.setPlaceholder("Phone number");
            phoneField.bindValue(phoneSignal, phoneSignal::set);

            Button removeBtn = new Button(VaadinIcon.MINUS.create());
            removeBtn.addClickListener(e -> phoneNumbers.remove(phoneSignal));

            row.add(phoneField, removeBtn);
            return row;
        });

        Button addBtn = new Button("Add phone number",
            e -> phoneNumbers.insertLast(""));

        add(phoneList, addBtn);
    }
}
```

### Transactions with verification

```java
Signal.runInTransaction(() -> {
    statusSignal.verifyValue("pending");
    statusSignal.set("shipped");
    shippedAtSignal.set(Instant.now());
    trackingSignal.set(trackingNumber);
});
```

## Anti-patterns

### Mutating objects in place

```java
// BAD: won't trigger reactivity
User u = userSignal.peek();
u.setName("New Name"); // mutation not detected!

// GOOD: create new immutable value
userSignal.update(u -> new User("New Name", u.getAge()));

// GOOD: use modify() for mutable objects
userSignal.modify(u -> u.setName("New Name"));
```

### Modifying signals inside effects

```java
// BAD: throws exception (read-only transaction)
Signal.effect(span, () -> {
    otherSignal.set(someSignal.get()); // ERROR!
});

// GOOD: use computed signal instead
Signal<String> derived = Signal.computed(() -> someSignal.get());

// OK (but dangerous): runWithoutTransaction if absolutely necessary
Signal.effect(component, () -> {
    String value = sourceSignal.get();
    Signal.runWithoutTransaction(() -> {
        derivedSignal.set(value); // risk of infinite loops!
    });
});
```

### Using local signals in transactions

```java
// BAD: local signals cannot participate in transactions
ValueSignal<String> localVal = new ValueSignal<>("value");

Signal.runInTransaction(() -> {
    localVal.set("new value"); // throws exception!
});

// GOOD: use shared signals for transactional updates
SharedValueSignal<String> sharedVal = new SharedValueSignal<>(String.class);

Signal.runInTransaction(() -> {
    sharedVal.set("new value"); // works
});
```

### Using shared signals without Push

```java
// BAD: other users won't see updates until they interact
SharedNumberSignal shared = new SharedNumberSignal();

// GOOD: enable Push on your AppShellConfigurator
@Push
public class AppConfig implements AppShellConfigurator {}
```

### Using get() outside reactive context

```java
// BAD: throws IllegalStateException
String value = signal.get(); // outside effect/computed

// GOOD: use peek() outside reactive contexts
String value = signal.peek();

// GOOD: get() inside reactive context
Signal.effect(component, () -> {
    String value = signal.get(); // inside effect, this is correct
});
```

### Effect for simple property binding

```java
// BAD: unnecessary boilerplate
Signal.effect(label, () -> label.setText(signal.get()));

// GOOD: use direct binding
label.bindText(signal);
```
