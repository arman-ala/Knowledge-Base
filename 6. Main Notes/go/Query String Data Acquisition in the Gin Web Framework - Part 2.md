# Query String Data Acquisition in the Gin Web Framework - Part 2

2025-07-29 09:37
Status: #DONE 
Tags: [[Gin]]

---
## Query String Data Acquisition in the Gin Web Framework: A Comprehensive Reference

| Method / Function | Explanation |
|---|---|
| `c.Query(key)` | Returns the *first* value associated with `key` as a string. If the key is absent an empty string is returned. |
| `c.DefaultQuery(key, defaultValue)` | Identical to `Query` but substitutes `defaultValue` when the key is missing. |
| `c.GetQuery(key)` | Returns the *first* value and a boolean indicating existence (true even if the value is empty). |
| `c.QueryArray(key)` | Returns a slice containing *all* values for `key`; yields an empty slice when the key is absent. |
| `c.GetQueryArray(key)` | Same as `QueryArray` but also returns an existence boolean. |
| `c.QueryMap(key)` | Converts repeated parameters of the form `key[sub]=value` into a `map[string]string`. |
| `c.GetQueryMap(key)` | Same as `QueryMap` plus an existence boolean. |
| `c.ShouldBindQuery(obj)` | Binds the entire query string into a struct using reflection and struct tags; returns an error if binding fails. |
| `c.MustBindQuery(obj)` | Identical to `ShouldBindQuery` but aborts the request with HTTP 400 on failure. |
| `c.ShouldBind(obj)` | Auto-selects binding engine based on `Content-Type`; for `GET` requests it behaves like `ShouldBindQuery`. |
| `c.MustBind(obj)` | Auto-selected binding engine that aborts on failure. |

---

### Tips, Tricks & Best Practices

| Tip | Rationale | Example Reference |
|---|---|---|
| Prefer `GetQuery*` when distinguishing missing vs empty | Avoids silent acceptance of omitted parameters | Example 2 |
| Validate early with struct binding | Centralizes validation logic and reduces boilerplate | Example 6 |
| Use typed struct fields to leverage automatic coercion | Gin converts query strings to native Go types | Example 6 |
| Combine middleware with struct binding for reusable validation | Promotes DRY code | Example 7 |
| Avoid `MustBind*` in middleware to allow graceful degradation | Middleware should not terminate the request chain prematurely | Example 7 |
| Leverage `QueryMap` for dynamic filters | Simplifies handling unknown key-value pairs | Example 4 |
| Cache expensive query transformations | Improves performance under load | Example 8 |
| Document query parameter contracts via struct tags | Improves API discoverability | Example 6 |
| Sanitize and escape user-provided query values | Prevents injection attacks | Example 9 |
| Use `DefaultQuery` for optional pagination parameters | Provides sensible defaults without extra checks | Example 1 |

---

### 1. Simple Key Retrieval via `Query`

```go
// GET /search?q=golang&limit=
r.GET("/search", func(c *gin.Context) {
    q := c.Query("q")         // "golang"
    limit := c.Query("limit") // ""  (empty string, key exists but no value)
    c.JSON(200, gin.H{"q": q, "limit": limit})
})
```

---

### 2. Distinguishing Absence from Empty with `GetQuery`

```go
// GET /filter?active=true
r.GET("/filter", func(c *gin.Context) {
    val, ok := c.GetQuery("active")
    if !ok {
        c.JSON(400, gin.H{"error": "missing required param 'active'"})
        return
    }
    active := val == "true"
    c.JSON(200, gin.H{"active": active})
})
```

---

### 3. Providing Safe Defaults via `DefaultQuery`

```go
// GET /items
r.GET("/items", func(c *gin.Context) {
    page := c.DefaultQuery("page", "1")
    size := c.DefaultQuery("size", "10")
    c.JSON(200, gin.H{"page": page, "size": size})
})
```

---

### 4. Collecting Repeated Keys with `QueryArray`

```go
// GET /tags?tag=go&tag=web&tag=framework
r.GET("/tags", func(c *gin.Context) {
    tags := c.QueryArray("tag") // []string{"go","web","framework"}
    c.JSON(200, gin.H{"tags": tags})
})
```

---

### 5. Building Maps from Prefixed Keys with `QueryMap`

```go
// GET /meta?filter[name]=gin&filter[year]=2024
r.GET("/meta", func(c *gin.Context) {
    filter := c.QueryMap("filter") // map[string]string{"name":"gin","year":"2024"}
    c.JSON(200, gin.H{"filter": filter})
})
```

---

### 6. Struct Binding for Validation & Type Safety

```go
type SearchRequest struct {
    Query string `form:"q" binding:"required"`
    Page  int    `form:"page" binding:"min=1"`
    Size  int    `form:"size" binding:"max=100"`
}

// GET /search?q=gin&page=2&size=5
r.GET("/search", func(c *gin.Context) {
    var req SearchRequest
    if err := c.ShouldBindQuery(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, gin.H{"query": req.Query, "page": req.Page, "size": req.Size})
})
```

---

### 7. Middleware-Level Validation without Abort

```go
type Pagination struct {
    Page int `form:"page" binding:"min=1"`
    Size int `form:"size" binding:"max=100"`
}

func PaginationMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        var p Pagination
        if err := c.ShouldBindQuery(&p); err == nil {
            c.Set("pagination", p) // pass downstream
        } // silently ignore errors to allow handlers to decide
        c.Next()
    }
}

r.Use(PaginationMiddleware())
r.GET("/list", func(c *gin.Context) {
    if v, ok := c.Get("pagination"); ok {
        p := v.(Pagination)
        c.JSON(200, gin.H{"page": p.Page, "size": p.Size})
    } else {
        c.JSON(200, gin.H{"page": 1, "size": 10})
    }
})
```

---

### 8. Caching Expensive Query Transformations

```go
var (
    cache   sync.Map
    cacheMu sync.RWMutex
)

type Filter struct {
    Tags []string `form:"tags"`
}

func cachedFilter(c *gin.Context) {
    raw := c.Request.URL.RawQuery
    if v, ok := cache.Load(raw); ok {
        c.JSON(200, v)
        return
    }
    var f Filter
    c.ShouldBindQuery(&f)
    sort.Strings(f.Tags) // normalize
    cacheMu.Lock()
    cache.Store(raw, gin.H{"tags": f.Tags})
    cacheMu.Unlock()
    c.JSON(200, gin.H{"tags": f.Tags})
}
```

---

### 9. Sanitizing User Input

```go
import "html"

// GET /echo?msg=<script>alert(1)</script>
r.GET("/echo", func(c *gin.Context) {
    msg := c.Query("msg")
    safe := html.EscapeString(msg)
    c.JSON(200, gin.H{"original": msg, "safe": safe})
})
```

How Gin treats **multiple values** for the **same query key**  
Gin never discards additional values.  
• `Query` / `DefaultQuery` / `GetQuery` – return **only the first** value.  
• `QueryArray` / `GetQueryArray` – return the **slice of all** values.  
• When you bind with `ShouldBindQuery`, a slice field receives every occurrence, a scalar field receives the **first** value.

Example:  
`GET /?tag=go&tag=web&tag=gin`  
```
c.Query("tag")            // "go"
c.QueryArray("tag")       // []string{"go","web","gin"}
```

---

How Gin treats **multiple distinct query keys**  
Each key is independent; the order of appearance is preserved only within the same key.

---

`ShouldBindQuery` with a struct – complete example

```go
type Search struct {
    Q       string   `form:"q" binding:"required"`
    Tags    []string `form:"tag"`          // multiple values collected
    Page    int      `form:"page,default=1"`
}

r.GET("/search", func(c *gin.Context) {
    var s Search
    if err := c.ShouldBindQuery(&s); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, gin.H{"query": s.Q, "tags": s.Tags, "page": s.Page})
})
```

Request:  
`GET /search?q=golang&tag=web&tag=framework&page=2`  
Response:  
`{"query":"golang","tags":["web","framework"],"page":2}`

---

Differences between the query helpers

| Function               | Returns | Empty value behaviour | Multiple-value support | Use-case |
|------------------------|---------|-----------------------|------------------------|----------|
| `Query(key)`           | string  | empty string if absent | first only             | quick read, single value |
| `DefaultQuery(k,d)`    | string  | supplied `default` if absent | first only | optional parameters |
| `GetQuery(k)`          | (string,bool) | bool indicates presence | first only | distinguish missing vs empty |
| `QueryArray(k)`        | []string | empty slice if absent | **all values** | repeated keys |
| `GetQueryArray(k)`     | ([]string,bool) | same as above | **all values** | same, plus existence flag |
| `QueryMap(prefix)`     | map[string]string | empty map if absent | grouped by sub-key | dynamic filters |
| `GetQueryMap(prefix)`  | (map[string]string,bool) | same as above | same | same, plus existence flag |