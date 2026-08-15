Refresh tokens solve the fundamental conflict between **security** and **user experience**: access tokens must be short-lived to minimize damage if stolen, but forcing users to re-type their password every 15 minutes creates a terrible experience.

  

### Why Refresh Tokens Are Used

- **Mitigating Access Token Theft:** Access tokens are stateless and sent with every single API call, making them vulnerable to network interception or client-side leaks. Keeping their lifespan short (5–15 minutes) ensures a stolen token becomes useless quickly.
    
      
    
- **Server-Side Control (Revocation):** Because access tokens are stateless, the server cannot easily revoke them before they expire. Refresh tokens, however, are stored on the server (in Redis or a database), allowing instant session revocation (e.g., clicking "Log out of all devices" or locking an account).
    
      
    
- **Silent Session Extension:** Users stay logged in seamlessly for days or weeks without exposing long-lived credentials to every API request.
    
      
    

### How Refresh Tokens Are Given (Login Flow)

```
[Client] ────── POST /api/v1/auth/login (username, password) ──────► [Auth Server]
                                                                          │
                                                                 1. Verify Credentials
                                                                 2. Generate Access Token (15m)
                                                                 3. Generate Refresh Token (7d)
                                                                 4. Save Refresh Token in Redis/DB
                                                                          │
[Client] ◄─── Access Token (JSON) + Refresh Token (HttpOnly Cookie) ──────┘
```

1. **Authentication:** The user sends credentials to `POST /api/v1/auth/login`.
    
      
    
2. **Dual-Token Generation:** Upon success, the server generates both an **Access Token** (short TTL, e.g., 15 mins) and a **Refresh Token** (long TTL, e.g., 7 days).
    
      
    
3. **Storage & Delivery:**
    
      
    - **Access Token:** Returned in the JSON response body. The client stores it in memory (or web app state).
        
          
        
    - **Refresh Token:** Sent back in a secure, server-set **`HttpOnly` Cookie** (`Set-Cookie: refreshToken=...; HttpOnly; Secure; SameSite=Strict`).
        
          
        
    - **Database Tracking:** The auth server saves the refresh token hash, associated `userId`, expiration, and `revoked` status in Redis or DB.
        
          
        

> **Why `HttpOnly` Cookies?** JavaScript cannot read `HttpOnly` cookies. If your web app suffers an XSS (Cross-Site Scripting) vulnerability, attackers can steal the access token from memory, but they **cannot** steal the refresh token from the cookie.
> 
>   

### How Refresh Tokens Are Taken / Used (Token Refresh Flow)

```
[Client] ────── GET /api/v1/orders (Expired Access Token) ──────► [Resource Server]
                                                                        │
[Client] ◄──────────────── 401 Unauthorized ───────────────────────────┘
   │
   │ (HTTP Interceptor catches 401)
   │
[Client] ────── POST /api/v1/auth/refresh ──────────────────────► [Auth Server]
         (Sends Refresh Token via HttpOnly Cookie automatically)         │
                                                                1. Verify Signature & Expiration
                                                                2. Lookup Token in Redis/DB
                                                                3. Check if revoked
                                                                4. Issue NEW Access Token
                                                                        │
[Client] ◄─────────────── 200 OK (New Access Token) ─────────────────────┘
   │
   │ (Retry original failed request)
   │
[Client] ────── GET /api/v1/orders (New Access Token) ─────────► [Resource Server]
```

1. **Failure Trigger:** The client sends an API request with an expired access token. The API returns `401 Unauthorized`.
    
      
    
2. **Intercept & Refresh:** A client-side HTTP interceptor catches the `401`, holds pending requests, and calls `POST /api/v1/auth/refresh`.
    
      
    
3. **Automatic Cookie Transmission:** The browser automatically attaches the `refreshToken` `HttpOnly` cookie to this request.
    
      
    
4. **Server Validation:**
    
      
    - Validates JWT signature and `exp` claim.
        
          
        
    - Checks Redis/DB to confirm the token exists and `revoked == false`.
        
          
        
5. **Issue & Retry:** The server issues a **new Access Token**. The client updates its state and retries the original failed API request transparently to the user.
    
      
    

### Key Interview Concepts

- **Refresh Token Rotation:** Every time a refresh token is used, the server revokes it and issues a _new_ refresh token along with the new access token. If an attacker steals a refresh token and tries to use it after the legitimate user already rotated it, the server detects duplicate usage, invalidates the entire token family, and forces a full re-login.
    
      
    
- **Logout Mechanism:** Logging out simply means calling `POST /api/v1/auth/logout`, which deletes the refresh token record from Redis/DB and clears the cookie from the browser (`Set-Cookie: refreshToken=; Max-Age=0`).