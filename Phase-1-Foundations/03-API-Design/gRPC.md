gRPC (Google Remote Procedure Call)
======

## gRPC (Google Remote Procedure Call)
gRPC is a **moder, high-performance** alternative to REST, designed for **service to service communication** inside distributed systems.

### How It Works:
- Instead of HTTP/JSON, gRPC uses **HTTP/2** and **binary data (Protocol Buffers / protobuf)**.
- You define your API in a `.proto` file:
```proto
service UserService{
    rpc GetUser (UserRequest) returns (UserResponse);
}
```
- gRPC automatically generates client and server code in many languages.

### Key Concepts:
- **Strong typing**: APIs are schema-defined via `.proto` files.
- **Bidirectional streaming** : both client and server can send messages continuously.
- **Compact and fast**: binary format -> less bandwith, lower latency.

### Pros:
- Extremely fast (uses HTTP/2 and protobuf).
- Great for microservices and backend to backend calls.
- Auto generates SDKs for multiple languages.
### Cons:
- Harder to debug (binary, not human readable).
- Not natively supported by browsers.
- Requires more setup and tooling.

### Use it for:
- Microservices inside your infrastructure.
- High performance systems (streaming, IoT, or internal RPC calls).

