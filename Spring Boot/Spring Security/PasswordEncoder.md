**`PasswordEncoder`** is a core interface in Spring Security (`org.springframework.security.crypto.password`) used to perform one-way hashing of plain-text passwords and safely verify raw passwords against stored hashes.

**Core Interface Methods**

- **`String encode(CharSequence rawPassword)`**: Hashes a raw password. The underlying algorithm automatically generates a unique salt for every invocation.
    
- **`boolean matches(CharSequence rawPassword, String encodedPassword)`**: Compares a raw password with an encoded hash using a constant-time comparison to prevent timing attacks.
    
- **`default boolean upgradeEncoding(String encodedPassword)`**: Checks whether an encoded password should be re-hashed (e.g., if the algorithm or work factor was upgraded). Returns `false` by default.
    

**Key Implementations**

- **`DelegatingPasswordEncoder`** (Default): Wraps multiple encoders and identifies the format using a prefix, e.g., `{bcrypt}$2a$10...` or `{argon2}...`. Allows upgrading security algorithms over time without forcing users to reset passwords.
    
- **`BCryptPasswordEncoder`**: Uses the BCrypt adaptive hashing function. Highly configurable via a strength parameter (log rounds, default `10`).
    
- **`Argon2PasswordEncoder` / `Pbkdf2PasswordEncoder`**: Hardware-resistant alternatives that protect against GPU/ASIC cracking attacks.
    
- **`NoOpPasswordEncoder`**: Deprecated encoder storing plain text. Used strictly for legacy or testing environments.
    

**Modern Configuration & Usage**

Java

```
@Configuration
public class SecurityConfig {

    // Explicit BCrypt bean with a custom work factor (strength = 12)
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}
```