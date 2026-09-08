https://docs.spring.io/spring-ai/reference/api/mcp/mcp-overview.html
> Spring AI 2.0.1 | Spring Boot | MCP Inspector

## 1. What is MCP?

**MCP (Model Context Protocol)** standardizes how an AI application/client interacts with external capabilities and context.

An MCP server can expose:
- **Tools** — executable capabilities/actions
- **Resources** — information/context
- **Prompts** — reusable instruction templates
- **Completions** — argument/autocomplete suggestions

```text
AI Client / Host
       |
       | MCP
       v
MCP Server
   |       |        |          |
 Tools  Resources  Prompts  Completions
   |       |        |          |
 APIs     DB/files Templates  Suggestions
```

**MCP is not an LLM.** The MCP server normally exposes capabilities/data that an AI client can use.

---

## 2. MCP Architecture

```text
MCP
 |
 | application-level protocol
 v
JSON-RPC
 |
 | message/RPC mechanism
 v
Transport
 |----------------------------|
 HTTP / Streamable HTTP       STDIO
 |
 v
TLS (when HTTPS)
 |
 v
TCP
```

- **MCP** = application protocol
- **JSON-RPC** = message/RPC mechanism
- **HTTP / STDIO** = transport
- **HTTPS** = HTTP + TLS

### MCP is NOT HTTP/2

Streamable HTTP does not mean HTTP/2. HTTP/1.1 vs HTTP/2 is an underlying HTTP concern.

---

## 3. Spring AI MCP Server

For a Spring Boot WebMVC MCP server:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server-webmvc</artifactId>
</dependency>
```

Spring AI provides annotations that turn Java methods into MCP capabilities. The annotation scanner is enabled by default.

---

## 4. MCP Tool

A **Tool** represents an executable capability/action.

```java
@McpTool(description = "Adds two numbers")
public int add(int a, int b) {
    return a + b;
}
```

A Tool does **not** have to modify data. A read-only operation can also be a Tool.

Possible implementations:
- Database
- REST API
- Redis
- Kafka
- File system
- External API
- Another microservice

> **Tool = executable capability/action**

---

## 5. Tool Metadata / Hints

```java
@McpTool(
    title = "ADD",
    description = "Adds two numbers",
    annotations = @McpTool.McpAnnotations(
        readOnlyHint = true,
        destructiveHint = false,
        idempotentHint = true,
        openWorldHint = false
    )
)
public int add(int a, int b) {
    return a + b;
}
```

| Hint | Meaning |
|---|---|
| `readOnlyHint` | Tool is intended to only read/observe |
| `destructiveHint` | Tool may destroy/change existing data |
| `idempotentHint` | Repeating the operation has the same intended effect |
| `openWorldHint` | Tool interacts with the outside/open world |

**Critical:** these are hints/metadata, not security controls.

```java
destructiveHint = false
```

does NOT prevent the method from deleting data. Real authorization must be implemented separately.

Typical CRUD hints:

```text
GET     -> readOnly=true,  destructive=false, idempotent=true
UPDATE  -> readOnly=false, destructive=false, idempotent=true
DELETE  -> readOnly=false, destructive=true,  idempotent=true
```

---

## 6. Idempotency

An operation is idempotent when repeating it has the same intended final effect as doing it once.

Common examples:

```text
PUT /users/10
DELETE /users/10
```

---

## 7. MCP Resource

A **Resource** represents information/context that an MCP client can retrieve.

```java
@McpResource(
    uri = "demo://user",
    name = "Demo User",
    description = "Provides demo user information"
)
public String getUser() {
    return "...";
}
```

A Resource can retrieve information from:

```text
Resource
   |
   +--> Database
   +--> REST API
   +--> Redis
   +--> File
   +--> Configuration
   +--> External service
```

> **Resource = information/context**

> **Tool = executable capability**

Both can internally interact with a database/API.

---

## 8. Resource Template

A Resource Template is a parameterized resource URI.

```java
@McpResource(
    uri = "demo://user/{id}",
    name = "User",
    description = "Provides information about a user"
)
public String getUserById(String id) {
    return "User information for ID: " + id;
}
```

Possible URIs:

```text
demo://user/101
demo://user/102
demo://user/103
```

In this Spring AI version, use `@McpResource(uri = ".../{id}")`; there is no separate `@McpResourceTemplate` annotation in the API used here.

---

## 9. MCP Prompt

A Prompt is a **reusable instruction/template**. It does **not** itself call an LLM.

```java
@McpPrompt(
    name = "analyzeStock",
    description = "Analyze a stock"
)
public GetPromptResult analyzeStock(
        @McpArg(
            name = "symbol",
            description = "Stock symbol",
            required = true
        )
        String symbol) {
    // build and return PromptMessage
}
```

If the client requests `analyzeStock("AAPL")`, the server can return a prepared instruction. The client/LLM can then use it.

> **Prompt = reusable instruction template, not an LLM call.**

---

## 10. @McpArg

In Spring AI 2.0.1, prompt arguments use:

```java
@McpArg(
    name = "symbol",
    description = "Stock symbol",
    required = true
)
String symbol
```

Do not use `@McpPromptArgument`; that is not part of the API used here.

---

## 11. Prompt vs Tool vs Resource vs Completion

| MCP primitive | Purpose | Example |
|---|---|---|
| Tool | Execute an operation | `getStockPrice("AAPL")` |
| Resource | Retrieve information/context | `stock://AAPL/profile` |
| Prompt | Provide reusable instructions | `analyzeStock("AAPL")` |
| Completion | Suggest argument values | `AAPL`, `AMZN`, `AMD` |

Simple mental model:

```text
Tool       = DO something
Resource   = GET information
Prompt     = TELL the AI how to approach something
Completion = SUGGEST valid input
```

---

## 12. Completion

Completion provides suggestions for arguments.

```java
@McpComplete(prompt = "analyzeStock")
public List<String> completeStockSymbol(String value) {
    return List.of("AAPL", "AMZN", "AMD", "ADBE",
                   "GOOGL", "GOOG", "MSFT", "META")
        .stream()
        .filter(s -> s.startsWith(value.toUpperCase()))
        .toList();
}
```

Input:

```text
AM
```

Possible result:

```text
AMZN
AMD
```

In Spring AI 2.0.1, `@McpComplete` has `prompt` and `uri`; it does not have an `argument` attribute.

---

## 13. MCP Server Metadata

```properties
spring.ai.mcp.server.name=demo-mcp-server
spring.ai.mcp.server.version=1.0.0
spring.ai.mcp.server.instructions=This is my Spring AI MCP learning server.
```

The client/Inspector can see server name, version and instructions.

---

## 14. MCP Capabilities

```properties
spring.ai.mcp.server.capabilities.tool=true
spring.ai.mcp.server.capabilities.resource=true
spring.ai.mcp.server.capabilities.prompt=true
spring.ai.mcp.server.capabilities.completion=true
```

If:

```properties
spring.ai.mcp.server.capabilities.tool=false
```

Tools are not exposed while other enabled capabilities can remain available.

> **Capabilities** describe what the server supports/exposes.

> **Tool annotations** describe individual tools.

---

## 15. MCP Inspector

MCP Inspector lets you test an MCP server without connecting an actual AI application.

It can inspect/test:

```text
Tools
Resources
Resource Templates
Prompts
Completions
Server metadata
Capabilities
```

For the WebMVC Streamable HTTP setup:

```text
http://localhost:8080/mcp
```

---

## 16. /mcp Is Not a Normal REST Endpoint

Although it looks like:

```text
http://localhost:8080/mcp
```

it is not a conventional REST endpoint such as:

```http
GET /users/10
POST /orders
PUT /users/10
```

`/mcp` is the entry point for the MCP protocol. The client sends MCP/JSON-RPC messages and the server responds according to MCP semantics.

---

## 17. Streamable HTTP

Streamable HTTP is the modern/recommended HTTP transport for MCP.

Spring AI WebMVC configuration:

```properties
spring.ai.mcp.server.protocol=STREAMABLE
```

The MCP endpoint is typically:

```text
/mcp
```

Streamable HTTP does not mean one permanent WebSocket-like connection. It supports normal request/response communication and streaming when needed.

Also:

```text
Streamable HTTP != HTTP/2
```

---

## 18. SSE Transport

SSE = **Server-Sent Events**.

It is an HTTP mechanism for streaming events from server to client.

Older MCP setups commonly used:

```text
/sse
/messages
```

SSE was an older MCP HTTP transport. For Spring AI WebMVC 2.x, Streamable HTTP is preferred.

---

## 19. Streamable HTTP vs SSE

| | Streamable HTTP | SSE |
|---|---|---|
| Modern MCP transport | Yes | Older |
| Spring AI 2.x direction | Recommended | Deprecated for WebMVC |
| Typical endpoint | `/mcp` | `/sse` + message endpoint |
| Streaming | Supported | Server → client |
| New projects | Prefer this | Usually avoid |

---

## 20. STDIO Transport

STDIO means **Standard Input / Standard Output**.

Instead of HTTP:

```text
MCP Client
    |
    | stdin/stdout
    v
MCP Server process
```

There is no HTTP endpoint such as `/mcp` when using STDIO.

The MCP client normally launches the MCP server as a local process.

STDIO is particularly useful for local MCP servers.

---

## 21. STDIO Configuration

For the dedicated Spring AI MCP server starter:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server</artifactId>
</dependency>
```

Enable STDIO:

```properties
spring.ai.mcp.server.stdio=true
```

For the WebMVC setup, protocol values are HTTP-oriented, such as `SSE`, `STREAMABLE` and `STATELESS`.

---

## 22. STDIO and Console Output

With STDIO, standard input/output carries the MCP protocol.

Avoid:

```java
System.out.println("Debug message");
```

because stdout can contain MCP protocol messages and interfere with the protocol stream.

Use logging configured appropriately instead.

---

## 23. HTTP vs STDIO

| | Streamable HTTP | STDIO |
|---|---|---|
| Transport | HTTP | stdin/stdout |
| Network | Yes | No |
| URL | `/mcp` | None |
| Typical use | Remote/network MCP | Local process MCP |
| TLS | Possible with HTTPS | Not applicable to transport |
| Client launches process | Usually no | Usually yes |

---

## 24. HTTPS and TLS

When using HTTPS, TLS provides:

- Encryption
- Server authentication
- Protection against network interception/tampering

But:

> **TLS does NOT automatically authorize a user to call tools.**

Separate concepts:

```text
HTTPS/TLS
    -> secures communication

Authentication
    -> identifies the client/user

Authorization
    -> determines what the client/user can do
```

---

## 25. Local vs Remote MCP

Local development:

```text
http://localhost:8080/mcp
```

is fine for testing.

For an internet-facing MCP server, consider:

- HTTPS/TLS
- Authentication
- Authorization
- Network security
- Tool permissions
- Input validation
- Rate limiting
- Audit logging

Do not assume MCP automatically secures the application.

---

## 26. MCP Client / Host / Server

```text
              AI Application / Host
                       |
                 MCP Client
                       |
                MCP protocol
                       |
                 MCP Server
            _________|_________
           |         |         |
         Tools    Resources  Prompts
           |         |         |
          APIs       DB      Templates
```

The MCP server exposes capabilities. The AI application/client decides when and how to use them.

---

## 27. Prompt Does Not Generate the Final Answer

Suppose the server has:

```text
analyzeStock("AAPL")
```

The Prompt can return detailed instructions. The Prompt itself does not magically produce the final answer; the client/LLM can use those instructions.

Similarly:

```text
Prompt
   -> produces detailed image-generation instructions

Image-generation Tool
   -> actually performs/calls image generation
```

---

## 28. Strong Interview Answers

### Tool vs Resource

> A Tool represents an executable capability that the model/client can invoke, while a Resource represents information or context that the client can retrieve. Both can internally interact with databases or APIs. The distinction is about how the capability is exposed through MCP, not whether it reads or writes a database.

### Tool Metadata

> MCP tool annotations such as read-only or destructive hints are metadata for the client. They help the client reason about the tool, but they don't enforce permissions. Real authorization must be implemented using application security.

### MCP Transport

> MCP can communicate over transports such as STDIO and HTTP-based transports. STDIO is commonly used when the client launches a local MCP server process, while Streamable HTTP is suitable for HTTP/network-based servers. SSE was an older HTTP transport and Streamable HTTP is preferred in modern Spring AI implementations.

---

## 29. MCP vs REST

### REST

```text
GET  /users/10
POST /orders
PUT  /users/10
DELETE /users/10
```

The API contract is application-specific.

### MCP

```text
MCP Client
    |
    v
MCP Server
    |
    +-- tools/list
    +-- tools/call
    +-- resources/list
    +-- resources/read
    +-- prompts/list
    +-- prompts/get
```

MCP standardizes discovery and interaction with AI-oriented capabilities.

---
## MCP Server Logging

With **Streamable HTTP**, normal application logs such as:

```
log.info("ADD tool called");
```

go to the Spring Boot console.

If you want the server to send a log message **through the MCP protocol to the connected client**, use `McpSyncRequestContext`.

```
@McpTool(description = "Adds two numbers")
public int add(McpSyncRequestContext context, int a, int b) {

    context.info("ADD tool called with a=" + a + ", b=" + b);

    int result = a + b;

    context.info("ADD tool calculated result=" + result);

    return result;
}
```

The flow is:

```
Tool execution
     |
     +--> log.info()      -> Application/console logs
     |
     +--> context.info()  -> MCP logging notification -> MCP Client/Inspector
```

### Important distinction

> `log.info()` = application logging.

> `context.info()` = MCP logging sent to the connected MCP client.

`McpSyncRequestContext` is a special parameter supplied by Spring AI when the MCP tool is invoked; it is not an argument that the client needs to provide.

---

## 30. Example: Trading / Finance MCP Server

### Tools

```text
getStockPrice(symbol)
placeOrder(symbol, quantity, side)
cancelOrder(orderId)
```

### Resources

```text
portfolio://current
stock://AAPL/profile
stock://AAPL/fundamentals
```

### Prompts

```text
analyzeStock(symbol)
reviewPortfolio()
```

### Completions

```text
"AAP" -> "AAPL"
"MS"  -> "MSFT"
```

This gives an AI client a standardized interface to a trading system.

---

## 31. Security Checklist

For production MCP servers, consider:

- Authentication
- Authorization
- Input validation
- Least-privilege tool access
- Rate limiting
- Audit logging
- HTTPS/TLS for network communication
- Protection against dangerous/destructive tools
- Protection of sensitive resources
- Validation of external data

Remember:

```text
destructiveHint=true
```

is **not** a security mechanism.

---

## 32. What We Built/Tested

### Tool

```java
@McpTool(description = "Adds two numbers")
public int add(int a, int b) {
    return a + b;
}
```

### Resource

```java
@McpResource(
    uri = "demo://user",
    name = "Demo User",
    description = "Provides demo user information"
)
public String getUser() {
    return "...";
}
```

### Resource Template

```java
@McpResource(
    uri = "demo://user/{id}",
    name = "User",
    description = "Provides information about a user"
)
public String getUserById(String id) {
    return "User information for ID: " + id;
}
```

### Prompt

```java
@McpPrompt(
    name = "analyzeStock",
    description = "Analyze a stock"
)
public GetPromptResult analyzeStock(
        @McpArg(
            name = "symbol",
            description = "Stock symbol",
            required = true
        )
        String symbol) {
    // return PromptMessage
}
```

### Completion

```java
@McpComplete(prompt = "analyzeStock")
public List<String> completeStockSymbol(String value) {
    // return suggestions
}
```

---

## 33. Last-Minute Interview Cheat Sheet

| Term | One-line answer |
|---|---|
| MCP | Standardized interface between an AI client and external capabilities/context |
| Tool | Executable capability/action |
| Resource | Retrievable information/context |
| Resource Template | Parameterized resource URI |
| Prompt | Reusable instruction/template; not an LLM call |
| Completion | Suggestions/autocomplete for arguments |
| JSON-RPC | Message/RPC mechanism used by MCP |
| Streamable HTTP | Modern HTTP transport supporting request/response and streaming |
| SSE | Older HTTP streaming transport |
| STDIO | Local process transport using stdin/stdout |
| Tool hints | Client-facing metadata, not security |
| Capabilities | Features the server exposes |
| `/mcp` | MCP protocol endpoint, not a normal REST endpoint |
| HTTPS | TLS-secured HTTP; does not provide authorization by itself |

---

## 34. One-Line Mental Model

```text
MCP = standardized interface between an AI client and external capabilities/context
```

Remember:

```text
Tool       = DO something
Resource   = GET information
Prompt     = TELL the AI how to approach something
Completion = SUGGEST valid input
Transport  = HOW MCP messages travel
```

---

## 35. Next Step

The next practical step is connecting the Spring Boot MCP server to an actual MCP client / AI application.

```text
Spring Boot MCP Server
        |
        | MCP
        v
Actual MCP Client / AI application
        |
        v
LLM
        |
        +--> discovers tools
        +--> decides when to call tools
        +--> reads resources
        +--> uses prompts
```

The goal is to move from:

```text
"I can expose an MCP tool"
```

to:

```text
"The AI can discover and actually use my MCP tool."
```
