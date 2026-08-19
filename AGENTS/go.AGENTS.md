# Go — AGENTS Protocol

## Go

### Minimum Required Linters/Analyzers

- **golangci-lint** — meta-linter that runs 50+ linters
- **go vet** — official analysis tool
- **staticcheck** — advanced static analysis
- **gofmt** / **goimports** — formatting (non-negotiable)

Install if missing:
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install honnef.co/go/tools/cmd/staticcheck@latest
```

Run:
```bash
gofmt -d . && go vet ./... && staticcheck ./... && golangci-lint run
```

### Error Handling: Check Every Error

Go makes error handling explicit. That is a feature, not a burden. Every
error return must be checked. Every. Single. One.

```go
// GOOD: Check and wrap with context
data, err := os.ReadFile(path)
if err != nil {
    return fmt.Errorf("loadConfig: reading %s: %w", path, err)
}

// BAD: Ignoring the error
data, _ := os.ReadFile(path)

// ALSO BAD: Check without context
data, err := os.ReadFile(path)
if err != nil {
    return err  // caller has no idea what path failed
}
```

### Use `errors.Is` and `errors.As` for Comparison

```go
// GOOD
if errors.Is(err, os.ErrNotExist) {
    return fmt.Errorf("config file missing: %s", path)
}

var pathErr *os.PathError
if errors.As(err, &pathErr) {
    log.Printf("path error on %s: %v", pathErr.Path, pathErr.Err)
}

// BAD: String comparison breaks when error messages change
if err.Error() == "file not found" {
    ...
}
```

### Defer for Cleanup

```go
// GOOD
func processFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("processFile: open %s: %w", path, err)
    }
    defer f.Close()

    // work with f...
    return nil
}

// BAD: Manual close, leaks on early return
func processFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }

    data, err := io.ReadAll(f)
    if err != nil {
        return err  // f is never closed
    }

    f.Close()
    return process(data)
}
```

### Do Not Overuse Goroutines

```go
// GOOD: Goroutine with proper lifecycle management
func serve(ctx context.Context, addr string) error {
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        return fmt.Errorf("serve: listen on %s: %w", addr, err)
    }
    defer listener.Close()

    var wg sync.WaitGroup
    for {
        conn, err := listener.Accept()
        if err != nil {
            if ctx.Err() != nil {
                break  // context cancelled, shutdown
            }
            log.Printf("accept error: %v", err)
            continue
        }
        wg.Add(1)
        go func() {
            defer wg.Done()
            handleConn(ctx, conn)
        }()
    }
    wg.Wait()
    return nil
}

// BAD: Fire-and-forget goroutines with no lifecycle
func serve(addr string) {
    listener, _ := net.Listen("tcp", addr)
    for {
        conn, _ := listener.Accept()
        go handleConn(conn)  // leaked goroutine, no shutdown, no error handling
    }
}
```

### Struct Initialization

```go
// GOOD: Named fields, clear intent
srv := &http.Server{
    Addr:         ":8080",
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    Handler:      mux,
}

// BAD: Positional initialization breaks when struct fields change
srv := &http.Server{":8080", nil, mux, ...}
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
