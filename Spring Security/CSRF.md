### What is CSRF?

CSRF = **Cross-Site Request Forgery**.

Imagine:

```
User logged into bank.com
        ↓
Browser has bank session cookie
        ↓
User visits malicious.com
        ↓
malicious.com causes request to bank.com
        ↓
Browser automatically sends cookie
```

The server thinks:

> "This request has a valid session cookie."

That's the CSRF problem.

### Why usually disable it for JWT REST APIs?

If your API uses:

```
Authorization: Bearer <JWT>
```

and the browser does **not automatically attach that token** to another site's request, the classic cookie-based CSRF scenario doesn't apply in the same way.

Therefore:

```
csrf(csrf -> csrf.disable())
```

is common for stateless REST APIs using Authorization headers.

### When NOT to disable CSRF?

If your application uses:

```
Browser
   ↓
Session Cookie
   ↓
Server
```

especially for browser-based applications, CSRF protection is important.

### Interview answer

> "CSRF primarily protects against unauthorized requests made using automatically attached browser credentials such as cookies. For a stateless REST API using bearer tokens in the Authorization header, CSRF is commonly disabled. For session-cookie-based applications, we generally keep CSRF protection enabled."