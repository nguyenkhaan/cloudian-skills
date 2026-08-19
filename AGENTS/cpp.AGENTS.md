# C++ — AGENTS Protocol

## C++

### Minimum Required Linters/Analyzers

- **clang-tidy** — with modernize-* and bugprone-* checks enabled
- **cppcheck** — deep static analysis
- **clang-format** — formatting enforcement
- **Compiler flags**: `-Wall -Wextra -Werror -pedantic -Wshadow -Wnon-virtual-dtor`

Install if missing:
```bash
# Debian/Ubuntu
apt install clang-tidy cppcheck clang-format

# macOS
brew install llvm cppcheck
```

### RAII or Death

Every resource (memory, file handle, socket, mutex) must be managed by
an RAII wrapper. If you write `new` without a corresponding smart pointer,
you have a bug. It might not crash today. It will crash in production.

```cpp
// GOOD: RAII with smart pointers
auto connection = std::make_unique<Connection>(host, port);
connection->send(data);
// automatically cleaned up when scope exits, even on exception

// BAD: Manual memory management in C++
Connection* connection = new Connection(host, port);
connection->send(data);  // if this throws, memory leaks
delete connection;       // never reached on exception
```

### No Raw `new`/`delete`

```cpp
// GOOD
auto items = std::make_unique<Item[]>(count);
auto shared_config = std::make_shared<Config>(load_config(path));
std::vector<int> data(size);  // stack-allocated, auto-freed

// BAD
Item* items = new Item[count];
Config* config = new Config(load_config(path));
// ... somewhere, maybe, hopefully, delete[] and delete are called
```

### Move Semantics

```cpp
// GOOD: Move large objects instead of copying
std::vector<LargeObject> process(std::vector<LargeObject> input) {
    std::vector<LargeObject> result;
    result.reserve(input.size());
    for (auto& obj : input) {
        result.push_back(transform(std::move(obj)));
    }
    return result;  // NRVO or move
}

// BAD: Unnecessary copy
std::vector<LargeObject> process(std::vector<LargeObject> input) {
    std::vector<LargeObject> result;
    for (const auto& obj : input) {
        result.push_back(transform(obj));  // copies every element
    }
    return result;
}
```

### Const Correctness

If it doesn't change, mark it `const`. This is not optional. It catches
bugs, enables optimizations, and communicates intent.

```cpp
// GOOD
void print_report(const std::vector<Record>& records) {
    for (const auto& record : records) {
        std::cout << record.name() << ": " << record.value() << "\n";
    }
}

// BAD: Takes mutable reference for no reason
void print_report(std::vector<Record>& records) {
    for (auto& record : records) {  // might accidentally modify
        std::cout << record.name() << ": " << record.value() << "\n";
    }
}
```

### Exceptions vs Error Codes

Pick one per project and be consistent. If using exceptions, **never** catch
and ignore. If using error codes, **always** check them.

```cpp
// GOOD: Exception with context
if (!file.is_open()) {
    throw std::runtime_error(
        std::format("Failed to open '{}': {}", path, strerror(errno))
    );
}

// GOOD: std::expected (C++23) / error code checked
auto result = parse_config(path);
if (!result) {
    log_error("parse_config failed: {}", result.error().message());
    return result.error();
}

// BAD: Catch and suppress
try {
    process();
} catch (...) {
    // "this shouldn't happen"
    // narrator: it did happen
}
```

### Use `std::string_view` for Non-Owning References

```cpp
// GOOD: No allocation for read-only access
void log_message(std::string_view msg, std::string_view level = "INFO") {
    std::cout << "[" << level << "] " << msg << "\n";
}

// BAD: Copies the string just to read it
void log_message(std::string msg, std::string level = "INFO") {
    std::cout << "[" << level << "] " << msg << "\n";
}
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
