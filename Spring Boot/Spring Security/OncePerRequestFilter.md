`OncePerRequestFilter` is an abstract base class in Spring (`org.springframework.web.filter`) that guarantees a custom filter executes **exactly once per incoming HTTP request**, preventing duplicate filter execution during internal Servlet dispatches.

**Why Use It Over Standard Servlet `Filter`?**

- **Servlet Forwards & Includes:** Standard Java `jakarta.servlet.Filter` implementations can trigger multiple times during a single HTTP request if the container performs an internal dispatch (e.g., forwarding to `/error` or an internal JSP/controller).
    
- **Async Requests:** Asynchronous request processing (e.g., Spring MVC `DeferredResult` or `Callable`) re-dispatches the request to the Servlet container when processing completes. `OncePerRequestFilter` uses an internal request attribute lock to track whether the filter has already run.
    

**How to Implement It**


```
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        String token = extractToken(request);
        
        if (token != null && validateToken(token)) {
            Authentication auth = getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        
        // CRITICAL: Always continue the chain
        filterChain.doFilter(request, response);
    }

    // Optional: Bypass filter execution for specific endpoints
    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        return request.getServletPath().startsWith("/api/public/");
    }
}
```

**Top Interview Talking Points & Pitfalls**

- **The `@Component` Double-Execution Bug:** If you mark your custom filter with `@Component` **and** register it manually in `SecurityFilterChain` via `.addFilterBefore(...)`, Spring Boot will automatically register the `@Component` as a global Servlet filter. Your filter will execute twice!
    
    - _Solution:_ Don't use `@Component`. Instantiate the filter via `new CustomFilter()` inside your `@Configuration` class, or register a `FilterRegistrationBean<CustomFilter>` bean with `registration.setEnabled(false)`.
        
- **`shouldNotFilter()` Overriding:** Explain that overriding `shouldNotFilter(HttpServletRequest request)` is cleaner than clogging `doFilterInternal()` with `if/else` checks to skip public paths.
    
- **SecurityContext Cleanup:** Point out that because `SecurityContextHolder` uses `ThreadLocal` storage, filters setting authentication must either run before `SecurityContextPersistenceFilter`/`SecurityContextHolderFilter` or rely on Spring's automated thread-clearing at the end of the chain.