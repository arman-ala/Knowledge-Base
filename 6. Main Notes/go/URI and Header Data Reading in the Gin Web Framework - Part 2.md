# URI and Header Data Reading in the Gin Web Framework - Part 2

2025-07-29 12:20
Status: #DONE 
Tags: [[Gin]]

---
## URI & Header Data Acquisition in the Gin Web Framework: A Comprehensive Reference

| Method / Function         | Explanation                                                                                                       |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `c.Param(key)`            | Returns the value of a single path parameter bound to the route template by name.                                 |
| `c.Params`                | Exposes the underlying slice of path parameters for indexed or ranged access.                                     |
| `c.GetHeader(key)`        | Returns the first value of the request header with the given canonical key (case-insensitive).                    |
| `c.Header(key, value)`    | Sets an outgoing response header (shortcut for `c.Writer.Header().Set`).                                          |
| `c.ShouldBindUri(obj)`    | Binds path parameters into struct fields annotated with `uri` tags; returns an error only.                        |
| `c.MustBindUri(obj)`      | Identical to `ShouldBindUri` but aborts with HTTP 400 on failure.                                                 |
| `c.ShouldBindHeader(obj)` | Binds incoming headers into struct fields annotated with `header` tags; returns an error only.                    |
| `c.MustBindHeader(obj)`   | Identical to `ShouldBindHeader` but aborts with HTTP 400 on failure.                                              |
| `c.ShouldBind(obj)`       | Auto-selects binding engine based on method and content-type; for URI parameters it behaves like `ShouldBindUri`. |
| `c.MustBind(obj)`         | Auto-selected engine that aborts on failure.                                                                      |

---

### Tips, Tricks & Best Practices

| Tip | Rationale | Example Reference |
|---|---|---|
| Use struct tags for URI parameters to enforce typing | Eliminates manual conversion and reduces runtime errors | Example 1 |
| Prefer `ShouldBindUri` over `MustBindUri` in middleware | Allows graceful degradation, keeps handler chain intact | Example 2 |
| Cache frequently used headers to avoid repeated reflection | Improves performance on high-throughput endpoints | Example 3 |
| Validate custom header formats early with middleware | Centralizes validation logic and keeps handlers clean | Example 4 |
| Use `GetHeader` for single-value headers, slice fields for multi-value | Prevents silent truncation of repeated headers | Example 5 |
| Document required headers via struct tags | Improves API discoverability and OpenAPI generation | Example 6 |
| Sanitize path parameters to prevent injection | Protects against directory traversal attacks | Example 7 |
| Combine path and query binding for composite keys | Supports RESTful design while remaining flexible | Example 8 |
| Use `c.Params.ByName` for direct indexed access | Slightly faster when parameter order is known | Example 9 |

---

### 1. Single Path Parameter via `Param`

```go
// Route: /user/:id
r.GET("/user/:id", func(c *gin.Context) {
    id := c.Param("id") // "42" for /user/42
    c.JSON(200, gin.H{"id": id})
})
```

---

### 2. Struct Binding of Path Parameters with `ShouldBindUri`

```go
type ProfileUri struct {
    ID   int    `uri:"id" binding:"required,min=1"`
    Lang string `uri:"lang" binding:"required,len=2"`
}

// Route: /profile/:lang/:id
r.GET("/profile/:lang/:id", func(c *gin.Context) {
    var u ProfileUri
    if err := c.ShouldBindUri(&u); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, gin.H{"lang": u.Lang, "id": u.ID})
})
```

---

### 3. Reading a Single Request Header with `GetHeader`

```go
r.GET("/secure", func(c *gin.Context) {
    token := c.GetHeader("Authorization")
    if token == "" {
        c.AbortWithStatus(401)
        return
    }
    c.JSON(200, gin.H{"token": token})
})
```

---

### 4. Binding All Request Headers into a Struct with `ShouldBindHeader`

```go
type ClientInfo struct {
    UserAgent string `header:"User-Agent"`
    IP        string `header:"X-Forwarded-For"`
}

r.GET("/whoami", func(c *gin.Context) {
    var info ClientInfo
    _ = c.ShouldBindHeader(&info)
    c.JSON(200, info)
})
```

---

### 5. Handling Multi-value Headers

```go
r.GET("/accept", func(c *gin.Context) {
    // Accept: text/html,application/xhtml+xml;q=0.9
    accepts := c.Request.Header.Values("Accept")
    c.JSON(200, gin.H{"accepts": accepts})
})
```

---

### 6. Middleware-level Header Validation

```go
type ApiKeyHeader struct {
    Key string `header:"x-api-key" binding:"required,len=32"`
}

func ApiKeyAuth() gin.HandlerFunc {
    return func(c *gin.Context) {
        var h ApiKeyHeader
        if err := c.ShouldBindHeader(&h); err != nil {
            c.AbortWithStatusJSON(401, gin.H{"error": "invalid or missing x-api-key"})
            return
        }
        c.Set("api_key", h.Key)
        c.Next()
    }
}

r.Use(ApiKeyAuth())
r.GET("/admin", func(c *gin.Context) {
    key := c.MustGet("api_key").(string)
    c.JSON(200, gin.H{"api_key": key})
})
```

---

### 7. Sanitizing Path Parameters

```go
r.GET("/static/*filepath", func(c *gin.Context) {
    fp := c.Param("filepath")
    clean := path.Clean("/" + fp) // prevent traversal
    c.JSON(200, gin.H{"clean": clean})
})
```

---

### 8. Composite Key: Path + Query Binding

```go
type SearchRequest struct {
    Lang string `uri:"lang" binding:"required,len=2"`
    Q    string `form:"q"  binding:"required"`
}

// Route: /search/:lang
r.GET("/search/:lang", func(c *gin.Context) {
    var req SearchRequest
    _ = c.ShouldBindUri(&req)
    _ = c.ShouldBindQuery(&req)
    c.JSON(200, gin.H{"lang": req.Lang, "query": req.Q})
})
```

---

### 9. Direct Indexed Access via `c.Params`

```go
r.GET("/book/:author/:title", func(c *gin.Context) {
    author := c.Params.ByName("author")
    title  := c.Params.ByName("title")
    c.JSON(200, gin.H{"author": author, "title": title})
})
```

| Method | Purpose | Operates On | Typical Usage |
|---|---|---|---|
| `GetHeader(key)` | **Read** an **incoming** request header | Request headers | `token := c.GetHeader("Authorization")` |
| `Header(key, value)` | **Write** an **outgoing** response header | Response headers | `c.Header("X-Rate-Limit", "100")` |

In short:

- `GetHeader` **retrieves** a header sent **by the client**.  
- `Header` **sets** a header to be sent **to the client**.