# Web Services

> **Version:** 1.0.0
> **Last Updated:** December 2025
> **Extends:** [General / Core](general-core.md)
> **Template:** [template-python-web-service](https://github.com/landerox/template-python-web-service)

Standards for building APIs, microservices, and web applications. Includes Docker containerization, Cloud Run deployment, and observability.

---

## Use Cases

| Type | Examples |
|------|----------|
| REST APIs | Public/private APIs, microservices |
| Web Applications | FastAPI + templates, Streamlit, Django |
| Webhooks | Event receivers, integrations |
| Backend for Frontend | BFF pattern |

---

## Stack

### Web Framework

| Tool | Description |
|------|-------------|
| **FastAPI** | High-performance async framework with automatic OpenAPI docs and Pydantic validation |

### ASGI Server

| Tool | Description | When to Use |
|------|-------------|-------------|
| **Granian** | High-performance ASGI server (Rust), integrated worker management | Recommended default |
| **Uvicorn** | Standard ASGI server, very mature | Fallback if Granian incompatibilities |

**Granian vs Uvicorn:**

| Aspect | Granian | Uvicorn |
|--------|---------|---------|
| Performance | Superior (Rust) | Very good (with uvloop) |
| Worker management | Integrated | Requires Gunicorn |
| Docker image | Simpler | More complex |
| Maturity | Newer | Very mature |

### HTTP Client

| Tool | Description |
|------|-------------|
| **httpx** | Async/sync HTTP client with HTTP/2 support, configurable timeouts |

### Configuration Tools

| Tool | Description |
|------|-------------|
| **pydantic-settings** | Typed configuration from environment variables |
| **python-dotenv** | `.env` file loading for local development |

### Observability Tools

| Tool | Description |
|------|-------------|
| **structlog** | Structured logging (JSON in production, readable in dev) |
| **OpenTelemetry** | Distributed tracing and metrics |
| **Sentry SDK** | Error tracking and performance monitoring |
| **prometheus-client** | Metrics exposition |

### Resilience

| Tool | Description |
|------|-------------|
| **tenacity** | Retries with backoff and circuit breaker patterns |

### High-Performance Serialization (Optional)

| Tool | Description | When to Use |
|------|-------------|-------------|
| **orjson** | Ultra-fast JSON (Rust), drop-in replacement | High-traffic APIs |
| **msgspec** | JSON/MessagePack, 2-5x faster than Pydantic | Critical performance |

> **Note:** Pydantic v2 is fast enough for most cases. Use orjson/msgspec only when profiling indicates serialization is a bottleneck.

### Inter-Service Communication

| Tool | Description | When to Use |
|------|-------------|-------------|
| **httpx** | HTTP/REST client | External APIs, most cases |
| **grpcio** | gRPC framework | High-frequency internal communication |

**gRPC vs REST:**

| Criterion | gRPC | REST |
|-----------|------|------|
| Performance | High (binary, HTTP/2) | Medium (JSON, HTTP/1.1) |
| Streaming | Native bidirectional | Limited (SSE, WebSocket) |
| Contracts | Strict (.proto) | Flexible (OpenAPI optional) |
| Browser support | Limited (grpc-web) | Native |
| Debugging | Harder (binary) | Easy (plain text) |

> **Recommendation:** REST for public APIs and frontends, gRPC for internal high-traffic microservices.

---

## Authentication

### Authentication Tools

| Tool | Description |
|------|-------------|
| **PyJWT** | JWT encoding/decoding |
| **python-jose** | JWT with JWS, JWE, JWK support |
| **passlib** | Password hashing (bcrypt, argon2) |
| **argon2-cffi** | Argon2 implementation |
| **authlib** | OAuth2 / OpenID Connect |

### Password Hashing

| Algorithm | Recommendation |
|-----------|----------------|
| **Argon2id** | Preferred (PHC winner) |
| **bcrypt** | Acceptable, widely used |
| **scrypt** | Acceptable |
| PBKDF2 | Legacy, avoid |
| MD5, SHA1, SHA256 | **Prohibited** for passwords |

### JWT Best Practices

| Practice | Description |
|----------|-------------|
| Algorithm | Use RS256 or ES256, avoid HS256 in production |
| Expiration | Access tokens: 15-60 min |
| Refresh tokens | To renew access tokens |
| Validate claims | Always verify `iss`, `aud`, `exp` |
| No sensitive data | JWT payload is visible (base64) |

---

## API Design

### Structure

| Component | Responsibility |
|-----------|----------------|
| Routers | Endpoint definition, input validation |
| Schemas | Pydantic models for request/response |
| Services | Business logic |
| Repositories | Data access |
| Dependencies | Component injection |

### Versioning

- Include version in path: `/api/v1/...`
- Maintain backward compatibility within a version
- Deprecate before removing

### REST Conventions

| Resource | Convention | Example |
|----------|------------|---------|
| Collection | Plural noun | `/users`, `/orders` |
| Item | Collection + ID | `/users/{id}` |
| Action | Verb (avoid) | `/orders/{id}/cancel` |
| Relationship | Nested resource | `/users/{id}/orders` |
| Filters | Query params | `/users?status=active` |

### HTTP Verbs

| Verb | Use | Idempotent |
|------|-----|------------|
| GET | Read resource(s) | Yes |
| POST | Create resource | No* |
| PUT | Replace complete resource | Yes |
| PATCH | Partially update | Yes |
| DELETE | Delete resource | Yes |

*POST can be idempotent with Idempotency-Key header

### Pydantic Schemas

| Practice | Description |
|----------|-------------|
| Separate schemas | Different schemas for Create/Update/Response |
| Validation in schema | Use Pydantic validators |
| `from_attributes` | Enable to map from ORM |
| Examples | Include examples for OpenAPI documentation |

---

## Error Handling

### Error Format (RFC 7807)

Use Problem Details standard for error responses:

| Field | Type | Description |
|-------|------|-------------|
| `type` | URI | Error type identifier |
| `title` | string | Human-readable error title |
| `status` | integer | HTTP status code |
| `detail` | string | Specific description |
| `instance` | URI | URI of the problem instance |

**Additional fields:**

| Field | Use |
|-------|-----|
| `errors` | Array of validation errors |
| `trace_id` | ID for log correlation |
| `code` | Internal error code |

---

## Pagination

| Strategy | Use | Pros/Cons |
|----------|-----|-----------|
| **Offset-based** | `?page=2&limit=20` | Simple, inefficient on large datasets |
| **Cursor-based** | `?cursor=abc123&limit=20` | Efficient, recommended for public APIs |
| **Keyset** | `?after_id=100&limit=20` | Similar to cursor, column-based |

**Standard response:**

| Field | Description |
|-------|-------------|
| `data` | Results array |
| `meta.total` | Total items (if efficient) |
| `meta.page` / `meta.cursor` | Current position |
| `meta.limit` | Items per page |
| `links.next` | Next page URL |
| `links.prev` | Previous page URL |

---

## Security

### HTTP Security Headers

| Header | Value | Purpose |
|--------|-------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Force HTTPS |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` | Prevent clickjacking |
| `Content-Security-Policy` | Per application | Control loadable resources |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referrer info |

### CORS

| Environment | Configuration |
|-------------|---------------|
| Development | Allow localhost with various ports |
| Staging | Specific staging domains |
| Production | Only production domains, explicit list |

**Best practices:**

| Practice | Description |
|----------|-------------|
| Explicit origin list | Never use `*` with credentials |
| Validate Origin header | Verify against whitelist |
| Limit methods | Only necessary ones |
| Reasonable Max-Age | 86400 (24h) is common |

### Rate Limiting

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Total limit |
| `X-RateLimit-Remaining` | Remaining requests |
| `X-RateLimit-Reset` | Reset timestamp |
| `Retry-After` | Seconds to retry (on 429) |

### Idempotency Keys

For state-mutating operations (POST, PUT):

| Scenario | Response |
|----------|----------|
| First time | Process and save result with key |
| Repeated key (success) | Return saved result |
| Repeated key (failure) | Retry operation |
| Expired key | 409 Conflict |

---

## Docker

### Base Image

| Image | Description |
|-------|-------------|
| `python:3.12-slim-bookworm` | Debian Bookworm (stable), slim variant, regular security updates |

**Why Bookworm slim:**

| Reason | Description |
|--------|-------------|
| Stable base | Debian stable, long-term support |
| Minimal size | ~150MB vs ~1GB full image |
| Security | Regular updates, smaller attack surface |
| Compatibility | glibc-based, broad library support |

### Multi-stage Builds (Mandatory)

| Stage | Base | Purpose |
|-------|------|---------|
| **builder** | `python:3.12-slim-bookworm` | Install dependencies with uv |
| **runtime** | `python:3.12-slim-bookworm` | Only virtualenv and source code |

**Benefits:**

| Benefit | Description |
|---------|-------------|
| Smaller image | No build tools, compilers in final image |
| Security | Reduced attack surface |
| Faster deploys | Less to push/pull |
| Layer caching | Dependencies cached separately |

### Build Optimization

| Practice | Description |
|----------|-------------|
| Copy `pyproject.toml` + `uv.lock` first | Leverage layer caching |
| Install dependencies before copying source | Cache dependencies layer |
| Use `--no-cache` for uv | Smaller image |
| Use `.dockerignore` | Faster context, smaller image |

### Environment Variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `PYTHONDONTWRITEBYTECODE` | 1 | Don't create .pyc files |
| `PYTHONUNBUFFERED` | 1 | Unbuffered stdout/stderr for logs |
| `PYTHONFAULTHANDLER` | 1 | Dump traceback on segfault |
| `UV_COMPILE_BYTECODE` | 1 | Compile bytecode during install (faster startup) |
| `UV_LINK_MODE` | copy | Copy files instead of hardlinks |

### Non-Root User (Mandatory)

| Practice | Description |
|----------|-------------|
| Create `appuser` | Dedicated non-root user |
| `chown` app directory | User owns application files |
| `USER appuser` | Switch before CMD/ENTRYPOINT |

**Security benefits:**

| Benefit | Description |
|---------|-------------|
| Least privilege | Process can't modify system |
| Container escape mitigation | Limits damage if compromised |
| Cloud Run requirement | Runs as non-root by default |

### .dockerignore (Mandatory)

Must exclude:

| Category | Examples |
|----------|----------|
| Git | `.git/`, `.gitignore` |
| Python | `__pycache__/`, `.venv/`, `*.egg-info/` |
| Testing | `.pytest_cache/`, `.coverage`, `.nox/` |
| IDEs | `.idea/`, `.vscode/` |
| CI/CD | `.github/` |
| Secrets | `*.pem`, `*.key`, `.env` |
| Docs | `docs/`, `*.md` |

### Graceful Shutdown

| Signal | Action |
|--------|--------|
| SIGTERM | Start graceful shutdown |
| SIGINT | Same as SIGTERM |

**Practices:**

| Practice | Description |
|----------|-------------|
| Finish in-flight requests | Wait for active requests |
| Close DB connections | Release pool connections |
| Flush logs/metrics | Send pending data |
| Shutdown timeout | Time limit (e.g., 30s) |

### ENTRYPOINT vs CMD

| Directive | Use |
|-----------|-----|
| `ENTRYPOINT` | The executable (e.g., `["granian"]`) |
| `CMD` | Default arguments (overridable) |

**Best practice:** Use exec form `["executable", "arg1"]` not shell form.

---

## Health Checks

| Endpoint | Type | What it Verifies |
|----------|------|------------------|
| `/health/live` | Liveness | App responds (simple ping) |
| `/health/ready` | Readiness | DB connected, dependencies OK |
| `/health/startup` | Startup | Complete initialization |

**Best practices:**

| Practice | Description |
|----------|-------------|
| Simple liveness | Don't verify external dependencies |
| Complete readiness | Verify DB, cache, critical services |
| Appropriate timeouts | Configure initialDelaySeconds, periodSeconds |
| Don't cache | Each check should be fresh |

---

## Observability and Monitoring

### Structured Logging

| Environment | Format |
|-------------|--------|
| Development | Readable with colors |
| Production | JSON for indexing |

### Log Levels

| Level | Use |
|-------|-----|
| DEBUG | Detailed debugging information |
| INFO | Normal operation events |
| WARNING | Unexpected but handled situations |
| ERROR | Errors preventing operation completion |
| CRITICAL | Errors requiring immediate attention |

### What to Log

| Event | Level | Information |
|-------|-------|-------------|
| Request received | INFO | method, path, request_id |
| Request completed | INFO | status_code, duration |
| Business operation | INFO | entity, action, identifiers |
| Recoverable error | WARNING | error, context |
| Unrecoverable error | ERROR | error, stack trace, context |

### Correlation IDs

| Header | Description |
|--------|-------------|
| `X-Request-ID` | Unique request ID |
| `X-Correlation-ID` | Common alternative |
| `traceparent` | W3C Trace Context standard |

**Behavior:**

| Scenario | Action |
|----------|--------|
| Header present | Use received ID |
| Header absent | Generate new UUID |
| Calls to other services | Propagate the ID |
| Logs | Include ID in each entry |
| Response | Return ID in header |

### GCP Cloud Logging

| Requirement | Description |
|-------------|-------------|
| JSON format | Mandatory for Cloud Logging |
| `severity` field | Map log level (INFO, ERROR, etc.) |
| `logging.googleapis.com/trace` | Trace ID for correlation |
| `spanId` | Span ID for correlation |

---

## Application Configuration

### 12-Factor App Principles

| Principle | Description |
|-----------|-------------|
| Env vars only | Configuration in environment, not code |
| Fail fast | Validate configuration at startup |
| Type safety | All configuration typed with pydantic-settings |
| Sensible defaults | Default values for development |
| Separate secrets | Never in code or versioned files |

### Organization

| Config Type | Example | Where to Define |
|-------------|---------|-----------------|
| Required | `DATABASE_URL` | Only in environment |
| With default | `DEBUG=false` | Default in code |
| Secrets | `API_KEY` | Secret manager or env |
| Feature flags | `ENABLE_FEATURE_X` | Environment variables |

### .env Files

- `.env.example` versioned with example values
- `.env` never versioned (in .gitignore)
- Use only for local development

---

## Deployment

### Cloud Run Services

| Consideration | Description |
|---------------|-------------|
| Stateless | Don't maintain state in memory between requests |
| Cold starts | Optimize startup time |
| Concurrency | Configure max_instances and min_instances |
| Timeouts | Maximum 60 minutes for HTTP |
| Memory | 256MB - 32GB available |

### GKE (Kubernetes)

| Component | Tool |
|-----------|------|
| Authentication | Workload Identity |
| Development | Skaffold |
| Secrets | External Secrets Operator |
| Observability | OpenTelemetry Collector |
| Metrics | prometheus-client |

**Considerations:**

| Aspect | Practice |
|--------|----------|
| Health checks | Separate liveness and readiness probes |
| Resources | Define requests and limits |
| HPA | Auto-scaling based on metrics |
| PodDisruptionBudget | Guarantee availability during updates |
| Network policies | Restrict traffic between pods |

---

## Testing

### Strategy by Test Type

| Test Type | Strategy | Speed | What to Test |
|-----------|----------|-------|--------------|
| **Unit** | Mocks | ~ms | Business logic, services, isolated functions |
| **Integration** | Testcontainers | ~s | API + real DB, queries, constraints |
| **E2E** | Real services | ~s | Full system, critical paths |

### Recommended Approach

| Layer | Tool | Description |
|-------|------|-------------|
| Unit tests | `pytest-mock` | Mock dependencies, test business logic in isolation |
| API tests | `httpx.AsyncClient` | FastAPI TestClient, test endpoints |
| DB tests | `testcontainers` | Real PostgreSQL/Redis in Docker |
| External APIs | `respx` | Mock httpx requests |

### Why Testcontainers over Mocks for DB

| Aspect | Mocks | Testcontainers |
|--------|-------|----------------|
| Speed | Fast (~ms) | Slower (~s) |
| SQL validation | ❌ No | ✅ Yes |
| Constraints/Types | ❌ No | ✅ Yes |
| Query bugs | ❌ Missed | ✅ Caught |
| CI/CD | Simple | Requires Docker |
| Confidence | Lower | Higher |

> **Recommendation:** Use mocks for unit tests (business logic), testcontainers for integration tests (DB queries, API endpoints with persistence).

### Testing Tools

| Tool | Purpose |
|------|---------|
| **pytest** | Test framework (from general-standards) |
| **pytest-asyncio** | Async test support |
| **pytest-mock** | unittest.mock wrapper |
| **httpx** | AsyncClient for FastAPI testing |
| **testcontainers** | PostgreSQL, Redis, etc. in Docker |
| **respx** | Mock httpx requests |
| **Faker** | Realistic fake data |
| **factory-boy** | Factories for test objects |
| **hypothesis** | Property-based testing for complex logic |

### Test Structure

```bash
tests/
├── unit/              # Fast, mocked, business logic
│   └── services/
├── integration/       # Real DB (testcontainers), API endpoints
│   └── api/
├── e2e/               # Full system (optional)
├── factories/         # factory-boy factories
├── fixtures/          # Shared fixtures
└── conftest.py        # Common fixtures, testcontainers setup
```

### What to Test Where

| What | Unit (Mocks) | Integration (Testcontainers) |
|------|--------------|------------------------------|
| Service business logic | ✅ | |
| Pydantic validation | ✅ | |
| API endpoint responses | | ✅ |
| DB queries (SQLAlchemy) | | ✅ |
| DB constraints/triggers | | ✅ |
| Transactions/rollbacks | | ✅ |
| Cache operations | | ✅ |
| External API calls | ✅ (respx) | |

### Fixtures Pattern

| Fixture | Scope | Purpose |
|---------|-------|---------|
| `db_engine` | session | Testcontainers PostgreSQL |
| `db_session` | function | Fresh session per test, auto-rollback |
| `client` | function | httpx AsyncClient with app |
| `authenticated_client` | function | Client with auth headers |

### Testing Best Practices

| Practice | Description |
|----------|-------------|
| Isolate tests | Each test independent, no shared state |
| Auto-rollback | Rollback DB after each test |
| Factories over fixtures | factory-boy for flexible test data |
| Test behavior, not implementation | Assert outcomes, not internal calls |
| Fast feedback | Run unit tests first, integration after |
| CI/CD markers | `@pytest.mark.integration` to separate |

---

## Additional Files

Beyond [General / Core](general-core.md), web services include:

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage container build |
| `.dockerignore` | Exclude files from build context |
| `compose.yaml` | Local development with services (DB, cache) |

---

## Databases

### ORM

| Tool | Description | When to Use |
|------|-------------|-------------|
| **SQLAlchemy 2.0** | Mature ORM with async support, typed queries | Complex projects, advanced queries |
| **SQLModel** | SQLAlchemy + Pydantic, less boilerplate | Simple CRUDs with FastAPI |

### Drivers

| Tool | Description |
|------|-------------|
| **asyncpg** | High-performance async PostgreSQL driver |
| **psycopg3** | Modern PostgreSQL driver (sync/async) |

### Migrations

| Tool | Description |
|------|-------------|
| **Alembic** | Database migrations for SQLAlchemy |

**Practices:**

| Practice | Description |
|----------|-------------|
| Auto-generated migrations | As starting point, always review |
| Versioned in Git | Track all migrations |
| Idempotent | When possible |

### Connection Pooling

| Parameter | Description |
|-----------|-------------|
| `pool_size` | Number of base connections |
| `max_overflow` | Additional allowed connections |
| `pool_pre_ping` | Verify connections before use |
| `pool_recycle` | Recycle connections periodically |

### Data Practices

| Practice | Description |
|----------|-------------|
| Soft deletes | Mark as deleted instead of physical delete |
| Auditing | `created_at`, `updated_at`, `created_by`, `updated_by` |
| Timezone-aware | Always use UTC timestamps |
| UUIDs | Consider for distributed systems |

### N+1 Prevention

| Strategy | When to Use |
|----------|-------------|
| `joinedload` | One-to-one, few records |
| `selectinload` | One-to-many, many records |
| `contains_eager` | Existing JOIN in query |
| Batch loading | Load IDs first, then batch |

---

## Caching

### Tools

| Tool | Description |
|------|-------------|
| **Redis** | Distributed cache, sessions, pub/sub |
| **Memorystore** | Managed Redis in GCP |
| **cachetools** | Local in-memory cache (LRU, TTL) |

### Cache Levels

| Level | Tool | Latency | Use |
|-------|------|---------|-----|
| L1 - In-process | cachetools | ~1μs | Hot data, immutable |
| L2 - Distributed | Redis | ~1ms | Sessions, shared data |
| L3 - CDN | Cloud CDN | ~10-50ms | Static assets |

### Strategies

| Strategy | Description |
|----------|-------------|
| Cache-aside | Application explicitly manages cache |
| Write-through | Simultaneous write to cache and backend |
| TTL | Expires after N seconds |

### Key Patterns

| Pattern | Example |
|---------|---------|
| Namespace | `users:123:profile` |
| Version | `v2:users:123` |
| Hash | `query:sha256(params)` |

### Best Practices

| Practice | Description |
|----------|-------------|
| Appropriate TTL | Define per data type |
| Cache stampede prevention | Use locking or TTL jitter |
| Graceful degradation | App should work without cache |
| Monitoring | Hit rate, miss rate, latency |

---

## Background Tasks

### Tools by Case

| Tool | Use Case |
|------|----------|
| **Cloud Tasks** | Managed task queues in GCP |
| **Cloud Scheduler** | Managed cron in GCP |
| **Celery** | High scale, many workers |
| **arq** | Simple tasks, native async, Redis broker |

### Background Task Best Practices

| Practice | Description |
|----------|-------------|
| Idempotency | Tasks should be re-executable |
| Timeouts | Define time limits |
| Dead letter queue | Handle permanent failures |
| Retries with backoff | Exponential backoff |
| Observability | Metrics and logs |

---

## GCP Services Integration

### Recommended Services

| Service | Use Case |
|---------|----------|
| **Cloud SQL** | Managed PostgreSQL/MySQL |
| **Memorystore** | Managed Redis |
| **Secret Manager** | Secrets storage |
| **Cloud Tasks** | Task queues |
| **Cloud Scheduler** | Cron jobs |
| **Pub/Sub** | Messaging, events |
| **Cloud Storage** | Object storage |
| **Cloud CDN** | Static content delivery |

### GCP Authentication

| Tool | Description |
|------|-------------|
| **Workload Identity** | Service account for Cloud Run/GKE |
| **Application Default Credentials** | Auto-detect credentials |

### Scaling Considerations

| Need | GCP Solution | Avoid |
|------|--------------|-------|
| More requests | Cloud Run auto-scaling | Threads for requests |
| CPU-bound tasks | Cloud Run Jobs | multiprocessing in API |
| Massive data | Dataflow | ThreadPoolExecutor |
| I/O-bound | async/await | threading |
