# SQL & Databases — AGENTS Protocol

## SQL: Every Query Is a Contract

SQL is not a scripting language you throw together. A query is a contract
with a database engine that manages someone's data. A bad query doesn't
just run slowly — it locks tables, corrupts state, or deletes production
data. Treat SQL with the same rigor as systems code.

### Minimum Required Tools

- **sqlfluff** — SQL linter and formatter (supports PostgreSQL, MySQL, SQLite, etc.)
- **pgFormatter** or **sql-formatter** — formatting enforcement
- **explain analyze** — you must read query plans, not guess at performance

Install if missing:
```bash
pip install sqlfluff
# or
npm install -g sql-formatter
```

Run:
```bash
sqlfluff lint queries/
sqlfluff fix queries/
```

### SQL Injection: The One Rule

**Never concatenate user input into SQL strings.** Not "usually". Not
"unless you sanitize first". Never. Use parameterized queries. Always.

```python
# GOOD: Parameterized query — input is data, not code
cursor.execute(
    "SELECT * FROM users WHERE email = %s AND status = %s",
    (email, "active")
)

# BAD: String concatenation — SQL injection in 3, 2, 1...
cursor.execute(
    f"SELECT * FROM users WHERE email = '{email}' AND status = 'active'"
)
# email = "'; DROP TABLE users; --"  congratulations

# ALSO BAD: f-string with "sanitization"
cursor.execute(
    f"SELECT * FROM users WHERE email = '{email.replace(chr(39), '')}'"
)
# There are a hundred ways to bypass naive sanitization. Don't try.
```

```javascript
// GOOD: Node.js with parameterized query
const result = await pool.query(
    "SELECT * FROM users WHERE email = $1 AND status = $2",
    [email, "active"]
);

// BAD
const result = await pool.query(
    `SELECT * FROM users WHERE email = '${email}'`
);
```

```go
// GOOD
row := db.QueryRow("SELECT * FROM users WHERE email = $1", email)

// BAD
row := db.QueryRow(fmt.Sprintf("SELECT * FROM users WHERE email = '%s'", email))
```

### Always Use Transactions for Multi-Statement Operations

```sql
-- GOOD: Atomic operation — either both happen or neither does
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- BAD: First succeeds, second fails — money vanishes
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
```

### Migrations Are Forward-Only and Versioned

Database migrations must be:
1. **Versioned** — numbered sequentially, never reordered
2. **Forward-only** — never edit a migration that has been applied
3. **Reversible** — every `up` has a corresponding `down`
4. **Tested** — run on a copy of production schema before deploying

```sql
-- migrations/003_add_user_email_index.up.sql
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);

-- migrations/003_add_user_email_index.down.sql
DROP INDEX CONCURRENTLY IF EXISTS idx_users_email;
```

Never use `IF NOT EXISTS` as a substitute for proper migration ordering.
It hides the fact that your migrations are running out of order or
being skipped.

### SELECT * Is Forbidden in Application Code

```sql
-- GOOD: Explicit columns — schema changes don't break your app
SELECT id, email, created_at FROM users WHERE status = 'active';

-- BAD: Schema change adds a 10MB blob column, your API now returns it
SELECT * FROM users WHERE status = 'active';
```

`SELECT *` is acceptable in ad-hoc queries and debugging. In application
code, it is a time bomb.

### Always EXPLAIN Before Shipping

Before any query goes to production, run `EXPLAIN ANALYZE` on realistic
data. Not on your dev database with 10 rows — on a copy of production
or a dataset of similar size.

```sql
-- Check your query plan
EXPLAIN ANALYZE
SELECT u.id, u.email, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.email;

-- Look for:
-- ✓ Index scans (not sequential scans on large tables)
-- ✓ Reasonable row estimates
-- ✗ Seq Scan on users (cost=0.00..15234.00 rows=500000)  ← ADD AN INDEX
-- ✗ Nested Loop with no index on inner table ← DANGER
```

### Index Discipline

```sql
-- GOOD: Index on columns used in WHERE, JOIN, ORDER BY
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_created_at ON orders (created_at);

-- GOOD: Composite index for common query pattern
CREATE INDEX idx_orders_user_status ON orders (user_id, status);

-- BAD: Index on every column "just in case"
-- (slows down writes, wastes storage, confuses the query planner)

-- BAD: No indexes at all on a table with millions of rows
-- (every query is a full table scan)
```

### NULL Handling

NULL is not empty string. NULL is not zero. NULL is not false.
NULL is *unknown*. Every comparison with NULL returns NULL (not false).

```sql
-- GOOD: Explicit NULL checks
SELECT * FROM users WHERE deleted_at IS NULL;
SELECT COALESCE(nickname, email) AS display_name FROM users;

-- BAD: This returns ZERO rows, always, even if deleted_at is NULL
SELECT * FROM users WHERE deleted_at = NULL;

-- BAD: This silently excludes NULL values
SELECT * FROM users WHERE status != 'banned';
-- Users with status = NULL are NOT returned. Probably a bug.
-- Fix: WHERE status IS DISTINCT FROM 'banned'
```

### ORM Discipline

ORMs are tools, not substitutes for understanding SQL. If you use an ORM:

1. **Log the generated SQL** during development. Read it. Know what your
   ORM is actually doing.
2. **Never use lazy loading** in web applications. It causes N+1 query
   problems that turn a 50ms page load into a 5-second one.
3. **Use raw SQL** for complex queries. An ORM query that takes 15 lines
   of builder chaining is less readable than the equivalent 5-line SQL.
4. **Run EXPLAIN** on ORM-generated queries the same way you would on
   hand-written SQL.

```python
# GOOD: Eager loading — one query with JOIN
users = session.query(User).options(joinedload(User.orders)).all()

# BAD: N+1 — one query for users, then one query per user for orders
users = session.query(User).all()
for user in users:
    print(user.orders)  # each access triggers a separate SELECT
```

### Row-Level Security (RLS): The Database Enforces Access

Application-level access checks are necessary but insufficient. A single
missed `WHERE telegram_id = :current_user` clause and you have a data leak.
PostgreSQL Row-Level Security pushes this enforcement into the database
itself — every query is filtered by policy, regardless of what the
application forgot to do.

The pattern: set session-level variables via `set_config()`, then define
policies that reference those variables.

```python
# At the start of every request, bind the security context to the session
async def bind_db_security_context(
    session: AsyncSession,
    context: SecurityDbContext,
) -> None:
    if context.telegram_id <= 0:
        raise RlsContextError("telegram_id must be positive")
    statements = [
        ("SELECT set_config('app.telegram_id', :value, true)", {"value": str(context.telegram_id)}),
        ("SELECT set_config('app.mode', :value, true)", {"value": context.mode}),
        ("SELECT set_config('app.scope_key', :value, true)", {"value": context.scope_key or ""}),
    ]
    for statement, params in statements:
        await session.execute(text(statement), params)
```

```sql
-- Policy: users can only see their own rows
ALTER TABLE user_memories ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_memories FORCE ROW LEVEL SECURITY;

CREATE POLICY user_memories_isolation ON user_memories
    USING (
        current_user = 'app_admin'
        OR (
            telegram_id = current_setting('app.telegram_id', true)::bigint
            AND scope = current_setting('app.mode', true)
            AND coalesce(scope_key, '') = current_setting('app.scope_key', true)
        )
    );
```

Key rules for RLS:
1. **`FORCE ROW LEVEL SECURITY`** — applies policies even to table owners.
2. **Admin override via `current_user` check** in the policy expression, not
   via `BYPASSRLS` role attribute. `BYPASSRLS` is a nuclear option — if
   compromised, it sees everything.
3. **Test RLS with real databases** — not mocks. Verify that user A cannot
   see user B's data, that an outsider sees zero rows, and that admin sees
   all rows. These are not unit tests — they are security contracts.

### Three-Role Separation Per Database

Never connect to the database with a single all-powerful user. Split into
three roles:

| Role | Purpose | Privileges |
|------|---------|------------|
| **owner** | DDL, migrations, bootstrap | `CREATE TABLE`, `ALTER`, schema changes |
| **runtime** | Application queries | `SELECT`, `INSERT`, `UPDATE`, `DELETE` on data tables, RLS-restricted |
| **admin** | Operational queries, admin panel | Same as runtime but policy includes admin override clause |

```yaml
# docker-compose.yml — separate databases per domain
services:
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: app_main_owner  # owner role creates the schema
      POSTGRES_PASSWORD: ${DB_OWNER_PASSWORD:?required}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_main_owner -d app"]

  security-postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: app_security_owner
      POSTGRES_PASSWORD: ${DB_SECURITY_OWNER_PASSWORD:?required}
```

The application never connects as `owner` at runtime. Bootstrap and
migrations run as owner, then the process switches to `runtime`. If the
runtime role is compromised, it cannot `DROP TABLE` or `ALTER COLUMN` —
it can only read/write data within RLS boundaries.

### Database-Per-Domain Isolation

Sensitive data that has different access patterns or compliance requirements
belongs in a separate database. Not a separate schema — a separate
PostgreSQL instance.

- **Main database** — user profiles, sessions, usage logs
- **Security database** — encryption keys, user salts, security profiles
- **Domain-specific databases** — chat messages, audit events

A compromised connection to the main database cannot query the security
database. This is defense in depth.

### Application-Layer Encryption for Sensitive Fields

RLS controls who sees which rows. Encryption controls what they see even if
they get the rows. Sensitive free-text fields (messages, prompts, user
memories) should be encrypted at the application layer with per-user derived
keys.

```python
# Encrypt before storage — database sees only ciphertext
protected = protect_free_text(
    plaintext=user_message,
    key_material=derived_key,
    field_kind="chat_text",
    scope="main",
    scope_key=scope_key,
    max_length=limits.free_text_large,
)
# stored_value is ChaCha20-Poly1305 ciphertext — useless without the key

# Decrypt on read — only the owning user's derived key works
plaintext = reveal_free_text(
    stored_value=row.ciphertext,
    key_material=derived_key,
    field_kind="chat_text",
    scope=row.scope,
    scope_key=row.scope_key,
)
```

Key discipline:
1. **Per-user salt** stored in a separate security database.
2. **Key version** tracked per user — enables key rotation without re-encrypting
   everything at once.
3. **Master secret** never stored in the database — only in environment variables.
4. **Scope binding** — ciphertext is bound to its scope, preventing cross-context
   decryption.

### Idempotent Writes: UPSERT Over INSERT

When the same operation might be retried (network timeout, user double-click,
queue redelivery), use `ON CONFLICT` to make writes idempotent.

```sql
-- GOOD: Idempotent — safe to retry
INSERT INTO security_user_profiles
    (telegram_id, user_salt_hex, key_version, cipher_algorithm, created_at, updated_at)
VALUES (:telegram_id, :salt, :version, :algorithm, NOW(), NOW())
ON CONFLICT (telegram_id) DO NOTHING;

-- GOOD: Upsert — update if exists, insert if not
INSERT INTO secrets (record_kind, record_key, field_name, ciphertext, updated_at)
VALUES (:kind, :key, :field, :cipher, NOW())
ON CONFLICT (record_kind, record_key, field_name)
DO UPDATE SET ciphertext = EXCLUDED.ciphertext, updated_at = NOW()
RETURNING secret_id;

-- BAD: Fails on retry, or worse — creates duplicates
INSERT INTO security_user_profiles
    (telegram_id, user_salt_hex, key_version, cipher_algorithm, created_at, updated_at)
VALUES (:telegram_id, :salt, :version, :algorithm, NOW(), NOW());
```

### ORM Model Discipline

If using an ORM (SQLAlchemy, Prisma, etc.), the models are contracts.
Every column must have an explicit type. Every lookup column must have an
explicit index.

```python
# GOOD: SQLAlchemy 2.0 style — explicit types, explicit indexes
class UserMemory(Base):
    __tablename__ = "user_memories"

    memory_id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    telegram_id: Mapped[int] = mapped_column(BigInteger, index=True)
    scope: Mapped[str] = mapped_column(String(32), default="main", index=True)
    scope_key: Mapped[str | None] = mapped_column(String(64), nullable=True, index=True)
    content: Mapped[str] = mapped_column(Text, default="")
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))
    updated_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))

# BAD: Implicit types, no indexes, no timezone
class UserMemory(Base):
    __tablename__ = "user_memories"

    memory_id = Column(Integer, primary_key=True)
    telegram_id = Column(Integer)  # no index, no BigInteger
    scope = Column(String)  # no max length, no index
    content = Column(String)  # no Text type, no default
    created_at = Column(DateTime)  # no timezone
```

Rules:
1. **`Mapped[T]` with explicit type** — the ORM model IS the schema documentation.
2. **`index=True`** on every column used in WHERE, JOIN, or ORDER BY.
3. **`DateTime(timezone=True)`** — always. Naive datetimes are bugs waiting to
   cross a timezone boundary.
4. **`String(N)` with max length** — unbounded strings invite unbounded storage.
5. **`__slots__ = ()` on DeclarativeBase** — prevents accidental attribute assignment.

### Audit Trails for Sensitive Operations

Any operation that changes user state, moves money, or grants access must
be logged to an immutable audit table. This is not optional — it is a
compliance and debugging requirement.

```python
class AdminEvent(Base):
    __tablename__ = "admin_events"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    admin_id: Mapped[str] = mapped_column(String(64))
    action: Mapped[str] = mapped_column(String(64))
    target_telegram_id: Mapped[int] = mapped_column(BigInteger)
    delta_credits: Mapped[float] = mapped_column(Float)
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))
```

The audit table is append-only. No UPDATE, no DELETE. If you need to
"correct" an entry, insert a new compensating record.

### E2E Database Tests: Real Postgres, Not Mocks

Database tests that mock the database are testing your mock, not your
queries. RLS policies, constraint violations, and query plans cannot be
mocked. Spin up real Postgres in Docker and test against it.

```python
# GOOD: Test RLS isolation with real database roles
@pytest.mark.asyncio
async def test_rls_user_isolation():
    owner_engine, owner_sf = build_session_factory("OWNER_DSN")
    runtime_engine, runtime_sf = build_session_factory("RUNTIME_DSN")

    try:
        # Owner inserts data for two users
        async with owner_engine.begin() as conn:
            await conn.execute(text(
                "INSERT INTO user_memories (telegram_id, scope, content, created_at, updated_at) "
                "VALUES (:tid, 'main', '', NOW(), NOW())"
            ), {"tid": USER_A_ID})
            await conn.execute(text(
                "INSERT INTO user_memories (telegram_id, scope, content, created_at, updated_at) "
                "VALUES (:tid, 'main', '', NOW(), NOW())"
            ), {"tid": USER_B_ID})

        # Runtime with User A context sees only User A's row
        async with runtime_sf() as session:
            await bind_db_security_context(session, SecurityDbContext(
                telegram_id=USER_A_ID, mode="main", role_kind="runtime",
            ))
            count = await scalar_count(session, "SELECT count(*) FROM user_memories", {})
            assert count == 1  # RLS filters to User A only

        # Runtime without context sees zero rows
        async with runtime_sf() as session:
            count = await scalar_count(session, "SELECT count(*) FROM user_memories", {})
            assert count == 0  # no context = no access
    finally:
        async with owner_engine.begin() as conn:
            await conn.execute(text("DELETE FROM user_memories WHERE telegram_id IN (:a, :b)"),
                {"a": USER_A_ID, "b": USER_B_ID})
        await owner_engine.dispose()
        await runtime_engine.dispose()
```

Key test patterns:
1. **Separate DSNs** for owner, runtime, admin — test each role's visibility.
2. **Cleanup in `finally`** — always delete test data regardless of pass/fail.
3. **Dispose engines** — prevent connection pool leaks between tests.
4. **Assert zero rows without context** — proves RLS is active, not permissive.
5. **Assert admin override** — admin sees all rows through policy, not `BYPASSRLS`.

### Connection Pools & Timeouts: Capacity Is a Feature

Agents often make one of two opposite mistakes:
1. Open a new DB connection for every request (connection storm).
2. Use an unbounded pool (slow leak until `max_connections` exhaustion).

Both fail in production. Every service must set explicit pool limits and
explicit timeouts.

#### Baseline Defaults (Start Here)

For a typical API service instance:
- `pool_size = 10`
- `max_overflow = 20`
- `pool_timeout = 30` seconds
- `pool_recycle = 1800` seconds
- `pool_pre_ping = true`

```python
# SQLAlchemy async engine (PostgreSQL/MySQL)
engine = create_async_engine(
    dsn,
    pool_size=10,
    max_overflow=20,
    pool_timeout=30,
    pool_recycle=1800,
    pool_pre_ping=True,
)
SessionFactory = async_sessionmaker(engine, expire_on_commit=False)
```

Never instantiate `create_async_engine()` per request. Create one engine
per process and reuse it.

#### Queue Timeouts vs Statement Timeouts

`pool_timeout` is how long a request waits for a free connection.
`statement_timeout` is how long SQL is allowed to execute.
You need both.

```python
# GOOD: Fail fast when DB is saturated and when query is pathological
await session.execute(text("SET statement_timeout = '30s'"))
await session.execute(text("SET lock_timeout = '5s'"))
await session.execute(text("SET idle_in_transaction_session_timeout = '60s'"))

# BAD: No timeout — one stuck query can hold a worker forever
await session.execute(text("SELECT heavy_join_without_index()"))
```

#### PostgreSQL Timeout Policy

Set these explicitly (at role level, database level, or per session):
- `statement_timeout = 30s` (or stricter for interactive endpoints)
- `lock_timeout = 5s`
- `idle_in_transaction_session_timeout = 60s`
- `connect_timeout = 5s` (driver/DSN)

These timeouts convert hanging behavior into controlled failure with clear
errors and bounded resource usage.

#### MySQL Timeout Policy

For MySQL-compatible systems, configure both client and server limits:
- Client: `connect_timeout`, `read_timeout`, `write_timeout`
- Server: `wait_timeout`, `interactive_timeout`

If client timeout is infinite while server timeout is strict, you'll get
random disconnects under load. If both are infinite, you'll leak resources
silently.

#### Capacity Math (Do This Before Deploy)

If you have `N` service instances, total potential DB connections are:

`N * (pool_size + max_overflow)`

That total must be **strictly below** database `max_connections`, with
reserved headroom for:
- migrations/bootstrap jobs
- admin sessions
- monitoring/backup tooling

Example:
- 8 replicas
- `pool_size=10`, `max_overflow=20`
- potential = `8 * 30 = 240`

If PostgreSQL `max_connections=300`, this may still be unsafe once workers,
cron jobs, and admin shells are added. Capacity planning is not optional.

#### Separate Pools by Workload Class

Use separate engines/pools when workloads differ:
- API request path (low latency, strict timeout)
- background workers (longer jobs, lower concurrency)
- admin/maintenance operations

Without isolation, one noisy background queue can starve request traffic.

#### Monitor the Pool

At minimum track:
- active connections
- idle connections
- waiting requests
- pool wait duration
- timeout count (pool timeout / statement timeout)

If you don't measure queue wait and timeout rate, you won't see saturation
until users do.

### Common Agent SQL Mistakes

1. **String concatenation in queries** — SQL injection. Always parameterize.
2. **No transactions** — partial updates corrupt data.
3. **SELECT * in app code** — schema changes break everything.
4. **Missing indexes on JOIN/WHERE columns** — queries that "work" on
   test data and timeout on production.
5. **Editing applied migrations** — creates schema drift between environments.
6. **Ignoring NULL semantics** — silent data exclusion bugs.
7. **N+1 queries through ORMs** — death by a thousand SELECTs.
8. **No LIMIT on user-facing queries** — one table scan away from OOM.
9. **Single database role for everything** — one compromised connection = full
   DDL access. Use owner/runtime/admin separation.
10. **Application-only access control** — one missed WHERE clause = data leak.
    Use RLS as defense in depth.
11. **Mocking the database in tests** — proves nothing about real queries, RLS,
    or constraints. Test against real Postgres.
12. **`DateTime` without timezone** — naive datetimes silently corrupt across
    time zones. Always `DateTime(timezone=True)`.
13. **Non-idempotent writes** — retries create duplicates. Use `ON CONFLICT`.
14. **No audit trail** — when something goes wrong with user data or money,
    you have no record of what happened.
15. **No pool/timeout limits** — either one connection per request or an
    unbounded pool. Both lead to resource exhaustion under load.

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
