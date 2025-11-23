Request Response Cycle
======

## Request "The Ask"
A **request** is the message a client sends to a server to ask for something — like fetching data, submitting a form, or calling an API.

### What matters:
- **Request type**: (GET, POST, etc.)
- **Request size**:  large payloads slow things down.
- **Headers & authentication**: add processing cost. 
- **Network path**: every hop (DNS, proxy, etc.) adds time. 

### As an engineer, always ask:
"How many requests are we making? How big are they? Are they necessary?"

Reducing unnecessary requests (batching, caching, compression) is one of the easiest ways to improve performance. 

## Response "The Answer"
A **response** is the server's reply. It includes:
- **Status code** (200, 400, 500, etc.)
- **Headers** (metadata — like content type or cache rules)
- **Body** (the actual data)

### What engineers focus on:
- **Response size**: affects download time.
- **Response time**: total delay before client receives data.
- **Cacheability**: can this response be reused later?

Optimizing response payloads (e.g., only sending needed fields, using compression like gzip or brotli) has a direct impact on perceived speed.

