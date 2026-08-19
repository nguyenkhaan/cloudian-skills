# Rust — AGENTS Protocol

## Rust

### Minimum Required Linters/Analyzers

- **clippy** — the official Rust linter (`cargo clippy`)
- **rustfmt** — the official formatter (`cargo fmt`)
- **cargo clippy -- -D warnings** — deny all warnings

Run:
```bash
cargo fmt --check && cargo clippy -- -D warnings
```

Clippy is non-negotiable. Every Clippy warning has a reason. Fix it. Do not
add `#[allow(clippy::...)]` unless you can write a multi-sentence
justification in a comment — and even then, reconsider.

### Embrace the Borrow Checker

The borrow checker is not your enemy. It is the only thing standing between
you and a use-after-free, a data race, or a double free. If the borrow
checker rejects your code, your code is probably wrong.

```rust
// GOOD: Borrow checker guides correct ownership
fn process(data: &[u8]) -> Result<Output, ProcessError> {
    let parsed = parse(data)?;
    let validated = validate(&parsed)?;
    transform(&validated)
}

// BAD: Fighting the borrow checker with clone()
fn process(data: Vec<u8>) -> Result<Output, ProcessError> {
    let parsed = parse(&data.clone())?;  // unnecessary clone
    let validated = validate(&parsed.clone())?;  // another one
    transform(&validated)
}
```

### Error Handling with `?` and Custom Types

```rust
// GOOD: Proper error type with context
use thiserror::Error;

#[derive(Error, Debug)]
pub enum ConfigError {
    #[error("failed to read config from '{path}': {source}")]
    ReadFailed {
        path: PathBuf,
        source: std::io::Error,
    },
    #[error("invalid config at '{path}': {details}")]
    Invalid {
        path: PathBuf,
        details: String,
    },
}

fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path).map_err(|e| ConfigError::ReadFailed {
        path: path.to_owned(),
        source: e,
    })?;
    parse_config(&content, path)
}

// BAD: String errors with no structure
fn load_config(path: &str) -> Result<Config, String> {
    let content = std::fs::read_to_string(path)
        .map_err(|e| format!("error: {}", e))?;
    parse_config(&content)
}

// TERRIBLE: unwrap() in library code
fn load_config(path: &str) -> Config {
    let content = std::fs::read_to_string(path).unwrap();  // panic on IO error
    toml::from_str(&content).unwrap()  // panic on parse error
}
```

### No `unwrap()` Outside of Tests

`unwrap()` is a panic waiting to happen. In production code, use `?`,
`expect()` with a descriptive message, or proper error handling.

```rust
// GOOD: expect() with context (acceptable for truly impossible cases)
let home = std::env::var("HOME")
    .expect("HOME environment variable must be set");

// GOOD: Proper error propagation
let home = std::env::var("HOME")
    .map_err(|_| AppError::MissingEnvVar("HOME"))?;

// BAD: unwrap() with no context
let home = std::env::var("HOME").unwrap();
```

### Use `Iterator` Methods, Not Manual Loops

```rust
// GOOD: Idiomatic, no off-by-one possible
let total: u64 = items
    .iter()
    .filter(|item| item.is_active())
    .map(|item| item.cost())
    .sum();

// BAD: Manual loop with mutable accumulator
let mut total: u64 = 0;
for i in 0..items.len() {
    if items[i].is_active() {
        total += items[i].cost();
    }
}
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
