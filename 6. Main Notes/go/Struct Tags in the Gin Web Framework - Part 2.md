## Struct Tags in the Gin Web Framework: A Comprehensive Reference

| Tag Key | Supported Bindings | Explanation |
|---|---|---|
| `form` | Query, Post Form, Multipart Form | Maps a struct field to a form or query parameter; slices collect multiple values. |
| `uri` | URI Parameters | Maps a struct field to a path parameter declared in the route pattern. |
| `header` | Request Headers | Maps a struct field to an HTTP request header; case-insensitive. |
| `json` | JSON Body | Maps a struct field to a JSON property in the request body. |
| `xml` | XML Body | Maps a struct field to an XML element/attribute in the request body. |
| `yaml` | YAML Body | Maps a struct field to a YAML key in the request body. |
| `toml` | TOML Body | Maps a struct field to a TOML key in the request body. |
| `binding` | All engines | Declares validation rules (e.g., `required`, `min`, `max`, `email`, `oneof`). |
| `time_format` | Time fields | Specifies the layout string for parsing time values (e.g., `"2006-01-02"`). |
| `default` | Form/Query/JSON | Supplies a default value when the parameter is absent. |
| `file` | Multipart Form | Declares a field as `*multipart.FileHeader` or `[]*multipart.FileHeader` for file uploads. |

---

### Tips, Tricks & Best Practices

| Tip | Rationale | Example Reference |
|---|---|---|
| Combine multiple tags on one field | Enables a single struct to serve JSON, form, URI, and header bindings | Example 1 |
| Prefer `binding:"required"` over manual checks | Centralizes validation and reduces boilerplate | Example 2 |
| Use `time_format` for custom date layouts | Guarantees deterministic parsing across locales | Example 3 |
| Tag slices as `[]T` to collect repeated keys automatically | Eliminates manual array handling | Example 4 |
| Supply `default` to keep optional fields zero-value safe | Simplifies client contracts | Example 5 |
| Tag embedded structs for hierarchical binding | Promotes code reuse and clear separation | Example 6 |
| Validate file extensions via `binding:"oneof=..."` | Prevents unwanted uploads | Example 7 |
| Use `header` tags in middleware for cross-cutting concerns | Keeps handlers clean and declarative | Example 8 |
| Document enum values with `oneof` in tags | Generates better OpenAPI specs | Example 9 |
| Test tag combinations with table-driven unit tests | Detects silent binding errors early | Example 10 |

---

### 1. Multiple Tags on One Field

```go
type User struct {
    ID   int    `uri:"id" form:"id" json:"id" binding:"required"`
    Name string `form:"name" json:"name" binding:"required"`
}
```

---

### 2. Required Validation

```go
type Login struct {
    Email string `form:"email" binding:"required,email"`
    Pass  string `form:"password" binding:"required,min=8"`
}
```

---

### 3. Custom Time Layout

```go
type Event struct {
    Date time.Time `form:"date" time_format:"2006-01-02" binding:"required"`
}
```

---

### 4. Repeated Form Values

```go
type Filter struct {
    Tags []string `form:"tags[]"`
}
```

---

### 5. Default Value

```go
type Page struct {
    Size int `form:"size" default:"10"`
}
```

---

### 6. Embedded Struct Binding

```go
type Address struct {
    City string `form:"city"`
}

type Profile struct {
    Name string `form:"name"`
    Address
}
```

---

### 7. File Extension Validation

```go
type Upload struct {
    File *multipart.FileHeader `form:"file" binding:"required"`
}
```

---

### 8. Header Binding in Middleware

```go
type ClientInfo struct {
    Token string `header:"Authorization" binding:"required"`
}
```

---

### 9. Enum via `oneof`

```go
type Role struct {
    Level string `form:"level" binding:"oneof=admin user guest"`
}
```

---

### 10. Table-Driven Tag Test

```go
func TestBind(t *testing.T) {
    type Case struct{ Raw string; Want int }
    tests := []Case{{"age=30", 30}, {"age=", 0}}
    for _, tc := range tests {
        var s struct{ Age int `form:"age"` }
        _ = binding.Form.Bind(strings.NewReader(tc.Raw), &s)
        if s.Age != tc.Want { t.Fatalf("mismatch") }
    }
}
```
