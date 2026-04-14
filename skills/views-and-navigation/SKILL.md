---
name: views-and-navigation
description: >
  Guide Claude on creating Vaadin 25 views with @Route, setting up router layouts
  (AppLayout, @Layout), navigation between views, and passing data via URL parameters.
  This skill should be used when the user asks to "create a view", "add a route",
  "use @Route", "set up navigation", "use AppLayout", "build a navigation menu",
  "use SideNav", "pass data between views", "use route parameters", "use query parameters",
  "create a master-detail view", "use @Layout", "use RouterLayout", "use @ParentLayout",
  "use @RoutePrefix", "use @Menu", "use RouterLink", or "navigate programmatically".
  Also trigger when the user needs help choosing between route parameters and query
  parameters, building a MainLayout with a drawer and navigation menu, or structuring
  nested router layouts.
version: 0.1.0
---

# Views & Navigation

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`, `get_component_styling`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## When to Use This Skill vs. Others

**This skill covers:** `@Route`, router layouts (`@Layout`, `RouterLayout`, `AppLayout`), navigation (`RouterLink`, `UI.navigate()`), URL parameters (route params, route templates, query params), `@Menu`, `SideNav`, `DrawerToggle`, nested layouts (`@ParentLayout`, `@RoutePrefix`), and master-detail patterns.

**Use `vaadin-layouts` instead** when the question is about HorizontalLayout, VerticalLayout, FlexLayout, alignment, spacing, padding, or flex-grow — i.e., arranging components *within* a view.

**Use `client-side-views` instead** when building React/Hilla views with TypeScript. This skill covers Java/Flow views only, though it explains how to navigate *to* React views from Java.

**Use `reusable-components` instead** when building custom components with Composite or HasValue. This skill covers *views* (route targets), not reusable components.

## Creating Views with @Route

A Vaadin view is a Java class annotated with `@Route` that extends `Component` or any subclass. The annotation's value is the URL path:

```java
@Route("customers")
@PageTitle("Customers")
@Menu(title = "Customers", order = 2, icon = "vaadin:users")
public class CustomersView extends VerticalLayout {

    public CustomersView() {
        // View content
    }
}
```

### Path Derivation

If you omit the `@Route` value, Vaadin derives the path from the class name: the name is converted to lower case and a trailing `View` suffix is removed. Special case: `MainView` and `Main` are mapped to root (`""`).

Examples:
- `MyEditor` -> `"myeditor"`
- `PersonView` -> `"person"`
- `CustomerListView` -> `"customerlist"`
- `MainView` -> `""`

Use an explicit path to avoid surprises.

An empty `@Route("")` maps to the application root.

### Route Aliases

Create multiple routes to the same view with `@RouteAlias`. A primary `@Route` is always required:

```java
@Route("")
@RouteAlias("home")
@RouteAlias("main")
public class HomeView extends Main {
    // ...
}
```

### Page Title

Set the browser tab title with `@PageTitle`:

```java
@Route("dashboard")
@PageTitle("Dashboard")
public class DashboardView extends Main { }
```

For dynamic titles (e.g., including a customer name), implement `HasDynamicTitle`:

```java
@Route("customer/:customerId")
public class CustomerDetailView extends Main
        implements HasDynamicTitle, BeforeEnterObserver {

    private String customerName;

    @Override
    public void beforeEnter(BeforeEnterEvent event) {
        var customerId = event.getRouteParameters().get("customerId").orElse("");
        customerName = lookupCustomerName(customerId);
    }

    @Override
    public String getPageTitle() {
        return customerName + " — Customer Details";
    }
}
```

### @Menu Annotation

The `@Menu` annotation registers a view in the application's navigation menu. It has three attributes:

- **`title`** — menu label (defaults to `@PageTitle` if unset)
- **`order`** — position in the menu (lower numbers appear first; unordered items appear after ordered ones)
- **`icon`** — icon string, interpreted by your menu-building code (typically a Vaadin icon name like `"vaadin:dashboard"`)

```java
@Route("settings")
@PageTitle("Settings")
@Menu(title = "Settings", order = 10, icon = "vaadin:cog")
public class SettingsView extends Main { }
```

## Router Layouts

Most applications have shared UI elements — a navigation menu, header, footer — that persist across views. A **router layout** wraps views so you don't duplicate these elements.

### The RouterLayout Interface

Router layouts implement `RouterLayout`, which provides:
- `showRouterLayoutContent(HasElement)` — shows the given view
- `removeRouterLayoutContent(HasElement)` — removes the given view

When navigating between views inside the same layout, the **existing layout instance is reused**.

### AppLayout as Your Standard Root

`AppLayout` is the built-in router layout for application shells. It provides three content areas: **navbar** (top bar), **drawer** (side panel), and **content** (main area, managed by the router). Use `DrawerToggle` for a hamburger menu button.

### Automatic Layouts with @Layout

The `@Layout` annotation creates an automatic layout applied to all views:

```java
@Layout
public class MainLayout extends AppLayout {
    // Applied to every view automatically
}
```

**Scope to a path** by passing a path parameter. The path requires a leading slash:

```java
@Layout("/admin")
public class AdminLayout extends AppLayout {
    // Only applies to views with routes starting with /admin
}
```

If multiple layouts match a route, the one with the longest matching path wins.

**Opting out:** A view can disable the automatic layout by setting `autoLayout = false`:

```java
@Route(value = "login", autoLayout = false)
public class LoginView extends Main { }
```

### Explicit Layouts with @Route(layout = ...)

Assign a specific layout to a view using the `layout` attribute. This also disables any automatic layout for that view:

```java
@Route(value = "hello", layout = MainLayout.class)
public class HelloView extends Main { }
```

### Full MainLayout Example

This is a complete, production-ready `MainLayout` using `@Layout`, `AppLayout`, `DrawerToggle`, `Scroller`, and a dynamic `SideNav` built from `@Menu` annotations:

```java
import com.vaadin.flow.component.Component;
import com.vaadin.flow.component.applayout.AppLayout;
import com.vaadin.flow.component.applayout.DrawerToggle;
import com.vaadin.flow.component.html.Span;
import com.vaadin.flow.component.icon.Icon;
import com.vaadin.flow.component.icon.VaadinIcon;
import com.vaadin.flow.component.orderedlayout.FlexComponent;
import com.vaadin.flow.component.orderedlayout.Scroller;
import com.vaadin.flow.component.orderedlayout.VerticalLayout;
import com.vaadin.flow.component.sidenav.SideNav;
import com.vaadin.flow.component.sidenav.SideNavItem;
import com.vaadin.flow.router.Layout;
import com.vaadin.flow.router.MenuConfiguration;
import com.vaadin.flow.router.MenuEntry;

@Layout
public class MainLayout extends AppLayout {

    public MainLayout() {
        addToNavbar(new DrawerToggle(), createTitle());
        setPrimarySection(Section.DRAWER);
        addToDrawer(createDrawerHeader(), new Scroller(createSideNav()));
    }

    private Span createTitle() {
        var title = new Span("My App");
        title.getStyle().setFontWeight("bold");
        return title;
    }

    private Component createDrawerHeader() {
        var logo = VaadinIcon.COGS.create();
        var name = new Span("My App");
        name.getStyle().setFontWeight("bold");

        var header = new VerticalLayout(logo, name);
        header.setAlignItems(FlexComponent.Alignment.CENTER);
        return header;
    }

    private SideNav createSideNav() {
        var nav = new SideNav();
        MenuConfiguration.getMenuEntries()
                .forEach(entry -> nav.addItem(createSideNavItem(entry)));
        return nav;
    }

    private SideNavItem createSideNavItem(MenuEntry entry) {
        var item = new SideNavItem(entry.title(), entry.path());
        item.setMatchNested(true);
        if (entry.icon() != null) {
            item.setPrefixComponent(new Icon(entry.icon()));
        }
        return item;
    }
}
```

Key points:
- **`setPrimarySection(Section.DRAWER)`** — drawer stays visible on wide screens
- **`Scroller`** — wraps `SideNav` so long menus scroll instead of overflowing
- **`setMatchNested(true)`** — highlights the nav item for nested paths (e.g., `/customers/123` highlights "Customers")
- **`MenuConfiguration.getMenuEntries()`** — dynamically builds the menu from `@Menu` annotations on views

## Navigation Between Views

### RouterLink (Preferred)

`RouterLink` creates an HTML `<a>` element. Prefer it over programmatic navigation because it **improves accessibility** and allows users to open links in new tabs:

```java
// Simple link
var link = new RouterLink("Home", HomeView.class);

// With a single route parameter
var link = new RouterLink("Customer Details",
        CustomerDetailView.class, "cu1234");

// With multiple route parameters
var link = new RouterLink("Edit Customer",
        CustomerDetailView.class,
        new RouteParameters(Map.of(
                "customerId", "cu1234",
                "mode", "edit"
        )));
```

### Programmatic Navigation with UI.navigate()

Use `UI.navigate()` in event handlers when a link isn't appropriate:

```java
// Navigate by class reference
UI.getCurrent().navigate(HomeView.class);

// With a single route parameter
UI.getCurrent().navigate(CustomerDetailView.class, "cu1234");

// With multiple route parameters
UI.getCurrent().navigate(CustomerDetailView.class,
        new RouteParameters(Map.of("customerId", "cu1234", "mode", "edit")));
```

### The "Your Own API" Pattern

Instead of scattering `UI.navigate()` calls throughout the codebase, encapsulate navigation logic in static methods on the target view. This improves readability and makes refactoring easier:

```java
@Route("customer/:customerId")
public class CustomerDetailView extends Main
        implements BeforeEnterObserver {

    public static void showCustomer(String customerId) {
        UI.getCurrent().navigate(CustomerDetailView.class, customerId);
    }

    public static RouterLink createLink(String text, String customerId) {
        return new RouterLink(text, CustomerDetailView.class, customerId);
    }

    @Override
    public void beforeEnter(BeforeEnterEvent event) {
        var customerId = event.getRouteParameters().get("customerId").orElse("");
        // Load customer data
    }
}
```

Callers use the clean API:

```java
// Programmatic
CustomerDetailView.showCustomer("cu1234");

// Link
layout.add(CustomerDetailView.createLink("View Customer", "cu1234"));
```

### Navigating to React Views

React views don't have a Java class, so use string-based navigation:

```java
// Link
var link = new Anchor("path/to/react/view", "React View");

// Programmatic
UI.getCurrent().navigate("path/to/react/view");
```

## Passing Data Between Views

### Route Parameters (HasUrlParameter)

For views that accept a **single** URL parameter, implement `HasUrlParameter<T>`:

```java
@Route("customer")
public class CustomerDetailView extends Main
        implements HasUrlParameter<String> {

    @Override
    public void setParameter(BeforeEvent event, String customerId) {
        // URL: /customer/cu1234
        // customerId = "cu1234"
    }
}
```

The parameter is **required** by default. Make it optional with `@OptionalParameter`:

```java
@Override
public void setParameter(BeforeEvent event,
        @OptionalParameter String customerId) {
    if (customerId == null) {
        // Show list or default view
    } else {
        // Show specific customer
    }
}
```

Use `@WildcardParameter` to capture the entire remaining path:

```java
@Override
public void setParameter(BeforeEvent event,
        @WildcardParameter String path) {
    // URL: /docs/a/b/c → path = "a/b/c"
}
```

### Route Templates

For **multiple** named parameters or regex constraints, use route templates in the `@Route` value:

```java
@Route("customer/:customerId/:mode?(edit|view)")
public class CustomerDetailView extends Main
        implements BeforeEnterObserver {

    private static final String PARAM_CUSTOMER_ID = "customerId";
    private static final String PARAM_MODE = "mode";

    @Override
    public void beforeEnter(BeforeEnterEvent event) {
        var params = event.getRouteParameters();
        var customerId = params.get(PARAM_CUSTOMER_ID).orElse("");
        var mode = params.get(PARAM_MODE).orElse("view");
        // ...
    }
}
```

Route template syntax:
- **`:name`** — required parameter, matches one segment
- **`:name?`** — optional parameter
- **`:name?(regex)`** — optional with regex constraint (e.g., `:mode?(edit|view)`)
- **`:name(regex)`** — required with regex constraint (e.g., `:id(\\d+)`)
- **`:name*`** — wildcard, captures remaining path (must be last)

### Query Parameters

Use query parameters for **optional filters, sorting, or pagination** that don't define the resource:

```java
@Route("customers")
public class CustomersView extends Main
        implements BeforeEnterObserver {

    @Override
    public void beforeEnter(BeforeEnterEvent event) {
        var queryParams = event.getLocation().getQueryParameters();
        var sort = queryParams.getSingleParameter("sort").orElse("name");
        var page = queryParams.getSingleParameter("page")
                .map(Integer::parseInt).orElse(0);
        // Apply sort and page
    }
}
```

**Updating query parameters** dynamically without a full navigation:

```java
private void updateFilters(String sort, int page) {
    var params = QueryParameters.merging()
            .add("sort", sort)
            .add("page", String.valueOf(page))
            .build();
    UI.getCurrent().navigate(getClass(), params);
}
```

### When to Use Which Approach

| Approach | Use when... | Example |
|----------|-------------|---------|
| `HasUrlParameter<T>` | Single parameter identifies the resource | `/customer/cu1234` |
| Route template | Multiple named params or regex constraints needed | `/customer/:id/:mode?(edit\|view)` |
| Query parameters | Optional filters, sorting, pagination | `/customers?sort=name&page=2` |

## Nested Layouts

Layouts can be nested for hierarchical UI structures. Use `@ParentLayout` to declare the parent and `@RoutePrefix` to add a path prefix:

```java
@Layout
public class MainLayout extends AppLayout {
    public MainLayout() {
        addToNavbar(new DrawerToggle(), new Span("My App"));
        addToDrawer(new Scroller(createSideNav()));
    }
    // ... SideNav creation
}

@ParentLayout(MainLayout.class)
@RoutePrefix("admin")
public class AdminLayout extends VerticalLayout implements RouterLayout {

    public AdminLayout() {
        add(new HorizontalLayout(
                new RouterLink("Users", AdminUsersView.class),
                new RouterLink("Groups", AdminGroupsView.class)
        ));
    }
}

@Route(value = "users", layout = AdminLayout.class)
@PageTitle("Admin Users")
@Menu(title = "Users", order = 1, icon = "vaadin:users")
public class AdminUsersView extends Main {
    // Route resolves to /admin/users
    // Rendered inside AdminLayout, which is inside MainLayout
}

@Route(value = "groups", layout = AdminLayout.class)
@PageTitle("Admin Groups")
@Menu(title = "Groups", order = 2, icon = "vaadin:group")
public class AdminGroupsView extends Main {
    // Route resolves to /admin/groups
}
```

### Opting Out of a Route Prefix

Set `absolute = true` on `@Route` or `@RoutePrefix` to ignore the prefix from the parent:

```java
@Route(value = "path", layout = AdminLayout.class, absolute = true)
public class MyView extends Main {
    // Route is /path, NOT /admin/path
}
```

## Master-Detail Pattern

A common pattern: a list (Grid) on the left, details on the right, with the selected item's ID in the URL. Use `SplitLayout` with `HasUrlParameter<Long>` and `@OptionalParameter`:

```java
@Route("customers")
@PageTitle("Customers")
@Menu(title = "Customers", order = 2, icon = "vaadin:users")
public class CustomersView extends Main
        implements HasUrlParameter<Long> {

    private final Grid<Customer> grid = new Grid<>(Customer.class);
    private final VerticalLayout detailPane = new VerticalLayout();

    public static void showCustomer(Long customerId) {
        UI.getCurrent().navigate(CustomersView.class, customerId);
    }

    public static void showList() {
        UI.getCurrent().navigate(CustomersView.class);
    }

    public CustomersView(CustomerService service) {
        var splitLayout = new SplitLayout(grid, detailPane);
        splitLayout.setSizeFull();
        splitLayout.setSplitterPosition(60);
        add(splitLayout);
        setSizeFull();

        grid.setItems(service.findAll());
        grid.addSelectionListener(e ->
                e.getFirstSelectedItem().ifPresent(
                        customer -> showCustomer(customer.getId())));
        detailPane.setVisible(false);
    }

    @Override
    public void setParameter(BeforeEvent event,
            @OptionalParameter Long customerId) {
        if (customerId != null) {
            showDetail(customerId);
        } else {
            detailPane.setVisible(false);
            grid.deselectAll();
        }
    }

    private void showDetail(Long customerId) {
        // Load and display customer details
        detailPane.setVisible(true);
        detailPane.removeAll();
        detailPane.add(new H3("Customer #" + customerId));
        // ... add detail fields
    }
}
```

## Best Practices

1. **Prefer `RouterLink` over `UI.navigate()`** — links are more accessible, support right-click "open in new tab", and work after session expiry.

2. **Use the "Your Own API" pattern** — add static `showXxx()` and `createLinkTo()` methods on target views. This centralizes route knowledge and makes refactoring safe.

3. **Prefer `@Layout` over explicit `layout =`** — automatic layouts reduce boilerplate. Use explicit assignment only when a view needs a non-default layout.

4. **Always set `setMatchNested(true)` on SideNavItems** — otherwise the nav item won't highlight when viewing a nested route like `/customers/123`.

5. **Use route parameters for resource identity, query parameters for filtering** — `/customer/123` identifies a resource; `?sort=name&page=2` filters a list. Don't put filter state in route parameters.

6. **Define parameter name constants** — avoid magic strings by declaring `private static final String PARAM_ID = "customerId"` and referencing it in both the route template and `getRouteParameters()`.

7. **Wrap drawer content in a `Scroller`** — `addToDrawer(new Scroller(sideNav))` ensures long navigation menus scroll rather than overflow.

8. **Use `@Menu` + `MenuConfiguration`** — define menu metadata on each view and build the nav dynamically. This avoids maintaining a separate menu configuration that can get out of sync.

## Anti-Patterns

1. **Building app shells with nested HorizontalLayout/VerticalLayout instead of AppLayout** — AppLayout handles responsive drawer collapsing, hamburger menus, and navbar placement automatically. Don't reinvent it.

2. **Hardcoding route strings** — `UI.getCurrent().navigate("customer/" + id)` is fragile. Use class references: `UI.getCurrent().navigate(CustomerDetailView.class, id)`. Or better, use the "Your Own API" pattern.

3. **Using `HasUrlParameter` for multiple parameters** — it only supports a single parameter. For multiple parameters, use route templates with `BeforeEnterObserver`.

4. **Storing navigation state in static fields** — static fields are shared across all users. Use URL parameters, session attributes, or Spring-scoped beans instead.

5. **Forgetting `autoLayout = false` on login/error views** — login and error pages should not render inside the main layout. Always set `@Route(value = "login", autoLayout = false)`.

6. **Using unconstrained route parameters for IDs** — `:id` matches any string. Use `:id(\\d+)` to constrain to numbers so invalid URLs get a 404 instead of a confusing error.

## Detailed Reference

For a quick-reference cheatsheet of routing annotations, navigation methods, parameter syntax, and copy-paste templates, see `references/navigation-patterns.md`.
