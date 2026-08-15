CORS is **browser-enforced**.
CORS = **Cross-Origin Resource Sharing**.

This is primarily a **browser security mechanism**.

Suppose:

```
React
http://localhost:3000

        ↓ API call

Spring Boot
http://localhost:8080
```

Different origins:

```
localhost:3000 ≠ localhost:8080
```

Browser may block the request unless the backend allows that origin.

---

## Preflight Request

For certain cross-origin requests, browser first sends:

```
OPTIONS /api/orders
Origin: http://localhost:3000
Access-Control-Request-Method: POST
```

This is called a **preflight request**.

Server responds with appropriate CORS headers:

```
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET,POST,PUT,DELETE
```

Then browser sends the actual request.

```
React
  ↓
OPTIONS /api/orders
  ↓
Spring Boot
  ↓
CORS allowed?
  ↓
YES
  ↓
POST /api/orders
```

Spring configuration might look like:

```
@Bean
CorsConfigurationSource corsConfigurationSource() {

    CorsConfiguration config = new CorsConfiguration();

    config.setAllowedOrigins(
        List.of("https://myfrontend.com")
    );

    config.setAllowedMethods(
        List.of("GET", "POST", "PUT", "DELETE")
    );

    config.setAllowedHeaders(
        List.of("Authorization", "Content-Type")
    );

    UrlBasedCorsConfigurationSource source =
        new UrlBasedCorsConfigurationSource();

    source.registerCorsConfiguration("/**", config);

    return source;
}
```

And:

```
http.cors(Customizer.withDefaults());
```

### Interview distinction

```
CSRF
 ↓
Protects against forged requests

CORS
 ↓
Controls which browser origins can access your API
```

They're **not the same thing**.