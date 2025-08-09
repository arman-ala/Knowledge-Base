# Understanding JWT: A Comprehensive Guide with Go Examples

2025-05-18 12:38
Status: 
Tags: [[Go]]

---
## Introduction

In modern web development, securing communication between clients and servers is paramount. One of the most popular methods for achieving this is through **JSON Web Tokens (JWT)**. JWTs provide a compact, secure way to transmit information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs are widely used for authentication and authorization in web applications due to their stateless nature and scalability.

In this guide, we will explore JWT in depth: what it is, how it works, and how to implement it using Go. We will also cover security considerations and best practices to ensure your JWT implementation is robust and secure.

---

## What is JWT?

**JSON Web Token (JWT)** is an open standard (RFC 7519) that defines a compact and self-contained way to securely transmit information between parties as a JSON object. This information is digitally signed, ensuring that it can be verified and trusted. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair (using RSA or ECDSA).

### Why Use JWT?

- **Stateless Authentication**: JWTs allow servers to be stateless because the token contains all the necessary information to verify the user. This eliminates the need for server-side storage of session data.
- **Scalability**: Since JWTs are self-contained, they are ideal for microservices architectures where multiple services need to verify user identity independently.
- **Cross-Domain Compatibility**: JWTs can be used across different domains, making them suitable for single sign-on (SSO) scenarios.

---

## Structure of a JWT

A JWT consists of three parts, separated by dots (`.`):

1. **Header**
2. **Payload**
3. **Signature**

These parts are Base64Url encoded and concatenated to form the JWT.

### 1. Header

The header typically contains two fields:

- `alg`: The signing algorithm used (e.g., HMAC SHA256 or RSA).
- `typ`: The type of token, which is `"JWT"`.

**Example:**

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

This JSON is then Base64Url encoded to form the first part of the JWT.

### 2. Payload

The payload contains the **claims**, which are statements about an entity (usually the user) and additional data. There are three types of claims:

- **Registered Claims**: Predefined claims like `iss` (issuer), `exp` (expiration time), `sub` (subject), and `aud` (audience).
- **Public Claims**: Custom claims that should be defined in the IANA JWT Registry or as a URI to avoid collisions.
- **Private Claims**: Custom claims agreed upon by the parties using the JWT.

**Example:**

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

This JSON is Base64Url encoded to form the second part of the JWT.

### 3. Signature

The signature is created by taking the encoded header, the encoded payload, and a secret key, then signing them using the algorithm specified in the header.

For HMAC SHA256, the signature is generated as:

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

The final JWT is the concatenation of the three parts:  
`<header>.<payload>.<signature>`

**Example JWT:**  
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`

---

## How JWT Works in Authentication

JWTs are commonly used for authentication in web applications. Here’s a step-by-step overview of the process:

1. **User Login**: The user provides their credentials (e.g., username and password) to the server.
2. **Token Creation**: If the credentials are valid, the server generates a JWT containing user information (e.g., user ID) and signs it with a secret key.
3. **Token Storage**: The client stores the JWT (typically in local storage or a cookie).
4. **Subsequent Requests**: For each request to a protected resource, the client includes the JWT in the `Authorization` header as `Bearer <token>`.
5. **Token Verification**: The server verifies the token’s signature and checks its claims (e.g., expiration time). If valid, the server processes the request.

This process ensures that the server can trust the token without needing to store session data.

---

## Implementing JWT in Go

To implement JWT in Go, we will use the `github.com/golang-jwt/jwt/v4` library, a popular and well-maintained package for handling JWTs.

### Step 1: Install the Library

```bash
go get github.com/golang-jwt/jwt/v4
```

### Step 2: Generate a JWT

We’ll define custom claims and generate a token using the HS256 algorithm.

```go
package main

import (
    "fmt"
    "time"
    "github.com/golang-jwt/jwt/v4"
)

type Claims struct {
    Username string `json:"username"`
    jwt.RegisteredClaims
}

func GenerateToken(username string, secret string) (string, error) {
    claims := Claims{
        Username: username,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            NotBefore: jwt.NewNumericDate(time.Now()),
        },
    }
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(secret))
}

func main() {
    token, err := GenerateToken("johndoe", "my_secret_key")
    if err != nil {
        fmt.Println("Error generating token:", err)
        return
    }
    fmt.Println("Generated Token:", token)
}
```

### Step 3: Verify a JWT

To verify a token, we parse it and check its signature and claims.

```go
func VerifyToken(tokenString string, secret string) (*Claims, error) {
    claims := &Claims{}
    token, err := jwt.ParseWithClaims(tokenString, claims, func(token *jwt.Token) (interface{}, error) {
        return []byte(secret), nil
    })
    if err != nil {
        return nil, err
    }
    if !token.Valid {
        return nil, fmt.Errorf("invalid token")
    }
    return claims, nil
}

func main() {
    // Assuming token is received from client
    claims, err := VerifyToken(token, "my_secret_key")
    if err != nil {
        fmt.Println("Invalid token:", err)
        return
    }
    fmt.Println("Username:", claims.Username)
}
```

---

## Examples

### Example 1: Simple Authentication Flow

1. **Login Handler**: Verifies user credentials and generates a JWT.
2. **Protected Handler**: Verifies the JWT from the `Authorization` header and grants access if valid.

```go
package main

import (
    "encoding/json"
    "net/http"
    "strings"
)

var secret = "my_secret_key"

func LoginHandler(w http.ResponseWriter, r *http.Request) {
    username := r.FormValue("username")
    password := r.FormValue("password")
    // Simulate credential verification
    if username == "johndoe" && password == "password123" {
        token, _ := GenerateToken(username, secret)
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]string{"token": token})
    } else {
        http.Error(w, "Invalid credentials", http.StatusUnauthorized)
    }
}

func ProtectedHandler(w http.ResponseWriter, r *http.Request) {
    authHeader := r.Header.Get("Authorization")
    if authHeader == "" {
        http.Error(w, "Missing token", http.StatusUnauthorized)
        return
    }
    parts := strings.Split(authHeader, " ")
    if len(parts) != 2 || parts[0] != "Bearer" {
        http.Error(w, "Invalid token format", http.StatusUnauthorized)
        return
    }
    claims, err := VerifyToken(parts[1], secret)
    if err != nil {
        http.Error(w, "Invalid token", http.StatusUnauthorized)
        return
    }
    w.Write([]byte("Welcome, " + claims.Username))
}
```

### Example 2: Token with Custom Claims

You can extend the `Claims` struct to include additional information, such as user roles.

```go
type CustomClaims struct {
    Username string `json:"username"`
    Role     string `json:"role"`
    jwt.RegisteredClaims
}

// Generate token with custom claims
claims := CustomClaims{
    Username: "johndoe",
    Role:     "admin",
    RegisteredClaims: jwt.RegisteredClaims{
        ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
    },
}
token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
```

---

## Security Considerations and Best Practices

While JWTs are powerful, they must be implemented carefully to ensure security:

1. **Do Not Store Sensitive Data in JWT**: Since the payload is only Base64 encoded, it can be decoded easily. Avoid including sensitive information like passwords.
2. **Use HTTPS**: Always transmit JWTs over HTTPS to prevent man-in-the-middle attacks.
3. **Set Reasonable Expiration Times**: Short expiration times reduce the risk if a token is compromised. Use refresh tokens for longer sessions.
4. **Secure the Secret Key**: The signing key must be kept secret. If compromised, attackers can forge tokens.
5. **Validate Claims**: Always verify claims like `exp` (expiration) and `aud` (audience) to ensure the token is valid for the intended use.
6. **Prevent XSS Attacks**: If storing JWTs in local storage, ensure your application is protected against cross-site scripting (XSS) attacks, as they can steal tokens.
7. **Consider Token Revocation**: For critical applications, maintain a blacklist of revoked tokens, though this introduces statefulness.

---

## Conclusion

JSON Web Tokens (JWT) provide a robust, stateless solution for authentication and authorization in modern web applications. By understanding their structure, implementation, and security considerations, developers can leverage JWTs to build scalable and secure systems. In this guide, we’ve covered the fundamentals of JWT, demonstrated how to implement it in Go, and highlighted best practices to ensure your authentication mechanism is both effective and secure.

For further reading, refer to the [JWT official documentation](https://jwt.io/introduction) and the [RFC 7519 specification](https://tools.ietf.org/html/rfc7519).