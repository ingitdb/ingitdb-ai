# Prompt 9: DALgo Client Integration

> **Agent**: Integration Engineer
> **Phase**: 3 — TypeScript Client (parallel)

Create a dal-go adapter for inGitDB with these requirements:

Overview:
- Implement dal-go interface for inGitDB
- Enable use of inGitDB with existing dal-go applications
- Provide type-safe database operations
- Support inGitDB-specific features

Repository: ingitdb/dalgo2ingitdb

dal-go Interface Implementation:
```go
package dalgo2ingitdb

import (
    "context"
    "github.com/dal-go/dalgo/dal"
    "github.com/ingitdb/ingitdb-go/client"
)

// Adapter implements dal.Database interface
type Adapter struct {
    client *client.InGitDBClient
    dbName string
}

// NewAdapter creates a new dal-go adapter for inGitDB
func NewAdapter(baseURL, dbName string, options ...Option) (*Adapter, error) {
    // Implementation
}

// Connection implements dal.Database.Connection
func (a *Adapter) Connection() dal.Connection {
    return &connection{adapter: a}
}

// Collection implements dal.Database.Collection
func (a *Adapter) Collection(name string) dal.Collection {
    return &collection{
        adapter: a,
        table:   name,
    }
}

// RunInTransaction implements dal.Database.RunInTransaction
func (a *Adapter) RunInTransaction(ctx context.Context, f func(context.Context) error, opts *dal.TransactionOptions) error {
    // Use inGitDB branch/commit mechanism for transactions
}
```

Key Features:
1. CRUD Operations:
   - Get, Set, Update, Delete
   - Batch operations
   - Query with filtering
   - Pagination support

2. Transaction Support:
   - Map to inGitDB branches
   - Atomic commits
   - Rollback capability

3. Type Safety:
   - Generic type support
   - Schema validation
   - Type conversion

4. InGitDB-Specific:
   - Branch operations
   - Version history access
   - Merge operations
   - Conflict resolution

Usage Example:
```go
package main

import (
    "context"
    "github.com/dal-go/dalgo/dal"
    "github.com/ingitdb/dalgo2ingitdb"
)

type User struct {
    ID    string `json:"id" dalgo:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func main() {
    // Create adapter
    adapter, err := dalgo2ingitdb.NewAdapter(
        "http://localhost:3000",
        "mydb",
        dalgo2ingitdb.WithAPIKey("your-api-key"),
    )
    if err != nil {
        panic(err)
    }

    // Get collection
    users := adapter.Collection("users")

    ctx := context.Background()

    // Insert record
    user := &User{
        ID:    "user1",
        Name:  "Alice",
        Email: "alice@example.com",
    }

    err = users.Set(ctx, dal.NewKeyWithID("user1"), user)
    if err != nil {
        panic(err)
    }

    // Get record
    var retrieved User
    err = users.Get(ctx, dal.NewKeyWithID("user1"), &retrieved)
    if err != nil {
        panic(err)
    }

    // Query records
    query := users.Query().
        Where("name", dal.Equal, "Alice").
        Limit(10)

    iterator := query.Run(ctx)
    defer iterator.Close()

    for iterator.Next() {
        var u User
        err := iterator.Decode(&u)
        if err != nil {
            panic(err)
        }
        println(u.Name)
    }

    // Transaction example
    err = adapter.RunInTransaction(ctx, func(ctx context.Context) error {
        // All operations in this function are atomic
        err := users.Set(ctx, dal.NewKeyWithID("user2"), &User{
            ID: "user2", Name: "Bob",
        })
        if err != nil {
            return err
        }

        return users.Delete(ctx, dal.NewKeyWithID("user1"))
    }, nil)
}
```

Advanced Features:
```go
// Access inGitDB-specific features
type ExtendedAdapter interface {
    dal.Database

    // Branch operations
    CreateBranch(ctx context.Context, name, from string) error
    SwitchBranch(ctx context.Context, name string) error
    MergeBranch(ctx context.Context, source, target string) error

    // Version history
    GetHistory(ctx context.Context, table, recordID string) ([]Version, error)
    GetCommits(ctx context.Context, limit int) ([]Commit, error)

    // Diff operations
    DiffBranches(ctx context.Context, base, compare string) (*Diff, error)
}

// Use extended features
if extended, ok := adapter.(ExtendedAdapter); ok {
    err := extended.CreateBranch(ctx, "feature", "main")

    history, err := extended.GetHistory(ctx, "users", "user1")
}
```

Configuration Options:
```go
type Options struct {
    BaseURL     string
    APIKey      string
    Branch      string
    Timeout     time.Duration
    RetryPolicy *RetryPolicy
    Logger      Logger
}

// Option functions
func WithAPIKey(key string) Option
func WithBranch(branch string) Option
func WithTimeout(timeout time.Duration) Option
func WithRetryPolicy(policy *RetryPolicy) Option
func WithLogger(logger Logger) Option
```

Implementation Requirements:
- Full dal.Database interface implementation
- Efficient query translation
- Connection pooling
- Error mapping
- Comprehensive logging
- Performance optimization
- Type-safe operations

Testing:
- Unit tests for all dal.Database methods
- Integration tests with real inGitDB server
- Compliance tests (dal-go test suite)
- Performance benchmarks
- Concurrent access tests

Documentation:
- API reference
- Usage examples
- Migration guide from other dal-go adapters
- Best practices
