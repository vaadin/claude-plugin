---
name: ui-unit-testing
description: >
  Guide Claude on writing fast, browser-free tests for Vaadin 25 Flow views with
  the Browserless Testing framework (formerly UI Unit Testing).
  This skill should be used when the user asks to "write a UI test", "unit test a view",
  "test without a browser", "use BrowserlessTest", "use UIUnitTest", "test a Vaadin component",
  "browser-free testing", "browserless testing", or needs help with the
  Vaadin browserless testing framework, component testers, navigation in tests,
  or mocking Spring beans in Vaadin view tests.
version: 0.2.0
---

# Browserless Testing in Vaadin 25

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## What Browserless Testing Is

Browserless tests (previously called "UI Unit Tests") run server-side Java code without a browser or servlet container. You interact directly with your server-side view classes and Vaadin components. The `BrowserlessTest` / `SpringBrowserlessTest` base classes set up the Vaadin session, UI, and routing — all in the same JVM as your JUnit tests.

This makes tests fast (milliseconds, not seconds), stable (no browser flakiness), and easy to run in CI.

**Free for all users since Vaadin 25.1.** Earlier versions required a commercial TestBench subscription.

## When to Use Browserless Tests vs. End-to-End Tests

**Browserless tests** — the default choice for most view testing:
- Testing view logic, navigation, form validation, component state
- TDD workflows where you run tests on every save
- Verifying that clicking a button shows the right notification
- Testing data binding and Binder behavior

**End-to-end tests (TestBench)** — for critical paths and client-side behavior:
- Login flows, checkout processes
- Testing JavaScript/client-side functionality
- Visual regression testing
- Cross-browser compatibility

Write many browserless tests and few end-to-end tests.

## Setup

### Maven dependency

The browserless testing framework ships in `browserless-test-junit6`. Vaadin 25 + Spring Boot 4 default to JUnit 6, which is API-compatible with JUnit 5.

```xml
<dependency>
    <groupId>com.vaadin</groupId>
    <artifactId>browserless-test-junit6</artifactId>
    <scope>test</scope>
</dependency>
```

> **Migrating from UI Unit Testing?** Replace the older `vaadin-testbench-unit` (JUnit 4) or `vaadin-testbench-unit-junit5` artifact with `browserless-test-junit6`, and rename the base classes (`UIUnitTest` → `BrowserlessTest`, `SpringUIUnitTest` → `SpringBrowserlessTest`). The testing API is otherwise unchanged.

## Writing Your First Test

Extend `SpringBrowserlessTest` for Spring Boot projects (and add `@SpringBootTest`), or `BrowserlessTest` otherwise:

```java
import com.vaadin.browserless.SpringBrowserlessTest;

@SpringBootTest
class HelloWorldViewTest extends SpringBrowserlessTest {

    @Test
    void clickButton_showsNotification() {
        // Navigate to the view
        HelloWorldView view = navigate(HelloWorldView.class);

        // Interact via component testers
        test(view.nameField).setValue("Marcus");
        test(view.sayHelloButton).click();

        // Assert on the result
        Notification notification = $(Notification.class).first();
        assertEquals("Hello Marcus", test(notification).getText());
    }
}
```

Key points:
- `navigate(ViewClass.class)` navigates to a route and returns the view instance
- `test(component)` returns a type-specific tester with simulated user actions
- `$(ComponentClass.class)` queries for components in the current UI (like jQuery for Vaadin)
- Fields on the view should be package-private (not private) so tests in the same package can access them

## Navigation

```java
// Simple navigation
MyView view = navigate(MyView.class);

// Navigation with URL parameter
DetailView view = navigate(DetailView.class, "123");

// Navigation with route template parameters
TemplateView view = navigate(TemplateView.class, Map.of("id", "456"));

// Navigation by path string (validates expected view type)
MyView view = navigate("my-view", MyView.class);
```

## Component Testers

The `test()` method returns a tester that simulates user interaction. Testers check that the component is visible, enabled, attached, and not behind a modal before allowing interaction.

```java
// TextField
test(textField).setValue("hello");      // simulates typing
String value = test(textField).getValue();

// Button
test(button).click();                   // simulates click

// Checkbox
test(checkbox).setValue(true);

// ComboBox
test(comboBox).selectItem("Option A");

// Grid — use GridTester for row interaction
GridTester<Person> gridTester = test(grid);
gridTester.clickRow(0);                 // click first row

// Notification
Notification n = $(Notification.class).first();
String text = test(n).getText();
```

For commercial components (Chart, etc.), implement `CommercialTesterWrappers` on your test class.

## Component Queries with $()

Find components in the current UI:

```java
// Find first component of type
Button btn = $(Button.class).first();

// Find by ID
TextField field = $(TextField.class).id("email");

// Find all of a type
List<Button> buttons = $(Button.class).all();

// Check existence
boolean hasGrid = $(Grid.class).exists();

// Nested query — find inside a specific component
VerticalLayout layout = $(VerticalLayout.class).id("content");
Button innerBtn = layout.$(Button.class).first();
```

## Restricting Package Scanning

By default, browserless tests scan the entire classpath for routes. For faster startup, restrict to specific packages with `@ViewPackages`. Prefer the `classes()` array — it survives IDE refactors when classes move:

```java
import org.springframework.boot.test.context.SpringBootTest;
import com.vaadin.browserless.ViewPackages;

@SpringBootTest
@ViewPackages(classes = {MyView.class, OtherView.class})
class MyViewTest extends SpringBrowserlessTest {
    // Scans the packages containing MyView and OtherView (and sub-packages)
}

// Or by package name:
@SpringBootTest
@ViewPackages(packages = {"com.example.app.feature1", "com.example.app.feature2"})
class MyViewTest extends SpringBrowserlessTest { }
```

## Non-Spring Projects

If you aren't using Spring, extend `BrowserlessTest` directly and drop `@SpringBootTest`:

```java
class MyViewTest extends BrowserlessTest {

    @Test
    void test() {
        MyView view = navigate(MyView.class);
        // ...
    }
}
```

For Quarkus, see the Vaadin docs for the Quarkus-specific browserless base class and dependency.

## Common Testing Patterns

### Test form validation

```java
@Test
void submitEmptyForm_showsValidationErrors() {
    EditView view = navigate(EditView.class);

    test(view.saveButton).click();

    // Check that required fields show errors
    assertTrue(view.nameField.isInvalid());
}
```

### Test navigation after action

```java
@Test
void saveSuccess_navigatesToList() {
    EditView view = navigate(EditView.class);
    test(view.nameField).setValue("Test");
    test(view.saveButton).click();

    // Verify navigation occurred
    assertTrue(getCurrentView() instanceof ListView);
}
```

### Test dialog content

```java
@Test
void deleteButton_showsConfirmDialog() {
    ListView view = navigate(ListView.class);
    test(view.deleteButton).click();

    ConfirmDialog dialog = $(ConfirmDialog.class).first();
    assertNotNull(dialog);
}
```

## Best Practices

1. **Make view fields package-private** — not private, so tests in the same package can access them directly. This avoids fragile reflection-based lookups.
2. **Use `@ViewPackages` to limit scanning** — speeds up test initialization significantly in large projects.
3. **One assertion focus per test** — test one behavior per method. Name tests descriptively: `action_expectedResult`.
4. **Use `test()` for interaction, not direct method calls** — `test(button).click()` checks visibility and enabled state; `button.click()` bypasses those checks.
5. **Prefer browserless tests over end-to-end** — they're 10-100x faster and more stable. Reserve end-to-end tests for client-side behavior and critical paths.
6. **Test the user's perspective** — verify what the user sees (notification text, navigation result, field errors) rather than internal implementation details.
