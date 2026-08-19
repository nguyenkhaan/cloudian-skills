# Docker — AGENTS Protocol

## Dockerfile Best Practices

### Minimum Required Tools

- **hadolint** — Dockerfile linter (the shellcheck of Docker)
- **docker scout** / **trivy** — image vulnerability scanning
- **dive** — analyze image layer efficiency

Install if missing:
```bash
# hadolint
wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64
chmod +x /usr/local/bin/hadolint

# trivy
apt install trivy  # or brew install trivy

# dive
wget https://github.com/wagoodman/dive/releases/latest/download/dive_*_linux_amd64.deb
dpkg -i dive_*.deb
```

Run:
```bash
hadolint Dockerfile
trivy image myapp:latest
dive myapp:latest
```

### Always Pin Base Image Versions

```dockerfile
# GOOD: Pinned digest — reproducible builds
FROM python:3.12.4-slim-bookworm@sha256:abc123...

# ACCEPTABLE: Pinned tag — mostly reproducible
FROM python:3.12.4-slim-bookworm

# BAD: Floating tag — build breaks randomly on Tuesday
FROM python:3.12

# CATASTROPHICALLY BAD: Latest is not a version
FROM python:latest
FROM python
```

A build that worked yesterday must work identically today. Floating tags
make that impossible. The `latest` tag is not a version — it is a prayer.

### Multi-Stage Builds Are Mandatory

Every production image must use multi-stage builds. The build tools,
compilers, and dev dependencies do not belong in the runtime image.

```dockerfile
# GOOD: Multi-stage — build tools stay in builder
FROM golang:1.22-bookworm AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/server

FROM gcr.io/distroless/static-debian12
COPY --from=builder /app /app
EXPOSE 8080
ENTRYPOINT ["/app"]

# BAD: 1.2GB image with gcc, make, and your source code
FROM golang:1.22
COPY . /app
WORKDIR /app
RUN go build -o server ./cmd/server
CMD ["./server"]
```

The bad version ships your source code, the entire Go toolchain, and every
intermediate build artifact. The good version ships a single static binary
in a minimal container.

### Layer Ordering: Dependencies Before Code

Docker caches layers. Put things that change rarely (dependencies) before
things that change often (your code). This makes rebuilds fast.

```dockerfile
# GOOD: Dependencies cached separately from code
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

# BAD: Any code change invalidates dependency cache
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
```

### Never Run as Root

```dockerfile
# GOOD: Create and switch to non-root user
RUN groupadd -r appuser && useradd -r -g appuser -d /home/appuser appuser
USER appuser

# BAD: Container runs as root — one exploit = full host access
# (no USER directive at all)
```

### No Secrets in Images

Secrets do not belong in Dockerfiles. Not as `ARG`. Not as `ENV`. Not
`COPY`'d as files. They end up in image layers, visible to anyone who
runs `docker history`.

```dockerfile
# GOOD: Mount secrets at runtime
# docker run -e API_KEY=xxx myapp
ENV API_KEY=""
# Code reads os.environ["API_KEY"] at runtime

# GOOD: BuildKit secrets for build-time needs
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm install

# BAD: Baked into image layer forever
ARG API_KEY=sk-live-abc123
ENV API_KEY=$API_KEY
COPY .env /app/.env
```

See [secrets.AGENTS.md](./secrets.AGENTS.md) for the full secrets policy.

### HEALTHCHECK Is Not Optional

```dockerfile
# GOOD: Container knows if it's healthy
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# BAD: Orchestrator has no idea if your app is alive or deadlocked
# (no HEALTHCHECK at all)
```

### Use .dockerignore

Every project with a Dockerfile must have a `.dockerignore`. Without it,
`docker build` copies your entire directory — including `.git`, `node_modules`,
`.env`, and every other thing that should not be in the image.

```
# .dockerignore — minimum
.git
.env
.env.*
node_modules
__pycache__
*.pyc
.pytest_cache
.mypy_cache
dist
build
*.md
LICENSE
```

## Docker Compose

### Pin Compose File Version

```yaml
# GOOD: Explicit and readable
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16.3-bookworm
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

### depends_on With Health Checks

Never use bare `depends_on`. It only waits for the container to *start*,
not for the service to be *ready*. Your app will crash trying to connect
to a database that hasn't finished initialization.

```yaml
# GOOD: Wait for actual readiness
depends_on:
  db:
    condition: service_healthy

# BAD: Container started ≠ service ready
depends_on:
  - db
```

### Named Volumes for Persistent Data

```yaml
# GOOD: Named volume — survives container recreation
volumes:
  pgdata:

# BAD: Bind mount to random host path
volumes:
  - ./data:/var/lib/postgresql/data  # permissions nightmare
```

## Common Agent Docker Mistakes

1. **Forgetting .dockerignore** — builds are slow, images are huge,
   secrets leak into layers.
2. **Using `latest` tag** — "it worked yesterday" is not a deployment
   strategy.
3. **Running as root** — every container exploit becomes a host exploit.
4. **No healthcheck** — orchestrator restarts healthy containers, ignores
   dead ones.
5. **`apt-get install` without `--no-install-recommends`** — installs
   200MB of X11 libraries for a headless server.
6. **Separate `RUN` for each command** — creates unnecessary layers.
   Chain with `&&`.

```dockerfile
# GOOD: Single layer, minimal install, clean cache
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# BAD: Three layers, recommended packages, cache left behind
RUN apt-get update
RUN apt-get install curl
RUN apt-get install ca-certificates
```

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
