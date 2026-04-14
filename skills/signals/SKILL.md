---
name: signals
description: >
  Guide Claude on using Vaadin Signals for reactive state management in Vaadin 25 Flow.
  This skill should be used when the user asks to "use signals", "manage state reactively",
  "share state between users", "use reactive state", "use ValueSignal",
  "use ListSignal", "use SharedValueSignal", "use SharedNumberSignal",
  "use SharedListSignal", "use SharedMapSignal", "use SharedNodeSignal",
  "use local signals", "use shared signals", "use computed signals",
  "bind signals to components", "bindText", "bindValue", "bindEnabled",
  "bindVisible", "bindChildren", or needs help with reactive UI updates,
  signal transactions, signal effects, signal bindings, or
  thread-safe state management in Vaadin Flow.
version: 0.2.0
---

# Reactive State Management with Signals in Vaadin 25

Use the Vaadin MCP tools (`search_vaadin_docs`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25.1"` and `ui_language` to `"java"`.

## What Signals Are

Signals are a reactive state management system for Vaadin Flow. A signal holds a value, and when that value changes, all dependent parts of the UI automatically update without manually adding and removing change listeners.

Key properties:
- **Reactive** -- changes propagate automatically to dependent UI
- **Automatic dependency tracking** -- effects detect which signals they depend on
- **Immutable values** -- signals work best with immutable types (String, Integer, Java records)
- **Thread-safe** -- signals can be updated from any thread without `ui.access()`
- **Two categories** -- local signals for single-session UI state, shared signals for multi-user/transactional state

## Local vs Shared: Which to Use

| | Local Signals | Shared Signals |
|---|---|---|
| Scope | Single UI instance | Multiple users/sessions |
| Cluster support | Single server only | Works across cluster |
| Transactions | No | Yes |
| Classes | `ValueSignal<T>`, `ListSignal<T>` | `SharedValueSignal<T>`, `SharedNumberSignal`, `SharedListSignal<T>`, `SharedMapSignal<T>`, `SharedNodeSignal` |
| Package | `com.vaadin.flow.signals.local` | `com.vaadin.flow.signals.shared` |

Use **local signals** when:
- State is only relevant to a single user's UI session
- Managing UI state like form visibility, panel expansion, local filters
- Dynamic lists for a single user (use `ListSignal`)

Use **shared signals** when:
- Multiple users need to see the same data in real-time
- You need transactional guarantees for state changes
- Building collaborative features like live dashboards or multi-user editing

When using shared signals for multi-user scenarios, enable `@Push` so changes propagate immediately to all connected UIs.

## Local Signals

### ValueSignal -- a single value

```java
import com.vaadin.flow.signals.local.ValueSignal;

ValueSignal<String> name = new ValueSignal<>("initial value");
name.set("new value");              // write
String current = name.peek();       // read (non-reactive, outside effects)
name.update(n -> n.toUpperCase());  // atomic read-modify-write
boolean ok = name.replace("OLD", "NEW"); // compare-and-set
```

Reading values:
- `get()` -- reactive read, registers dependency. Must only be called inside a reactive context (effect, computed, transaction). Throws `IllegalStateException` outside reactive context.
- `peek()` -- non-reactive read, no dependency. Use in click listeners, initialization, or any code outside a reactive context.

Custom equality checkers to control when updates are skipped:

```java
ValueSignal<String> name = new ValueSignal<>("John",
    (a, b) -> a != null && a.equalsIgnoreCase(b));
name.set("john"); // no update triggered (considered equal)
```

Read-only view for encapsulation:

```java
Signal<String> readOnly = name.asReadonly();
```

Working with mutable values -- use `modify()`:

```java
ValueSignal<User> userSignal = new ValueSignal<>(new User("Jane", 25));
userSignal.modify(user -> user.setAge(26)); // correct
// Do NOT mutate objects directly -- changes won't be detected
```

Transforming values with `map()`:

```java
Signal<String> upper = name.map(String::toUpperCase);
```

### ListSignal -- ordered list with per-entry reactivity

```java
import com.vaadin.flow.signals.local.ListSignal;

ListSignal<String> tags = new ListSignal<>();
ValueSignal<String> last = tags.insertLast("item");   // add to end
ValueSignal<String> first = tags.insertFirst("item");  // add to beginning
ValueSignal<String> mid = tags.insertAt(1, "middle");  // add at index

tags.remove(last);   // remove specific entry
tags.clear();        // remove all
tags.moveTo(first, 2); // reorder without recreating

List<ValueSignal<String>> entries = tags.peek(); // snapshot
last.set("updated"); // update individual entry (only its dependents re-render)
```

Each entry is an independent `ValueSignal`, so updating one entry only triggers re-renders for components bound to that entry, not the entire list.

## Shared Signals

### SharedValueSignal -- single value, shared across users

```java
import com.vaadin.flow.signals.shared.SharedValueSignal;

SharedValueSignal<String> name = new SharedValueSignal<>(String.class);
name.set("John Doe");
String current = name.peek();
name.update(n -> n.toUpperCase());
name.replace("expected", "newValue");
```

### SharedNumberSignal -- numeric with atomic arithmetic

```java
import com.vaadin.flow.signals.shared.SharedNumberSignal;

SharedNumberSignal counter = new SharedNumberSignal();
counter.set(5);
counter.incrementBy(1);
counter.incrementBy(-2);
int count = counter.getAsInt();
```

### SharedListSignal -- ordered list, shared across users

```java
import com.vaadin.flow.signals.shared.SharedListSignal;
import com.vaadin.flow.signals.shared.SharedListSignal.ListPosition;

SharedListSignal<Person> people = new SharedListSignal<>(Person.class);
SharedValueSignal<Person> entry = people.insertLast(new Person("Jane", 25)).signal();
people.insertFirst(new Person("John", 30));

// Precise positioning with ListPosition
people.insertAt("item", ListPosition.after(entry));
people.insertAt("item", ListPosition.before(entry));
people.insertAt("item", ListPosition.first());
people.insertAt("item", ListPosition.last());

// Reorder
people.moveTo(entry, ListPosition.first());

// Read
List<SharedValueSignal<Person>> list = people.peek();
list.get(0).set(new Person("Updated", 26));
```

### SharedMapSignal -- key-value pairs with string keys

```java
import com.vaadin.flow.signals.shared.SharedMapSignal;

SharedMapSignal<String> config = new SharedMapSignal<>(String.class);
config.put("theme", "dark");
config.putIfAbsent("language", "en");
config.remove("language");

Map<String, SharedValueSignal<String>> map = config.peek();
SharedValueSignal<String> themeSignal = map.get("theme");
```

### SharedNodeSignal -- tree structure (value + list + map children)

```java
import com.vaadin.flow.signals.shared.SharedNodeSignal;

SharedNodeSignal user = new SharedNodeSignal();
user.putChildWithValue("name", "John Doe");
user.putChildWithValue("age", 30);
user.insertChildWithValue("Reading", ListPosition.last());

user.peek().mapChildren().get("name").asValue(String.class).peek(); // "John Doe"
user.peek().listChildren().getLast().asValue(String.class).peek();  // "Reading"

SharedMapSignal<String> mapChildren = user.asMap(String.class);
```

## Effects and Computed Signals

### Effects -- reactive callbacks tied to component lifecycle

```java
Signal.effect(component, () -> {
    // Re-runs automatically when any signal read with get() changes
    // Active while component is attached, inactive while detached
    System.out.println("Name: " + firstName.get() + " " + lastName.get());
});
```

Returns a `Registration` that can be used to remove the effect:

```java
Registration reg = Signal.effect(component, () -> { ... });
reg.remove(); // stop the effect
```

Contextual effects with `EffectContext`:

```java
Signal.effect(component, ctx -> {
    String value = priceSignal.get();
    span.setText("$" + value);
    if (!ctx.isInitialRun() && ctx.isBackgroundChange()) {
        span.getElement().flashClass("highlight");
    }
});
```

Standalone effects (not tied to a component -- must clean up manually):

```java
Registration cleanup = Signal.unboundEffect(() -> {
    System.out.println("Counter: " + counter.get());
});
cleanup.remove(); // required to avoid memory leaks
```

### Computed signals -- derived values

```java
Signal<String> fullName = Signal.computed(() ->
    firstName.get() + " " + lastName.get());
```

Computed signals are read-only, cached, and recalculate only when dependencies change.

### Signal.not() -- negate a boolean signal

```java
Signal<Boolean> notLoading = Signal.not(loading);
```

### peek() inside effects -- read without tracking

```java
Signal.effect(component, () -> {
    String name = nameSignal.get();   // tracked dependency
    int count = countSignal.peek();   // NOT tracked, effect won't re-run for this
});
```

### Signal.untracked() -- block of code without tracking

```java
Signal.effect(component, () -> {
    String tracked = trackedSignal.get();
    Signal.untracked(() -> {
        String notTracked = anotherSignal.get(); // not tracked
    });
});
```

## Component and Element Bindings

### Component-level bindings

**Text binding:**

```java
span.bindText(nameSignal);
span.bindText(counter.map(c -> String.format("Count: %.0f", c)));
span.bindText(() -> firstName.get() + " " + lastName.get()); // lambda variant
```

HTML components (`Span`, `Paragraph`, `H1`--`H6`) also accept signals in constructors:

```java
Paragraph p = new Paragraph(signal); // shorthand for new + bindText
```

**Visibility binding:**

```java
detailsPanel.bindVisible(showDetails);
noResults.bindVisible(searchText.map(String::isEmpty));
```

**Enabled state binding:**

```java
submitButton.bindEnabled(formValid);
submitButton.bindEnabled(Signal.computed(() ->
    !email.get().isEmpty() && password.get().length() >= 8));
```

**Two-way form field binding:**

```java
TextField field = new TextField("Name");
field.bindValue(nameSignal, nameSignal::set);
// User types -> signal updates; signal changes -> field updates
```

Works with all `HasValue` fields: `TextField`, `TextArea`, `Checkbox`, `NumberField`, `ComboBox`, `DatePicker`, etc.

**Two-way binding to record properties:**

```java
record Todo(String text, boolean done) {
    Todo withDone(boolean done) { return new Todo(this.text, done); }
}

ValueSignal<Todo> todoSignal = new ValueSignal<>(new Todo("Write docs", false));

Checkbox checkbox = new Checkbox();
checkbox.bindValue(todoSignal.map(Todo::done), todoSignal.updater(Todo::withDone));
```

**Two-way binding to mutable bean properties:**

```java
TextField nameField = new TextField("Name");
nameField.bindValue(userSignal.map(User::getName), userSignal.modifier(User::setName));
```

**Read-only, required indicator, placeholder, helper text:**

```java
field.bindReadOnly(lockedSignal);
field.bindRequiredIndicatorVisible(requiredSignal);
field.bindPlaceholder(placeholderSignal);
field.bindHelperText(remaining.map(r -> r + " characters remaining"));
```

**Dynamic children from list signal:**

```java
container.bindChildren(items, itemSignal -> {
    Span itemView = new Span();
    itemView.bindText(itemSignal);
    return itemView;
});
```

The factory runs once per item. Adding/removing items only affects those items. Reordering moves components, not recreates them. Updating an entry value updates only that entry's bindings.

**Binding items to data components (Grid, ComboBox):**

```java
Signal.effect(grid, () -> grid.setItems(items.getValues().toList()));
```

**Size, class names, themes, styles:**

```java
panel.bindWidth(widthSignal);
panel.bindHeight(heightSignal);
panel.bindClassName("highlighted", highlightedSignal);
panel.bindClassNames(classListSignal);
panel.bindThemeName("compact", compactSignal);
layout.getThemeList().bind("dark", darkModeSignal);
panel.getStyle().bind("background-color", bgColorSignal);
```

**Change callbacks on bindings:**

```java
span.bindText(priceSignal.map(p -> "$" + p))
    .onChange(ctx -> {
        if (ctx.isBackgroundChange()) {
            ctx.getElement().flashClass("highlight");
        }
    });
```

### Element-level bindings

For custom components or fine-grained DOM control:

```java
element.bindText(signal);
element.bindAttribute("aria-label", labelSignal);
element.bindProperty("hidden", hiddenSignal, null);       // read-only
element.bindProperty("hidden", hiddenSignal, hiddenSignal::set); // two-way
element.bindVisible(visibleSignal);
element.bindEnabled(enabledSignal);
element.getClassList().bind("active", isActiveSignal);
element.getClassList().bind(classListSignal);              // group binding
element.getStyle().bind("color", colorSignal);
element.flashClass("highlight");                           // trigger CSS animation
```

While a signal is bound to an element property, manual changes to that property throw `BindingActiveException`. Unbind with `bindText(null)`, `bindProperty(null)`, etc.

## Signal Scope Patterns

### View-scoped (per component instance)

Declare signals as private instance fields. Each navigation creates a new instance:

```java
@Route("dashboard")
public class DashboardView extends VerticalLayout {
    private final ValueSignal<Integer> counter = new ValueSignal<>(0);
    // ...
}
```

### Session-scoped

Use `@Component @VaadinSessionScope` beans. One instance per HTTP session, shared across tabs:

```java
@Component
@VaadinSessionScope
public class UserPreferences {
    private final ValueSignal<String> theme = new ValueSignal<>("light");

    public ValueSignal<String> getThemeSignal() { return theme; }
}
```

### Application-scoped (global)

Use `@Component` (singleton) beans. Shared by all users:

```java
@Component
public class SystemStatus {
    private final SharedValueSignal<String> status = new SharedValueSignal<>(String.class);

    public Signal<String> getStatus() { return status.asReadonly(); }
    public void setStatus(String s) { status.set(s); }
}
```

| Scope | Declaration | Lifetime |
|---|---|---|
| View | Private instance field | View instance lifetime, new per navigation |
| Session | `@Component @VaadinSessionScope` bean | HTTP session lifetime |
| Application | `@Component` (singleton) bean or static field | Application lifetime, shared by all users |

## Transactions

Transactions group multiple shared signal operations into a single atomic unit. Observers see all changes or none:

```java
Signal.runInTransaction(() -> {
    firstNameSignal.set("John");
    lastNameSignal.set("Doe");
    ageSignal.set(30);
});
```

Verification methods for conditional updates:

```java
Signal.runInTransaction(() -> {
    statusSignal.verifyValue("pending");
    statusSignal.set("processing");
});
```

Transactions can return values:

```java
TransactionOperation<String> txOp = Signal.runInTransaction(() -> {
    statusSignal.verifyValue("pending");
    statusSignal.set("confirmed");
    return "Order confirmed";
});
String result = txOp.returnValue();
```

**Local signals (`ValueSignal`, `ListSignal`) cannot participate in transactions.** Using local signals inside `runInTransaction()` throws an exception. Use shared signals if you need transactional guarantees.

## Complete Example: Shared Counter

```java
@Push
public class SharedCounter extends VerticalLayout {
    private final SharedNumberSignal counter = new SharedNumberSignal();

    public SharedCounter() {
        Button button = new Button();
        button.addClickListener(click -> counter.incrementBy(1));
        button.bindText(counter.map(c -> String.format("Clicked %.0f times", c)));
        add(button);
    }
}
```

All users see the same counter value. Clicking in any browser updates all connected UIs. Requires `@Push` on `AppShellConfigurator`.

## Complete Example: Local Todo List

```java
public class TodoList extends VerticalLayout {
    record Todo(String text, boolean done) {
        Todo withDone(boolean done) { return new Todo(this.text, done); }
    }

    private final ListSignal<Todo> todos = new ListSignal<>();
    private final ValueSignal<String> newTaskText = new ValueSignal<>("");

    public TodoList() {
        TextField input = new TextField("New todo");
        input.bindValue(newTaskText, newTaskText::set);

        Button addBtn = new Button("Add");
        addBtn.bindEnabled(newTaskText.map(t -> !t.isBlank()));
        addBtn.addClickListener(e -> {
            todos.insertLast(new Todo(newTaskText.peek(), false));
            newTaskText.set("");
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

            Button deleteBtn = new Button("Delete",
                e -> todos.remove(todoSignal));

            row.add(checkbox, text, deleteBtn);
            return row;
        });

        add(new HorizontalLayout(input, addBtn), list);
    }
}
```

## Best Practices

1. **Use immutable values** -- Strings, primitives, Java records. Mutating an object directly won't trigger reactivity. Always create a new value with `update()`, or use `modify()` for mutable beans:

   ```java
   // GOOD: new immutable record
   user.update(u -> new User(u.name(), u.age() + 1));

   // GOOD: modify() for mutable beans
   userSignal.modify(u -> u.setAge(26));

   // BAD: mutating in place without modify()
   User u = user.peek();
   u.setAge(u.getAge() + 1); // change not detected
   ```

2. **Prefer direct bindings over effects** -- `bindText()`, `bindVisible()`, `bindEnabled()`, `bindValue()` are more concise and efficient than writing a full `Signal.effect()` for simple property bindings.

3. **Use `peek()` outside reactive contexts** -- In click listeners, initialization code, or anywhere outside an effect/computed, use `peek()`. Using `get()` outside a reactive context throws `IllegalStateException`.

4. **Use transactions for multi-signal atomic updates** (shared signals only) -- prevents observers from seeing partial state.

5. **Use `update()` for atomic read-modify-write** -- `counter.update(c -> c + 1)` is atomic; reading and then setting is not.

6. **Don't modify signals inside effects or computed callbacks** -- they run in read-only transactions. If you must, use `Signal.runWithoutTransaction()`, but beware of infinite loops.

7. **Use `peek()` inside effects to read without tracking** -- `signal.peek()` reads the value without creating a dependency.

8. **Enable `@Push` for shared signals** -- when multiple users share signals, enable server push so changes propagate immediately.

9. **Store signals as class fields** -- keep all reactive state together at the top of the class for clarity. Computed signals can be declared alongside their sources.

10. **Local signals cannot participate in transactions** -- use shared signals if you need `runInTransaction()`.
