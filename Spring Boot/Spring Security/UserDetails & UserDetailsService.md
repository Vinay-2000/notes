```
UserDetails
    ↓
"Information needed to authenticate/describe the user"

Authentication
    ↓
"Result/current state of authentication for this request"
```

`UserDetails` is a Spring Security interface that represents the **user information needed by Spring Security for authentication and authorization**.

It provides things like:

```
Username
Password
Authorities
Account status
```

The interface conceptually provides:

```
public interface UserDetails {

    String getUsername();

    String getPassword();

    Collection<? extends GrantedAuthority>
        getAuthorities();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

`UserDetailsService` is an interface used to **load user information**.

It has one main method:

```
UserDetails loadUserByUsername(String username)
```

So when Spring Security needs to authenticate:

```
username = vinay
```

it can do:

```
UserDetails user =
    userDetailsService.loadUserByUsername("vinay");
```

### 1. `UserDetails` (The Model)

`UserDetails` is an interface in `org.springframework.security.core.userdetails`. It encapsulates user attributes required for Spring Security to make authentication and authorization decisions.

**Key Methods:**

- **`getAuthorities()`**: Returns `Collection<? extends GrantedAuthority>` (e.g., `ROLE_USER`, `READ_PRIVILEGE`).
    
- **`getPassword()` & `getUsername()`**: Credentials and identifier.
    
- **Boolean Account Flags**:
    
    - `isAccountNonExpired()`
        
    - `isAccountNonLocked()`
        
    - `isCredentialsNonExpired()`
        
    - `isEnabled()`
        
        _(If any return `false`, authentication automatically fails with a specific exception like `LockedException`)._
        

### 2. `UserDetailsService` (The DAO Adapter)

`UserDetailsService` is a functional interface with a single method:

Java

```
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

It acts as a bridge between your custom user storage (JPA, Mongo, Redis, external API) and Spring Security’s `DaoAuthenticationProvider`.