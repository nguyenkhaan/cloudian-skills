# Secrets and Credentials: Nothing Hardcoded, Ever — AGENTS Protocol

## Secrets and Credentials: Nothing Hardcoded, Ever

A secret in source code is not a shortcut. It is a security incident
waiting to happen. The moment a token, password, API key, or private key
appears in a file that is (or could be) committed to version control, it
is compromised. It does not matter if the repo is private. It does not
matter if "nobody else has access". It does not matter if you "plan to
remove it later". The secret is in the git history forever unless
someone force-pushes, and force-pushing a production repo is its own
disaster.

### The Absolute Rules

1. **No secrets in source code.** Not in Python strings. Not in JSON
   configs. Not in YAML. Not in shell scripts. Not in Dockerfiles. Not
   in comments. Not "temporarily". Not "just for testing". **Never.**

2. **No secrets in logs.** When an error occurs and you log the context,
   redact credentials before printing. An agent's instinct to "log
   everything for debugging" will dump API keys into stdout, log files,
   and CI outputs where anyone can read them.

3. **Use environment variables and `.env` files.** Secrets live in `.env`
   (which is in `.gitignore`) or in the deployment platform's secret
   store. The code reads them from the environment at runtime.

4. **`.env` is always in `.gitignore`.** If `.gitignore` doesn't exist,
   create it. If it exists but doesn't contain `.env`, add it. This is
   the first thing you do before touching any secret.

### What This Looks Like

```python
# GOOD: Read from environment, fail hard if missing
import os

def get_api_key() -> str:
    key = os.environ.get("OPENAI_API_KEY")
    if not key:
        raise RuntimeError(
            "OPENAI_API_KEY environment variable is not set. "
            "Create a .env file or export it in your shell."
        )
    return key

# BAD: Hardcoded key — now in git history forever
API_KEY = "sk-proj-abc123..."  # "I'll move this to .env later"
# narrator: they did not move it to .env later

# CATASTROPHICALLY BAD: Key in error message / log
try:
    response = call_api(api_key="sk-proj-abc123...")
except APIError as e:
    logger.error(f"API call failed with key={api_key}: {e}")
    # congratulations, your key is now in CloudWatch / Datadog / stdout
```

```javascript
// GOOD: Node.js with dotenv
import "dotenv/config";

const dbUrl = process.env.DATABASE_URL;
if (!dbUrl) {
    throw new Error(
        "DATABASE_URL is not set. Copy .env.example to .env and fill in values."
    );
}

// BAD
const dbUrl = "postgresql://admin:supersecret@prod-db:5432/myapp";
```

```go
// GOOD
apiKey := os.Getenv("API_KEY")
if apiKey == "" {
    log.Fatal("API_KEY environment variable is required")
}

// BAD
const apiKey = "sk-live-abc123..."
```

```rust
// GOOD
let api_key = std::env::var("API_KEY").unwrap_or_else(|_| {
    eprintln!("ERROR: API_KEY environment variable is not set");
    std::process::exit(1);
});

// BAD
let api_key = "sk-live-abc123...";
```

### Logging Redaction

When logging request/response context near credentials, redact:

```python
# GOOD: Redact before logging
def log_request_context(headers: dict[str, str]) -> None:
    safe_headers = {
        k: ("***REDACTED***" if k.lower() in ("authorization", "x-api-key") else v)
        for k, v in headers.items()
    }
    logger.debug(f"Request headers: {safe_headers}")

# BAD: Dumps auth header into logs
def log_request_context(headers: dict[str, str]) -> None:
    logger.debug(f"Request headers: {headers}")  # includes Authorization: Bearer sk-...
```

### The `.env.example` Pattern

Every project with secrets must include a `.env.example` file that lists
all required variables with placeholder values and comments:

```bash
# .env.example — commit this file, NOT .env
# Copy to .env and fill in real values

# OpenAI API key (required)
OPENAI_API_KEY=sk-your-key-here

# Database connection string (required)
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# Redis URL (optional, defaults to localhost)
REDIS_URL=redis://localhost:6379
```

If the agent creates a project that uses any external service, it **must**
create `.env.example`, add `.env` to `.gitignore`, and document which
variables are required vs optional.

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
