Performance & Metrics Pack
==================

Topics on this file : **Request, Response, Latency, Throughput**

## 1. Request "The Ask"
A **request** is the message a client sends to a server to ask for something — like fetching data, submitting a form, or calling an API.

### What matters:
- **Request type**: (GET, POST, etc.)
- **Request size**:  large payloads slow things down.
- **Headers & authentication**: add processing cost. 
- **Network path**: every hop (DNS, proxy, etc.) adds time. 

### As an engineer, always ask:
"How many requests are we making? How big are they? Are they necessary?"

Reducing unnecessary requests (batching, caching, compression) is one of the easiest ways to improve performance. 

## 2. Response "The Answer"
A **response** is the server's reply. It includes:
- **Status code** (200, 400, 500, etc.)
- **Headers** (metadata — like content type or cache rules)
- **Body** (the actual data)

### What engineers focus on:
- **Response size**: affects download time.
- **Response time**: total delay before client receives data.
- **Cacheability**: can this response be reused later?

Optimizing response payloads (e.g., only sending needed fields, using compression like gzip or brotli) has a direct impact on perceived speed.

## 3. Latency "How Long It Takes"
**Latency** is the **delay** between when a request is sent and when a response starts coming back.\
It's measured in **milliseconds (ms)** and is the key number that defines how "fast" a system feels.

### Main components of latency:
1. **Network latency**: time it takes data to travel between client and server.
2. **Processing latency**: time the server spends handling the request.
3. **Queueing latency**: time spent waiting if the server is busy.

### You should:
- Understand where latency comes from (network, server database).
- You should know how to measure it (monitoring tools like New Relic, Datadog, or logs).
- You should know how to reduce it:
    - Caching responses
    - Using CDNs
    - Placing servers closer to user (geo-distribution)
    - Reducing request dependencies (less round trips)

### Analogy:
Latency is like the time between pressing a button and seeing the result — even if the action is correct, high latency feels "laggy".

## 4. Throughput "How Much We Can Handle"
**Throughput** measures **how many requests** a system can handle in a given time — usally requests per second (RPS) or transactions per second (TPS).

### In simple terms:
Latency = how long one request takes.\
Throughput = how many you can handle at once.

### Example:
- One server handles 100 requests/second — throughput = 100 RPS.
- If each request takes 100ms — latency = 100ms.

### As a system grows, you need to:
- **Scale up** (make one server faster)
- **Scale out** (add more servers behind a load balancer)

### Engineering takeaway:
- High latency — slow experience.
- Low throughput — bottlenecks under load.

Both must be monitored and balanced.

### Rule of Thumb:
If you reduce latency, throughput usually increases — but not always.\
Some optimizations (like caching) improve both, others (like encryption) may trade a little latency for better security.