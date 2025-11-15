GraphQL
======

## GraphQL (Flexible Data Queries)
GraphQL (by Facebook) is a **query language for APIs** that lets clients ask for exactly what they need — no more, no less.

Instead of having fixed endpoints like `/users` or `/posts`, you have **one endpoint** (usually `/graphql`), and the client defines what fields it wants.

### Example:
#### Request:
```graphql
{
  user(id: 123) {
    name
    posts {
      title
    }
  }
}
```
#### Response:
```graphql
{
  "user": {
    "name": "Alice",
    "posts": [
      {"title": "My first post"},
      {"title": "GraphQL is great"}
    ]
  }
}
```

### Key Ideas:
- Client defines the shape of the response.
- One endpoint handles all queries.
- Uses **schema and resolvers** (like a type system for your data).

### Pros:
- No over-fetching or under-fetching.
- Great for frontend flexibility.
- Strongly typed — clients know exactly what's avaliable.
### Cons:
- More complex to cache (every query is unique).
- Harder to secure and monitor.
- Needs extra server logic to resolve queries.

### Use it for:
- Public APIs or frontend-heavy apps (like mobile or dashboards).
- System with lots of relational data (users, posts, comments, etc.).
