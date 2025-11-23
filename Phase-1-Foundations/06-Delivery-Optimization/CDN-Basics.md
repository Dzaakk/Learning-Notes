CDN (Content Delivery Network)
====


## CDN (Content Delivery Network) — The Global Speed Booster

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