# Security Patterns Quick Reference

## Security Annotations

| Annotation | Access Level | Typical Use |
|---|---|---|
| `@AnonymousAllowed` | Anyone (no login required) | Login view, public landing page |
| `@PermitAll` | Any authenticated user | Dashboard, user profile |
| `@RolesAllowed("ADMIN")` | Users with specified role(s) | Admin panel, user management |
| `@DenyAll` | Nobody | Default when no annotation is present |

> `@AnonymousAllowed` is Vaadin-specific (`com.vaadin.flow.server.auth`). The rest are Jakarta annotations (`jakarta.annotation.security`). `@Secured` and `@PreAuthorize` are **not supported** on Vaadin views.

> **Vaadin-specific behavior:** In standard Jakarta Security, `@PermitAll` allows all callers including unauthenticated ones. Vaadin overrides this: `@PermitAll` requires authentication. Use `@AnonymousAllowed` for truly public access.

## Annotation Resolution Rules

| Rule | Behavior |
|---|---|
| No annotation on view | `@DenyAll` applies (access denied) |
| Inherited from superclass | Closest annotated parent class wins |
| Child class annotated | Overrides parent class annotation |
| Interfaces | Not checked for annotations |
| Layout + View | Both must independently grant access |
| Layout without annotation | `@DenyAll` applies to the layout |
| Override priority | `@DenyAll` > `@AnonymousAllowed` > `@RolesAllowed` > `@PermitAll` |

## Login View Checklist

- [ ] `@Route(value = "login", autoLayout = false)` — prevents embedding in main layout
- [ ] `@AnonymousAllowed` — allows unauthenticated access
- [ ] `extends Main` (or another `Component` subclass)
- [ ] `implements BeforeEnterObserver` — for error parameter handling
- [ ] `login.setAction("login")` — POSTs to Spring Security's form login endpoint
- [ ] Check `?error` query parameter in `beforeEnter()` and call `login.setError(true)`
- [ ] Register in SecurityConfig: `configurer.loginView(LoginView.class)`

## Copy-Paste: SecurityConfig (Form Login)

```java
import com.vaadin.flow.spring.security.VaadinSecurityConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsManager;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

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

## Copy-Paste: LoginView

```java
import com.vaadin.flow.component.html.Main;
import com.vaadin.flow.component.login.LoginForm;
import com.vaadin.flow.router.BeforeEnterEvent;
import com.vaadin.flow.router.BeforeEnterObserver;
import com.vaadin.flow.router.PageTitle;
import com.vaadin.flow.router.Route;
import com.vaadin.flow.server.auth.AnonymousAllowed;
import com.vaadin.flow.theme.lumo.LumoUtility;

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

## OAuth2 Provider Configuration

| Property | Google | Keycloak | GitHub |
|---|---|---|---|
| `registration.{id}.client-id` | `${GOOGLE_CLIENT_ID}` | `my-client-id` | `${GITHUB_CLIENT_ID}` |
| `registration.{id}.client-secret` | `${GOOGLE_CLIENT_SECRET}` | `${KEYCLOAK_SECRET}` | `${GITHUB_CLIENT_SECRET}` |
| `registration.{id}.scope` | `openid,profile,email` | `openid,profile` | `user:email` |
| `registration.{id}.authorization-grant-type` | *(auto)* | `authorization_code` | *(auto)* |
| `provider.{id}.issuer-uri` | *(auto — common provider)* | `http://host/realms/app` | *(auto — common provider)* |
| `oauth2LoginPage` URL | `/oauth2/authorization/google` | `/oauth2/authorization/keycloak` | `/oauth2/authorization/github` |

> Google and GitHub are Spring Security "common providers" — issuer-uri and grant-type are auto-configured. Custom providers like Keycloak require full configuration.

## Copy-Paste: SecurityConfig (OAuth2)

```java
import com.vaadin.flow.spring.security.VaadinSecurityConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

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

## Logout Approaches

| Approach | When to Use | Key Code |
|---|---|---|
| `AuthenticationContext.logout()` | Recommended for all cases | `authContext.logout()` |
| `SecurityContextLogoutHandler` | When `AuthenticationContext` unavailable | `new SecurityContextLogoutHandler().logout(request, null, null)` |
| Custom post-logout URL | Redirect to specific page after logout | `configurer.loginView(LoginView.class, "/goodbye")` |

## AuthenticationContext API

| Method | Purpose |
|---|---|
| `isAuthenticated()` | Check if user is logged in |
| `getAuthenticatedUser(Class<U>)` | Get user details (e.g., `UserDetails.class`) |
| `getGrantedRoles()` | Get roles without `ROLE_` prefix |
| `hasRole(String)` | Check single role |
| `hasAnyRole(String...)` | Check if user has any of the roles |
| `hasAllRoles(String...)` | Check if user has all roles |
| `logout()` | Log out and redirect |

> Declare `AuthenticationContext` fields as `transient` — it is not `Serializable`.

## Copy-Paste: MainLayout with Logout Button

```java
import com.vaadin.flow.component.applayout.AppLayout;
import com.vaadin.flow.component.button.Button;
import com.vaadin.flow.component.html.H1;
import com.vaadin.flow.component.orderedlayout.FlexComponent;
import com.vaadin.flow.component.orderedlayout.HorizontalLayout;
import com.vaadin.flow.spring.security.AuthenticationContext;
import com.vaadin.flow.theme.lumo.LumoUtility;
import org.springframework.security.core.userdetails.UserDetails;

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
