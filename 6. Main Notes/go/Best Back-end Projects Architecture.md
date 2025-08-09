# Designing a Scalable Go Project Architecture: A Practical Guide 🚀

2025-05-01 17:52  
Status: #STARTED  
Tags: [[Go]]

---

If you’re diving into building a back-end project with Go, one of the first hurdles you’ll face is figuring out how to structure your codebase. A well-organized project isn’t just about keeping things tidy—it’s about making your code scalable, maintainable, and easy to collaborate on. In this post, I’ll walk you through a practical approach to designing a Go project’s architecture, breaking down each component with examples and explaining _why_ this structure works. Whether you’re building a small API or a complex microservice, this guide will give you a solid foundation. Let’s get started! 🛠️

---

## Why Architecture Matters in Go Projects

Go is loved for its simplicity and performance, but without a clear structure, even a small project can become a tangled mess. A good architecture:

- **Improves maintainability**: You can quickly find and update code.
- **Enhances scalability**: Adding new features or services is painless.
- **Facilitates collaboration**: Team members can jump in without deciphering a chaotic codebase.
- **Supports testing**: Isolated components are easier to test.

The structure I’m sharing here is inspired by real-world Go projects and follows principles like separation of concerns and dependency injection. It’s flexible enough to adapt to different project sizes while keeping things clean. Let’s dive into the key components of the architecture.

---

## 1. The `entity` Package: Defining Your Core Data Models 📋

The `entity` package is where you define the core data structures (or structs) that represent the key objects in your system. Think of these as the building blocks of your application—whether it’s a `User`, `Order`, or `Product`. Each struct comes with its fields and, sometimes, methods to encapsulate behavior.

### Why Use an `entity` Package?

- **Centralized models**: All your data structures live in one place, making them easy to find and update.
- **Reusability**: These structs can be used across different parts of your application (services, repositories, etc.).
- **Encapsulation**: By attaching methods to structs, you can keep related logic close to the data.

### Example: Creating a `User` Entity

Let’s say we’re building a user management system. We’ll define a `User` struct in the `entity` package.

```bash
cd entity
touch user.go
```

Here’s what the `user.go` file might look like:

```go
package entity

import (
    "time"
)

// User represents a user in the system.
type User struct {
    ID          uint
    PhoneNumber string
    Name        string
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```

This struct is simple but powerful. It captures the essential fields for a user and uses Go’s `time.Time` for timestamps. Notice that we’re keeping it minimal—no validation logic or database-specific code here. That’s deliberate! The `entity` package should focus solely on defining the data model, leaving behavior and persistence to other layers.

---

## 2. The `pkg` Package: Reusable Utilities and Custom Types 🧰

The `pkg` package is your toolbox for reusable code that doesn’t belong to a specific domain but is used across your project. This could include validation functions, custom types, or utility helpers. For example, you might want to validate a user’s name or phone number in a consistent way.

### Why Use a `pkg` Package?

- **Reusability**: Avoid duplicating code by centralizing shared logic.
- **Modularity**: Keep utilities separate from business logic.
- **Testability**: Isolated utilities are easier to test.

### Example: Validating a User’s Name

Let’s create a custom `Name` type with validation logic. We’ll store it in the `pkg/name` directory.

```bash
cd pkg
mkdir name
cd name
touch name.go
```

Here’s the code for `name.go`:

```go
package name

import "errors"

// ErrInvalidNameLength is returned when the name is too short.
var ErrInvalidNameLength = errors.New("INVALID_NAME: name length must be more than 3")

// Name is a custom string type for user names.
type Name string

// IsValid checks if the name meets the length requirement.
func (n Name) IsValid() (bool, error) {
    if len(n) < 3 {
        return false, ErrInvalidNameLength
    }
    return true, nil
}
```

Alternatively, if you don’t need a custom type, you could write a standalone validation function:

```go
package name

import "errors"

// ErrInvalidNameLength is returned when the name is too short.
var ErrInvalidNameLength = errors.New("INVALID_NAME: name length must be more than 3")

// IsValid checks if the name meets the length requirement.
func IsValid(name string) (bool, error) {
    if len(name) < 3 {
        return false, ErrInvalidNameLength
    }
    return true, nil
}
```

Both approaches work, but the custom type (`Name`) is great when you want to attach methods or enforce type safety. For example, you could add more methods like `ToUpper()` or `TrimSpace()` to the `Name` type later. The `pkg` package is also a good place for other utilities, like phone number validators or logging helpers.

---

## 3. The `service` Package: Implementing Business Logic ⚙️

The `service` package is where your application’s business logic lives. Each service corresponds to a specific domain (e.g., user management, order processing) and implements the use cases defined during your system design phase. Services act as the glue between your entities and external systems (like databases or APIs).

### Why Use a `service` Package?

- **Separation of concerns**: Business logic is isolated from data models and storage.
- **Testability**: Services are easy to mock and test since they depend on interfaces.
- **Reusability**: Services can be reused across different parts of the application (e.g., HTTP handlers, CLI commands).

### Example: Building a User Registration Service

Let’s implement a user registration service using a more detailed example. We’ll create a `userservice` subdirectory under `service`.

```bash
cd service
mkdir userservice
cd userservice
touch service.go
```

Here’s the complete code for `service.go`:

```go
package userservice

import (
    "fmt"
    "go_cast/S11P01-game/entity"
    "go_cast/S11P01-game/pkg/name"
    "go_cast/S11P01-game/pkg/phonenumber"
)

type Service struct {
    repo Repository
}

type Repository interface {
    // usually the storage is external service and we may face some errors
    // so we need to return error too
    IsPhoneNumberUnique(phoneNumber string) (bool, error)
    Register(user entity.User) (entity.User, error)
}

type RegisterRequest struct {
    Name        string
    PhoneNumber string
}

type RegisterResponse struct {
    User entity.User
}

func New(repo Repository) Service {
    return Service{
        repo: repo,
    }
}

func (s Service) Register(req RegisterRequest) (res RegisterResponse, err error) {
    // TODO - we should verify the phone number by verifying the code sent to the phone number

    // phone number validation
    err = phonenumber.IsValid(req.PhoneNumber)
    if err != nil {
        return
    }

    // check phone number uniqueness
    if isUnique, err := s.repo.IsPhoneNumberUnique(req.PhoneNumber); err != nil || !isUnique {
        if err != nil {
            return RegisterResponse{}, err
        }
        if !isUnique {
            return RegisterResponse{}, fmt.Errorf("phone number is not unique")
        }
    }

    // validate name
    if isValid, err := name.IsValid(req.Name); err != nil || !isValid {
        if err != nil {
            return RegisterResponse{}, err
        }
        if !isValid {
            return RegisterResponse{}, fmt.Errorf("name is not valid")
        }
    }
    // create new user
    user := entity.User{
        ID:          0,
        Name:        req.Name,
        PhoneNumber: req.PhoneNumber,
    }
    // create new user in the storage (file, database, etc.)
    if createdUser, err := s.repo.Register(user); err != nil {
        return RegisterResponse{}, fmt.Errorf("unexpected error: %v", err)
    } else {
        res.User = createdUser
    }
    // return created user
    return res, nil
}
```

### Breaking Down the Code

1. **Service Struct**: The `Service` struct holds a `Repository` interface, which allows us to inject different storage implementations (e.g., MySQL, in-memory, or mock for testing). This is an example of dependency injection, which makes the service layer more flexible and easier to test.
    
2. **Repository Interface**: This defines the contract for storage operations (`IsPhoneNumberUnique` and `Register`). By using an interface, we decouple the service from the storage layer, meaning we can change the underlying storage without modifying the service code.
    
3. **Request/Response Structs**: `RegisterRequest` and `RegisterResponse` clearly define the input and output of the `Register` method, making the API explicit and easy to understand.
    
4. **Business Logic**: The `Register` method performs several steps:
    
    - **Phone Number Validation**: It uses the `phonenumber.IsValid` function from the `pkg/phonenumber` package to check if the phone number is valid. If not, it returns an error early using Go’s implicit return feature (since `err` is a named return value).
    - **Uniqueness Check**: It calls the repository’s `IsPhoneNumberUnique` method to ensure the phone number isn’t already in use. If there’s an error (e.g., database failure) or the number isn’t unique, it returns an appropriate error.
    - **Name Validation**: It uses the `name.IsValid` function from the `pkg/name` package to validate the user’s name, returning an error if validation fails.
    - **User Creation**: If all validations pass, it creates a new `entity.User` struct with the provided name and phone number. The `ID` is set to 0 since the repository will assign it.
    - **Saving the User**: It calls the repository’s `Register` method to save the user to the storage system (e.g., a database). If successful, it assigns the created user to the response; otherwise, it wraps the error with context for debugging.

This structure ensures that the service layer is clean, testable, and independent of the underlying storage. For example, you could easily swap out a MySQL repository for a PostgreSQL one without touching the service code. Additionally, the use of interfaces makes it straightforward to mock the repository for unit testing.

---

## 4. The `repository` Package: Handling Data Persistence 💾

The `repository` package is the bridge between your application’s business logic and its data storage, whether that’s a database, file system, or external API. Each repository is tailored to a specific entity (like `User`) and implements the methods defined in the service’s `Repository` interface. This abstraction ensures that the business logic (in the `service` package) remains independent of the storage mechanism, making it easy to switch databases or mock data for testing.

### Why Use a `repository` Package?

- **Abstraction**: By hiding the details of the storage layer, repositories make it simple to swap out one storage system for another (e.g., MySQL to PostgreSQL) without changing the service layer.
- **Single Responsibility**: Repositories focus solely on data access, keeping business logic and validation out of the storage layer.
- **Testability**: The use of interfaces allows you to mock repositories for unit tests, ensuring that tests are fast and independent of external systems.
- **Consistency**: Standardized interfaces ensure that data operations are predictable and uniform across the application.

### Example: MySQL Repository for Users

To demonstrate, let’s implement a MySQL-based repository for our `User` entity. This repository will handle checking phone number uniqueness and saving new users to a MySQL database. We’ll organize the code in a `mysql` subdirectory under `repository`.

```bash
cd repository
mkdir mysql
cd mysql
touch db.go
touch user.go
```

#### Step 1: Setting Up the Database Connection (`db.go`)

The `db.go` file is responsible for initializing and configuring the MySQL database connection. This is a critical step because a poorly configured connection can lead to performance issues, such as connection leaks or timeouts.

```go
package mysql

import (
    "database/sql"
    "fmt"
    "time"
    _ "github.com/go-sql-driver/mysql"
)

// MySQLDB holds the database connection.
type MySQLDB struct {
    db *sql.DB
}

// New initializes a new MySQL database connection.
func New() *MySQLDB {
    connection, err := sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        panic(fmt.Errorf("cannot open mysql db: %w", err))
    }

    // Configure connection pooling
    connection.SetConnMaxLifetime(time.Minute * 3)
    connection.SetMaxOpenConns(10)
    connection.SetMaxIdleConns(10)

    return &MySQLDB{db: connection}
}
```

**What Happened and Why?**

- **Database Connection**: The `sql.Open` function establishes a connection to the MySQL database using the connection string `"user:password@/dbname"`. The `_` import of `github.com/go-sql-driver/mysql` registers the MySQL driver with Go’s `database/sql` package, allowing us to use MySQL without directly referencing the driver in our code. This is a common Go idiom for database drivers.
- **Error Handling**: If the connection fails (e.g., due to incorrect credentials or an unavailable database), we panic with a wrapped error. Panicking is appropriate here because a failure to connect to the database is a critical error that prevents the application from starting. In a production environment, you might want to handle this more gracefully (e.g., retrying or exiting with a status code).
- **Connection Pooling**: The `SetConnMaxLifetime`, `SetMaxOpenConns`, and `SetMaxIdleConns` methods configure the connection pool:
    - `SetConnMaxLifetime(time.Minute * 3)` ensures that connections are closed after 3 minutes to prevent stale connections, which is particularly important for long-running applications.
    - `SetMaxOpenConns(10)` limits the number of open connections to 10, preventing the application from overwhelming the database with too many simultaneous connections.
    - `SetMaxIdleConns(10)` allows up to 10 idle connections to remain open, improving performance by reusing existing connections instead of opening new ones for each query.
- **Why This Matters**: Proper connection management is crucial for performance and reliability. Without these settings, the application could exhaust database resources, leading to errors or slowdowns. The defaults provided here are reasonable for a small to medium-sized application, but you’d adjust them based on your database’s capacity and workload.

#### Step 2: Implementing the User Repository (`user.go`)

The `user.go` file implements the `Repository` interface defined in the `userservice` package. This includes methods to check if a phone number is unique and to register a new user.

```go
package mysql

import (
    "go_cast/S11P01-game/entity"
)

// IsPhoneNumberUnique checks if the phone number is already in use.
func (d *MySQLDB) IsPhoneNumberUnique(phoneNumber string) (bool, error) {
    var count int
    err := d.db.QueryRow("SELECT COUNT(*) FROM users WHERE phone_number = ?", phoneNumber).Scan(&count)
    if err != nil {
        return false, fmt.Errorf("failed to check phone number: %w", err)
    }
    return count == 0, nil
}

// Register saves a new user to the database.
func (d *MySQLDB) Register(user entity.User) (entity.User, error) {
    result, err := d.db.Exec(
        "INSERT INTO users (phone_number, name, created_at, updated_at) VALUES (?, ?, ?, ?)",
        user.PhoneNumber, user.Name, user.CreatedAt, user.UpdatedAt,
    )
    if err != nil {
        return entity.User{}, fmt.Errorf("failed to insert user: %w", err)
    }

    id, err := result.LastInsertId()
    if err != nil {
        return entity.User{}, fmt.Errorf("failed to get last insert ID: %w", err)
    }

    user.ID = uint(id)
    return user, nil
}
```

**What Happened and Why?**

- **Checking Phone Number Uniqueness (`IsPhoneNumberUnique`)**:
    - **Query Execution**: The `QueryRow` method executes a SQL query to count the number of rows in the `users` table where the `phone_number` matches the input. The `?` placeholder prevents SQL injection by safely binding the `phoneNumber` parameter.
    - **Result Scanning**: The `Scan` method extracts the count into the `count` variable. If the query fails (e.g., due to a database error or network issue), an error is returned, wrapped with context for easier debugging.
    - **Logic**: If `count == 0`, the phone number is unique, and the method returns `true`. Otherwise, it returns `false`. This ensures that the service layer can enforce unique phone numbers for users.
    - **Why This Approach?**: Using a `SELECT COUNT(*)` query is a standard way to check for existence in a database. It’s efficient for small to medium datasets, though for very large tables, you might optimize with an index on `phone_number` to speed up lookups.
- **Registering a User (`Register`)**:
    - **Insert Query**: The `Exec` method runs an `INSERT` query to add a new user to the `users` table, binding the user’s `PhoneNumber`, `Name`, `CreatedAt`, and `UpdatedAt` fields to the query’s placeholders. This ensures safe and efficient data insertion.
    - **Error Handling**: If the insertion fails (e.g., due to a duplicate phone number or database downtime), the method returns an empty `entity.User` and a wrapped error. This allows the service layer to handle the error appropriately (e.g., by informing the user that the phone number is already taken).
    - **Retrieving the Inserted ID**: The `LastInsertId` method retrieves the auto-incremented ID assigned to the new user by the database. This ID is critical because it uniquely identifies the user and is used in subsequent operations (e.g., fetching the user later).
    - **Updating the User Struct**: The `user.ID` field is updated with the retrieved ID, and the modified `user` struct is returned. This ensures that the service layer receives the complete user entity, including the database-generated ID.
    - **Why This Matters**: The `Register` method encapsulates the complexity of database operations, providing a clean interface for the service layer. By returning the updated `user` struct, it ensures that the service layer has all the necessary data (like the ID) without needing to query the database again.
- **Database Schema Assumptions**: The code assumes a `users` table with columns `phone_number`, `name`, `created_at`, and `updated_at`, and an auto-incrementing `id` column. In a real project, you’d need to ensure this schema exists (e.g., via a migration script) before running the application.

---

## Tying It All Together: The `main.go` File

Now that we have our `entity`, `pkg`, `service`, and `repository` packages set up, it’s time to bring everything together in the `main.go` file. This file serves as the entry point for our application and is responsible for setting up the HTTP server, handling routes, and wiring up the dependencies.

### Example: Setting Up the HTTP Server

We’ll place the `main.go` file in the `cmd` directory to keep it separate from the business logic.

```bash
mkdir cmd
cd cmd
touch main.go
```

Here’s the code for `main.go`:

```go
package main

import (
    "encoding/json"
    "fmt"
    "go_cast/S11P01-game/repository/mysql"
    "go_cast/S11P01-game/service/userservice"
    "io"
    "net/http"
)

func main() {
    http.HandleFunc("/users/register", userRegisterHandler)
    http.HandleFunc("/health-check", healthCheckHandler)
    http.ListenAndServe(":8080", nil)
}

func userRegisterHandler(res http.ResponseWriter, req *http.Request) {
    if req.Method != http.MethodPost {
        res.WriteHeader(http.StatusMethodNotAllowed)
        fmt.Fprintf(res, "Invalid Method: %s", req.Method)
        return
    }

    data, err := io.ReadAll(req.Body)
    if err != nil {
        res.Write([]byte(fmt.Sprintf(`{"error": "%s"}`, err.Error())))
        return
    }

    var requestBody userservice.RegisterRequest
    err = json.Unmarshal(data, &requestBody)
    if err != nil {
        res.Write([]byte(fmt.Sprintf(`{"error": "%s"}`, err.Error())))
        return
    }

    mysqlRepo := mysql.New()
    userSvc := userservice.New(mysqlRepo)
    response, err := userSvc.Register(requestBody)
    if err != nil {
        res.Write([]byte(fmt.Sprintf(`{"error": "%s"}`, err.Error())))
        return
    }

    res.WriteHeader(http.StatusCreated)
    res.Write([]byte(fmt.Sprintf(`{"user": "%s"}`, response.User)))
}

func healthCheckHandler(res http.ResponseWriter, req *http.Request) {
    if req.Method != http.MethodGet {
        http.Error(res, "Invalid Method", http.StatusMethodNotAllowed)
        return
    }
    res.WriteHeader(http.StatusOK)
    fmt.Fprint(res, "OK")
}
```

### Breaking Down the Code

1. **Setting Up Routes**: In the `main` function, we use `http.HandleFunc` to define two routes:
    
    - `/users/register`: Handles user registration via a POST request.
    - `/health-check`: A simple GET endpoint to check if the server is running.
2. **User Registration Handler**:
    
    - **Method Check**: Ensures the request method is POST. If not, it returns a 405 Method Not Allowed error with a message showing the invalid method—helpful for debugging client issues.
    - **Reading Request Body**: Uses `io.ReadAll` to read the request body. If there’s an error (e.g., the body is malformed or too large), it returns a JSON error response.
    - **Unmarshaling JSON**: Converts the JSON request body into a `userservice.RegisterRequest` struct. If the JSON is invalid (e.g., missing fields), it returns an error response.
    - **Creating Repository and Service**: Instantiates a MySQL repository using `mysql.New()` and passes it to the user service via `userservice.New(mysqlRepo)`. This is dependency injection in action, keeping the service decoupled from the storage.
    - **Calling the Service**: Calls the `Register` method on the user service with the request data. If there’s an error (e.g., invalid phone number or database issue), it returns a JSON error response.
    - **Returning the Response**: If successful, it sets the status to 201 Created and returns the created user as a JSON string. This status code signals to the client that a new resource was created.
3. **Health Check Handler**: A simple endpoint that returns "OK" for GET requests, useful for monitoring the server’s status. If the method isn’t GET, it returns a 405 error.
    

### Why This Approach?

- **Separation of Concerns**: The `main.go` file focuses on setting up the HTTP server and handling requests, while the business logic is delegated to the `service` package. This keeps the code clean and maintainable.
- **Dependency Injection**: By creating the repository and passing it to the service, we ensure that the service layer remains decoupled from the storage implementation. This makes it easier to switch to a different database or mock the repository for testing.
- **Error Handling**: The handler checks for common errors (e.g., invalid method, JSON parsing issues) and returns appropriate HTTP status codes and messages. This improves the API’s usability and helps clients understand what went wrong.
- **Simplicity**: Using the standard `net/http` package keeps the setup straightforward, which is ideal for small to medium-sized projects. For larger applications, you might consider using a more feature-rich router like `gorilla/mux`.

**Note**: In the `main.go` file, there’s a commented-out section showing an alternative way to set up the HTTP server using `http.NewServeMux()` and `http.Server`. This approach gives you more control over the server’s configuration, such as setting timeouts or adding middleware. For most projects, the simpler `http.HandleFunc` and `http.ListenAndServe` are sufficient, but it’s good to know the options.

---

## Putting It All Together: How the Layers Interact 🔗

Here’s how the components work together:

1. The **client** (e.g., a user sending a request via Postman or a web app) sends a POST request to `/users/register` with the user’s name and phone number in the request body.
2. The **HTTP handler** (`userRegisterHandler`) in `main.go` reads the request, validates the method, and parses the JSON body into a `RegisterRequest` struct.
3. The handler creates an instance of the MySQL repository and injects it into the user service.
4. The **service** (`userservice.Service`) validates the input using utilities from the `pkg` package, checks for phone number uniqueness via the repository, creates an `entity.User`, and saves it using the repository.
5. The **repository** interacts with the MySQL database to check uniqueness and insert the new user.
6. The **service** returns the created user (or an error), which the handler sends back to the client as a JSON response.

This layered architecture ensures each component has a single responsibility and can be tested independently. For example, you can mock the `Repository` interface to test the `Service` without touching the database.

---

## Tips for Scaling Your Go Project 🌱

As your project grows, here are some tips to keep your architecture robust:

- **Add a `cmd` package**: Store your application’s entry points (e.g., `main.go`) in a `cmd` package to keep them separate from business logic.
- **Use dependency injection**: Pass dependencies (like repositories) to services via constructors or interfaces to make testing easier.
- **Organize by domain**: Group related entities, services, and repositories under domain-specific directories (e.g., `user/entity`, `user/service`).
- **Add logging and metrics**: Use the `pkg` package to create reusable logging or monitoring utilities.
- **Write tests**: Create unit tests for your services and repositories, and use integration tests to verify database interactions.

---

## Wrapping Up

Structuring a Go project doesn’t have to be intimidating. By organizing your code into `entity`, `pkg`, `service`, and `repository` packages, you create a clean, scalable foundation that’s easy to maintain and extend. This architecture isn’t set in stone—feel free to tweak it based on your project’s needs. The key is to keep your code modular, testable, and focused on a single responsibility.

What’s your favorite way to structure Go projects? Have you tried this approach or something different? Let me know in the comments—I’d love to hear your thoughts! 🚀