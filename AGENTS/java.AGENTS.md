# Java — AGENTS Protocol

## Java

### Minimum Required Linters/Analyzers

- **Checkstyle** — style enforcement
- **SpotBugs** (or FindBugs) — bug pattern detection
- **PMD** — static analysis for common problems
- **Error Prone** — compile-time bug detection by Google

Compiler flags: `-Xlint:all -Werror`

### No Null Returns for Collections

```java
// GOOD: Return empty collection, never null
public List<User> findUsers(String query) {
    List<User> results = repository.search(query);
    return results != null ? results : List.of();
}

// BAD: Null return forces null checks on every caller
public List<User> findUsers(String query) {
    return repository.search(query);  // might return null
}
```

### Use Optional, Not Null

```java
// GOOD
public Optional<User> findById(long id) {
    User user = repository.get(id);
    return Optional.ofNullable(user);
}

// Caller:
User user = findById(42)
    .orElseThrow(() -> new UserNotFoundException("User 42 not found"));

// BAD
public User findById(long id) {
    return repository.get(id);  // returns null if not found
}

// Caller has to remember to check:
User user = findById(42);
if (user == null) { ... }  // but they won't remember
```

### Close Resources with Try-with-Resources

```java
// GOOD
try (var conn = dataSource.getConnection();
     var stmt = conn.prepareStatement(sql);
     var rs = stmt.executeQuery()) {
    while (rs.next()) {
        results.add(mapRow(rs));
    }
}  // all three closed automatically, even on exception

// BAD: Manual close, leak on exception
Connection conn = dataSource.getConnection();
PreparedStatement stmt = conn.prepareStatement(sql);
ResultSet rs = stmt.executeQuery();
while (rs.next()) {
    results.add(mapRow(rs));  // if this throws, rs/stmt/conn leak
}
rs.close();
stmt.close();
conn.close();
```

### Specific Exception Types

```java
// GOOD
public Config loadConfig(Path path) throws ConfigurationException {
    if (!Files.exists(path)) {
        throw new ConfigurationException(
            String.format("Config file not found: %s", path.toAbsolutePath())
        );
    }
    try {
        String content = Files.readString(path);
        return parseConfig(content);
    } catch (IOException e) {
        throw new ConfigurationException(
            String.format("Failed to read config from %s", path.toAbsolutePath()), e
        );
    }
}

// BAD: Throws generic Exception
public Config loadConfig(Path path) throws Exception {
    return parseConfig(Files.readString(path));
}

// FORBIDDEN: Empty catch block
try {
    riskyOperation();
} catch (Exception e) {
    // swallowed silently
}
```

### Immutability by Default

```java
// GOOD: Immutable record (Java 16+)
public record UserDTO(String name, String email, List<String> roles) {
    public UserDTO {
        Objects.requireNonNull(name, "name must not be null");
        Objects.requireNonNull(email, "email must not be null");
        roles = List.copyOf(roles);  // defensive copy, immutable
    }
}

// BAD: Mutable POJO with getters/setters for everything
public class UserDTO {
    private String name;
    private List<String> roles;  // mutable, shared reference

    public void setName(String name) { this.name = name; }
    public List<String> getRoles() { return roles; }  // exposes internal state
}
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
