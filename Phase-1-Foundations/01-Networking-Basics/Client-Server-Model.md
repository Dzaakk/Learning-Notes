The Client-Server Model
======
Internet runs on a simple pattern: **a client asks, a server answers**

- **Client** = the requester (e.g. your phone, browser, or app).
- **Server** = the provider (the machine hosting the data or service).

## Flow
1. Client sends a request.
2. Server processes that request.
3. Server sends a response back.

## Analogy
Think of it like ordering food:
- **Client** = you (the customer).
- **Server** = The restaurant kitchen.
- **Request** = your order.
- **Response** = your food.

## Visual Diagram
![request-response](./images/request-response.excalidraw.png)

## Real-World Examples
- **Netflix**: Your app  (client) requests movie data from Netflix servers.
- **WhatsApp**: Your phone (client) sends messages to WhatsApp's servers, which forward them to your friend's phone.
- **Google Search**: Browser (client) sends search query to Google servers, gets search results back

## Trade-offs
|Aspect|Centralized (Traditional)|Decentralized (P2P)|
|:------|:------|:------|
|Control|Server has full control|No single authority|
|Scalability|Need to scale servers|Scales with users|
|Single Point of Failure|Server down = service down|More resilient|
|Examples|Instagram, Twitter|BitTorrent, Blockchain|