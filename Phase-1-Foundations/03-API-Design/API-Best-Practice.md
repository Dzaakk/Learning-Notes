API Best Practices
======

## API Versioning (Managing Change)
**Problem**: Your API evolves, but old clients still use it. Breaking changes = angry users!\
**Solution**: Version your API so old and new versions coexist.

### Analogy
Think of iPhone IOS updates:
- iOS 16 users can still use old apps.
- iOS 17 introduces new features.
- Apps specify minimum iOS version.

Similarly:
- API v1 still works for old clients.
- API v2 has new features.
- Client specify which version they want.

### Versioning Strategies
1. URI Versioning (Most Common)
Good:\
    - GET /v1/users
    - GET /v2/users

    #### Why it's popular:
    - Clear and explicit.
    - Easy to route.
    - Caching-friendly.

    #### Real-World Examples:
    - **Twiter API**: `api.twitter.com/1.1/statuses` -> `api.twitter.com/2/tweets`
    - **Stripe**: `api.stripe.com/v1/charges`
    - **GitHub**: `api.github.com/v3/users`


2. Header Versioning
GET /users HTTP/1.1\
Accept: application/vnd.myapi.v2+json

    Why use it:
    - Cleaner URLs
    - Same endpoint, different versions

    #### Used by:
    - GitHub API
    - Azure API


3. Query Parameter Versioning
GET /users?version=2

    Why it's less common:
    - Easy to forget
    - Caching issues
    - Not RESTful

### Version Comparison
|Method|Pros|Cons|Example|
|:-----|:---|:---|:------|
|URI|Clear, Simple|URL changes|`/v1/users`|
|Header|Clean URLs|Harder to test|`Accept: v2`|
|Query|Easy|Messy, caching issues| `?v=2`|

### Versioning Trade-offs
No Versioning
├─ Pros: Simple
└─ Cons: Breaking changes break everything💥

Too Many Versions (v1, v1.1, v1.2, v2, v2.1...)
├─ Pros: Gradual migration
└─ Cons: Maintenance hell (20 version to support)

Just Right (v1, v2, v3 for major changes)
├─ Pros: Manageable
└─ Cons: Requires planning

### Best Practices
#### ✅DO:
- Version from day 1 (even if it's v1)
- Use major versions only (v1, v2, not v1.2.3)
- Deprecate old versions gradually (give 6-12 months)
- Document breaking changes clearly

#### ❌DON'T:
- Remove versions without warning
- Version every tiny change
- Mix versioning strategies


## Pagination (Handling Large Data)
**Problem**: Returning 1 million users in one response = 💥 server crash + slow client\
**Solution**: Split data into pages.

### Analogy
Google search results:
- Page 1: Results 1-10
- Page 2: Results 11-20
- You don't load all 1 million results at once!

### Pagination Strategies
1. Offset-Based Pagination (Simple)

    GET /users?limit=20&offset=0 // Page 1 (users 1-20)\
    GET /users?limit=20&offset=20 // Page 2 (users 21-40)\
    GET /users?limit=20&offset=40 // Page 3 (users 41-60)

    #### How it works:
    ```sql
    SELECT * FROM users
    LIMIT 20 OFFSET 40;
    ```
    #### Response:
    ```json
    {
    "data": [...],
    "pagination": {
        "total": 1000,
        "page": 3,
        "per_page": 20,
        "total_pages": 50
        }
    }
    ```
    #### ✅Pros:
    - Simple to implement
    - Client controls page size
    - Can jump to any page
    
    #### ❌Cons:
    - **Slow for large offsets** (database scans first N rows)
    - **Inconsistent if data changes (new item added -> duplicates/skips)

    **Used By** : GitHub, Reddit, most simple APIs

2. Cursor-Based Pagination (Efficient) 

    GET /users?limit=20 // First page\
    GET /users?limit=20&cursor=abc123 //Next page\
    GET /users?limit=20&cursor=xyz789 //Page after that\

    #### How it works:
    Cursor = encoded pointer to last item\
    Example: cursor "eyJ1c2VyX2lkIjoxMjN9" (base64 encoded {user_id:123})
    Query: 
    ```sql
    SELECT * FROM users 
    WHERE id > 123 
    Limit 20;
    ```
    #### Response:
    ```json
        {
    "data": [...],
    "pagination": {
        "next_cursor": "xyz789",
        "has_more": true
        }
    }
    ```
    #### ✅Pros:
    - Fast (no offset scan)
    - Consistent (even if data changes)
    - Efficient for large datasets
    #### ❌Cons:
    - Can't jump to specific page
    - More complex to implement

    **Used by**: Twitter, Facebook, Instagram, Stripe (real-time feeds)
    
3. Page-Based Pagination (User-Friendly)

    Get /users?page=1&per_page=20 // Page1\
    Get /users?page=2&per_page=20 // Page2\

    #### Response:
    ```json
        {
    "data": [...],
    "page": 2,
    "per_page": 20,
    "total_pages": 50,
    "total_items": 1000
    }
    ```

    #### ✅Pros:
    - Intuitive for users
    - Shows total pages (good for UI)

    #### ❌Cons:
    - Same issues as offset-based
    
    **Used by**: WordPress API, e-commerce sites

### Pagination Comparison
|Method|Speed|Consistency|Jump to Page?|Best For|
|:-----|-----|-----------|-------------|---------|
|Offset|Slow (large offset)|❌|✅Yes|Small datasets|
|Cursor|⚡Fast|✅|❌No|Large datasets, feeds|
|Page|Slow (large page)|❌|✅Yes|User-facing UIs|

### When to Use What?

#### Offset-Based:
✅ Admin dashboards (small datasets)\
✅ Search results (need page numbers)\
✅ Simple CRUD apps

#### Cursor-Based:
✅ Twitter feed (millions of tweets, real-time)\
✅ Chat messages (constantly growing)\
✅ Activity logs

#### Page-Based:
✅ E-commerce product listings\
✅ Blog posts\
✅ User wants to see "Page 5"

## Rate Limiting (Preventing Abuse)
**Problem**: A malicious user sends 1 million requests/sec -> server crashes\
**Solution**: Limit requests per user.

### Analogy
Free sample at supermarket:
- "One free sample per customer"
- If you come back 100 times, they say "Sorry, you already had one!"

Rate limit = "100 requests per minute per user"

### How Rate Limiting Works
![rate-limit-flow](./images/rate-limit-process.excalidraw.png)

### Rate Limit Response
**Request:**
```http
Get /api/users HTTP/1.1
Authorization: Bearer token123
```
**Response (if within limit):**
```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000 // Max Requests per hour
X-RateLimit-Remaining: 999 // Requests left
X-RateLimit-Reset: 164000000 // When limit resets (Unix timestamp)

{"data": [...]}
```
**Response (if exceeded):**
```http
HTTP/1.1 429 Too Many Requests 
X-RateLimit-Limit: 1000 
X-RateLimit-Remaining: 0 
X-RateLimit-Reset: 164000000 
Retry-After: 3600 // Retry after 1 hour (seconds)

{"error": "Rate limit exceeded. Try again in 1 hour."}
```

### Rate Limiting Algorithms
1. Fixed Window

    Every hour = new window

    Window 1 (10:00 - 11:00): 1000 requests allowed\
    Window 2 (11:00 - 12:00): 1000 requests allowed

    **Problem**: Burst at window edge
    - 10:59 -> 1000 requests
    - 11:00 -> 1000 requests

    = 2000 requests in 1 minute!

2. Sliding Window (Better)
    
    Tracks last 60 minutes continuously
    - At 10:30, checks requests from 9:30-10:30
    - At 10:31, checks requests from 9:31-10:31

    More accurate, prevents burst

3. Token Bucket (Most Common)

    Bucket starts with 1000 tokens
    - Each request = token
    - Tokens refill at constant rate (10 tokens/minute)
    - If bucket empty -> rate limited

    Allows short bursts, smooth over time

### Real-World Rate Limits
|API|Free Tier|Paid Tier|Per|
|:--|:--------|:--------|:--|
|Twitter|15 requests|450 requests|15 min|
|GitHub|60 requests|5000 requests|hour|
|Stripe|100 requests|No limit (per account)|second|
|Google Maps|40,000 requests|Custom|month|

### Rate Limiting Trade-offs
|Approach|Pros|Cons|
|-|-|-|
|IP-based|Simple|Shared IPs (company networks)|
|User-based|Accurate|Requires authentication|
|API Key-based|Tracks per app|Users can create multiple keys|
|No rate limit|User-friendly|Abuse, DDoS risk💥|

### Best Practices
#### ✅DO:
- Return clear error messages
- Include Retry-After header
- Document limits clearly
- Use 429 status code
- Different limits for different endpoints\
(GET /users -> 1000/hour, POST /users -> 100/hour)

#### ❌DON'T:
- Return 403 or 500 (use 429)
- Hide rate limit info
- Make limits too strict (frustrates users)
- Apply same limit to all endpoints
