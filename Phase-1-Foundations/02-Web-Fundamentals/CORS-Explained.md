CORS (Cross-Origin Resource Sharing)
======

### The Problem:
Browser have a security policy: **Same-Origin Policy**
- Origin = `protocol + domain + port`
- Example origins: 
    - `https://example.com:443`
    - `https://localhost:3000`
    - `https://api.example.com`

**Same-Origin Policy says**: JavaScript at `https://example.com` **CANNOT** make requests to `https://api.example.com`\
**why?** Protection from malicious scripts stealing data from other sites.

---
### The Solution: CORS
Servers can **explicitly allow certain origins to access their resources via **CORS headers**.\
**Server response with**:
```http
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Method: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type, Authorization
```
**Meaning**:
- `example.com` can access this API
- Can use GET, POST, PUT methods
- Can include Content-Type and Authorization headers
---

**Preflight Request (The OPTIONS Check)**\
For "complex" requests (non-simple), browser sends a **preflight request** first:

#### 1. Browser sends OPTION request:
```http
OPTIONS /api/users HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type
```
#### 2. Server replies:
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type
```
#### 3. If OK, browser then sends the actual request
---

### Simple vs Complex Requests

#### Simple requests (no preflight):
- Methods : GET, POST, HEAD.
- Headers: only Accept, Content-Type(limited), etc.
- Content-Type: `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`.

#### Complex requests (need preflight):
- Custom headers (like `Authorization`).
- Methods: PUT, DELETE, PATCH.
- Content-Type: `application/json`.
---

### Common CORS Issues & Solutions:
#### Issue 1: "No Access-Control-Allow-Origin-Header"
- **Problem**: Server hasn't set CORS headers.
- **Solution**: Backend must add CORS headers.
#### Issue 2: "Credentials mode is 'include'"
- **Problem**: Sending cookies cross-origin but server doesn't allow.
- **Solution**: Server set `Access-Control-Allow-Credentials: true`.
#### Issue 3: Wildcard (`*`) doesn't work with credentials
- **Problem**: `Access-Control-Allow-Origin: *` + cookies = forbidden.
- **Solution**: Specify exact origin: `Access-Control-Allow-Origin: https://example.com`.


