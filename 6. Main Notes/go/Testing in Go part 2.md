# Testing in Go

2025-07-26 01:31
Status: #DONE 
Tags: [[Go]]

---
### Key Points
- Go's `testing` package is a robust, built-in framework for writing unit tests, benchmarks, and fuzz tests, seamlessly integrated with the `go test` command.
- It offers types like `T`, `B`, `F`, and `TB`, each with methods to manage test state, report errors, and control execution, alongside standalone functions for additional functionality.
- Best practices include writing tests for all public functions, using table-driven tests, ensuring test independence, and leveraging tools like mocking and the race detector.
- While the standard library is powerful, supplementary tools like Testify or Ginkgo can enhance testing capabilities, though their use is debated among developers.
- Examples and structured approaches, such as table-driven tests and subtests, improve test readability and maintainability.

### Overview of Go Testing
The Go programming language provides a built-in testing framework through the `testing` package, designed to ensure code reliability and quality. This framework supports unit tests, benchmarks, and fuzz tests, all executable via the `go test` command. Its simplicity and integration with Go's toolchain make it accessible for developers to write tests without external dependencies. The package includes types like `T` for unit tests, `B` for benchmarks, and `F` for fuzz tests, each offering methods to manage test execution and report results. Additionally, standalone functions provide utilities for tasks like measuring code coverage and allocations.

### Core Components
The `testing` package revolves around four key types:
- **T**: Manages unit test state and reporting.
- **B**: Handles benchmark tests for performance measurement.
- **F**: Supports fuzz testing to explore edge cases.
- **TB**: A common interface for shared methods across `T`, `B`, and `F`.

These types, along with standalone functions, enable developers to write comprehensive tests. Tests are typically written in files ending with `_test.go` and can access unexported identifiers if in the same package or only exported ones in a separate `_test` package (black-box testing).

### Best Practices
To write effective tests, developers should:
- Test all public functions to ensure complete coverage of exported APIs.
- Use table-driven tests to organize test cases clearly.
- Keep tests independent to avoid unintended side effects.
- Employ subtests for grouping related test cases.
- Mock dependencies to isolate code under test.
- Test error conditions to verify robust error handling.
- Use the `-race` flag to detect concurrency issues and `-cover` to measure test coverage.
- Write benchmarks for performance-critical code.

### Tips and Tricks
- Use `t.Parallel()` to speed up tests, but ensure thread safety.
- Leverage `t.Cleanup()` for resource cleanup to prevent leaks.
- Write descriptive test names for clarity.
- Consider libraries like Testify for enhanced assertions, though some prefer the standard library for simplicity.
- Use fuzz testing to uncover edge cases and property-based testing for broader validation.

### Examples
Below are examples illustrating common testing patterns:
- **Unit Test**: Tests a simple `Add` function.
- **Table-Driven Test**: Organizes multiple test cases in a structured table.
- **Subtest**: Groups related tests for better organization.
- **Benchmark**: Measures the performance of a function.
- **Fuzz Test**: Explores edge cases with random inputs.

```go
package math

import "testing"

func Add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}

func TestAddTableDriven(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {2, 3, 5},
        {0, 0, 0},
        {-1, 1, 0},
    }
    for _, tt := range tests {
        t.Run("table-driven", func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}

func TestAddSubtests(t *testing.T) {
    t.Run("positive numbers", func(t *testing.T) {
        if got := Add(2, 3); got != 5 {
            t.Errorf("Add(2, 3) = %d; want 5", got)
        }
    })
    t.Run("zero", func(t *testing.T) {
        if got := Add(0, 0); got != 0 {
            t.Errorf("Add(0, 0) = %d; want 0", got)
        }
    })
}

func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

func FuzzAdd(f *testing.F) {
    f.Add(2, 3)
    f.Fuzz(func(t *testing.T, a, b int) {
        Add(a, b) // Basic fuzz test; add checks as needed
    })
}
```

---

## Comprehensive Guide to Testing in Go: Library, Best Practices, and Examples

### Abstract
This paper provides an in-depth exploration of Go's `testing` package, a cornerstone of the language's built-in testing framework. It details the package's types, methods, and functions, alongside best practices, tips, and tricks for effective testing. Practical examples illustrate key concepts, making this guide a valuable resource for both novice and experienced Go developers. By adhering to the outlined practices, developers can ensure robust, maintainable, and high-quality codebases.

### 1. Introduction
Testing is fundamental to software development, ensuring code reliability, maintainability, and performance. Go's `testing` package, part of the standard library, provides a lightweight yet powerful framework for writing unit tests, benchmarks, and fuzz tests. Integrated with the `go test` command, it requires minimal setup, making it accessible for developers. This framework supports test-driven development (TDD), facilitates regression testing, and enhances code documentation. Its simplicity, combined with features like coverage analysis and benchmarking, makes it a preferred choice for Go developers, though some opt for supplementary libraries like Testify or Ginkgo for additional functionality [1, 14].

### 2. The Go Testing Package
The `testing` package is designed for automated testing, executed via `go test`. Tests are written in files ending with `_test.go`, either in the same package for internal testing or a separate `_test` package for black-box testing [1]. The package includes four primary types and several standalone functions.

#### 2.1 Types
- **T**: Manages unit test state, offering methods for error reporting, logging, and test control.
- **B**: Handles benchmark tests, extending `T` with performance-specific methods.
- **F**: Supports fuzz testing, introduced in Go 1.18, for exploring edge cases.
- **TB**: A common interface for shared methods across `T`, `B`, and `F`.

#### 2.2 Methods and Functions
The following table details the methods and functions of the `testing` package, as documented in the official Go documentation [1].

| Type/Function       | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| **T Methods**       |                                                                             |
| `Chdir(dir string)` | Changes the current working directory to the named directory.               |
| `Cleanup(func())`   | Registers a function to be called when the test and all its subtests complete. |
| `Context() context.Context` | Returns the context for the test.                                          |
| `Deadline() (deadline time.Time, ok bool)` | Returns the time when the test will fail.                                  |
| `Error(args ...interface{})` | Reports an error.                                                          |
| `Errorf(format string, args ...interface{})` | Reports an error with a formatted message.                                 |
| `Fail()`            | Marks the test as failed but continues execution.                          |
| `FailNow()`         | Marks the test as failed and stops execution.                              |
| `Failed() bool`     | Reports whether the test has failed.                                       |
| `Fatal(args ...interface{})` | Reports an error and stops the test.                                       |
| `Fatalf(format string, args ...interface{})` | Reports an error with a formatted message and stops the test.              |
| `Helper()`          | Indicates that the caller is a helper function, excluding it from stack traces. |
| `Log(args ...interface{})` | Prints to the test log, visible with `-v` flag.                            |
| `Logf(format string, args ...interface{})` | Prints to the test log with a formatted message.                           |
| `Name() string`     | Returns the name of the test.                                              |
| `Parallel()`        | Marks the test to run in parallel with other parallel tests.               |
| `Run(name string, f func(*T)) *T` | Runs a subtest with the given name.                                       |
| `Setenv(key, value string)` | Sets an environment variable for the test.                                |
| `Skip(args ...interface{})` | Skips the test with a message.                                             |
| `SkipNow()`         | Skips the test immediately.                                                |
| `Skipf(format string, args ...interface{})` | Skips the test with a formatted message.                                   |
| `Skipped() bool`    | Reports whether the test was skipped.                                      |
| `TempDir() string`  | Creates a new temporary directory, cleaned up after the test.              |
| **B Methods**       |                                                                             |
| All methods of `T`  | Inherits all `T` methods.                                                  |
| `ReportAllocs()`    | Reports memory allocations during the benchmark.                           |
| `ReportMetric(value float64, unit string)` | Reports a custom metric (e.g., operations per second).                     |
| `ResetTimer()`      | Resets the benchmark timer to zero.                                        |
| `SetBytes(n int64)` | Sets the number of bytes processed per iteration for throughput metrics.   |
| `SetParallelism(p int)` | Sets the number of goroutines for parallel benchmarks.                     |
| `StartTimer()`      | Starts the benchmark timer.                                                |
| `StopTimer()`       | Stops the benchmark timer, useful for excluding setup time.                |
| **F Methods**       |                                                                             |
| All methods of `T`  | Inherits all `T` methods.                                                  |
| `Add(additional []byte)` | Adds seed data to the fuzz target.                                         |
| `Fuzz(t *T)`        | Defines the fuzz target function for random input testing.                 |
| **Standalone Functions** |                                                                             |
| `AllocsPerRun(f func()) int64` | Reports average allocations per run of `f`.                               |
| `CoverMode() string` | Returns the coverage mode ("set", "count", "atomic") or empty if disabled. |
| `Coverage() float64` | Returns the fraction of statements covered [0, 1].                         |
| `Init()`            | Registers testing flags, used for benchmarks outside `go test`.            |
| `Main(...)`         | Internal function for `go test` implementation.                            |
| `RegisterCover(...)` | Registers coverage data accumulators (internal use).                       |
| `RunBenchmarks(...)` | Runs benchmarks (internal use).                                            |
| `RunExamples(...)`  | Runs examples (internal use).                                              |
| `RunTests(...)`     | Runs tests (internal use).                                                 |
| `Short() bool`      | Reports whether the `-test.short` flag is set.                             |
| `Testing() bool`    | Reports whether the code is running in a test environment.                 |
| `Verbose() bool`    | Reports whether the `-test.v` flag is set.                                 |

### 3. Best Practices for Testing in Go
Effective testing enhances code quality and maintainability. The following practices, drawn from various sources [2, 3, 8, 12, 13, 15, 20], are widely recommended:

- **Test All Public Functions**: Ensure every exported function has corresponding tests to verify the public API [2].
- **Table-Driven Tests**: Use a slice of structs to define test cases, improving readability and scalability [3, 12].
- **Independent Tests**: Tests should not depend on each other’s state to avoid fragile test suites [8].
- **Subtests**: Group related tests using `t.Run` for better organization and isolation [10].
- **Mock Dependencies**: Use libraries like `gomock` or `mockery` to isolate code from external dependencies [5, 13].
- **Test Error Cases**: Verify that error conditions are handled correctly [8].
- **Use `-race` Detector**: Run `go test -race` to detect concurrency issues [12].
- **Measure Test Coverage**: Use `go test -cover` to aim for 60-80% coverage, balancing thoroughness with practicality [5, 13].
- **Benchmark Performance**: Write benchmarks for performance-critical code to identify bottlenecks [16].

### 4. Tips and Tricks
Advanced techniques can further enhance testing efficiency [9, 17, 19, 24]:

- **Parallel Testing**: Use `t.Parallel()` to speed up tests, but ensure thread safety to avoid race conditions [19].
- **Resource Cleanup**: Leverage `t.Cleanup()` to manage resources like temporary files or database connections [17].
- **Descriptive Test Names**: Use clear names (e.g., `TestAdd_NegativeNumbers`) to improve readability [24].
- **Assertion Libraries**: Libraries like Testify provide concise assertions, though some developers prefer the standard library for simplicity [15, 19].
- **Fuzz Testing**: Use `F` type methods to test edge cases with random inputs, introduced in Go 1.18 [1, 3].
- **Property-Based Testing**: Libraries like Gopter enable testing general properties of code, complementing specific test cases [20].

### 5. Examples
The following examples demonstrate common testing patterns, using a simple `Add` function for illustration.

```go
package math

import "testing"

func Add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}

func TestAddTableDriven(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {2, 3, 5},
        {0, 0, 0},
        {-1, 1, 0},
    }
    for _, tt := range tests {
        t.Run("table-driven", func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}

func TestAddSubtests(t *testing.T) {
    t.Run("positive numbers", func(t *testing.T) {
        if got := Add(2, 3); got != 5 {
            t.Errorf("Add(2, 3) = %d; want 5", got)
        }
    })
    t.Run("zero", func(t *testing.T) {
        if got := Add(0, 0); got != 0 {
            t.Errorf("Add(0, 0) = %d; want 0", got)
        }
    })
}

func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

func FuzzAdd(f *testing.F) {
    f.Add(2, 3)
    f.Fuzz(func(t *testing.T, a, b int) {
        Add(a, b) // Basic fuzz test; add checks as needed
    })
}
```

### 6. Code Examples for Methods and Functions
Below are code snippets for each method and function in the `testing` package, demonstrating their usage.

#### T Methods
- **Chdir**:
  ```go
  t.Chdir("/new/dir")
  ```
- **Cleanup**:
  ```go
  t.Cleanup(func() { os.RemoveAll("/tmp/test") })
  ```
- **Context**:
  ```go
  ctx := t.Context()
  ```
- **Deadline**:
  ```go
  deadline, ok := t.Deadline()
  ```
- **Error**:
  ```go
  t.Error("an error occurred")
  ```
- **Errorf**:
  ```go
  t.Errorf("expected %v, got %v", expected, got)
  ```
- **Fail**:
  ```go
  t.Fail()
  ```
- **FailNow**:
  ```go
  t.FailNow()
  ```
- **Failed**:
  ```go
  if t.Failed() {
      // Handle failure
  }
  ```
- **Fatal**:
  ```go
  t.Fatal("test failed")
  ```
- **Fatalf**:
  ```go
  t.Fatalf("expected %v, got %v", expected, got)
  ```
- **Helper**:
  ```go
  t.Helper()
  ```
- **Log**:
  ```go
  t.Log("logging message")
  ```
- **Logf**:
  ```go
  t.Logf("logging %v", value)
  ```
- **Name**:
  ```go
  name := t.Name()
  ```
- **Parallel**:
  ```go
  t.Parallel()
  ```
- **Run**:
  ```go
  t.Run("subtest", func(t *testing.T) {
      // Subtest code
  })
  ```
- **Setenv**:
  ```go
  t.Setenv("ENV_VAR", "value")
  ```
- **Skip**:
  ```go
  t.Skip("skipping test")
  ```
- **SkipNow**:
  ```go
  t.SkipNow()
  ```
- **Skipf**:
  ```go
  t.Skipf("skipping %v", reason)
  ```
- **Skipped**:
  ```go
  if t.Skipped() {
      // Handle skip
  }
  ```
- **TempDir**:
  ```go
  dir := t.TempDir()
  ```

#### B Methods
- **ReportAllocs**:
  ```go
  b.ReportAllocs()
  ```
- **ReportMetric**:
  ```go
  b.ReportMetric(100, "ops")
  ```
- **ResetTimer**:
  ```go
  b.ResetTimer()
  ```
- **SetBytes**:
  ```go
  b.SetBytes(1024)
  ```
- **SetParallelism**:
  ```go
  b.SetParallelism(4)
  ```
- **StartTimer**:
  ```go
  b.StartTimer()
  ```
- **StopTimer**:
  ```go
  b.StopTimer()
  ```

#### F Methods
- **Add**:
  ```go
  f.Add([]byte{1, 2, 3})
  ```
- **Fuzz**:
  ```go
  func FuzzExample(f *testing.F) {
      f.Fuzz(func(t *testing.T, input []byte) {
          // Fuzz logic
      })
  }
  ```

#### Standalone Functions
- **AllocsPerRun**:
  ```go
  allocs := testing.AllocsPerRun(100, func() { /* code */ })
  ```
- **CoverMode**:
  ```go
  mode := testing.CoverMode()
  ```
- **Coverage**:
  ```go
  coverage := testing.Coverage()
  ```
- **Init**:
  ```go
  testing.Init()
  ```
- **Short**:
  ```go
  if testing.Short() {
      // Short test logic
  }
  ```
- **Testing**:
  ```go
  if testing.Testing() {
      // Test-specific logic
  }
  ```
- **Verbose**:
  ```go
  if testing.Verbose() {
      // Verbose logging
  }
  ```

*Note*: `Main`, `RegisterCover`, `RunBenchmarks`, and `RunTests` are internal functions not typically used directly by developers.

### 7. Conclusion
Go's `testing` package offers a comprehensive framework for ensuring code quality through unit tests, benchmarks, and fuzz tests. Its integration with the `go test` command and support for coverage and concurrency analysis make it a powerful tool for developers. By adhering to best practices like table-driven tests, independent test cases, and proper resource management, developers can create robust test suites. Supplementary tools, while optional, can enhance testing capabilities, though the standard library remains sufficient for most use cases. The provided examples and detailed method descriptions serve as a practical guide for implementing effective testing strategies in Go.

### Key Points
- Go's `testing` package provides a comprehensive framework for unit tests, benchmarks, and fuzz tests, enabling robust validation of real-world project code.
- Examples should reflect realistic scenarios, such as testing HTTP handlers, database interactions, or complex business logic, to mirror production use cases.
- Best practices, such as table-driven tests, independent test cases, and proper resource cleanup, remain critical for maintaining test quality.
- The following examples demonstrate the use of all `testing.T`, `testing.B`, `testing.F`, and standalone functions in the context of a realistic project, such as a REST API server.

### Overview
This response extends the previous exploration of Go’s `testing` package by providing detailed, project-like examples for all methods and functions, focusing on a realistic scenario: a REST API server for managing user profiles. The examples include testing HTTP handlers, database operations, and performance benchmarks, reflecting common patterns in production-grade Go applications. Each example is designed to demonstrate practical usage while adhering to best practices like table-driven tests, resource cleanup, and concurrency safety.

### Realistic Project Context
Consider a Go project for a user management API with the following components:
- **User Service**: Handles CRUD operations for user profiles (e.g., `CreateUser`, `GetUser`).
- **HTTP Handler**: Exposes REST endpoints (e.g., `/users`).
- **Database**: Uses an in-memory store or a mock database for testing.
- **Business Logic**: Validates user data, such as email formats and age constraints.

The examples below test these components using all methods and functions from the `testing` package, ensuring coverage of realistic scenarios like HTTP request handling, database queries, and performance-critical operations.

### Best Practices and Tips
The best practices and tips from the previous response remain applicable:
- **Test All Public Functions**: Ensure coverage of exported APIs.
- **Table-Driven Tests**: Organize test cases in a structured table.
- **Independent Tests**: Avoid shared state between tests.
- **Subtests**: Group related test cases using `t.Run`.
- **Mock Dependencies**: Use interfaces or libraries like `gomock` to isolate dependencies.
- **Test Error Cases**: Verify error handling for robustness.
- **Use `-race` and `-cover`**: Detect concurrency issues and measure coverage.
- **Parallel Testing**: Use `t.Parallel()` for efficiency, ensuring thread safety.
- **Resource Cleanup**: Use `t.Cleanup()` to manage resources.
- **Descriptive Test Names**: Enhance readability with clear names.
- **Fuzz Testing**: Explore edge cases with random inputs.

### Examples
Below is a comprehensive set of examples for all `testing.T`, `testing.B`, `testing.F`, and standalone functions, implemented in the context of a user management API. The code includes a simplified user service, HTTP handler, and mock database, with tests demonstrating each method and function.

```go
package main

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"net/http"
	"net/http/httptest"
	"os"
	"path/filepath"
	"strings"
	"sync"
	"testing"
	"time"
)

// User represents a user profile.
type User struct {
	ID    int
	Name  string
	Email string
	Age   int
}

// UserService manages user operations.
type UserService struct {
	db map[int]User // Mock database
	mu sync.RWMutex
}

// NewUserService creates a new UserService.
func NewUserService() *UserService {
	return &UserService{db: make(map[int]User)}
}

// CreateUser adds a user to the database.
func (s *UserService) CreateUser(ctx context.Context, user User) error {
	if user.Name == "" || user.Email == "" || user.Age < 0 {
		return fmt.Errorf("invalid user data")
	}
	if !strings.Contains(user.Email, "@") {
		return fmt.Errorf("invalid email")
	}
	s.mu.Lock()
	defer s.mu.Unlock()
	s.db[user.ID] = user
	return nil
}

// GetUser retrieves a user by ID.
func (s *UserService) GetUser(ctx context.Context, id int) (User, error) {
	s.mu.RLock()
	defer s.mu.RUnlock()
	user, exists := s.db[id]
	if !exists {
		return User{}, fmt.Errorf("user %d not found", id)
	}
	return user, nil
}

// UserHandler handles HTTP requests for users.
func UserHandler(s *UserService) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
		if err := s.CreateUser(r.Context(), user); err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
			return
		}
		w.Write([]byte("User created"))
	}
}

// Tests for UserService and UserHandler
func TestMain(m *testing.M) {
	// Simulate setup for external resources (e.g., database connection)
	os.Exit(m.Run())
}

func TestUserService(t *testing.T) {
	// Chdir: Change working directory for test
	t.Run("Chdir", func(t *testing.T) {
		originalDir, _ := os.Getwd()
		t.Chdir(t.TempDir())
		defer t.Chdir(originalDir)
		// Verify directory change
		newDir, _ := os.Getwd()
		if filepath.Base(newDir) == filepath.Base(originalDir) {
			t.Errorf("Chdir failed to change directory")
		}
	})

	// Cleanup: Ensure temporary files are removed
	t.Run("Cleanup", func(t *testing.T) {
		f, err := os.CreateTemp("", "testfile")
		if err != nil {
			t.Fatal(err)
		}
		t.Cleanup(func() {
			os.Remove(f.Name())
		})
		// Write to file to simulate usage
		f.WriteString("test")
	})

	// Context: Use test context
	t.Run("Context", func(t *testing.T) {
		ctx := t.Context()
		s := NewUserService()
		user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
		if err := s.CreateUser(ctx, user); err != nil {
			t.Errorf("CreateUser failed: %v", err)
		}
	})

	// Deadline: Check test deadline
	t.Run("Deadline", func(t *testing.T) {
		_, ok := t.Deadline()
		if ok {
			t.Log("Test has a deadline")
		} else {
			t.Log("No deadline set")
		}
	})

	// Error: Report a non-fatal error
	t.Run("Error", func(t *testing.T) {
		s := NewUserService()
		user := User{ID: 1, Name: "", Email: "alice@example.com", Age: 30}
		if err := s.CreateUser(context.Background(), user); err == nil {
			t.Error("Expected error for empty name, got none")
		}
	})

	// Errorf: Report a formatted error
	t.Run("Errorf", func(t *testing.T) {
		s := NewUserService()
		user := User{ID: 1, Name: "Alice", Email: "invalid", Age: 30}
		if err := s.CreateUser(context.Background(), user); err == nil {
			t.Errorf("CreateUser(%v) should have failed", user)
		}
	})

	// Fail: Mark test as failed
	t.Run("Fail", func(t *testing.T) {
		s := NewUserService()
		if len(s.db) != 0 {
			t.Fail()
		}
	})

	// FailNow: Stop test on failure
	t.Run("FailNow", func(t *testing.T) {
		s := NewUserService()
		if len(s.db) != 0 {
			t.FailNow()
		}
	})

	// Failed: Check if test failed
	t.Run("Failed", func(t *testing.T) {
		s := NewUserService()
		if err := s.CreateUser(context.Background(), User{ID: 1, Name: "", Email: "alice@example.com", Age: 30}); err == nil {
			t.Error("Expected error")
		}
		if t.Failed() {
			t.Log("Test marked as failed")
		}
	})

	// Fatal: Stop test with error
	t.Run("Fatal", func(t *testing.T) {
		s := NewUserService()
		if err := s.CreateUser(context.Background(), User{ID: 1, Name: "", Email: "alice@example.com", Age: 30}); err == nil {
			t.Fatal("Expected error for empty name")
		}
	})

	// Fatalf: Stop test with formatted error
	t.Run("Fatalf", func(t *testing.T) {
		s := NewUserService()
		user := User{ID: 1, Name: "Alice", Email: "invalid", Age: 30}
		if err := s.CreateUser(context.Background(), user); err == nil {
			t.Fatalf("CreateUser(%v) should have failed", user)
		}
	})

	// Helper: Mark helper function
	t.Run("Helper", func(t *testing.T) {
		helper := func(t *testing.T) {
			t.Helper()
			t.Error("Error from helper")
		}
		helper(t)
	})

	// Log: Log message
	t.Run("Log", func(t *testing.T) {
		t.Log("Running user creation test")
		s := NewUserService()
		user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
		s.CreateUser(context.Background(), user)
	})

	// Logf: Log formatted message
	t.Run("Logf", func(t *testing.T) {
		s := NewUserService()
		user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
		s.CreateUser(context.Background(), user)
		t.Logf("Created user with ID %d", user.ID)
	})

	// Name: Get test name
	t.Run("Name", func(t *testing.T) {
		t.Logf("Running test: %s", t.Name())
	})

	// Parallel: Run test in parallel
	t.Run("Parallel", func(t *testing.T) {
		t.Parallel()
		s := NewUserService()
		user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
		if err := s.CreateUser(context.Background(), user); err != nil {
			t.Errorf("CreateUser failed: %v", err)
		}
	})

	// Run: Run subtests
	t.Run("Run", func(t *testing.T) {
		s := NewUserService()
		t.Run("ValidUser", func(t *testing.T) {
			user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
			if err := s.CreateUser(context.Background(), user); err != nil {
				t.Errorf("CreateUser failed: %v", err)
			}
		})
		t.Run("InvalidUser", func(t *testing.T) {
			user := User{ID: 2, Name: "", Email: "bob@example.com", Age: 25}
			if err := s.CreateUser(context.Background(), user); err == nil {
				t.Error("Expected error for empty name")
			}
		})
	})

	// Setenv: Set environment variable
	t.Run("Setenv", func(t *testing.T) {
		t.Setenv("TEST_ENV", "test_value")
		if os.Getenv("TEST_ENV") != "test_value" {
			t.Error("Setenv failed to set environment variable")
		}
	})

	// Skip: Skip test with message
	t.Run("Skip", func(t *testing.T) {
		if os.Getenv("CI") == "true" {
			t.Skip("Skipping test in CI environment")
		}
	})

	// SkipNow: Skip test immediately
	t.Run("SkipNow", func(t *testing.T) {
		t.SkipNow()
	})

	// Skipf: Skip test with formatted message
	t.Run("Skipf", func(t *testing.T) {
		reason := "test not implemented"
		t.Skipf("Skipping test: %s", reason)
	})

	// Skipped: Check if test was skipped
	t.Run("Skipped", func(t *testing.T) {
		t.Skip("Skipping test")
		if t.Skipped() {
			t.Log("Test was skipped")
		}
	})

	// TempDir: Create temporary directory
	t.Run("TempDir", func(t *testing.T) {
		dir := t.TempDir()
		f, err := os.Create(filepath.Join(dir, "testfile.txt"))
		if err != nil {
			t.Fatal(err)
		}
		f.Close()
	})
}

func TestUserHandler(t *testing.T) {
	// Table-driven test for HTTP handler
	tests := []struct {
		name           string
		user           User
		expectedStatus int
		expectedBody   string
	}{
		{
			name:           "ValidUser",
			user:           User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30},
			expectedStatus: http.StatusOK,
			expectedBody:   "User created",
		},
		{
			name:           "InvalidEmail",
			user:           User{ID: 2, Name: "Bob", Email: "invalid", Age: 25},
			expectedStatus: http.StatusBadRequest,
			expectedBody:   "invalid email",
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			s := NewUserService()
			req := httptest.NewRequest(http.MethodPost, "/users", nil)
			req = req.WithContext(context.WithValue(req.Context(), "user", tt.user))
			rr := httptest.NewRecorder()
			handler := UserHandler(s)
			handler.ServeHTTP(rr, req)
			if status := rr.Code; status != tt.expectedStatus {
				t.Errorf("Handler returned wrong status code: got %v want %v", status, tt.expectedStatus)
			}
			if strings.TrimSpace(rr.Body.String()) != tt.expectedBody {
				t.Errorf("Handler returned unexpected body: got %v want %v", rr.Body.String(), tt.expectedBody)
			}
		})
	}
}

func BenchmarkCreateUser(b *testing.B) {
	s := NewUserService()
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
	b.StopTimer()
	b.StartTimer()
	for b.Loop() {
		s.CreateUser(context.Background(), user)
	}
}

func BenchmarkCreateUserAllocs(b *testing.B) {
	s := NewUserService()
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
	b.ReportAllocs()
	for b.Loop() {
		s.CreateUser(context.Background(), user)
	}
}

func BenchmarkCreateUserMetric(b *testing.B) {
	s := NewUserService()
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
	for b.Loop() {
		s.CreateUser(context.Background(), user)
	}
	b.ReportMetric(float64(b.N), "users_created")
}

func BenchmarkCreateUserResetTimer(b *testing.B) {
	s := NewUserService()
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
	// Simulate warm-up
	for i := 0; i < 100; i++ {
		s.CreateUser(context.Background(), user)
	}
	b.ResetTimer()
	for b.Loop() {
		s.CreateUser(context.Background(), user)
	}
}

func BenchmarkCreateUserSetBytes(b *testing.B) {
	s := NewUserService()
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
	// Approximate size of user struct in bytes
	b.SetBytes(int64(8 + len(user.Name) + len(user.Email) + 8))
	for b.Loop() {
		s.CreateUser(context.Background(), user)
	}
}

func BenchmarkCreateUserParallel(b *testing.B) {
	s := NewUserService()
	user := User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
	b.SetParallelism(4)
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			s.CreateUser(context.Background(), user)
		}
	})
}

func FuzzCreateUser(f *testing.F) {
	s := NewUserService()
	f.Add([]byte(`{"id":1,"name":"Alice","email":"alice@example.com","age":30}`))
	f.Fuzz(func(t *testing.T, data []byte) {
		var user User
		if err := json.Unmarshal(data, &user); err != nil {
			t.Skip()
		}
		s.CreateUser(context.Background(), user)
	})
}

func TestStandaloneFunctions(t *testing.T) {
	// AllocsPerRun
	t.Run("AllocsPerRun", func(t *testing.T) {
		s := NewUserService()
		allocs := testing.AllocsPerRun(100, func() {
			s.CreateUser(context.Background(), User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30})
		})
		t.Logf("Allocations per run: %f", allocs)
	})

	// CoverMode
	t.Run("CoverMode", func(t *testing.T) {
		mode := testing.CoverMode()
		t.Logf("Coverage mode: %s", mode)
	})

	// Coverage
	t.Run("Coverage", func(t *testing.T) {
		coverage := testing.Coverage()
		t.Logf("Coverage: %f", coverage)
	})

	// Short
	t.Run("Short", func(t *testing.T) {
		if testing.Short() {
			t.Skip("Skipping in short mode")
		}
	})

	// Testing
	t.Run("Testing", func(t *testing.T) {
		if testing.Testing() {
			t.Log("Running in test environment")
		}
	})

	// Verbose
	t.Run("Verbose", func(t *testing.T) {
		if testing.Verbose() {
			t.Log("Verbose mode enabled")
		}
	})
}
```

