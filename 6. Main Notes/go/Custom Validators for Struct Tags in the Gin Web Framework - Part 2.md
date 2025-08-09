# Custom Validators for Struct Tags in the Gin Web Framework - Part 2

2025-07-30 00:31
Status: #DONE 
Tags: [[Gin]]

---
## Custom Validators via Struct Tags in the Gin Web Framework: A Comprehensive Reference

| Method / Function | Explanation |
|---|---|
| `binding.Validator.Engine()` | Returns the underlying `*validator.Validate` instance provided by go-playground/validator. |
| `engine.RegisterValidation(tag, fn, callValidationEvenIfNull ...bool)` | Registers a Go function as a reusable validation rule linked to a struct-tag key. |
| `engine.RegisterAlias(alias, tags)` | Creates a composite alias that expands to multiple built-in or custom validators. |
| `RegisterTranslation(tag, trans, registerFn, translateFn)` | Maps a validator tag to a human-readable error message (i18n ready). |
| `StructLevel` validator | Allows cross-field or cross-struct validation by receiving the whole struct instance. |
| `FieldLevel` validator | Operates on a single field value; receives `FieldLevel` interface. |
| `validation.FieldError` | Interface returned by validation failures; exposes field, tag, param, namespace, and translated message. |
| `binding:"-"` | Explicitly skips validation for a field. |
| `binding:"<custom_tag>"` | Invokes the registered custom validator when the tag is present. |
| `binding:"<custom_tag>=param"` | Passes a parameter string to the custom validator. |

---

### Tips, Tricks & Best Practices

| Tip | Rationale | Example Reference |
|---|---|---|
| Register validators once at `init()` | Prevents race conditions and keeps registration centralized | Example 1 |
| Use `RegisterAlias` for common rule sets | Reduces tag verbosity and enforces consistency | Example 2 |
| Prefer `FieldLevel` for single-value checks | Simpler signature and higher performance | Example 3 |
| Use `StructLevel` for cross-field logic | Access sibling fields without reflection hacks | Example 4 |
| Cache compiled regex inside validators | Eliminates repeated compilation overhead | Example 5 |
| Provide translations for all custom tags | Enables clean localization of error messages | Example 6 |
| Include unit tests for every validator | Detects silent regressions early | Example 7 |
| Document accepted parameter syntax | Prevents client confusion and support overhead | Example 8 |
| Combine tag with `omitempty` when optional | Avoids unnecessary validation on zero values | Example 9 |
| Chain multiple custom tags with commas | Creates expressive, declarative rules | Example 10 |

---

### 1. Single-Field Custom Validator Registration

```go
package main

import (
    "regexp"
    "github.com/gin-gonic/gin"
    "github.com/gin-gonic/gin/binding"
    "github.com/go-playground/validator/v10"
)

func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterValidation("e164", func(fl validator.FieldLevel) bool {
            return regexp.MustCompile(`^\+?[1-9]\d{1,14}$`).MatchString(fl.Field().String())
        })
    }
}

type Contact struct {
    Phone string `json:"phone" binding:"required,e164"`
}

func main() {
    r := gin.Default()
    r.POST("/contact", func(c *gin.Context) {
        var ct Contact
        if err := c.ShouldBindJSON(&ct); err != nil {
            c.JSON(400, gin.H{"error": err.Error()})
            return
        }
        c.JSON(200, gin.H{"phone": ct.Phone})
    })
    r.Run()
}
```

---

### 2. Alias for Reusable Rule Set

```go
func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterAlias("short_strong_password", "min=8,max=32,alphanum,containsany=!@#$%")
    }
}

type Cred struct {
    Password string `json:"password" binding:"required,short_strong_password"`
}
```

---

### 3. Field-Level Validator with Parameter

```go
func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterValidation("divisible_by", func(fl validator.FieldLevel) bool {
            divisor := parseInt(fl.Param())
            return fl.Field().Int()%divisor == 0
        })
    }
}

type Payload struct {
    Amount int `json:"amount" binding:"required,divisible_by=100"`
}
```

---

### 4. Cross-Field Validation via StructLevel

```go
type Schedule struct {
    Start time.Time `json:"start" binding:"required"`
    End   time.Time `json:"end" binding:"required"`
}

func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterStructValidation(func(sl validator.StructLevel) {
            s := sl.Current().Interface().(Schedule)
            if s.End.Before(s.Start) {
                sl.ReportError(s.End, "End", "end", "end_after_start", "")
            }
        }, Schedule{})
    }
}
```

---

### 5. Cached Regex Validator

```go
var isoOnce sync.Once
var isoRegex *regexp.Regexp

func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        isoOnce.Do(func() {
            isoRegex = regexp.MustCompile(`^[A-Z]{2}\d{4}$`)
        })
        _ = v.RegisterValidation("iso_code", func(fl validator.FieldLevel) bool {
            return isoRegex.MatchString(fl.Field().String())
        })
    }
}

type Country struct {
    Code string `json:"code" binding:"required,iso_code"`
}
```

---

### 6. Translated Error Messages

```go
import ut "github.com/go-playground/universal-translator"

func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterValidation("even", func(fl validator.FieldLevel) bool {
            return fl.Field().Int()%2 == 0
        })

        en := ut.New(en.New()).GetFallback()
        _ = v.RegisterTranslation("even", en,
            func(ut ut.Translator) error { return ut.Add("even", "{0} must be even", true) },
            func(ut ut.Translator, fe validator.FieldError) string {
                t, _ := ut.T("even", fe.Field())
                return t
            },
        )
    }
}
```

---

### 7. Unit-Tested Validator

```go
func TestCreditCard(t *testing.T) {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterValidation("luhn", func(fl validator.FieldLevel) bool {
            return luhn.Valid(fl.Field().String())
        })
    }
    var cc struct{ Card string `binding:"luhn"` }
    assert.Error(t, binding.Validator.ValidateStruct(&cc))
}
```

---

### 8. Parameter Documentation

```go
func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterValidation("enum", func(fl validator.FieldLevel) bool {
            allowed := strings.Split(fl.Param(), " ")
            for _, a := range allowed {
                if a == fl.Field().String() {
                    return true
                }
            }
            return false
        })
    }
}

type Request struct {
    Status string `json:"status" binding:"required,enum=pending active closed"`
}
```

---

### 9. Optional Field with `omitempty`

```go
type PatchUser struct {
    Nickname *string `json:"nickname,omitempty" binding:"omitempty,min=3"`
}
```

---

### 10. Chained Custom Tags

```go
func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        _ = v.RegisterValidation("no_spaces", func(fl validator.FieldLevel) bool {
            return !strings.Contains(fl.Field().String(), " ")
        })
    }
}

type User struct {
    Username string `json:"username" binding:"required,no_spaces,min=4"`
}
```

### How Struct-Level Validators Work in Gin  

A **Struct-Level validator** is registered once (typically in `init`) and is invoked **after** all field-level validators succeed.  
It receives a `validator.StructLevel` interface (`sl`) that exposes:  

- `sl.Current()` – the struct instance (value).  
- `sl.ReportError(...)` – registers a validation failure.  
- `sl.Top()` – root struct (useful for cross-struct checks).  

Because Gin unwraps pointers before calling the validator, **modifications made to the value inside the function are not reflected back** unless the struct is passed as a pointer and you use a workaround (e.g., re-binding manually) .  

### Example: Cross-Field Validation via Struct-Level  

```go
type Schedule struct {
	Start time.Time `form:"start" binding:"required"`
	End   time.Time `form:"end" binding:"required"`
}

func init() {
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		v.RegisterStructValidation(func(sl validator.StructLevel) {
			s := sl.Current().Interface().(Schedule)
			if s.End.Before(s.Start) {
				sl.ReportError(s.End, "End", "end", "end_after_start", "")
			}
		}, Schedule{})
	}
}
```  

### Example: Chaining Multiple Custom Tags  

```go
func init() {
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		// register two custom validators
		v.RegisterValidation("no_spaces", func(fl validator.FieldLevel) bool {
			return !strings.Contains(fl.Field().String(), " ")
		})
		v.RegisterValidation("starts_upper", func(fl validator.FieldLevel) bool {
			r, _ := utf8.DecodeRuneInString(fl.Field().String())
			return unicode.IsUpper(r)
		})
	}
}

type User struct {
	Username string `json:"username" binding:"required,no_spaces,starts_upper"`
}
```
Handling Nested-Struct Validation  
Gin’s default validator (go-playground/validator) automatically descends into **embedded** or **pointer-to-struct** fields.  
Rules:  
1. Tags on the parent field propagate to the child.  
2. Validation is evaluated **depth-first**; any failure on a leaf field aborts further descent.  
3. Use `binding:"dive"` on slices/maps to apply rules to every element.  

Best-practice pattern:  
```go
type Address struct {
    City string `json:"city" binding:"required"`
}

type User struct {
    Name    string  `json:"name" binding:"required"`
    Address Address `json:"address" binding:"required"` // nested struct
}
```

---

Field-Level vs Struct-Level Validators  

| Aspect | Field-Level | Struct-Level |
|---|---|---|
| **Scope** | Single field value | Entire struct (cross-field logic) |
| **Signature** | `func(fl validator.FieldLevel) bool` | `func(sl validator.StructLevel)` |
| **Access** | `fl.Field()` only | `sl.Current()`, `sl.ReportError(...)` |
| **Registration** | `RegisterValidation(tag, fn)` | `RegisterStructValidation(fn, T)` |
| **Use-case** | Regex, enum, range | Business rules spanning ≥2 fields |

---

Date Validation – Best Practice  

1. **Tag** the field with `time_format` (layout) and `binding:"required"`.  
2. **Register** a reusable validator for **semantic** checks (e.g., “date must be in future”).  
3. **Store** as `time.Time` to avoid string parsing in handlers.  

Example:

```go
func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        v.RegisterValidation("future", func(fl validator.FieldLevel) bool {
            t := fl.Field().Interface().(time.Time)
            return t.After(time.Now())
        })
    }
}

type Event struct {
    Date time.Time `json:"date" time_format:"2006-01-02" binding:"required,future"`
}
```

This pattern centralizes layout and business rules, provides clear errors, and keeps handlers free of manual date parsing.

Applying custom validation to nested structs in Gin is done **once at registration** and automatically recurses; no extra tags are needed beyond `dive` for collections.

### 1. Register a Struct-Level validator for the nested type

```go
type Address struct {
    City  string `json:"city" binding:"required"`
    ZIP   string `json:"zip"`
}

func init() {
    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
        v.RegisterStructValidation(func(sl validator.StructLevel) {
            a := sl.Current().Interface().(Address)
            if a.City == "Berlin" && !strings.HasPrefix(a.ZIP, "1") {
                sl.ReportError(a.ZIP, "ZIP", "zip", "berlin_zip", "")
            }
        }, Address{})
    }
}
```

### 2. Embed or reference the nested struct

```go
type User struct {
    Name    string  `json:"name" binding:"required"`
    Address Address `json:"address" binding:"required"` // validation runs automatically
}
```

### 3. For slices/maps of nested structs

```go
type Order struct {
    Items []Item `json:"items" binding:"required,dive"` // `dive` applies rules to each element
}
```

The validator descends into **every** `Address` or `Item` encountered, executing the registered struct-level logic without additional code in handlers.