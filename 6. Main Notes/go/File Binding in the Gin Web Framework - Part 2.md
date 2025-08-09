# File Binding in the Gin Web Framework - Part 2

2025-07-29 12:53
Status: #DONE 
Tags: [[Gin]]

---
File Binding in the Gin Web Framework: A Comprehensive Reference

| Method / Function | Explanation |
|---|---|
| `c.FormFile(name)` | Returns the first uploaded file for the given multipart field name; returns `*multipart.FileHeader`. |
| `c.MultipartForm()` | Returns the complete parsed `*multipart.Form` containing both files and values. |
| `c.SaveUploadedFile(src, dst)` | Copies the uploaded file from memory or temporary storage to the local filesystem at `dst`. |
| `c.ShouldBind(obj)` | Auto-detects `multipart/form-data` and binds form fields and files into a struct annotated with `form` tags. |
| `c.MustBind(obj)` | Same as `ShouldBind` but aborts the request with HTTP 400 on any binding failure. |
| `c.ShouldBindWith(obj, binding.Form)` | Explicitly binds a multipart body into a struct; returns an error only. |
| `c.MustBindWith(obj, binding.Form)` | Explicit binding that terminates the handler chain on failure. |
| `form:"field"` tag | Associates a struct field with a multipart form field; slices receive multiple values. |
| `form:"file"` tag + `*multipart.FileHeader` | Declares a struct field that will receive a single uploaded file descriptor. |
| `form:"files[]"` tag + `[]*multipart.FileHeader` | Declares a slice field that will receive multiple uploaded file descriptors. |

---

### Tips, Tricks & Best Practices

| Tip | Rationale | Example Reference |
|---|---|---|
| Always check returned errors from `FormFile` and `MultipartForm` | Prevents nil-pointer panics on malformed requests | Example 1 |
| Validate file size and MIME type before saving | Mitigates DoS via large uploads and enforces security policy | Example 2 |
| Use `ShouldBind` rather than `MustBind` in reusable middleware | Enables downstream recovery or alternative responses | Example 3 |
| Prefer direct `SaveUploadedFile` for single-file endpoints | Reduces boilerplate and leverages Gin’s optimized copy | Example 4 |
| Stream large files with `MultipartReader` to minimize memory usage | Avoids buffering multi-MB payloads into RAM | Example 5 |
| Sanitize filenames with `filepath.Base` or UUID renaming | Prevents path traversal and name collisions | Example 6 |
| Bind mixed form-data (fields + files) in a single struct | Encapsulates entire upload contract and simplifies validation | Example 7 |
| Set `maxMemory` in `c.MultipartForm()` to cap RAM usage | Guards against unbounded resource consumption | Example 2 |
| Document expected form fields via struct tags | Improves auto-generated OpenAPI specs | Example 7 |
| Run virus scanning in a separate goroutine | Keeps upload latency low while maintaining security | Example 8 |

---

### 1. Single-File Upload via `FormFile`

```go
r.POST("/avatar", func(c *gin.Context) {
    file, err := c.FormFile("avatar")
    if err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    dst := "./uploads/" + file.Filename
    if err := c.SaveUploadedFile(file, dst); err != nil {
        c.JSON(500, gin.H{"error": "save failed"})
        return
    }
    c.JSON(200, gin.H{"path": dst})
})
```

---

### 2. Size & MIME Validation Before Save

```go
const maxMemory = 32 << 20 // 32 MB

r.POST("/doc", func(c *gin.Context) {
    form, _ := c.MultipartForm()
    files := form.File["docs"]
    for _, f := range files {
        if f.Size > 5<<20 {
            c.JSON(413, gin.H{"error": "file too large"})
            return
        }
        if f.Header.Get("Content-Type") != "application/pdf" {
            c.JSON(415, gin.H{"error": "only PDF allowed"})
            return
        }
    }
    c.JSON(200, gin.H{"count": len(files)})
})
```

---

### 3. Middleware Using `ShouldBind`

```go
type UploadForm struct {
    Title string                `form:"title" binding:"required"`
    File  *multipart.FileHeader `form:"file" binding:"required"`
}

func ValidateUpload() gin.HandlerFunc {
    return func(c *gin.Context) {
        var uf UploadForm
        if err := c.ShouldBind(&uf); err != nil {
            c.AbortWithStatusJSON(400, gin.H{"error": err.Error()})
            return
        }
        c.Set("upload", uf)
        c.Next()
    }
}

r.Use(ValidateUpload())
r.POST("/upload", func(c *gin.Context) {
    uf := c.MustGet("upload").(UploadForm)
    c.JSON(200, gin.H{"title": uf.Title})
})
```

---

### 4. Direct Save with `SaveUploadedFile`

```go
r.POST("/cover", func(c *gin.Context) {
    file, _ := c.FormFile("cover")
    dst := filepath.Join("covers", uuid.New().String()+filepath.Ext(file.Filename))
    _ = c.SaveUploadedFile(file, dst)
    c.JSON(200, gin.H{"cover": dst})
})
```

---

### 5. Streaming Large Files with `MultipartReader`

```go
r.POST("/stream", func(c *gin.Context) {
    reader, _ := c.Request.MultipartReader()
    part, _ := reader.NextPart()
    for part != nil && part.FormName() != "bigfile" {
        part, _ = reader.NextPart()
    }
    if part == nil {
        c.JSON(400, gin.H{"error": "missing bigfile"})
        return
    }
    defer part.Close()
    written, _ := io.Copy(os.Stdout, part) // stream to stdout
    c.JSON(200, gin.H{"bytes": written})
})
```

---

### 6. Sanitized Filename & Directory

```go
r.POST("/image", func(c *gin.Context) {
    file, _ := c.FormFile("img")
    safe := uuid.New().String() + filepath.Ext(filepath.Base(file.Filename))
    _ = c.SaveUploadedFile(file, filepath.Join("img", safe))
    c.JSON(200, gin.H{"filename": safe})
})
```

---

### 7. Mixed Form-Data Binding

```go
type Article struct {
    Title   string                  `form:"title" binding:"required"`
    Body    string                  `form:"body"`
    Attach  *multipart.FileHeader   `form:"attach"`
    Gallery []*multipart.FileHeader `form:"gallery[]"`
}

r.POST("/post", func(c *gin.Context) {
    var art Article
    _ = c.ShouldBind(&art)
    c.JSON(200, gin.H{
        "title":   art.Title,
        "attach":  art.Attach.Filename,
        "gallery": len(art.Gallery),
    })
})
```

---

### 8. Async Virus Scanning

```go
r.POST("/scan", func(c *gin.Context) {
    file, _ := c.FormFile("file")
    dst := filepath.Join("tmp", file.Filename)
    _ = c.SaveUploadedFile(file, dst)

    go func(path string) {
        // expensive scan
        _ = exec.Command("clamscan", path).Run()
        _ = os.Remove(path)
    }(dst)

    c.JSON(202, gin.H{"status": "scanning"})
})
```