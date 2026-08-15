**JSON Web Token (JWT)** is an open standard (RFC 7519) that defines a compact, self-contained method for securely transmitting information between parties as a JSON object.

  

### 1. JWT Structure

A JWT is a single string composed of three distinct Base64URL-encoded parts separated by dots (`.`): `Header.Payload.Signature`.

  

- **Header**: Contains token metadata—typically the hashing algorithm used (`alg`, e.g., `HS256` or `RS256`) and the token type (`typ`: `JWT`).
    
      
    
- **Payload**: Contains the **claims** (statements about the authenticated user and metadata).
    
      
    
- **Signature**: Cryptographic hash that verifies the token wasn't tampered with in transit.
    
      
    

### 2. Claims

Claims are key-value pairs inside the payload. They are split into three types:

  

- **Registered Claims**: Standardized, recommended keys:
    
      
    - `sub` (subject - usually the User ID or username)
        
          
        
    - `iss` (issuer - authority that generated the token)
        
          
        
    - `iat` (issued at - timestamp)
        
          
        
    - `exp` (expiration time - timestamp)
        
          
        
- **Public Claims**: Custom claims collision-proofed by URI naming conventions.
    
      
    
- **Private Claims**: Custom application-specific data (e.g., `roles: ["ROLE_ADMIN"]`, `email: "user@domain.com"`).
    
      
    

> **Crucial Rule:** JWT payloads are Base64URL encoded, **not encrypted**. Never store sensitive secrets (passwords, PINs, SSNs) inside claims.
> 
>   

### 3. Signature & Expiration

- **Signature Calculation**: Calculated by combining the encoded header, encoded payload, and a secret/private key:
    
      
    
    $$\text{Signature} = \text{Algorithm}(\text{Base64Url}(\text{Header}) + "." + \text{Base64Url}(\text{Payload}), \text{Secret})$$
    
- **Expiration (`exp`)**: Defines when the token becomes invalid. During validation, the server verifies if $\text{currentTime} > \text{exp}$. Spring Security JWT parsers automatically throw `ExpiredJwtException` when a token passes this threshold.
    
      
    

### 4. Access Token vs. Refresh Token

|**Feature**|**Access Token**|**Refresh Token**|
|---|---|---|
|**Purpose**|Grants access to protected endpoints|Obtains a new Access Token when expired|
|**Lifetime**|Short-lived (e.g., 5 to 15 minutes)|Long-lived (e.g., 7 to 30 days)|
|**Storage**|Client memory or HTTP Headers|Secure `HttpOnly`, `SameSite` Cookie or DB|
|**Revocation**|Difficult (stateless; relies on fast expiration)|Easy (stored in Redis/DB; can be deleted on logout)|

### 5. Token Generation (Java / JJWT Library)

Java

```
public String generateAccessToken(UserDetails userDetails) {
    Map<String, Object> extraClaims = Map.of(
        "roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList())
    );

    return Jwts.builder()
        .claims(extraClaims)
        .subject(userDetails.getUsername())
        .issuedAt(new Date())
        .expiration(new Date(System.currentTimeMillis() + 15 * 60 * 1000)) // 15 mins
        .signWith(getSigningKey(), SignatureAlgorithm.HS256)
        .compact();
}
```

### 6. Token Validation Workflow

Within a custom Spring Security `OncePerRequestFilter`:

  

1. **Header Parsing:** Extract the header (`Authorization: Bearer <token>`).
    
      
    
2. **Signature Verification & Decoding:** Parse claims using the signing key.
    
      
    
3. **Exception Handling:** Catch exceptions (`ExpiredJwtException`, `MalformedJwtException`, `SignatureException`).
    
      
    
4. **Context Injection:** Load `UserDetails`, verify username match, and set authentication:
    
      
    
    Java
    
    ```
    UsernamePasswordAuthenticationToken authToken = 
        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
    SecurityContextHolder.getContext().setAuthentication(authToken);
    ```
    

### Top Interview Talking Points & Pitfalls

- **Symmetric vs. Asymmetric Signing:**
    
      
    - **HS256 (Symmetric):** The auth server and resource server share the exact same secret key. Best for monolithic apps.
        
          
        
    - **RS256 (Asymmetric):** Auth server signs tokens with a **Private Key**; microservices verify tokens using a public **Public Key** (`jwks_uri`). Best for distributed systems.
        
          
        
- **Stateless Logout Problem:** To invalidate a JWT before its `exp` time, you must track blacklisted tokens in an in-memory cache like Redis or rely strictly on low TTL access tokens combined with Refresh Token revocation.