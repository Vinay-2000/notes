> `SecurityContextHolder` gives you the current `SecurityContext`, and that contains the current `Authentication`. The `Authentication` can have the user's `UserDetails` as its principal.

So the chain is:

```
SecurityContextHolder
        ↓
SecurityContext
        ↓
Authentication
        ↓
Principal
        ↓
UserDetails   ← commonly, but not guaranteed
```

For example:

```
Authentication authentication =
    SecurityContextHolder
        .getContext()
        .getAuthentication();

UserDetails userDetails =
    (UserDetails) authentication.getPrincipal();

String username = userDetails.getUsername();
```

Or if you only need the username:

```
String username =
    authentication.getName();
```