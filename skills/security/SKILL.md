---
name: security
description: >
  Guide Claude on securing Vaadin 25 applications with Spring Security.
  This skill should be used when the user asks to "add security", "add login",
  "create a login view", "create a login form", "use Spring Security",
  "secure a view", "add authentication", "add authorization",
  "use @RolesAllowed", "use @PermitAll", "use @AnonymousAllowed", "use @DenyAll",
  "use VaadinSecurityConfigurer", "add OAuth2", "use OAuth2 login",
  "use Google login", "use Keycloak", "use GitHub login",
  "add logout", "add a logout button", "use AuthenticationContext",
  "protect a view", "role-based access", "configure SecurityFilterChain",
  or needs help with view access control, login forms, OAuth2 providers,
  or logout handling in Vaadin Flow.
version: 0.1.0
---

# Security with Spring Security in Vaadin 25

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`, `get_component_styling`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## When to Use This Skill vs. Others

**This skill covers:** Spring Security configuration with `VaadinSecurityConfigurer`, login views with `LoginForm`, view access control annotations (`@AnonymousAllowed`, `@PermitAll`, `@RolesAllowed`, `@DenyAll`), `AuthenticationContext`, logout handling, and OAuth2/OpenID Connect integration with providers like Google, Keycloak, GitHub, and Okta.

**Use `views-and-navigation` instead** when the question is about `@Route`, `@Layout`, `AppLayout`, `SideNav`, or URL parameters. This skill covers how to *secure* views, not how to create or navigate between them.

**Use `client-side-views` instead** when securing React/Hilla views with `ViewConfig.loginRequired` and `ViewConfig.rolesAllowed`. This skill covers Java/Flow view security, though the annotation-based approach also applies to `@BrowserCallable` endpoints.

## Setting Up Spring Security

Add the Spring Security starter dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Create a security configuration class that uses `VaadinSecurityConfigurer`:

```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.with(VaadinSecurityConfigurer.vaadin(), configurer -> {
            configurer.loginView(LoginView.class);
        });
        return http.build();
    }

    @Bean
    public UserDetailsManager userDetailsManager() {
        // WARNING: In-memory users for development only.
        // Use JDBC, LDAP, or OAuth2 in production.
        var user = User.withUsername("user")
                .password("{noop}user")
                .roles("USER")
                .build();
        var admin = User.withUsername("admin")
                .password("{noop}admin")
                .roles("ADMIN")
                .build();
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

`VaadinSecurityConfigurer.vaadin()` automatically handles:

- **CSRF** — enabled with exemptions for Vaadin internal requests
- **Logout** — configured with Vaadin-aware logout handlers
- **Request cache** — Vaadin-specific request cache for redirect after login
- **Exception handling** — proper error responses for Vaadin requests
- **Authorized requests** — permits Vaadin framework requests (client engine, push, etc.)
- **Navigation access control** — enforces view access annotations

## Login View

Use the built-in `LoginForm` component to create a login page. It provides a form with username and password fields, is compatible with password managers, and handles CSRF tokens automatically.

```java
@Route(value = "login", autoLayout = false)
@PageTitle("Login")
@AnonymousAllowed
public class LoginView extends Main implements BeforeEnterObserver {

    private final LoginForm login;

    public LoginView() {
        login = new LoginForm();
        login.setAction("login");

        addClassNames(LumoUtility.Display.FLEX,
                LumoUtility.JustifyContent.CENTER,
                LumoUtility.AlignItems.CENTER);
        setSizeFull();
        add(login);
    }

    @Override
    public void beforeEnter(BeforeEnterEvent event) {
        if (event.getLocation()
                .getQueryParameters()
                .getParameters()
                .containsKey("error")) {
            login.setError(true);
        }
    }
}
```

Key points:

- **`autoLayout = false`** — prevents the login view from rendering inside the application's main layout (e.g., `AppLayout` with navigation menu)
- **`@AnonymousAllowed`** — required so unauthenticated users can access the page
- **`login.setAction("login")`** — makes the form POST to Spring Security's `/login` endpoint
- **`BeforeEnterObserver`** — checks for the `?error` query parameter that Spring Security adds after a failed login attempt
- **Register in SecurityConfig** — `configurer.loginView(LoginView.class)` tells VaadinSecurityConfigurer which view is the login page

## View Access Control

Control who can access each view using Jakarta and Vaadin security annotations on the view class:

| Annotation | Access Level | Typical Use |
|---|---|---|
| `@AnonymousAllowed` | Anyone (no login required) | Login view, public landing page |
| `@PermitAll` | Any authenticated user | Dashboard, user profile |
| `@RolesAllowed("ADMIN")` | Users with specified role(s) | Admin panel, user management |
| `@DenyAll` | Nobody | Default when no annotation is present |

> **Note on `@PermitAll`:** Vaadin's use of `@PermitAll` differs from the Jakarta Security standard. In standard Jakarta Security, `@PermitAll` means "anyone, including unauthenticated users" — similar to Vaadin's `@AnonymousAllowed`. In Vaadin, `@PermitAll` means "any **authenticated** user." Developers familiar with standard Jakarta security may be confused when access is denied to unauthenticated users on a view they explicitly "permitted all" — use `@AnonymousAllowed` for truly public views.

```java
@Route("public")
@AnonymousAllowed
public class PublicView extends VerticalLayout { }

@Route("dashboard")
@PermitAll
public class DashboardView extends VerticalLayout { }

@Route("admin")
@RolesAllowed("ADMIN")
public class AdminView extends VerticalLayout { }
```

### Annotation Resolution Rules

- **No annotation on a view** — `@DenyAll` applies (access denied by default)
- **Superclass annotations** — inherited from the closest annotated parent class
- **Child class annotated** — overrides parent class annotations
- **Interfaces** — annotations on interfaces are not checked
- **Layouts are checked independently** — both the layout and the view must grant access. A view with `@PermitAll` inside a layout with no annotation (default `@DenyAll`) is inaccessible
- **Override priority when multiple annotations exist** — `@DenyAll` > `@AnonymousAllowed` > `@RolesAllowed` > `@PermitAll`

### Role Constants

Define role names as constants to avoid typos in `@RolesAllowed` annotations:

```java
public final class Roles {
    public static final String ADMIN = "ADMIN";
    public static final String USER = "USER";

    private Roles() {
    }
}

// Usage:
@RolesAllowed(Roles.ADMIN)
public class AdminView extends VerticalLayout { }
```

### Programmatic Access Checks

Inject `AuthenticationContext` to check roles or get user information within a view:

```java
@Route("settings")
@PermitAll
public class SettingsView extends VerticalLayout {

    public SettingsView(AuthenticationContext authContext) {
        authContext.getAuthenticatedUser(UserDetails.class)
                .ifPresent(user -> add(new H2("Welcome " + user.getUsername())));

        if (authContext.hasRole(Roles.ADMIN)) {
            add(new Button("Admin Settings", event -> {
                // show admin-only settings
            }));
        }
    }
}
```

## Logout

### AuthenticationContext (Recommended)

The simplest and most reliable way to log out. `AuthenticationContext` handles session invalidation and redirect automatically. Add a logout button to your `MainLayout`:

```java
public class MainLayout extends AppLayout {

    private final transient AuthenticationContext authContext;

    public MainLayout(AuthenticationContext authContext) {
        this.authContext = authContext;

        var title = new H1("My App");
        title.addClassNames(LumoUtility.FontSize.LARGE, LumoUtility.Margin.NONE);
        var logout = new Button("Logout", event -> authContext.logout());

        var header = new HorizontalLayout(title, logout);
        header.setWidthFull();
        header.setJustifyContentMode(FlexComponent.JustifyContentMode.BETWEEN);
        header.setAlignItems(FlexComponent.Alignment.CENTER);
        header.addClassNames(LumoUtility.Padding.Horizontal.MEDIUM);

        addToNavbar(header);
    }
}
```

`AuthenticationContext` must be declared `transient` because it is not `Serializable`.

By default, logout redirects to `/`. To customize the post-logout redirect, specify a second parameter when registering the login view:

```java
configurer.loginView(LoginView.class, "/goodbye");
```

### SecurityContextLogoutHandler (Alternative)

When `AuthenticationContext` is not available, use `SecurityContextLogoutHandler` directly. The redirect must happen before the handler invalidates the session:

```java
public void logout() {
    UI.getCurrent().getPage().setLocation("/");
    var logoutHandler = new SecurityContextLogoutHandler();
    logoutHandler.logout(
            VaadinServletRequest.getCurrent().getHttpServletRequest(),
            null, null);
}
```

## OAuth2 / OpenID Connect

To authenticate users via an external identity provider (Google, GitHub, Keycloak, Okta, Azure AD), use Spring Security's OAuth2 client support.

### Dependencies

Add the OAuth2 client starter alongside Spring Security:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

### Google (Common Provider)

Google is a Spring Security "common provider", so minimal configuration is needed. Only the client ID and secret are required:

```properties
spring.security.oauth2.client.registration.google.client-id=${GOOGLE_CLIENT_ID}
spring.security.oauth2.client.registration.google.client-secret=${GOOGLE_CLIENT_SECRET}
spring.security.oauth2.client.registration.google.scope=openid,profile,email
```

### Keycloak (Custom Provider)

Providers that are not built into Spring Security require full configuration including the issuer URI:

```properties
spring.security.oauth2.client.registration.keycloak.provider=keycloak
spring.security.oauth2.client.registration.keycloak.client-id=my-client-id
spring.security.oauth2.client.registration.keycloak.client-secret=${KEYCLOAK_CLIENT_SECRET}
spring.security.oauth2.client.registration.keycloak.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.keycloak.scope=openid,profile

spring.security.oauth2.client.provider.keycloak.issuer-uri=http://keycloak.local:8180/realms/my-app
```

### SecurityConfig for OAuth2

Replace `loginView()` with `oauth2LoginPage()`, pointing to the provider's authorization URI:

```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.with(VaadinSecurityConfigurer.vaadin(), configurer -> {
            configurer.oauth2LoginPage(
                    "/oauth2/authorization/google",
                    "{baseUrl}");
        });
        return http.build();
    }
}
```

The `{baseUrl}` template variable resolves to the application's base URL and is used as the post-logout redirect URI. Other supported variables: `{baseScheme}`, `{baseHost}`, `{basePort}`, `{basePath}`.

When using OAuth2, no `LoginView` class is needed — the identity provider handles the login UI. Users are redirected to the provider's login page and back to the application after authentication.

### OAuth2 Logout

For OAuth2 applications, use `AuthenticationContext.logout()` the same way as with form login. The `VaadinSecurityConfigurer` handles the OAuth2-specific logout flow.

### Other Providers

The same pattern works for GitHub, Okta, and Azure AD. The only differences are:

- **`application.properties`** — the registration and provider keys change per provider
- **`oauth2LoginPage()` URL** — use `/oauth2/authorization/{registrationId}` where `{registrationId}` matches the key in `application.properties`
- **Common providers** (Google, GitHub, Facebook, Okta) need only `client-id`, `client-secret`, and `scope`
- **Custom providers** (Keycloak, Azure AD) also need `issuer-uri` and `authorization-grant-type`

## Best Practices

1. **Never hard-code credentials** — use environment variables or a secrets manager. The `UserDetailsManager` with `{noop}` passwords is for development only.
2. **Annotate every view and layout** — views without an access annotation default to `@DenyAll`. Layouts are checked independently; both must grant access.
3. **Use `AuthenticationContext` for logout and user info** — it integrates with Spring Security and handles session cleanup correctly. Declare the field `transient`.
4. **Use `autoLayout = false` on the login view** — the login view should not render inside the application's main layout.
5. **Prefer `VaadinSecurityConfigurer` over manual Spring Security config** — it handles CSRF, logout, request caching, and exception handling for Vaadin automatically.
6. **Externalize OAuth2 credentials** — use `application.properties` with environment variable references (`${GOOGLE_CLIENT_SECRET}`) or Spring profiles.
7. **Define role constants** — create a `Roles` class with `public static final String` fields to avoid typos in `@RolesAllowed` annotations.
8. **Leverage Vaadin's server-side security model** — UI state lives on the server, so attackers cannot tamper with it from the browser. Combined with annotation-based access control, this provides defense in depth.

## Anti-Patterns

1. **Forgetting `autoLayout = false` on the login view** — the login form renders inside the app shell with the navigation menu, which is confusing and may cause access control issues.
2. **Forgetting to annotate the layout** — a view marked `@PermitAll` inside a layout with no annotation (default `@DenyAll`) is inaccessible. Both must independently grant access.
3. **Using `@Secured` or `@PreAuthorize` on views** — these Spring Security annotations are not supported on Vaadin views. Use `@AnonymousAllowed`, `@PermitAll`, `@RolesAllowed`, or `@DenyAll`.
4. **Hard-coding passwords in `SecurityConfig`** — in-memory `{noop}` passwords are for prototyping only. Production applications must use JDBC, LDAP, or OAuth2 authentication.
5. **Calling `SecurityContextLogoutHandler` without redirecting first** — the logout handler invalidates the session, so `UI.getCurrent().getPage().setLocation()` must happen before the handler call. Prefer `AuthenticationContext.logout()` which handles this correctly.
6. **Using URL-pattern security for view access control** — Vaadin uses a single servlet endpoint for all views; the browser URL is updated client-side but all requests go to the same server endpoint. This means URL-pattern rules in `HttpSecurity` (e.g., `.requestMatchers("/admin/**").hasRole("ADMIN")`) do not control access to Vaadin views. Use annotation-based access control (`@RolesAllowed`, `@PermitAll`, etc.) exclusively for view security. URL-pattern rules are still appropriate for non-Vaadin endpoints such as REST APIs.

## Detailed Reference

For a quick-reference cheatsheet of security annotations, OAuth2 provider configurations, login view checklist, and SecurityConfig templates, see `references/security-patterns.md`.
