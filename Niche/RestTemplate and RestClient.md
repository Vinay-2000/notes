`RestTemplate` and `RestClient` are both synchronous HTTP clients in Spring, but **RestClient is the modern replacement** introduced in **Spring Framework 6.1 (Spring Boot 3.2)**.

Since you're using **Spring Boot 3.2**, you should prefer **RestClient** for new applications.

## Quick Comparison

|Feature|RestTemplate|RestClient|
|---|---|---|
|Status|Maintenance mode|Recommended|
|Introduced|Spring 3|Spring 6.1 / Boot 3.2|
|API Style|Verbose|Fluent|
|Builder Pattern|Limited|Yes|
|Readability|Moderate|Excellent|
|Request Customization|Harder|Easier|
|Future Development|No new features|Active development|

---

# RestTemplate

You create a `RestTemplate` object and call methods like `getForObject()`, `postForEntity()`.

```java
RestTemplate restTemplate = new RestTemplate();

User user = restTemplate.getForObject(
    "https://api.example.com/users/1",
    User.class
);
```

POST example

```java
User user = new User("Vinay");

ResponseEntity<User> response =
    restTemplate.postForEntity(
        "https://api.example.com/users",
        user,
        User.class
    );
```

---

## Problems with RestTemplate

Methods are scattered.

```
getForObject()

getForEntity()

postForObject()

postForEntity()

exchange()

delete()

put()

patchForObject()
```

As APIs become more complex, most people end up using `exchange()` everywhere.

```java
ResponseEntity<User> response =
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        entity,
        User.class
    );
```

Lots of parameters.

Hard to read.

---

# RestClient

Spring redesigned the API to be fluent.

```java
RestClient restClient = RestClient.create();

User user = restClient
        .get()
        .uri("https://api.example.com/users/1")
        .retrieve()
        .body(User.class);
```

Reads almost like English.

```
get()

uri()

retrieve()

body()
```

---

POST

```java
User created = restClient
        .post()
        .uri("/users")
        .body(user)
        .retrieve()
        .body(User.class);
```

---

Adding headers

### RestTemplate

```java
HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth(token);

HttpEntity<?> entity = new HttpEntity<>(headers);

ResponseEntity<User> response =
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        entity,
        User.class
    );
```

---

### RestClient

```java
User user = restClient
        .get()
        .uri(url)
        .headers(h -> h.setBearerAuth(token))
        .retrieve()
        .body(User.class);
```

Much cleaner.

---

# Error Handling

### RestTemplate

Usually with `try-catch`

```java
try {
    User user = restTemplate.getForObject(url, User.class);
}
catch (HttpClientErrorException ex) {
    ...
}
```

Or implement

```java
ResponseErrorHandler
```

---

### RestClient

Built into the fluent chain.

```java
User user = restClient
        .get()
        .uri(url)
        .retrieve()
        .onStatus(
            HttpStatusCode::is4xxClientError,
            (request, response) -> {
                throw new RuntimeException("Client Error");
            }
        )
        .body(User.class);
```

---

# Configuration

### RestTemplate

```java
@Bean
RestTemplate restTemplate() {
    return new RestTemplate();
}
```

---

### RestClient

```java
@Bean
RestClient restClient(RestClient.Builder builder) {
    return builder
            .baseUrl("https://api.example.com")
            .defaultHeader("API-KEY", "123")
            .build();
}
```

Everything can be configured in one place.

---

# Timeouts

Both support

- Connect Timeout
    
- Read Timeout
    

But with `RestClient`, configuration is cleaner because it uses the builder.

---

# Under the Hood

An important interview point:

> **RestClient is not a completely new HTTP implementation.** It uses the same underlying infrastructure as `RestTemplate` (message converters, request factories, interceptors, etc.) but exposes a modern fluent API.

So if you already know `RestTemplate`, learning `RestClient` is straightforward.

---

# RestClient vs WebClient

Many people confuse these.

|RestClient|WebClient|
|---|---|
|Synchronous|Asynchronous & Reactive|
|Thread blocks until response|Non-blocking|
|Spring MVC|Spring WebFlux|
|Easier for traditional apps|Best for high concurrency|

If you're building a typical Spring Boot REST API (like your current projects), **RestClient** is generally the right choice. Use **WebClient** when you're building reactive applications or need non-blocking I/O.


---

# Which should you use?

- Existing project already using `RestTemplate` → Continue using it unless you're refactoring.
    
- New Spring Boot 3.2+ project → Use `RestClient`.
    
- Reactive application or very high concurrency → Use `WebClient`.
    

### Interview answer (30 seconds)

> `RestTemplate` is Spring's older synchronous HTTP client and is now in maintenance mode. `RestClient`, introduced in Spring Framework 6.1, is its modern replacement. Both are synchronous and share the same underlying infrastructure, but `RestClient` provides a fluent, builder-based API, making request creation, header configuration, and error handling cleaner and more readable. For new Spring Boot 3.2+ applications, Spring recommends using `RestClient`, while `WebClient` is the preferred choice for reactive, non-blocking applications.

## Common interview methods (the ones you'll use most)

```
.get()
.post()
.put()
.delete()
.patch()

.uri()

.header()
.headers()

.contentType()
.accept()

.body()

.retrieve()

.body(Class)
.toEntity()
.toBodilessEntity()

.onStatus()

.exchange()

.baseUrl()
.defaultHeader()
.build()
```

In practice, **90% of your code** will look like this:

```
User user = restClient
        .get()
        .uri("/users/{id}", id)
        .accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .body(User.class);
```

or

```
Order created = restClient
        .post()
        .uri("/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .body(orderRequest)
        .retrieve()
        .body(Order.class);
```

These patterns cover the vast majority of REST calls in typical Spring Boot applications.

### Why do we use `exchange()` in restTemplate

## 1. Need to send custom headers (Most Common)

The convenience methods don't let you pass headers.

### Doesn't work

```
restTemplate.getForObject(
    url,
    User.class
);
```

What if the API requires:

- Authorization Bearer Token
- API Key
- Custom Headers

You need `exchange()`.

```
HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth(token);

HttpEntity<?> entity = new HttpEntity<>(headers);

ResponseEntity<User> response =
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        entity,
        User.class
    );
```

---

## 2. Need the HTTP status code

`getForObject()` returns only the body.

```
User user = restTemplate.getForObject(url, User.class);
```

Suppose you want

```
Status : 201

Headers : Location

Body : User
```

Need

```
ResponseEntity<User> response =
        restTemplate.exchange(...);
```

Now you can access

```
response.getStatusCode();

response.getHeaders();

response.getBody();
```

---

## 3. Need Generic Types

Suppose API returns

```
[
   { ... },
   { ... }
]
```

If you write

```
List<User> users =
    restTemplate.getForObject(url, List.class);
```

You actually get

```
List<LinkedHashMap>
```

To preserve generic type information:

```
ResponseEntity<List<User>> response =
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        null,
        new ParameterizedTypeReference<List<User>>() {}
    );
```

---

## 4. Using HTTP methods without convenience methods

`RestTemplate` has helpers for common operations, but for arbitrary or less common methods (or when you want one consistent API), you use:

```
restTemplate.exchange(
    url,
    HttpMethod.PATCH,
    entity,
    User.class
);
```

or

```
HttpMethod.OPTIONS

HttpMethod.HEAD
```

---

## 5. Sending headers with POST/PUT/PATCH

Suppose you need

- Content-Type
- Authorization
- Body

```
HttpHeaders headers = new HttpHeaders();

headers.setContentType(MediaType.APPLICATION_JSON);

headers.setBearerAuth(token);

HttpEntity<User> entity =
        new HttpEntity<>(user, headers);

restTemplate.exchange(
        url,
        HttpMethod.POST,
        entity,
        User.class
);
```

---

## 6. Need conditional requests

Headers like

```
If-None-Match

If-Modified-Since

ETag
```

must go in the request.

Again

```
exchange()
```

---

## 7. Downloading files

Suppose you're downloading a PDF.

You need

```
ResponseEntity<byte[]> response =
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        entity,
        byte[].class
    );
```

Now you can inspect

- Content-Type
- Content-Length
- File Name
- Body

---

## 8. Multipart requests

Uploading files generally requires

```
HttpEntity<MultiValueMap<String,Object>>
```

which is used with

```
exchange()
```

---

## 9. Need complete control

Sometimes you simply want everything.

```
Request

↓

Headers

↓

Body

↓

Method

↓

Response Headers

↓

Status Code

↓

Response Body
```

`exchange()` exposes all of these.