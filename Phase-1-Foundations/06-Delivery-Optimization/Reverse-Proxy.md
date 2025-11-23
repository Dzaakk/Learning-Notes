Reverse Proxy
======

## Reverse Proxy — The Traffic Manager
A **reverse proxy** is a server that sits **in front of one or moce backend servers** and handles incoming client requests before they reach the actual servers.

### Think of it as a receptionist:
Client never talk directly to the backend — they talk to the receptionist (the proxy), who decides *which* server should handle each request.

### What It Does
1. Load Balancing
    - Distributes incoming traffic among multiple servers (using round robin, least connections, etc.).
    - Prevents overload on a single machine.

2. Security
    - Hides backend server IPs -> attackers can't target them directly.
    - Can enforce HTTPS, authentication, and firewall rules.

3. Caching
    - stores frequent response (like static files or API results) so it doesn't need to hit the backend every time.

4. Compression & Optimization
    - Compress responses, manages headers, or even rewrites URLs before sending data to clients.

5. SSL Termination
    - Handles encryption/decryption at the proxy layer, so backend servers can focus on logic, not security handshakes.

