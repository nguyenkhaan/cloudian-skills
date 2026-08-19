# General Principles Across All Languages — AGENTS Protocol

## General Principles Across All Languages

### Functions Do One Thing

A function has one job. If you need the word "and" to describe what it
does, it's doing too much. `validate_and_save_config()` is two functions
pretending to be one.

### Names Must Be Descriptive

```
GOOD:  retry_count, max_connections, parse_header, is_authenticated
BAD:   rc, mc, ph, ia, temp, data, result, handle, process
```

"data" is not a name. Everything is data. "result" is not a name. Every
function returns a result. "handle" is not a name. Handle what? Be
specific. The few extra characters cost nothing to type and save minutes
of reading.

Exceptions: `i`, `j`, `k` for loop indices. `n` for count. `x`, `y`, `z`
for coordinates. These are universal conventions with centuries of
mathematical precedent.

### Comments Explain Why, Not What

```python
# GOOD: Explains WHY
# Retry with exponential backoff because the payment API
# rate-limits to 10 requests/second and returns 429
for attempt in range(max_retries):
    ...

# BAD: Explains WHAT (the code already says this)
# increment counter by 1
counter += 1

# TERRIBLE: Lies
# This function is thread-safe
def update_global_state():  # narrator: it was not thread-safe
    ...
```

### No Magic Numbers

```python
# GOOD
MAX_RETRY_ATTEMPTS = 5
SOCKET_TIMEOUT_SECONDS = 30
HTTP_STATUS_NOT_FOUND = 404

# BAD
for i in range(5):
    socket.settimeout(30)
    if response.status == 404:
        ...
```

### Encoding Is Always UTF-8

Unless there is a documented, specific, unavoidable reason otherwise. When
opening files, specify encoding explicitly. Do not rely on system defaults.

```python
# GOOD
with open(path, "r", encoding="utf-8") as f:
    ...

# BAD: Uses system default encoding which varies by platform
with open(path, "r") as f:
    ...
```

### Tests Are Not Optional

If you write a function, write a test. The test proves the function works.
Without the test, you have a hypothesis. With the test, you have evidence.

Tests cover:
1. The happy path (valid inputs → expected outputs)
2. Edge cases (empty inputs, boundary values, zeros)
3. Error cases (invalid inputs → expected exceptions)

This is not abstract advice. Here is what a proper test looks like:

```python
import pytest
from myapp.auth import hash_password, verify_password, AuthError


class TestHashPassword:
    """Tests for hash_password() — covers happy path, edge cases, errors."""

    def test_valid_password_returns_hash(self) -> None:
        result = hash_password("correcthorsebatterystaple")
        assert isinstance(result, str)
        assert len(result) == 60  # bcrypt hash length
        assert result.startswith("$2b$")

    def test_hash_is_not_plaintext(self) -> None:
        password = "secret123"
        assert hash_password(password) != password

    def test_same_password_produces_different_hashes(self) -> None:
        h1 = hash_password("same")
        h2 = hash_password("same")
        assert h1 != h2  # salt must differ

    def test_empty_password_raises(self) -> None:
        with pytest.raises(AuthError, match="password must not be empty"):
            hash_password("")

    def test_none_password_raises(self) -> None:
        with pytest.raises(TypeError):
            hash_password(None)  # type: ignore[arg-type]

    def test_whitespace_only_password_raises(self) -> None:
        with pytest.raises(AuthError, match="password must not be empty"):
            hash_password("   ")

    def test_unicode_password(self) -> None:
        result = hash_password("пароль_密码_パスワード")
        assert verify_password("пароль_密码_パスワード", result)

    def test_max_length_password(self) -> None:
        long_pw = "a" * 72  # bcrypt truncates at 72 bytes
        result = hash_password(long_pw)
        assert verify_password(long_pw, result)

    def test_beyond_max_length_warns(self) -> None:
        with pytest.warns(UserWarning, match="truncated to 72 bytes"):
            hash_password("a" * 100)
```

Notice: the test for empty string, the test for `None`, the test for
whitespace, the test for Unicode, the test for boundary length. These are
not paranoia. These are the inputs your function **will** receive in
production. If you didn't test them, you don't know if they work.

The same discipline applies in every language:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn parse_port_valid() {
        assert_eq!(parse_port("8080"), Ok(8080));
    }

    #[test]
    fn parse_port_zero_is_invalid() {
        assert!(parse_port("0").is_err());
    }

    #[test]
    fn parse_port_negative_is_invalid() {
        assert!(parse_port("-1").is_err());
    }

    #[test]
    fn parse_port_above_65535_is_invalid() {
        assert!(parse_port("65536").is_err());
    }

    #[test]
    fn parse_port_non_numeric() {
        assert!(parse_port("abc").is_err());
    }

    #[test]
    fn parse_port_empty_string() {
        assert!(parse_port("").is_err());
    }

    #[test]
    fn parse_port_boundary_values() {
        assert_eq!(parse_port("1"), Ok(1));
        assert_eq!(parse_port("65535"), Ok(65535));
    }
}
```

```go
func TestParsePort(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int
        wantErr bool
    }{
        {name: "valid", input: "8080", want: 8080},
        {name: "min valid", input: "1", want: 1},
        {name: "max valid", input: "65535", want: 65535},
        {name: "zero", input: "0", wantErr: true},
        {name: "negative", input: "-1", wantErr: true},
        {name: "overflow", input: "65536", wantErr: true},
        {name: "non-numeric", input: "abc", wantErr: true},
        {name: "empty", input: "", wantErr: true},
        {name: "leading space", input: " 80", wantErr: true},
        {name: "float", input: "80.0", wantErr: true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParsePort(tt.input)
            if tt.wantErr {
                if err == nil {
                    t.Errorf("ParsePort(%q) = %d, want error", tt.input, got)
                }
                return
            }
            if err != nil {
                t.Errorf("ParsePort(%q) unexpected error: %v", tt.input, err)
                return
            }
            if got != tt.want {
                t.Errorf("ParsePort(%q) = %d, want %d", tt.input, got, tt.want)
            }
        })
    }
}
```

The pattern is always the same: valid input, boundary values, invalid
input, empty input, type abuse. Every function. Every time. No shortcuts.

### Tests Call Real Code and Fail Hard

A test that doesn't call the real code is a bedtime story. It makes you
feel safe, but it protects nothing.

Tests **must** import and invoke the actual production functions, classes,
and modules. Not copies. Not reimplementations. Not mocks of the thing
being tested. The real thing. If the real thing is hard to test, that's
a design problem — fix the design, don't fake the test.

Mocks are acceptable **only** for external boundaries: network calls,
databases, filesystems, third-party APIs. Everything else runs for real.

When a test fails, it must **crash loudly** with a message that tells you
exactly what broke. No soft assertions. No "log a warning and continue".
No test that passes green while the underlying code is broken.

```python
# GOOD: Hard failure with diagnostic info
def test_create_user_returns_valid_id(db: Database) -> None:
    user = create_user(db, name="Alice", email="alice@example.com")
    assert user.id > 0, f"Expected positive user ID, got {user.id}"
    assert user.name == "Alice"

    # Verify it actually persisted — call the REAL read path
    fetched = get_user_by_id(db, user.id)
    assert fetched is not None, f"User {user.id} not found after creation"
    assert fetched.email == "alice@example.com"

# BAD: Mock the thing you're testing
def test_create_user(mock_db):
    mock_db.insert.return_value = Mock(id=1, name="Alice")
    user = create_user(mock_db, name="Alice", email="alice@example.com")
    assert user.id == 1  # you tested that Mock returns what you told it to
    # congratulations, you tested nothing
```

### The Testing Pyramid: White Box, Grey Box, Black Box

Tests exist at three levels. All three are mandatory for any non-trivial
project. The ratio should be roughly 70/20/10.

**White box (unit)** — tests individual functions with full knowledge of
internals. Fast, many, cover edge cases:

```python
# White box: directly tests the hash validation logic
def test_validate_hash_rejects_truncated_input() -> None:
    truncated = "$2b$12$abcdef"  # valid prefix, invalid length
    assert not is_valid_bcrypt_hash(truncated)
```

**Grey box (integration)** — tests that components work together correctly.
Knows the architecture, tests real interactions between modules:

```python
# Grey box: tests that the auth module correctly talks to the real database
@pytest.fixture
def app_db(tmp_path: Path) -> Database:
    db = Database(tmp_path / "test.db")
    db.run_migrations()
    yield db
    db.close()

def test_login_flow_with_real_db(app_db: Database) -> None:
    create_user(app_db, name="bob", email="bob@test.com", password="secret")
    token = authenticate(app_db, email="bob@test.com", password="secret")
    assert token is not None
    assert isinstance(token, str)
    assert len(token) > 32

    user = verify_token(app_db, token)
    assert user.email == "bob@test.com"
```

**Black box (end-to-end)** — tests the entire system as a user would see
it. Knows nothing about internals. Talks to real HTTP endpoints, real UI,
real databases (or created for the test).

### Spin Up Real Environments

If the project has a Docker setup, tests **must** use it. Do not simulate
what Docker provides. Start the actual containers, wait for health checks,
run tests against the live system, tear everything down.

```python
# GOOD: Real environment, real HTTP calls, real assertions
import subprocess
import time
import httpx
import pytest


@pytest.fixture(scope="session")
def live_server():
    """Start the full application stack via docker-compose."""
    subprocess.run(
        ["docker", "compose", "up", "-d", "--build", "--wait"],
        check=True,
        timeout=120,
    )
    # Wait for the health endpoint
    base_url = "http://localhost:8080"
    for attempt in range(30):
        try:
            r = httpx.get(f"{base_url}/health", timeout=2)
            if r.status_code == 200:
                break
        except httpx.ConnectError:
            pass
        time.sleep(1)
    else:
        pytest.fail("Server did not become healthy within 30 seconds")

    yield base_url

    subprocess.run(["docker", "compose", "down", "-v"], check=True)


class TestAPIBlackBox:
    """Black-box tests: no knowledge of internals, only HTTP contract."""

    def test_health_endpoint(self, live_server: str) -> None:
        r = httpx.get(f"{live_server}/health")
        assert r.status_code == 200
        assert r.json()["status"] == "ok"

    def test_create_and_fetch_user(self, live_server: str) -> None:
        # Create
        r = httpx.post(
            f"{live_server}/api/users",
            json={"name": "Alice", "email": "alice@test.com"},
        )
        assert r.status_code == 201, f"Create failed: {r.status_code} {r.text}"
        user_id = r.json()["id"]
        assert isinstance(user_id, int)

        # Fetch — verify it actually persists
        r = httpx.get(f"{live_server}/api/users/{user_id}")
        assert r.status_code == 200
        assert r.json()["name"] == "Alice"
        assert r.json()["email"] == "alice@test.com"

    def test_create_user_invalid_email_returns_422(self, live_server: str) -> None:
        r = httpx.post(
            f"{live_server}/api/users",
            json={"name": "Bob", "email": "not-an-email"},
        )
        assert r.status_code == 422, (
            f"Expected 422 for invalid email, got {r.status_code}: {r.text}"
        )

    def test_get_nonexistent_user_returns_404(self, live_server: str) -> None:
        r = httpx.get(f"{live_server}/api/users/999999")
        assert r.status_code == 404
```

For frontend testing, use **Playwright** or equivalent. Not fake DOM
libraries. Real browser, real clicks, real assertions:

```python
# GOOD: Real browser, real UI interaction
from playwright.sync_api import Page, expect


def test_login_page_rejects_empty_password(page: Page, live_server: str) -> None:
    page.goto(f"{live_server}/login")
    page.fill("[data-testid=email]", "user@test.com")
    page.fill("[data-testid=password]", "")
    page.click("[data-testid=submit]")
    expect(page.locator("[data-testid=error]")).to_have_text("Password is required")


def test_successful_login_redirects_to_dashboard(
    page: Page, live_server: str
) -> None:
    # Precondition: user exists (created via API in fixture)
    page.goto(f"{live_server}/login")
    page.fill("[data-testid=email]", "alice@test.com")
    page.fill("[data-testid=password]", "correctpassword")
    page.click("[data-testid=submit]")
    page.wait_for_url(f"{live_server}/dashboard")
    expect(page.locator("h1")).to_have_text("Welcome, Alice")
```

### The Test Coverage Contract

| Test Level       | What It Proves                          | Runs Against        | Speed     | Count   |
|------------------|-----------------------------------------|---------------------|-----------|---------|
| **White box**    | Individual logic is correct             | Functions/classes    | Fast      | Many    |
| **Grey box**     | Components integrate correctly          | Real modules + DB   | Medium    | Moderate|
| **Black box**    | System works as the user sees it        | Live HTTP/UI        | Slow      | Few     |

All three levels must exist. A project with only unit tests has proven
that its bricks are solid but has no evidence the building stands. A
project with only E2E tests has proven the happy path works but will take
hours to diagnose when something breaks deep inside.

The agent must write tests at the appropriate level for the change being
made. A bugfix gets a regression test at the level where the bug
manifested. A new API endpoint gets white-box tests for the handler logic,
grey-box tests for database integration, and black-box tests for the HTTP
contract.

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
