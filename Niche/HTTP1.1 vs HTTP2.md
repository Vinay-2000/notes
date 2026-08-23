**HTTP/1.1** (released in 1997) was designed for simple HTML documents and text files. As modern web apps grew to require hundreds of assets per page, HTTP/1.1 hit a performance wall.

**HTTP/2** (released in 2015) completely overhauled how data is framed and transported across TCP connections to fix these bottlenecks.

## Visualizing the Main Difference: Connection Handling
![[Http2 vs Http1.png]]

## 1. Multiplexing vs. Head-of-Line (HoL) Blocking

- **HTTP/1.1 (Sequential Processing):**
    A single TCP connection can only handle **one request/response cycle at a time**. If a client needs 50 assets (JS, CSS, images), it must wait for Request 1 to complete before sending Request 2 on that connection.
    - _Workaround:_ Browsers open up to **6 parallel TCP connections** per domain, but each connection carries the overhead of a TCP 3-way handshake and TLS negotiation.
    - _The Problem:_ If a heavy request gets stuck, all subsequent requests behind it on that connection are blocked (**Application Head-of-Line Blocking**).
- **HTTP/2 (True Multiplexing):**
    Uses a **single TCP connection** to send and receive **multiple requests simultaneously**. Requests and responses are chopped into tiny binary frames, tagged with a stream ID, and sent intermingled over the wire. The receiving end reassembles them instantly based on their tags.

## 2. Text Format vs. Binary Framing

- **HTTP/1.1 (Plain Text):**
    Transfers data as uncompressed plain text (ASCII). Parsing raw text strings requires complex CPU processing and is prone to formatting/whitespace errors.
- **HTTP/2 (Binary Layer):**
    Breaks all communication into a **Binary Framing Layer**. Headers, data payloads, and stream settings are converted into compact binary `0`s and `1`s, making parsing simpler, more reliable, and much faster for the CPU.

## 3. Header Overhead (Uncompressed vs. HPACK)

- **HTTP/1.1:**
    Sends headers (User-Agent, Cookies, Accept-Encoding) as plain text with **every single request**. Over 50–100 requests, this sends kilobytes of redundant text over the network.
- **HTTP/2 (HPACK Compression):**
    Maintains a stateful index table between the client and server. If the headers don't change between requests, HTTP/2 sends only the **index values** (a few bytes) instead of re-transmitting full header strings.

## 4. Resource Delivery (Polling vs. Server Push)

- **HTTP/1.1:**
    Strictly client-initiated. The browser fetches `index.html`, parses it, discovers a CSS file reference, and sends a second request for `style.css`.
- **HTTP/2 (Server Push):**
    Allows the server to **proactively push resources** to the client's cache before the client even requests them. When `index.html` is requested, the server can push `style.css` along with it in anticipation.

## Direct Architectural Comparison

| **Feature**               | **HTTP/1.1**                           | **HTTP/2**                               |
| ------------------------- | -------------------------------------- | ---------------------------------------- |
| **Protocol Format**       | Text-based                             | Binary Framing Layer                     |
| **TCP Connections**       | Multiple connections (Max ~6 per host) | **Single connection** per origin         |
| **Multiplexing**          | ❌ No (Sequential pipeline)            | ✅ Yes (Asynchronous streams)            |
| **Head-of-Line Blocking** | 🔴 High (At request level)             | 🟡 Lower (Solved at application level)\* |
| **Header Handling**       | Uncompressed text                      | **HPACK** compressed binary              |
| **Server Push**           | ❌ Not supported                       | ✅ Supported                             |

> _\*Note on HTTP/2 TCP HoL Blocking:_ While HTTP/2 solves application-level blocking, a lost TCP packet still pauses all streams on that connection while TCP retransmits the missing packet. This inspired **HTTP/3 (QUIC)**, which runs over **UDP** to eliminate TCP-level blocking entirely!


HTTP/3 over QUIC solves Head-of-Line (HoL) blocking by **replacing TCP with UDP** and moving stream management directly down to the transport layer.

To understand how HTTP/3 achieves this, we need to compare how TCP and QUIC handle lost data packets.

## The Problem: TCP's Transport-Layer HoL Blocking (HTTP/2)

While HTTP/2 solved application-level HoL blocking (allowing multiple requests to share a single connection), it remained vulnerable to **TCP-level HoL blocking**.

1. **TCP Knows Nothing About Streams:** TCP treats all data passing through a connection as a **single, continuous byte stream**. It doesn't know that the stream contains three separate HTTP requests (`styles.css`, `app.js`, `logo.png`).
    
2. **Strict In-Order Delivery:** TCP guarantees that bytes are delivered to the receiving application in the exact order they were sent.
    
3. **The Bottleneck:** If a packet carrying part of Stream A (`styles.css`) is lost over a shaky Wi-Fi or cellular network, TCP **stalls the entire connection buffer**. Even if packets for Stream B (`app.js`) and Stream C (`logo.png`) arrive perfectly, the operating system holds them back until the missing packet for Stream A is retransmitted and received.
    

> **Result in HTTP/2:** A single dropped packet halts **all** multiplexed streams on that TCP connection.

## The Solution: How HTTP/3 + QUIC Eliminates the Bottleneck

HTTP/3 runs on **QUIC**, an underlying transport protocol built on top of **UDP** (User Datagram Protocol). UDP does not enforce connection state or strict byte ordering, giving QUIC full control over reliability.

### 1. Independent Per-Stream Delivery

In QUIC, streams are first-class citizens at the transport layer. QUIC assigns sequence numbers and flow control **to individual streams**, rather than the entire connection.

- If a packet for Stream A is lost, **only Stream A is paused** to wait for retransmission.
    
- The OS immediately hands the packets for Stream B and Stream C over to the browser or application without delay.
    

### 2. Independent Encryption Frames

HTTP/2 uses TLS over TCP, where TLS frames span across the entire TCP stream. A dropped TCP packet breaks TLS decryption for subsequent data.

QUIC integrates TLS 1.3 directly into its packet structure. Each QUIC packet is independently encrypted and decrypted, allowing the receiver to unpack valid stream packets even if neighboring packets are missing.

## Summary Comparison

| **Feature**                           | **HTTP/2 (over TCP)**                    | **HTTP/3 (over QUIC/UDP)**                            |
| ------------------------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **Transport Protocol**                | TCP                                      | UDP + QUIC                                            |
| **Stream Awareness**                  | Application layer only (HTTP layer)      | **Transport layer** (Built into QUIC)                 |
| **Impact of Dropped Packet**          | Stalls **ALL** streams on the connection | Stalls **ONLY** the stream that lost the packet       |
| **Connection Handshake**              | 2–3 RTTs (TCP Handshake + TLS)           | **0-RTT to 1-RTT** (Combined Transport + TLS 1.3)     |
| **Network Switching (Wi-Fi ↔ 4G/5G)** | Breaks TCP connection (IP changes)       | **Smooth connection migration** (Uses Connection IDs) |