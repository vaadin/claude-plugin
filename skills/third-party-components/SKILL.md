---
name: third-party-components
description: >
  Guide Claude on integrating third-party Web Components and React components
  from npm into Vaadin 25 Flow applications. This skill should be used when
  the user asks to "integrate a web component", "wrap a web component",
  "third-party component", "@Tag", "@NpmPackage", "@JsModule", "@DomEvent",
  "@EventData", "@Synchronize", "integrate a React component",
  "wrap a React component", "ReactAdapterComponent", "ReactAdapterElement",
  "npm component", "PropertyDescriptor", "callJsFunction",
  "synchronize properties", or "listen to DOM events".
version: 0.1.0
---

# Integrating Third-Party Components in Vaadin 25

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## Choosing Your Integration Path

There are two paths for integrating an npm component into Vaadin Flow. The right choice depends on what the npm package exports.

| Criterion | Web Component Path | React Path |
|-----------|-------------------|------------|
| **npm package exports** | A custom element (`customElements.define()`) | React components (`export function/class`) |
| **Java base class** | `Component` (or `AbstractSinglePropertyField`) | `ReactAdapterComponent` |
| **Client-side code needed?** | No (unless writing a custom element yourself) | Yes — a thin `.tsx` adapter file |
| **State sync mechanism** | Element properties + `@Synchronize` / `@DomEvent` | `setState()` / `getState()` / `hooks.useState()` |
| **When to use** | The package already ships a web component | The package only ships React components |

If the npm package exports **both** a web component and a React wrapper, prefer the Web Component path — it avoids the extra `.tsx` adapter layer.

## Path 1: Web Component Integration

This path is for npm packages that export a custom HTML element (e.g., `<fancy-slider>`, `<leaflet-map>`). You write a Java class that maps to the element's tag, properties, and events.

### Core Annotations

Every web component wrapper needs three annotations:

```java
@Tag("fancy-slider")                                         // 1
@NpmPackage(value = "fancy-slider", version = "2.1.0")       // 2
@JsModule("fancy-slider/fancy-slider.js")                    // 3
public class FancySlider extends Component {
}
```

1. `@Tag` — must match the tag name registered by `customElements.define()` in the npm package.
2. `@NpmPackage` — triggers `npm install` of the package. Pin the version.
3. `@JsModule` — imports the JavaScript module that registers the custom element. For packages that use a different entry point, check the package's docs.

The `@JsModule` path is relative to `node_modules/` for npm packages, or relative to `src/main/frontend/` when prefixed with `./`.

### Properties

Read and write element properties to configure the web component:

```java
// Direct element API
public void setMin(int min) {
    getElement().setProperty("min", min);
}

public int getMin() {
    return getElement().getProperty("min", 0);
}
```

For cleaner code with multiple properties, use `PropertyDescriptor`:

```java
private static final PropertyDescriptor<Integer, Integer> minProperty =
        PropertyDescriptors.propertyWithDefault("min", 0);

private static final PropertyDescriptor<Integer, Integer> maxProperty =
        PropertyDescriptors.propertyWithDefault("max", 100);

public void setMin(int min) {
    minProperty.set(this, min);
}

public int getMin() {
    return minProperty.get(this);
}

public void setMax(int max) {
    maxProperty.set(this, max);
}

public int getMax() {
    return maxProperty.get(this);
}
```

`PropertyDescriptor` ensures the property name is defined once and reused in both getter and setter.

### Property Synchronization

By default, property changes in the browser are **not** sent to the server. Two approaches to sync them:

**Approach 1: `@Synchronize` annotation** — use on getter methods for properties on the **root element**:

```java
@Synchronize("value-changed")  // DOM event that triggers sync
public int getValue() {
    return getElement().getProperty("value", 0);
}
```

`@Synchronize` only works for events from the root element (or events that bubble up to it). It does **not** work for events on child elements.

**Approach 2: `Element.addPropertyChangeListener()`** — more flexible, works programmatically:

```java
public FancySlider() {
    getElement().addPropertyChangeListener("value", "value-changed", event -> {
        // React to value change, or use a no-op listener just to enable sync
    });
}
```

Use this approach when you need to react to changes directly, or when `@Synchronize` doesn't fit.

### DOM Events

Map client-side DOM events to typed Java events using `@DomEvent` and `@EventData`:

```java
@DomEvent("slide-end")
public static class SlideEndEvent extends ComponentEvent<FancySlider> {

    private final int value;

    public SlideEndEvent(FancySlider source, boolean fromClient,
            @EventData("event.detail.value") int value) {
        super(source, fromClient);
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

The first two constructor parameters (`source` and `fromClient`) are always required and filled automatically. All subsequent parameters must be annotated with `@EventData`.

`@EventData` expressions are evaluated as JavaScript in the browser:
- `event.XXX` — accesses properties on the DOM event
- `event.detail.XXX` — accesses Custom Event detail
- `element.XXX` — accesses properties on the element itself

Expose the event with an `addXxxListener()` method:

```java
public Registration addSlideEndListener(
        ComponentEventListener<SlideEndEvent> listener) {
    return addListener(SlideEndEvent.class, listener);
}
```

### Calling JavaScript Functions

Some web components expose methods on the element (e.g., `open()`, `refresh()`). Call them from Java:

```java
public void open() {
    getElement().callJsFunction("open");
}

public void scrollToIndex(int index) {
    getElement().callJsFunction("scrollToIndex", index);
}
```

Supported parameter types: `String`, `Boolean`, `Integer`, `Double`, primitives, `JsonValue`, `Element`, and `Component`. The method returns a `PendingJavaScriptResult` for async return values.

### Child Elements

For layout-type web components that accept children:

**Option 1: Implement `HasComponents`** — provides public `add()`, `remove()`, `removeAll()`:

```java
@Tag("fancy-panel")
@NpmPackage(value = "fancy-panel", version = "1.0.0")
@JsModule("fancy-panel/fancy-panel.js")
public class FancyPanel extends Component implements HasComponents {
    // add() and remove() provided by the interface
}
```

**Option 2: Use the Element API** — for controlled internal children:

```java
public void setIcon(Component icon) {
    getElement().removeAllChildren();
    getElement().appendChild(icon.getElement());
}
```

Only implement `HasComponents` when arbitrary children make sense. Use explicit methods for structured content.

### Making It a Binder Field: `AbstractSinglePropertyField`

When the web component represents an input value, extend `AbstractSinglePropertyField` to integrate with Vaadin's `Binder`:

```java
@Tag("fancy-slider")
@NpmPackage(value = "fancy-slider", version = "2.1.0")
@JsModule("fancy-slider/fancy-slider.js")
public class FancySlider extends AbstractSinglePropertyField<FancySlider, Integer> {

    public FancySlider() {
        super("value", 0, false);  // property name, default value, nullable
    }

    @Synchronize("change")
    @Override
    public Integer getValue() {
        return super.getValue();
    }
}
```

Constructor parameters for `AbstractSinglePropertyField`:
1. Property name on the element (e.g., `"value"`)
2. Default value (also used by `clear()` and `isEmpty()`)
3. Whether `setValue(null)` is allowed

The `@Synchronize` annotation on `getValue()` tells Flow which DOM event triggers server-side value updates.

### Complete Web Component Example

A full integration of a hypothetical `<star-rating>` web component:

```java
@Tag("star-rating")
@NpmPackage(value = "@example/star-rating", version = "3.0.0")
@JsModule("@example/star-rating/star-rating.js")
public class StarRating extends AbstractSinglePropertyField<StarRating, Integer> {

    private static final PropertyDescriptor<Integer, Integer> maxStarsProperty =
            PropertyDescriptors.propertyWithDefault("max", 5);

    public StarRating() {
        super("value", 0, false);
    }

    @Synchronize("rating-changed")
    @Override
    public Integer getValue() {
        return super.getValue();
    }

    public void setMaxStars(int max) {
        maxStarsProperty.set(this, max);
    }

    public int getMaxStars() {
        return maxStarsProperty.get(this);
    }

    public void setReadOnly(boolean readOnly) {
        getElement().setProperty("readonly", readOnly);
    }

    @DomEvent("rating-changed")
    public static class RatingChangedEvent extends ComponentEvent<StarRating> {

        private final int rating;

        public RatingChangedEvent(StarRating source, boolean fromClient,
                @EventData("event.detail.value") int rating) {
            super(source, fromClient);
            this.rating = rating;
        }

        public int getRating() {
            return rating;
        }
    }

    public Registration addRatingChangedListener(
            ComponentEventListener<RatingChangedEvent> listener) {
        return addListener(RatingChangedEvent.class, listener);
    }
}
```

Usage:

```java
StarRating rating = new StarRating();
rating.setMaxStars(10);
rating.addValueChangeListener(e ->
    Notification.show("Rating: " + e.getValue()));

// Works with Binder
binder.forField(rating)
    .asRequired("Please rate")
    .bind(Review::getRating, Review::setRating);
```

### Writing Your Own Web Component

When no npm package provides what you need, write the client-side code yourself using Lit. Create a JavaScript file in `frontend/` (e.g., `frontend/my-rating.js`), use `@JsModule("./my-rating.js")` with a `./` prefix, and omit `@NpmPackage`. Properties, events, and `callJsFunction()` work the same way.

## Path 2: React Component Integration

This path is for npm packages that export React components (not custom elements). You create a thin adapter layer: a Java class extending `ReactAdapterComponent` and a `.tsx` file extending `ReactAdapterElement`.

The integration uses an intermediate web component as a bridge: Java (server) <-> Web Component adapter <-> React (client).

### Server-Side: `ReactAdapterComponent`

```java
@NpmPackage(value = "react-colorful", version = "5.6.1")     // 1
@JsModule("./components/color-picker.tsx")                     // 2
@Tag("color-picker")                                           // 3
public class ColorPicker extends ReactAdapterComponent {

    public ColorPicker() {
        setColor(new RgbaColor(255, 0, 0, 1.0));  // 4 — always initialize state
    }

    public RgbaColor getColor() {
        return getState("color", RgbaColor.class);
    }

    public void setColor(RgbaColor color) {
        setState("color", color);
    }

    public void addColorChangeListener(SerializableConsumer<RgbaColor> listener) {
        addStateChangeListener("color", RgbaColor.class, listener);
    }
}
```

1. `@NpmPackage` — installs the React library from npm.
2. `@JsModule` — path to **your** `.tsx` adapter file (prefixed with `./` since it lives in `src/main/frontend/`).
3. `@Tag` — must match the tag name in `customElements.define()` in the `.tsx` file.
4. Always call `setState` in the constructor to initialize. This ensures `@PreserveOnRefresh` works correctly.

State sync API on `ReactAdapterComponent`:
- `setState(String name, T value)` — sends state from server to client
- `getState(String name, Class<T> type)` — reads current client value
- `addStateChangeListener(String name, Class<T> type, Consumer<T> listener)` — reacts to client changes

### Non-Primitive Types

For complex state objects, use Java records (or beans). The state is serialized as JSON:

```java
public record RgbaColor(int r, int g, int b, double a) {}
```

On the TypeScript side, define a matching interface:

```typescript
interface RgbaColor {
    r: number;
    g: number;
    b: number;
    a: number;
}
```

Records, beans, lists, and maps are all supported — anything representable as JSON.

### Client-Side: `ReactAdapterElement`

Create a `.tsx` file in `src/main/frontend/` (e.g., `src/main/frontend/components/color-picker.tsx`):

```tsx
import { ReactAdapterElement, type RenderHooks } from 'Frontend/generated/flow/ReactAdapter';
import { RgbaColorPicker, type RgbaColor } from 'react-colorful';
import type { ReactElement } from 'react';

class ColorPickerElement extends ReactAdapterElement {
    protected override render(hooks: RenderHooks): ReactElement | null {
        const [color, setColor] = hooks.useState<RgbaColor>('color');  // 1

        return <RgbaColorPicker color={color} onChange={setColor} />;  // 2
    }
}

customElements.define('color-picker', ColorPickerElement);             // 3
```

1. `hooks.useState<T>(name)` — binds to the named state property, matching `setState`/`getState` on the Java side.
2. Wire the React component's props and callbacks to the adapter state.
3. `customElements.define()` — the tag name must match `@Tag` on the Java class.

**`RenderHooks` API:**
- `hooks.useState<T>(name)` — returns `[value, setter]` like React's `useState`, but synchronized with the server.
- `hooks.useCustomEvent<T>(name, init?)` — returns a callback that dispatches a `CustomEvent` on the element.
- `hooks.useContent(name)` — returns a placeholder for embedding Flow components (see below).

### Firing Custom Events

When a user action isn't a simple state change, use `hooks.useCustomEvent`:

```tsx
protected override render(hooks: RenderHooks): ReactElement | null {
    const fireSubmit = hooks.useCustomEvent<{ name: string }>('submit');

    return <button onClick={() => fireSubmit({ name: 'John' })}>Submit</button>;
}
```

Listen for it in Java:

```java
getElement().addEventListener("submit", event -> {
    JsonObject detail = event.getEventData();
    // handle submit
}).addEventData("event.detail");
```

### Making It a Binder Field

To use a React component as a form field with `Binder`, extend `AbstractSinglePropertyField` and dispatch a `value-changed` CustomEvent from the `.tsx` adapter:

**Client-side (`.tsx`):**

```tsx
import { ReactAdapterElement, type RenderHooks } from 'Frontend/generated/flow/ReactAdapter';
import type { ReactElement } from 'react';

class ReactSliderElement extends ReactAdapterElement {
    protected override render(hooks: RenderHooks): ReactElement | null {
        const [value, setValue] = hooks.useState<number>('value');

        const handleChange = (newValue: number) => {
            setValue(newValue);
            this.dispatchEvent(new CustomEvent('value-changed', {
                detail: { value: newValue }
            }));
        };

        return <input type="range" value={value ?? 0}
                   onChange={e => handleChange(Number(e.target.value))} />;
    }
}

customElements.define('react-slider', ReactSliderElement);
```

**Server-side (Java):**

```java
@Tag("react-slider")
@JsModule("./components/react-slider.tsx")
public class ReactSlider extends AbstractSinglePropertyField<ReactSlider, Integer> {

    public ReactSlider() {
        super("value", 0, false);
    }
}
```

The `value-changed` CustomEvent is the convention `AbstractSinglePropertyField` listens for by default. No `@Synchronize` is needed — the base class handles it.

### Embedding Flow Components in React

You can place Vaadin Flow components inside a React adapter using `getContentElement()` on the server and `hooks.useContent()` on the client:

**Server-side:**

```java
@JsModule("./components/react-panel.tsx")
@Tag("react-panel")
public class ReactPanel extends ReactAdapterComponent {

    public ReactPanel() {
        Div toolbar = new Div(new Button("Save"), new Button("Cancel"));
        getContentElement("toolbar").appendChild(toolbar.getElement());
    }
}
```

**Client-side:**

```tsx
class ReactPanelElement extends ReactAdapterElement {
    protected override render(hooks: RenderHooks): ReactElement | null {
        const toolbar = hooks.useContent('toolbar');

        return (
            <div>
                <h2>My Panel</h2>
                <div>{toolbar}</div>
            </div>
        );
    }
}

customElements.define('react-panel', ReactPanelElement);
```

### Complete React Integration Example

Integrating a hypothetical `react-star-rating` npm package:

**Java (`StarRating.java`):**

```java
@NpmPackage(value = "react-star-rating", version = "4.0.0")
@JsModule("./components/star-rating.tsx")
@Tag("star-rating-adapter")
public class StarRating extends ReactAdapterComponent {

    public StarRating() {
        setState("rating", 0);
        setState("maxStars", 5);
    }

    public int getRating() {
        return getState("rating", Integer.class);
    }

    public void setRating(int rating) {
        setState("rating", rating);
    }

    public void setMaxStars(int max) {
        setState("maxStars", max);
    }

    public void addRatingChangeListener(SerializableConsumer<Integer> listener) {
        addStateChangeListener("rating", Integer.class, listener);
    }
}
```

**TypeScript (`src/main/frontend/components/star-rating.tsx`):**

```tsx
import { ReactAdapterElement, type RenderHooks } from 'Frontend/generated/flow/ReactAdapter';
import { StarRating } from 'react-star-rating';
import type { ReactElement } from 'react';

class StarRatingElement extends ReactAdapterElement {
    protected override render(hooks: RenderHooks): ReactElement | null {
        const [rating, setRating] = hooks.useState<number>('rating');
        const [maxStars] = hooks.useState<number>('maxStars');

        return (
            <StarRating
                value={rating ?? 0}
                count={maxStars ?? 5}
                onChange={setRating}
            />
        );
    }
}

customElements.define('star-rating-adapter', StarRatingElement);
```

**Usage:**

```java
StarRating rating = new StarRating();
rating.setMaxStars(10);
rating.addRatingChangeListener(value ->
    Notification.show("Rated: " + value));
```

## Best Practices

1. **Match `@Tag` with `customElements.define()`** — the tag name in Java must exactly match the name registered on the client. A mismatch silently breaks the component.

2. **Pin npm versions** — always specify an exact version in `@NpmPackage` (e.g., `"2.1.0"`, not `"^2.1.0"`). The annotation value is written directly to `package.json`.

3. **Initialize state in the constructor** — for `ReactAdapterComponent`, call `setState` for all properties in the Java constructor. This ensures correct behavior with `@PreserveOnRefresh` and avoids `null` in the `.tsx` adapter on first render.

4. **Keep the `.tsx` adapter thin** — the adapter should only bridge between the React component's API and the Web Component state/events. No business logic, no data fetching, no complex state management.

5. **Use `PropertyDescriptor`** — for Web Component wrappers with many properties, `PropertyDescriptor` reduces duplication and ensures property names are consistent between getter and setter.

6. **Prefer `@DomEvent` over raw `addEventListener`** — `@DomEvent` integrates with Vaadin's event system, supports `@EventData`, and automatically handles disabled/hidden state.

7. **Use `AbstractSinglePropertyField` for form fields** — if the component has a single value property, this base class provides `HasValue`, `Binder` integration, `clear()`, `isEmpty()`, and value change events out of the box.

8. **Place `.tsx` adapter files consistently** — put them in `src/main/frontend/components/` or a descriptive subdirectory. Reference with `@JsModule("./components/my-adapter.tsx")`.

## Anti-Patterns

1. **Mismatched tag names** — `@Tag("my-slider")` in Java but `customElements.define('my-fancy-slider', ...)` in JavaScript. The component renders as an empty unknown element with no errors in the Java console.

2. **Missing `@NpmPackage`** — the `@JsModule` import fails at build time because the package isn't installed. Always pair `@JsModule` with `@NpmPackage` for npm dependencies.

3. **`@Synchronize` on child element events** — `@Synchronize` only listens to events on the root element. If the relevant event fires on a child, use `getElement().addPropertyChangeListener()` instead.

4. **No initial `setState` in `ReactAdapterComponent`** — the `.tsx` adapter receives `undefined` for all state on first render. Always initialize every state property in the Java constructor.

5. **Business logic in `.tsx` adapter** — the adapter should be a passthrough. Put validation, formatting, and data logic in Java. The adapter only renders the React component and forwards state/events.

6. **Choosing the wrong path** — using `ReactAdapterComponent` for a package that already exports a web component adds unnecessary complexity. Check what the npm package actually exports before deciding.
