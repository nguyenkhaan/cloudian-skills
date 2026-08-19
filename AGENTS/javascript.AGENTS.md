# JavaScript / Node.js / Electron — AGENTS Protocol

## JavaScript / Node.js / Electron

### Minimum Required Linters/Analyzers

- **ESLint** — with `eslint:recommended` at minimum
- **Prettier** — formatting (no debates about style)
- For Electron: **electron-builder** security audit

Install if missing:
```bash
npm install -D eslint prettier eslint-config-prettier
```

Run:
```bash
npx eslint . && npx prettier --check .
```

### Always Use `===` and `!==`

```javascript
// GOOD
if (value === null) { ... }
if (count !== 0) { ... }

// BAD: Type coercion surprises
if (value == null) { ... }  // also matches undefined
if (count != 0) { ... }    // "0" != 0 is false
```

### Async/Await Error Handling

```javascript
// GOOD: try/catch with context
async function fetchUser(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
            throw new Error(
                `fetchUser: HTTP ${response.status} for userId=${userId}`
            );
        }
        return await response.json();
    } catch (error) {
        throw new Error(
            `fetchUser(${userId}) failed: ${error.message}`,
            { cause: error }
        );
    }
}

// BAD: Unhandled rejection
async function fetchUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();  // no status check, no error handling
}

// FORBIDDEN: .catch(() => {})
fetch(url).catch(() => {});  // silently swallowed
```

### Electron: Memory and Security

Electron apps are notorious memory hogs. Agents make this worse.

```javascript
// GOOD: Proper IPC, no nodeIntegration in renderer
const mainWindow = new BrowserWindow({
    webPreferences: {
        nodeIntegration: false,     // MANDATORY for security
        contextIsolation: true,     // MANDATORY for security
        preload: path.join(__dirname, "preload.js"),
    },
});

// BAD: Security nightmare
const mainWindow = new BrowserWindow({
    webPreferences: {
        nodeIntegration: true,      // full Node access from web content
        contextIsolation: false,    // no isolation
    },
});
```

```javascript
// GOOD: Clean up listeners to prevent memory leaks
class JsonStreamProcessor {
    #controller = new AbortController();

    start(stream) {
        stream.on("data", this.#onData, { signal: this.#controller.signal });
    }

    #onData = (chunk) => {
        // process chunk
    };

    destroy() {
        this.#controller.abort();  // removes all listeners at once
    }
}

// BAD: Listener leak
stream.on("data", (chunk) => {
    // anonymous function, can never be removed
});
```

### No `var`, Use `const` by Default

```javascript
// GOOD
const MAX_RETRIES = 3;
let currentRetry = 0;  // let only when reassignment is needed

// BAD
var MAX_RETRIES = 3;  // function-scoped, hoisted, reassignable
var currentRetry = 0;
```

### Handle Process Signals

```javascript
// GOOD: Graceful shutdown
process.on("SIGTERM", async () => {
    console.log("SIGTERM received, shutting down gracefully");
    await server.close();
    await database.disconnect();
    process.exit(0);
});

process.on("unhandledRejection", (reason, promise) => {
    console.error("Unhandled Rejection at:", promise, "reason:", reason);
    process.exit(1);  // CRASH, do not continue in undefined state
});

// BAD: No signal handling, process killed mid-transaction
```

---

## TypeScript

### Minimum Required Linters/Analyzers

- **ESLint** with `@typescript-eslint/eslint-plugin`
- **Prettier** — formatting
- **tsc** with `"strict": true` in tsconfig.json — non-negotiable

```json
// tsconfig.json — minimum strict settings
{
    "compilerOptions": {
        "strict": true,
        "noUncheckedIndexedAccess": true,
        "noImplicitReturns": true,
        "noFallthroughCasesInSwitch": true,
        "forceConsistentCasingInFileNames": true,
        "exactOptionalPropertyTypes": true
    }
}
```

### No `any`

`any` is the TypeScript equivalent of giving up. It disables type checking
for everything it touches, and it spreads — `any` infects every expression
it participates in.

```typescript
// GOOD: Proper types
interface ApiResponse<T> {
    data: T;
    status: number;
    timestamp: Date;
}

async function fetchUsers(): Promise<ApiResponse<User[]>> {
    const response = await fetch("/api/users");
    if (!response.ok) {
        throw new HttpError(`GET /api/users failed: ${response.status}`, response);
    }
    return response.json() as Promise<ApiResponse<User[]>>;
}

// BAD: any everywhere
async function fetchUsers(): Promise<any> {
    const response: any = await fetch("/api/users");
    return response.json();
}
```

### Use `unknown` for Truly Unknown Types

```typescript
// GOOD: unknown forces you to narrow before using
function parseInput(raw: unknown): Config {
    if (typeof raw !== "object" || raw === null) {
        throw new TypeError(`parseInput: expected object, got ${typeof raw}`);
    }
    if (!("version" in raw) || typeof (raw as Record<string, unknown>).version !== "number") {
        throw new TypeError("parseInput: missing or invalid 'version' field");
    }
    // now safe to use
    return raw as Config;
}

// BAD: any lets you skip all checks
function parseInput(raw: any): Config {
    return raw;  // no validation, no safety
}
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
