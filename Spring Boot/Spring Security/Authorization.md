**Authorization** is the process of determining whether an already authenticated user has permission to perform a specific action or access a specific resource.

In Spring Security, authorization happens at two distinct levels: **Request Level** (HTTP URIs) and **Method Level** (Java methods).

### 1. Roles vs. Authorities (Crucial Interview Question)

In Spring Security, permissions are fundamentally represented as `GrantedAuthority` objects.

- **Authority (Privilege/Permission):** A fine-grained string representing a specific action (e.g., `READ_DOCUMENT`, `DELETE_USER`).
    
- **Role:** A coarse-grained group of authorities. By default, Spring Security convention prefixes roles with **`ROLE_`** (e.g., `ROLE_ADMIN`, `ROLE_USER`).
    

**The Prefix Convention Rule:**

- `.hasRole("ADMIN")` automatically appends the `ROLE_` prefix under the hood and checks if the user has `ROLE_ADMIN`.
    
- `.hasAuthority("ADMIN")` checks for the literal, exact string `"ADMIN"`.
    
- `.hasAuthority("ROLE_ADMIN")` is strictly equivalent to `.hasRole("ADMIN")`.
    

### 2. Request-Level Authorization

Configured inside `SecurityFilterChain` using `authorizeHttpRequests`. Evaluation stops at the **first matching rule**, so order rules from most specific to least specific.

Java

```
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers(HttpMethod.POST, "/api/products").hasAuthority("WRITE_PRODUCTS")
            .requestMatchers("/api/admin/**").hasRole("ADMIN") // Checks for ROLE_ADMIN
            .anyRequest().authenticated() // Wildcard matcher MUST be last
        );
    return http.build();
}
```

### 3. Method-Level Authorization (Method Security)

Method security enables securing service, repository, or controller layer methods directly using Spring Expression Language (SpEL).

To enable it in Spring Security 6+, annotate a configuration class with `@EnableMethodSecurity`:

Java

```
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // Replaces deprecated @EnableGlobalMethodSecurity
public class SecurityConfig { ... }
```

#### Core Annotations:

- **`@PreAuthorize`**: Evaluates authorization **before** the method executes. Most common choice.
    
    Java
    
    ```
    // Check role or permission
    @PreAuthorize("hasRole('ADMIN') or hasAuthority('DOCUMENT_WRITE')")
    public void deleteDocument(Long id) { ... }
    
    // Check if current authenticated user owns the resource (SpEL parameter binding)
    @PreAuthorize("#username == authentication.principal.username or hasRole('ADMIN')")
    public UserProfile getProfile(String username) { ... }
    ```
    
- **`@PostAuthorize`**: Executes the method first, then evaluates the expression **before returning the result**. Uses `returnObject` to inspect the method output.
    
    Java
    
    ```
    // Executes method, then checks if current user owns the returned object
    @PostAuthorize("returnObject.ownerId == authentication.principal.id")
    public AccountDetails getAccount(Long accountId) { ... }
    ```
    
- **`@PreFilter` & `@PostFilter`**: Used for filtering collections/arrays.
    
    - `@PreFilter`: Filters input collection parameters before method execution.
        
    - `@PostFilter`: Filters the returned collection, removing elements the user isn't authorized to view.
        
    
    Java
    
    ```
    @PostFilter("filterObject.owner == authentication.name")
    public List<Order> getOrders() { ... }
    ```
    

### 4. Under the Hood (Spring Security 6 Architecture)

1. **AuthorizationManager Interface:** Replaced the legacy `AccessDecisionManager` and `AccessDecisionVoter` components.
    
2. **Request Interception:** `AuthorizationFilter` sits near the end of the `SecurityFilterChain`, delegates to `RequestMatcherDelegatingAuthorizationManager`, and checks matching request rules.
    
3. **Method Interception:** Method security uses Spring AOP proxies (`MethodSecurityInterceptor`). When a method with `@PreAuthorize` is invoked, the proxy intercepts the call and evaluates the SpEL expression against the current `Authentication` in `SecurityContextHolder`.
    

### Top Interview Talking Points & Pitfalls

- **Spring AOP Proxy Limitation:** `@PreAuthorize` uses Spring AOP proxies. If `Method A` in a class calls `Method B` (which has `@PreAuthorize`) _within the same class_ (`this.methodB()`), the authorization check will **not** trigger because the call bypasses the Spring AOP proxy.
    
- **Overusing `@PostAuthorize` Side Effects:** Warn interviewers that if a method in `@PostAuthorize` performs a mutation or expensive database write, that write **still happens** even if authorization fails—Spring Security will simply throw an `AccessDeniedException` and prevent the client from receiving the return object.
    
- **HTTP 401 vs. HTTP 403:**
    
    - **401 Unauthorized:** User is NOT authenticated (missing/invalid token). Handled by `AuthenticationEntryPoint`.
        
    - **403 Forbidden:** User IS authenticated, but lacks sufficient roles/authorities. Handled by `AccessDeniedHandler`.