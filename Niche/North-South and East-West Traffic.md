The terms **North-South** and **East-West** describe the **directional flow of data packets** relative to a system boundary (like a data center, Kubernetes cluster, or private cloud VPC).

Understanding this distinction is critical because each flow requires entirely different network protocols, security models, and performance optimizations.

## The Directional mental model

Plaintext

```
               EXTERNAL CLIENTS / PUBLIC INTERNET
                                │
                                │  ▲
                   North-South  │  │ (Ingress / Egress)
                        Traffic │  │
                                ▼  │
         ┌──────────────────────────────────────────┐
         │              API GATEWAY                 │
         └────────────────────┬─────────────────────┘
                              │
  ┌───────────────────────────┼───────────────────────────┐
  │                           │                           │
  │   ┌────────────────┐      │      ┌────────────────┐   │
  │   │ Auth Service   │◄────►│◄────►│ Order Service  │   │
  │   └───────┬────────┘ East-West   └───────┬────────┘   │  PRIVATE
  │           │           Traffic            │            │ DATA CENTER
  │           ▼                              ▼            │  / CLUSTER
  │   ┌────────────────┐             ┌────────────────┐   │
  │   │  User Database │             │ Payment Broker │   │
  │   └────────────────┘             └────────────────┘   │
  └───────────────────────────────────────────────────────┘
```

## ↕️ North-South Traffic (In & Out)

**Definition:** Data moving **between the external world and the internal infrastructure** (Ingress from external clients into the data center, or Egress from internal services out to third-party APIs like Stripe or AWS S3).

- **Primary Focus:** **Security Perimeter, Authentication, & Protocol Adaptability.**
    
- **Typical Entry Point:** Edge Routers, Load Balancers, WAFs (Web Application Firewalls), API Gateways, and Ingress Controllers.
    
- **Network Environment:** Public internet or WAN—unpredictable, lossy, high-latency, and untrusted.
    
- **Optimal Protocols:**
    
    - **QUIC / HTTP/3:** Ideal for mobile and web clients due to 0-RTT handshakes and connection migration over lossy networks.
        
    - **REST / GraphQL / JSON over HTTPS:** Universally accessible for public-facing consumers.
        

## ↔️ East-West Traffic (Side-to-Side)

**Definition:** Data moving **internally between servers, containers, or microservices** inside the same data center, cloud region, or Kubernetes cluster.

In modern microservice architectures, **East-West traffic accounts for 70% to 80%+ of all network volume** in a system. A single North-South request from a user can trigger dozens of internal East-West RPC calls between microservices, caches, and databases.

- **Primary Focus:** **Ultra-low Latency, High Throughput, & Zero-Trust Security.**
    
- **Typical Routers/Components:** Service Meshes (Istio, Linkerd), Internal L4 Load Balancers, Sidecar Proxies (Envoy).
    
- **Network Environment:** Private VPC / High-Speed LAN—highly reliable, zero packet loss, high-bandwidth (10G/100G+), and low-latency.
    
- **Optimal Protocols:**
    
    - **gRPC / HTTP/2 over TCP:** Binary Protobuf framing minimizes CPU overhead for high-frequency internal calls.
        
    - **Unix Domain Sockets / Shared Memory:** For co-located sidecar communication.
        

## Direct Architectural Comparison

|**Dimension**|**North-South Traffic**|**East-West Traffic**|
|---|---|---|
|**Direction**|Client $\leftrightarrow$ Internal System|Internal Service $\leftrightarrow$ Internal Service|
|**Volume Share**|~15% to 30% of total network bytes|**~70% to 85% of total network bytes**|
|**Network Quality**|Untrusted, high-latency, lossy internet|Trusted LAN / VPC, low-latency, zero-loss|
|**Security Model**|Perimeter Security (WAF, TLS, OAuth)|**Zero Trust** (mTLS, Service-to-Service IAM)|
|**Primary Bottleneck**|Network Latency & Handshakes|**CPU Utilization & Serialization Overhead**|
|**Ideal Protocols**|QUIC (HTTP/3), REST, GraphQL|**gRPC (HTTP/2), Kafka, Thrift**|
|**Routing Component**|API Gateway / Ingress Controller|Service Mesh / Internal L4 Load Balancers|

## Key Security Trade-off: Zero Trust

Historically, organizations treated North-South as untrusted and East-West as completely trusted ("hard shell, soft interior").

If an attacker breached the North-South perimeter (e.g., via an unpatched API flaw), they could move laterally without resistance through the internal East-West network. Modern architectures enforce **Zero Trust Security** internally using **mTLS (Mutual TLS)** and service meshes to encrypt and authenticate all East-West microservice traffic.