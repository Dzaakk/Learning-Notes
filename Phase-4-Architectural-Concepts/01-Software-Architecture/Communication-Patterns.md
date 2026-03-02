Communication Patterns
===

## Overview
In distributed systems and microservices, services need to communicate with each other. THe way they communicate significantly impacts system performance, reliability, and complexity.

**Two main categories:**
1. **Synchronous** - Caller waits for response
2. **Asynchronous** - Caller doesn't wait, continues immediately

## Synchronous Communication

### 1. REST API (HTTP/HTTPS)
**REST (Representational State Transfer)** uses HTTPS methods to perform operations on resources identified by URLs.

**HTTP Methods**
- **GET** - Read data 
- **POST** - Create data
- **PUT** - Update/replace data
- **PATCH** - Partial update
- **DELETE** - Remove data

**Implementation Example**
```go
// User Service - REST API Server
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "sync"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

type UserStore struct {
    users map[int]*User
    mu    sync.RWMutex
    nextID int
}

func NewUserStore() *UserStore {
    return &UserStore{
        users: make(map[int]*User),
        nextID: 1,
    }
}

// GET /users
func (us *UserStore) GetAllUsers(w http.ResponseWriter, r *http.Request) {
    us.mu.RLock()
    defer us.mu.RUnlock()
    
    users := make([]*User, 0, len(us.users))
    for _, user := range us.users {
        users = append(users, user)
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

// GET /users/{id}
func (us *UserStore) GetUser(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }
    
    us.mu.RLock()
    user, exists := us.users[id]
    us.mu.RUnlock()
    
    if !exists {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

// POST /users
func (us *UserStore) CreateUser(w http.ResponseWriter, r *http.Request) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, "Invalid request body", http.StatusBadRequest)
        return
    }
    
    us.mu.Lock()
    user.ID = us.nextID
    us.nextID++
    us.users[user.ID] = &user
    us.mu.Unlock()
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}

// PUT /users/{id}
func (us *UserStore) UpdateUser(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }
    
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, "Invalid request body", http.StatusBadRequest)
        return
    }
    
    us.mu.Lock()
    defer us.mu.Unlock()
    
    if _, exists := us.users[id]; !exists {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    user.ID = id
    us.users[id] = &user
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

// DELETE /users/{id}
func (us *UserStore) DeleteUser(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }
    
    us.mu.Lock()
    defer us.mu.Unlock()
    
    if _, exists := us.users[id]; !exists {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    delete(us.users, id)
    w.WriteHeader(http.StatusNoContent)
}

func main() {
    store := NewUserStore()
    
    http.HandleFunc("/users", func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case "GET":
            store.GetAllUsers(w, r)
        case "POST":
            store.CreateUser(w, r)
        default:
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })
    
    http.HandleFunc("/users/", func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case "GET":
            store.GetUser(w, r)
        case "PUT":
            store.UpdateUser(w, r)
        case "DELETE":
            store.DeleteUser(w, r)
        default:
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })
    
    log.Println("User service running on :8001")
    log.Fatal(http.ListenAndServe(":8001", nil))
}
```

**REST Client Example**
```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

type UserClient struct {
    baseURL    string
    httpClient *http.Client
}

func NewUserClient(baseURL string) *UserClient {
    return &UserClient{
        baseURL: baseURL,
        httpClient: &http.Client{
            Timeout: 10 * time.Second,
        },
    }
}

// Get all users
func (c *UserClient) GetUsers() ([]*User, error) {
    resp, err := c.httpClient.Get(c.baseURL + "/users")
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("status: %d", resp.StatusCode)
    }
    
    var users []*User
    if err := json.NewDecoder(resp.Body).Decode(&users); err != nil {
        return nil, err
    }
    
    return users, nil
}

// Get single user
func (c *UserClient) GetUser(id int) (*User, error) {
    url := fmt.Sprintf("%s/users/?id=%d", c.baseURL, id)
    resp, err := c.httpClient.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    if resp.StatusCode == http.StatusNotFound {
        return nil, fmt.Errorf("user not found")
    }
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("status: %d", resp.StatusCode)
    }
    
    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        return nil, err
    }
    
    return &user, nil
}

// Create user
func (c *UserClient) CreateUser(user *User) (*User, error) {
    body, err := json.Marshal(user)
    if err != nil {
        return nil, err
    }
    
    resp, err := c.httpClient.Post(
        c.baseURL+"/users",
        "application/json",
        bytes.NewReader(body),
    )
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusCreated {
        return nil, fmt.Errorf("status: %d", resp.StatusCode)
    }
    
    var created User
    if err := json.NewDecoder(resp.Body).Decode(&created); err != nil {
        return nil, err
    }
    
    return &created, nil
}

func main() {
    client := NewUserClient("http://localhost:8001")
    
    // Create user
    newUser := &User{
        Name:  "John Doe",
        Email: "john@example.com",
    }
    
    created, err := client.CreateUser(newUser)
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Created user: %+v\n", created)
    
    // Get user
    user, err := client.GetUser(created.ID)
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Retrieved user: %+v\n", user)
}
```
**Advantages:**
- Simple and well-understood
- Human readable (JSON)
- Cacheable (HTTP caching)
- Stateless
- Wide tooling support

**Disadvantages:**
- Chatty (multiple requests for complex operations)
- Over fetching/under fetching data
- Tight coupling (client waits)
- Not ideal for real time

**Best For:** CRUD operations, public APIs, web applications

### 2. gRPC (Remote Procedure Call)
**gRPC** is a high performance RPC framework using Protocol Buffers (binary format) over HTTP/2

**Define Service (Protobuf)**
```proto
// user.proto
syntax = "proto3";

package user;
option go_package = "github.com/example/user";

service UserService {
    rpc GetUser(GetUserRequest) returns (UserResponse);
    rpc CreateUser(CreateUserRequest) returns (UserResponse);
    rpc ListUsers(ListUsersRequest) returns (stream UserResponse);
}

message GetUserRequest {
    int32 id = 1;
}

message CreateUserRequest {
    string name = 1;
    string email = 2;
}

message ListUsersRequest {
    int32 page = 1;
    int32 page_size = 2;
}

message UserResponse {
    int32 id = 1;
    string name = 2;
    string email = 3;
}
```

**Generate Go Code**
```bash
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    user.proto
```

**gRPC Server Implementation**
```go
package main

import (
    "context"
    "fmt"
    "log"
    "net"
    "sync"
    
    pb "github.com/example/user"
    "google.golang.org/grpc"
)

type userServer struct {
    pb.UnimplementedUserServiceServer
    users  map[int32]*pb.UserResponse
    nextID int32
    mu     sync.RWMutex
}

func newUserServer() *userServer {
    return &userServer{
        users:  make(map[int32]*pb.UserResponse),
        nextID: 1,
    }
}

func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.UserResponse, error) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    user, exists := s.users[req.Id]
    if !exists {
        return nil, fmt.Errorf("user not found")
    }
    
    return user, nil
}

func (s *userServer) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.UserResponse, error) {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    user := &pb.UserResponse{
        Id:    s.nextID,
        Name:  req.Name,
        Email: req.Email,
    }
    
    s.users[s.nextID] = user
    s.nextID++
    
    return user, nil
}

func (s *userServer) ListUsers(req *pb.ListUsersRequest, stream pb.UserService_ListUsersServer) error {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    for _, user := range s.users {
        if err := stream.Send(user); err != nil {
            return err
        }
    }
    
    return nil
}

func main() {
    listener, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("Failed to listen: %v", err)
    }
    
    grpcServer := grpc.NewServer()
    pb.RegisterUserServiceServer(grpcServer, newUserServer())
    
    log.Println("gRPC server running on :50051")
    if err := grpcServer.Serve(listener); err != nil {
        log.Fatalf("Failed to serve: %v", err)
    }
}
```

**gRPC Client**
```go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "time"
    
    pb "github.com/example/user"
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
)

func main() {
    // Connect to server
    conn, err := grpc.Dial("localhost:50051", 
        grpc.WithTransportCredentials(insecure.NewCredentials()))
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    client := pb.NewUserServiceClient(conn)
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    // Create user
    createResp, err := client.CreateUser(ctx, &pb.CreateUserRequest{
        Name:  "Alice",
        Email: "alice@example.com",
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Created: %+v\n", createResp)
    
    // Get user
    getResp, err := client.GetUser(ctx, &pb.GetUserRequest{
        Id: createResp.Id,
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Retrieved: %+v\n", getResp)
    
    // List users (streaming)
    stream, err := client.ListUsers(ctx, &pb.ListUsersRequest{})
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("All users:")
    for {
        user, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("- %+v\n", user)
    }
}
```

**Advantages:**
- High performance (binary protocol)
- Strong typing (Protocol Buffers)
- Bi-directional streaming
- Code generation (client & server)
- Built-in error handling

**Disadvantages:**
- Not human readable
- Limited browser support
- More complex setup
- Requires protobuf knowledge

**Best For:** Service to service communication, real time streaming, performance critical paths

### 3. GraphQL
**GraphQL** allows clients to request exactly data they need in a single query.

**Server Implementation**
```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    
    "github.com/graphql-go/graphql"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

var users = []User{
    {ID: 1, Name: "Alice", Email: "alice@example.com"},
    {ID: 2, Name: "Bob", Email: "bob@example.com"},
}

var userType = graphql.NewObject(
    graphql.ObjectConfig{
        Name: "User",
        Fields: graphql.Fields{
            "id": &graphql.Field{
                Type: graphql.Int,
            },
            "name": &graphql.Field{
                Type: graphql.String,
            },
            "email": &graphql.Field{
                Type: graphql.String,
            },
        },
    },
)

var queryType = graphql.NewObject(
    graphql.ObjectConfig{
        Name: "Query",
        Fields: graphql.Fields{
            "user": &graphql.Field{
                Type: userType,
                Args: graphql.FieldConfigArgument{
                    "id": &graphql.ArgumentConfig{
                        Type: graphql.Int,
                    },
                },
                Resolve: func(p graphql.ResolveParams) (interface{}, error) {
                    id, ok := p.Args["id"].(int)
                    if !ok {
                        return nil, nil
                    }
                    
                    for _, user := range users {
                        if user.ID == id {
                            return user, nil
                        }
                    }
                    return nil, nil
                },
            },
            "users": &graphql.Field{
                Type: graphql.NewList(userType),
                Resolve: func(p graphql.ResolveParams) (interface{}, error) {
                    return users, nil
                },
            },
        },
    },
)

var schema, _ = graphql.NewSchema(
    graphql.SchemaConfig{
        Query: queryType,
    },
)

func executeQuery(query string) *graphql.Result {
    result := graphql.Do(graphql.Params{
        Schema:        schema,
        RequestString: query,
    })
    return result
}

func main() {
    http.HandleFunc("/graphql", func(w http.ResponseWriter, r *http.Request) {
        var params struct {
            Query string `json:"query"`
        }
        
        if err := json.NewDecoder(r.Body).Decode(&params); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }
        
        result := executeQuery(params.Query)
        json.NewEncoder(w).Encode(result)
    })
    
    log.Println("GraphQL server running on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**GraphQL Query Examples**
```graphql
# Get specific fields only
{
  user(id: 1) {
    name
    email
  }
}

# Get all users
{
  users {
    id
    name
  }
}
```
**Advantages:**
- Client specifies exact data needed
- Single request for complex dqata
- No over fetching/under fetching
- Strong typing 
- Great for frontend

**Disadvantages:**
- Complex to implement
- Difficult to cache
- Can expose too much data
- Query complexity issues

**Best For:** Frontend heavy apps, mobile apps (reduce bandwith), complex data requirements