### Core Concepts

```
OAuth2
→ Framework/protocol for authorization and token issuance

JWT
→ Token format commonly used for OAuth2 access tokens

JWKS
→ Endpoint containing public keys used to verify JWT signatures
```

### Important JWT Claims

```
iss → Issuer
      Who issued the token?
      Example: Authorization Server

aud → Audience
      Which service/API is this token intended for?

sub → Subject
      Who does this token represent?

exp → Expiration
      When does the token expire?
```

### JWKS + `kid`

With asymmetric signing like **RS256**:

```
Authorization Server
    │
    ├── Private Key → Signs JWT
    │
    └── JWKS → Publishes Public Key(s)
```

JWT header:

```
{
  "alg": "RS256",
  "kid": "key-2026"
}
```

`kid` = **Key ID**.

Resource server:

```
JWT
 ↓
kid = key-2026
 ↓
JWKS
 ↓
Find matching public key
 ↓
Verify JWT signature
```

This also supports **signing-key rotation**.

### Spring Security

With OAuth2 Resource Server:

```
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth.company.com
```

Spring Security can use the issuer metadata to discover the JWKS endpoint and validate JWTs.

### Complete Flow

```
User
 ↓
OAuth2 Authorization Server
 ↓
JWT signed with Private Key
 ↓
Spring Boot Resource Server
 ↓
Read kid
 ↓
Get matching Public Key from JWKS
 ↓
Verify Signature
 ↓
Validate iss / aud / exp
 ↓
Create Authentication
 ↓
SecurityContext
 ↓
Authorization
```

### Interview Answer

> **`iss` identifies the token issuer, `aud` identifies the intended API, and `kid` identifies the signing key. With RS256, the Authorization Server signs the JWT using a private key and publishes the corresponding public keys through JWKS. Spring Security uses the issuer/JWKS information to select the correct public key and validate the JWT before creating the authenticated SecurityContext.**