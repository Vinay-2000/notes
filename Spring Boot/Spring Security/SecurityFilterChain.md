`SecurityFilterChain` is the core mechanism in Spring Security responsible for intercepting, evaluating, and applying security controls to incoming HTTP requests through a series of Servlet Filters.

**What is SecurityFilterChain?** It is an interface that matches an incoming `HttpServletRequest` against a specific set of security filters (e.g., `UsernamePasswordAuthenticationFilter`, `BearerTokenAuthenticationFilter`, `ExceptionTranslationFilter`). Instead of applying one monolithic filter, Spring routes requests through an ordered list of modular filters.

**Why is it used?**

- **Replaced Legacy Adapters**: Replaced the deprecated `WebSecurityConfigurerAdapter` (removed in Spring Security 6.0) in favor of a clean, component-based, functional bean definition.
    
- **Targeted Security Rules**: Enables defining multiple filter chains with different priorities (`@Order`) for different URL patterns (e.g., one chain for REST APIs using JWT, another for web MVC using session cookies).
    
- **Decoupled Architecture**: Separates filter execution logic from Spring Boot’s main Servlet container lifecycle, delegating control entirely to the Spring application context.
    

**How It Works Under the Hood**

1. **DelegatingFilterProxy**: The Servlet container hands the incoming request to Spring's `DelegatingFilterProxy`.
    
2. **FilterChainProxy**: `DelegatingFilterProxy` delegates execution to `FilterChainProxy`, which manages all registered `SecurityFilterChain` instances.
    
3. **Chain Selection**: `FilterChainProxy` inspects the request URI and selects the **first** matching `SecurityFilterChain`.
```
HTTP Request
     │
     ↓
FilterChainProxy
     │
     ├── SecurityFilterChain #1
     │       matcher: /api/**
     │
     ├── SecurityFilterChain #2
     │       matcher: /admin/**
     │
     └── SecurityFilterChain #3
             matcher: /**
     │
     ↓
First matching chain
     ↓
Its filters execute
     ↓
Controller
```
4. **Execution**: The request traverses each filter in that specific chain sequentially before hitting the `DispatcherServlet`.
    

**How to Implement It (Spring Security 6+)**



```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**") // Applies this chain only to /api/**
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(new CustomJwtFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

**Key Interview Talking Points**

- **Filter Ordering**: Standard Spring Security filters run in a strict, predefined order. Use `.addFilterBefore()` or `.addFilterAfter()` when inserting custom filters.
    
- **Multiple Filter Chains**: Always give narrower URL matchers higher precedence using `@Order(1)` so broad matchers (like `/**`) don't shadow them.


**`securityMatcher`** determines whether an incoming request enters a specific **`SecurityFilterChain`**, whereas **`requestMatchers`** determines path-level authorization rules (like `permitAll()` or `hasRole()`) for endpoints _within_ that chain.


```
@Bean
@Order(1) // High priority API chain
public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
    http
        // 1. CHAIN SELECTOR: Only /api/** requests enter this chain
        .securityMatcher("/api/**") 
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            // 2. ENDPOINT RULES: Applied only to requests that entered this chain
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        );

    return http.build();
}
```

                 Request
                    ↓
            FilterChainProxy
                    ↓
            securityMatcher
             "Which chain?"
                    ↓
             Selected Chain
                    ↓
            requestMatchers
             "What access?"
                    ↓
              Authorization