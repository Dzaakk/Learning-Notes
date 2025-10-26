Delivery Optimization Pack
==================

Topics on this file : **Reverse Proxy, CDN**

## 1. The Big Idea
When a user in Indonesia visit a website hosted in the U.S., every request has to cross oceans — that's **latency**.\
When millions of users visit at once, servers get **overloaded** — that's a **scalability** problem.

**Reverse Proxies** and **CDNs** exist to solve those two problems.\
They're the middlemen that **stand between clients and servers**, optimizing how requests are handled and responses are delivered.

## 2. Reverse Proxy — The Traffic Manager
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

## 3. CDN (Content Delivery Network) — The Global Speed Booster
A **CDN** is a network of servers distributed around the world that stores copies of your content closers to users.

When someone requests a file (like an image, video, or JS file), the CDN serves it from the nearest serveer instead of the origin.

### How It Works
1. A user request `example.com/image.jpg`.
2. The DNS routes them to the **nearest CDN edge server**.
3. If the edge server already has the file (cached), it returns it immediately.
4. If not, it fetches it once from the **origin server**, stores it, and reuses it for future users nearby.

### Result:
- Lower latency (shorter physical distance).
- Reduce load on origin servers.
- Better reliability and redundancy.

### What CDNs Can Deliver
- Static assets (HTML, JS, CSS, images, videos).
- Dynamic APIs (via edge caching or compute).
- Even full websites (using edge servers like Cloudflare, Akamai, Fastly)

### Benfits of CDNs
- **Speed**: milliseconds instead of seconds, especially for global users.
- **Scalability**: handles traffic spikes automatically.
- **Resilience**: if one server fails, others take over.
- **Security**: CDNs block DDoS attacks before they reach your origin.