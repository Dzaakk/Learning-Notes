Status Codes
======

## HTTP Status Codes
Status codes are **3 digit numbers** that the server sends to the client to indicate "how did your request go?".
- First digit = category (2xx, 3xx, 4xx, 5xx).
- Last two digits = specific detail.

### 2xx - Success (Request Successful)
#### 200 OK
- Request succeeded, this is the most common response.
- GET successfully retrieved data, POST successfully submitted data.
- *When to use*: Default success response.
#### 201 Created
- New resource was successfully created.
- *When to use*: After POST that creates a new user, article, etc.
- *Best practice*: Include `Location` header with the new resource's URL.
#### 204 No Content
- Request succeeded, but server sends no data back.
- *When to use*: Successful DELETE, or PUT/PATCH that doesn't need to return data.
- *Why*: Saves bandwidth.
---
### 3xx - Redirection (Moved Location)
#### 301 Moved Permanently
- Resource permanently moved to new URL.
- *When to use*: Wesite changed domain, old URL deprecated.
- *impact*: Browser will cache this redirect, future requests go directly to new URL.
#### 302 Found (Temporary Redirect)
- Resource temporarily at different location.
- *When to use*: Maintenance mode, A/B testing.
- *Impact*: Browser doesn't cache, still checks original URL next time.
#### 304 Not Modified
- Resource hasn't changed since client's last request.
- *When to use*: Browser ask "any updates?" with `If-None-Match` or `If-Modified-Since` header.
- *Why*: CACHING - browser can use cached version, no need to download again.
---
### 4xx - Client Error (Your Fault!)
#### 400 Bad Request
- Request has wrong format or missing required field.
- *Example*: Malformed JSON, missing parameter.
- *Best practice*: Provide clear error message.
#### 401 Unauthorized
- Not logged in or token invalid/expired.
- *When to use*: User hasn't authenticated.
- *Response*: Usually frontend redirects to login page.
#### 403 Forbidden
- Logged in, but doesn't have access.
- *Example*: Regular user tries to access admin panel.
- *Difference from 401*: 401 = "who are you?", 403 = "I know who you are, but you can't do this".
#### 404 Not Found
- Resource doesn't exist.
- *Example*: `/users/9999` but that user ID doesn't exists.
- *Engineering note*: Don't expose internal errors in 404 (security).
#### 429 Too Many Requests
- Client sent too many requests (rate limit exceeded).
- *Best practice*: Include `Retry-After` header (in seconds).
- *Use case*: API rate limiting, DDoS protection.
---
### 5xx - Server Error (Our Fault!)
#### 500 Internal Server Error
- Server error, something went wrong in backend.
- *Example*: Uncaught exception, database crash.
- *Best practice*: DON'T expose stack trace to client (security risk).
#### 502 Bad Gateway
- Server acting as gateway/proxy, but got invalid response from upstream server.
- *Example*: Load balancer can't reach backend server.
- *Common scenario*: Deployment error, backend server down.
#### 503 Service Unavailable
- Server temporarily unavailable (maintenance, overload).
- *Best practice*: Include `Retry-After` header.
- *Use case*: Planned maintenance, server overload.
#### 504 Gateway Timeout
- Gateway/proxy timeout waiting for upstream server.
- *Example*: Database query took too long (>30s).
- *Engineering note*: This hints to optimize query or increase timeout.

### Engineering Tips:
1. **Always handle errors properly** - don't jsut assume 200.
2. **Use specific status code** - don't make everything 200 or 500.
3. **Log 5xx errors** - these are bugs that need fixing.
4. **Monitor 4xx rates** - could indicate API design issues if too high.
5. **Use 429 for rate limiting** - better than blocking completely.

