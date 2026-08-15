`Authentication` represents the **authentication information of the current user**.
Conceptually:

```
Authentication
├── Principal
├── Credentials
├── Authorities
└── Authenticated?
```

For example:

```
Principal:   vinay
Credentials: password
Authorities: ROLE_USER
Authenticated: true
```

After successful authentication, Spring Security can have something like:

```
Authentication authentication;
```
and:
```
authentication.getPrincipal();
authentication.getAuthorities();
authentication.isAuthenticated();
```
this gets stored in the `SecurityContext`
```
Controller
   │
   │ credentials
   ↓
UsernamePasswordAuthenticationToken
   │
   ↓
AuthenticationManager
   │
   ↓
AuthenticationProvider
   │
   ├── UserDetailsService
   │
   └── PasswordEncoder
   │
   ↓
Authenticated
   │
   ↓
JWT
```

```
@PostMapping("/login")
public LoginResponse login(
        @RequestBody LoginRequest request) {

    Authentication token =
            new UsernamePasswordAuthenticationToken(
                    request.getUsername(),
                    request.getPassword()
            );

    Authentication authentication =
            authenticationManager.authenticate(token);

    String jwt = jwtService.generateToken(authentication);

    return new LoginResponse(jwt);
}
```

### Core Components

- **`SecurityContextHolder`**: Storage location for current security details. Uses a `ThreadLocal` strategy by default to store the `SecurityContext` for the duration of a request.
    
- **`Authentication`**: Interface holding the user's details:
    
    - `Principal`: User identity (e.g., `UserDetails` object or username string).
        
    - `Credentials`: Secret proving identity (e.g., password; usually erased after authentication).
        
    - `Authorities`: Granted permissions/roles (`GrantedAuthority` list, e.g., `ROLE_ADMIN`).
        
    - `authenticated`: Boolean flag (`true` once verified).
        
- **`AuthenticationManager`**: Primary interface for initiating authentication. The standard implementation is **`ProviderManager`**.
    
- **`AuthenticationProvider`**: Performs actual validation. Each provider handles a specific authentication type (e.g., `DaoAuthenticationProvider` for username/password, `JwtAuthenticationProvider` for tokens).
    
- **`UserDetailsService`**: Core interface with a single method, `loadUserByUsername(String username)`, used by `DaoAuthenticationProvider` to retrieve user data from DB/storage.
    
- **`PasswordEncoder`**: Hashes and matches passwords securely (e.g., `BCryptPasswordEncoder`).
    

### Step-by-Step Authentication Flow

1. **Extraction:** A filter (like `UsernamePasswordAuthenticationFilter` or custom JWT filter) extracts credentials from the HTTP request and creates an **unauthenticated** `Authentication` object (e.g., `UsernamePasswordAuthenticationToken`).
    
2. **Delegation:** The filter calls `AuthenticationManager.authenticate(unauthenticatedToken)`.
    
3. **Provider Selection:** `ProviderManager` iterates through configured `AuthenticationProvider`s to find one that supports the token type via `supports()`.
    
4. **Verification:** The `AuthenticationProvider`:
    
    - Fetches user data via `UserDetailsService`.
        
    - Verifies credentials via `PasswordEncoder.matches()`.
        
5. **Context Population:** Upon success, a **fully populated authenticated token** is returned and saved:

  ```
    SecurityContextHolder.getContext().setAuthentication(authenticatedToken);
  ```
