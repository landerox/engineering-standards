# Python Development Standards

> **Version:** 1.0.0
> **Last Updated:** December 2025
> **Applies to:** Python 3.12+

---

## Table of Contents

1. [Philosophy and Principles](#1-philosophy-and-principles)
1. [Core Technology Stack](#2-core-technology-stack)
1. [Tools by Domain](#3-tools-by-domain)
1. [Project Structure](#4-project-structure)
1. [Dependency Management](#5-dependency-management)
1. [Code Quality](#6-code-quality)
1. [Type Checking](#7-type-checking)
1. [Testing](#8-testing)
1. [Security](#9-security)
1. [Documentation](#10-documentation)
1. [Git and Version Control](#11-git-and-version-control)
1. [CI/CD](#12-cicd)
1. [Docker and Containers](#13-docker-and-containers)
1. [Application Configuration](#14-application-configuration)
1. [Logging and Observability](#15-logging-and-observability)
1. [Patterns and Best Practices](#16-patterns-and-best-practices)
1. [Databases](#17-databases)
1. [APIs and Web Services](#18-apis-and-web-services)
1. [Caching](#19-caching)
1. [Background Tasks and Scheduling](#20-background-tasks-and-scheduling)
1. [Data Processing](#21-data-processing)
1. [Messaging and Events](#22-messaging-and-events)
1. [CLI and Scripts](#23-cli-and-scripts)
1. [Local Development](#24-local-development)
1. [Environment Management](#25-environment-management)
1. [Monorepo and Workspaces](#26-monorepo-and-workspaces)
1. [Use Cases by Project Type](#27-use-cases-by-project-type)
1. [New Project Checklist](#28-new-project-checklist)

---

## 1. Philosophy and Principles

### 1.1 Fundamental Principles

This document establishes the standards governing Python application development in the organization. The principles guiding these decisions are:

- **Simplicity over complexity**: Prefer straightforward, easy-to-understand solutions over unnecessary abstractions.
- **Explicit over implicit**: Code should clearly communicate its intent without requiring hidden knowledge.
- **Consistency**: All projects should follow the same patterns and conventions to facilitate collaboration.
- **Automation**: Repetitive processes should be automated to reduce human error and increase productivity.
- **Security by default**: Security considerations are not optional and must be integrated from the start.
- **Reproducibility**: Development, testing, and production environments must be deterministically reproducible.
- **Observability**: Applications should be observable by default (logs, metrics, traces).

### 1.2 Document Scope

This standard applies to all new Python projects and should be progressively adopted in existing projects. It covers from tool selection to deployment practices, providing a complete guide for the development lifecycle.

---

## 2. Core Technology Stack

### 2.1 Mandatory Tools (Cross-cutting)

The following tools constitute the main stack and their use is **mandatory** in all new projects.

#### 2.1.1 Runtime: Python 3.12+

**Description**: Minimum required Python interpreter version.

**Justification**: Python 3.12 introduces significant performance improvements, better support for generic types without additional imports, and more descriptive error messages. Previous versions are reaching or have reached end of life.

---

#### 2.1.2 Dependency and Environment Management: uv

**Description**: All-in-one tool for dependency management, virtual environments, and Python versions. Developed by Astral (Ruff creators), written in Rust.

**Justification**:

- 10-100x faster than pip, poetry, or pipenv
- Manages Python versions (replaces pyenv)
- Manages dependencies and lock files (replaces pip, poetry, pipenv, pip-tools)
- Manages virtual environments (replaces venv, virtualenv)
- Single tool for the entire lifecycle
- Deterministic lock file for total reproducibility

**Replaces**: pip, poetry, pipenv, pip-tools, pyenv, venv, virtualenv.

---

#### 2.1.3 Linting and Formatting: Ruff

**Description**: Extremely fast linter and code formatter, written in Rust. Unifies multiple tools into one.

**Justification**:

- 10-100x faster than traditional alternatives
- Replaces Flake8, Black, isort, pyupgrade, autoflake, and many Flake8 plugins
- Includes over 800 linting rules
- Unified configuration in pyproject.toml
- Auto-fix for most issues
- Supports security rules (equivalent to basic Bandit)

**Replaces**: Flake8, Black, isort, autopep8, yapf, pyupgrade, autoflake, pylint (partially).

---

#### 2.1.4 Type Checking: Pyright

**Description**: Static type checker for Python, developed by Microsoft. It's the engine behind Pylance in VS Code.

**Justification**:

- Faster than mypy on large projects
- Better type inference
- More complete strict mode
- Native VS Code integration through Pylance
- Better support for modern typing features

**Acceptable alternative**: mypy, when specific technical reasons exist.

---

#### 2.1.5 Testing: pytest

**Description**: Testing framework for Python with extensible plugin architecture.

**Justification**:

- Simple and expressive syntax
- Powerful fixture system for setup/teardown
- Parallelization with pytest-xdist
- Extensive plugin ecosystem
- Excellent async code support
- Integration with coverage.py

---

#### 2.1.6 Task Runner: just

**Description**: Modern command runner, similar to Make but with cleaner syntax and better user experience.

**Justification**:

- More readable syntax than Makefile
- Cross-platform without specific shell dependencies
- Support for arguments and variables
- No tabs vs spaces issues
- Automatic command documentation

**Replaces**: Makefile, scattered bash scripts.

---

#### 2.1.7 Git Hooks: pre-commit

**Description**: Framework for managing and running Git hooks consistently.

**Justification**:

- Ensures code meets standards before commit
- Configuration versioned in the repository
- Consistent execution across all developers
- Extensive catalog of available hooks
- Support for custom hooks

---

#### 2.1.8 Validation and Serialization: Pydantic v2

**Description**: Data validation library using Python type hints. Version 2 was rewritten with a Rust core for maximum performance.

**Justification**:

- Declarative type-based data validation
- Automatic serialization/deserialization
- Native FastAPI integration
- Excellent performance (Rust core)
- Descriptive error messages
- Support for complex types and custom validators

**Includes**:

- **pydantic**: Models and validation
- **pydantic-settings**: Configuration from environment variables

---

#### 2.1.9 Matrix Testing: nox

**Description**: Automation tool for running tests in multiple environments and Python versions.

**Justification**:

- Pure Python configuration (not INI like tox)
- Easy to understand and maintain
- Integration with uv for speed
- Supports custom sessions (lint, docs, tests)
- Ideal for libraries supporting multiple versions

**Use cases**:

- Public or internal libraries that must support Python 3.11, 3.12, 3.13
- Verify compatibility before releases
- CI/CD with version matrix

**Replaces**: tox (cleaner configuration).

---

### 2.2 Legacy Tools (Prohibited in New Projects)

The following tools **should not be used** in new projects. For existing projects, progressive migration is recommended.

| Legacy Tool | Replaced by | Reason for Change |
|-------------|-------------|-------------------|
| pip | uv | Speed, native lock files |
| poetry | uv | Speed, simplicity |
| pipenv | uv | Speed, better UX |
| pip-tools | uv | Integrated in uv |
| pyenv | uv | uv manages Python versions |
| Flake8 | ruff | Speed, unification |
| Black | ruff format | Speed, unification |
| isort | ruff | Integrated in ruff |
| autopep8 | ruff format | Speed, unification |
| yapf | ruff format | Speed, unification |
| pylint | ruff | Speed, most rules covered |
| mypy | pyright | Speed, better inference |
| Makefile | just | Better syntax, cross-platform |
| argparse | Typer | Better UX, type hints |
| tox | nox | Python configuration, more flexible |

---

## 3. Tools by Domain

The following tools are used according to project type or specific needs.

### 3.1 Security

| Tool | Description | Use Case |
|------|-------------|----------|
| **Bandit** | Static security analysis for Python code | Cross-cutting, CI/CD |
| **Semgrep** | Advanced static analysis with customizable rules | Projects with strict security requirements |
| **pip-audit** | Vulnerability scanning in dependencies | Cross-cutting, CI/CD |
| **Safety** | Alternative to pip-audit for vulnerability scanning | Alternative to pip-audit |
| **Trivy** | Vulnerability scanning in Docker images | Containerized projects |
| **gitleaks** | Secret detection in code | Cross-cutting, CI/CD |
| **detect-secrets** | Secret detection with baseline | Cross-cutting, pre-commit |

### 3.2 Testing

| Tool | Description | Use Case |
|------|-------------|----------|
| **pytest-cov** | Coverage integration with pytest | Cross-cutting |
| **pytest-asyncio** | Support for async tests | Projects with async code |
| **pytest-xdist** | Parallel test execution | Projects with many tests |
| **pytest-mock** | Mocking integration with pytest | Cross-cutting |
| **pytest-benchmark** | Code benchmarking | Performance optimization |
| **hypothesis** | Property-based testing | Complex business logic |
| **Faker** | Realistic fake data generation | Tests with diverse data |
| **factory-boy** | Factories for creating test objects | Tests with complex models |
| **testcontainers** | Docker containers for integration tests | Integration tests with DBs, caches |
| **nox** | Multi-version matrix testing | Public libraries |

### 3.3 Documentation

| Tool | Description | Use Case |
|------|-------------|----------|
| **MkDocs** | Documentation site generator | Cross-cutting |
| **mkdocs-material** | Modern theme for MkDocs | Cross-cutting |
| **mkdocstrings** | Automatic doc generation from docstrings | APIs, libraries |

### 3.4 APIs and Web Services

| Tool | Description | Use Case |
|------|-------------|----------|
| **FastAPI** | Modern high-performance web framework | REST APIs, microservices |
| **Pydantic** | Data validation and serialization | Cross-cutting |
| **httpx** | Async/sync HTTP client | External API consumption |
| **Granian** | High-performance ASGI server (Rust) | **Recommended** for Cloud Run |
| **Uvicorn** | Standard ASGI server | Fallback if incompatibilities exist |

**Granian vs Uvicorn**:

| Aspect | Granian | Uvicorn |
|--------|---------|---------|
| Performance | Superior (native Rust) | Very good (with uvloop) |
| Worker management | Integrated | Requires Gunicorn for multi-worker |
| Docker image | Simpler (no Gunicorn) | More complex (Gunicorn + Uvicorn) |
| Maturity | Newer | Very mature, widely used |
| Compatibility | May have edge cases | Maximum compatibility |

**Recommendation**:

- **New Cloud Run projects**: Use Granian by default
- **Existing projects**: Migrate to Granian gradually
- **Incompatibilities**: Fallback to Uvicorn

**Granian benefits in Cloud Run**:

- Eliminates the need for Gunicorn (fewer dependencies)
- Better performance without changing application code
- Integrated worker management
- Smaller Docker image size

### 3.5 Databases

| Tool | Description | Use Case |
|------|-------------|----------|
| **SQLAlchemy 2.0** | Mature ORM with async support | Default for complex projects |
| **SQLModel** | Combines SQLAlchemy + Pydantic, less boilerplate | Simple CRUDs with FastAPI |
| **Alembic** | Database migrations | Projects with SQLAlchemy |
| **asyncpg** | High-performance async PostgreSQL driver | PostgreSQL with async |
| **psycopg3** | Modern PostgreSQL driver (sync/async) | Alternative to asyncpg |

**When to use each ORM**:

| ORM | Use when |
|-----|----------|
| **SQLAlchemy 2.0** | Complex projects, advanced queries, maximum control |
| **SQLModel** | Simple CRUD APIs, less boilerplate, FastAPI creator |
| **Raw asyncpg** | Critical performance, highly optimized queries |

### 3.6 Caching

| Tool | Description | Use Case |
|------|-------------|----------|
| **redis-py** | Official Redis client | Distributed cache, sessions |
| **redis-om** | Object Mapping for Redis | Models in Redis |
| **cachetools** | In-memory caching utilities | Simple local cache |

### 3.7 Background Tasks and Scheduling

| Tool | Description | Use Case |
|------|-------------|----------|
| **Celery** | Distributed task queue system | Complex async tasks, high scale |
| **arq** | Simple async task queue based on Redis | Simple async tasks |
| **dramatiq** | Simpler Celery alternative | Async tasks with less overhead |
| **APScheduler** | Task scheduling | Cron jobs in application |

### 3.8 Data Processing

| Tool | Description | Use Case |
|------|-------------|----------|
| **Polars** | High-performance DataFrames | ETL, data analysis |
| **Pandera** | DataFrame validation | Data schema validation |
| **fastavro** | Efficient Avro serialization | Data pipelines, streaming |
| **Apache Beam** | SDK for data pipelines | Dataflow, streaming, batch |

### 3.9 Messaging and Events

| Tool | Description | Use Case |
|------|-------------|----------|
| **google-cloud-pubsub** | GCP Pub/Sub client | Messaging in GCP |
| **FastStream** | Framework for reactive microservices (FastAPI style) | Modern event consumers |
| **fastavro** | Avro serialization/deserialization | Schemas in Pub/Sub |

### 3.10 Observability

| Tool | Description | Use Case |
|------|-------------|----------|
| **structlog** | Structured logging | Cross-cutting |
| **OpenTelemetry** | Distributed tracing and metrics | Microservices, distributed systems |
| **prometheus-client** | Prometheus metrics exposition | Application metrics |
| **Sentry SDK** | Error tracking and performance monitoring | Exception capture, APM |
| **google-cloud-profiler** | Production profiling | Applications in GCP |
| **py-spy** | Sampling profiler without overhead | Performance debugging |

### 3.11 CLI

| Tool | Description | Use Case |
|------|-------------|----------|
| **Typer** | Framework for modern CLIs | Scripts, CLI tools |
| **Rich** | Enriched terminal output | CLIs with better UX |

### 3.12 Configuration

| Tool | Description | Use Case |
|------|-------------|----------|
| **pydantic-settings** | Typed configuration from env vars | Cross-cutting |
| **python-dotenv** | .env file loading | Local development |

### 3.13 Resilience

| Tool | Description | Use Case |
|------|-------------|----------|
| **tenacity** | Retries with backoff and circuit breaker | External service calls |

### 3.14 High-Performance Serialization

| Tool | Description | Use Case |
|------|-------------|----------|
| **orjson** | Ultra-fast JSON parsing (Rust) | High-traffic APIs, replaces json stdlib |
| **msgspec** | JSON/MessagePack serialization 2-5x faster than Pydantic | Critical performance |
| **ujson** | Fast JSON (C) | Alternative to orjson |

**Note**: For most cases, Pydantic v2 is fast enough. Use orjson/msgspec only when profiling indicates serialization is a bottleneck.

### 3.15 Feature Flags

| Tool | Description | Use Case |
|------|-------------|----------|
| **LaunchDarkly** | Enterprise feature flags platform | Large teams, complex releases |
| **Unleash** | Open source feature flags | Self-hosted, full control |
| **Simple env vars** | Boolean environment variables | Simple cases |

### 3.16 Internationalization

| Tool | Description | Use Case |
|------|-------------|----------|
| **Babel** | i18n/l10n for Python | Multi-language applications |
| **gettext** | Translation standard | Integrated with Babel |

### 3.17 Build and Distribution

| Tool | Description | Use Case |
|------|-------------|----------|
| **Hatch/Hatchling** | Modern build backend | Libraries, packages |
| **Artifact Registry** | Private package repository (GCP) | Internal distribution |
| **Keyring** | Transparent authentication for repositories | Access to private repos |

### 3.18 Authentication and Web Security

| Tool | Description | Use Case |
|------|-------------|----------|
| **PyJWT** | JSON Web Token encoding/decoding | Stateless authentication |
| **python-jose** | JWT with JWS, JWE, JWK support | Advanced JWT features |
| **passlib** | Password hashing (bcrypt, argon2) | Secure password storage |
| **argon2-cffi** | Argon2 implementation (PHC winner) | Modern password hashing |
| **authlib** | OAuth2 / OpenID Connect client and server | IdP integration |
| **fastapi-users** | Ready-made authentication for FastAPI | Quick auth in FastAPI |

**Recommended password hashing algorithms**:

| Algorithm | Recommendation |
|-----------|----------------|
| **Argon2id** | Preferred (Password Hashing Competition winner) |
| **bcrypt** | Acceptable, widely used |
| **scrypt** | Acceptable |
| **PBKDF2** | Legacy, avoid in new projects |
| MD5, SHA1, SHA256 | **Prohibited** for passwords |

**JWT Best Practices**:

| Practice | Description |
|----------|-------------|
| Secure algorithm | Use RS256 or ES256, avoid HS256 in production |
| Short expiration | Access tokens: 15-60 min |
| Refresh tokens | To renew access tokens |
| Validate claims | Always verify iss, aud, exp |
| Don't store sensitive data | JWT payload is visible (base64) |

### 3.19 Inter-Service Communication

| Tool | Description | Use Case |
|------|-------------|----------|
| **grpcio** | gRPC framework for Python | Efficient communication between microservices |
| **grpcio-tools** | Code generator from .proto | Compile Protocol Buffers |
| **grpc-interceptor** | Interceptors for logging, auth, etc. | Middleware in gRPC |
| **betterproto** | More Pythonic gRPC code generator | Alternative to grpcio-tools |

**When to use gRPC vs REST**:

| Criterion | gRPC | REST |
|-----------|------|------|
| Performance | High (binary, HTTP/2) | Medium (JSON, HTTP/1.1) |
| Streaming | Native bidirectional | Limited (SSE, WebSocket) |
| Contracts | Strict (.proto) | Flexible (OpenAPI optional) |
| Browser support | Limited (grpc-web) | Native |
| Debugging | More difficult (binary) | Easy (plain text) |
| Ecosystem | Smaller | Very extensive |

**Recommendation**: REST for public APIs and frontends, gRPC for internal communication between high-traffic microservices.

---

## 4. Project Structure

### 4.1 Directory Layout: src-layout

**Description**: All projects must use src-layout, where source code resides within a `src/` directory.

**Justification**:

- Avoids accidental imports of local code during development
- Forces package installation to run tests
- Clear separation between source code and other files
- Officially recommended by PyPA (Python Packaging Authority)

### 4.2 Mandatory Directories

| Directory | Purpose |
|-----------|---------|
| `src/<package_name>/` | Application source code |
| `tests/` | Automated tests |
| `.github/workflows/` | CI/CD pipelines |

### 4.3 Recommended Directories

| Directory | Purpose | When to use |
|-----------|---------|-------------|
| `.devcontainer/` | DevContainers configuration | Teams using VS Code |
| `docs/` | Project documentation | Projects with extensive documentation |
| `scripts/` | Development and operations scripts | When there are auxiliary scripts |
| `infra/` | Infrastructure as Code (Terraform, K8s) | Projects with their own infrastructure |

### 4.4 Mandatory Files

| File | Purpose |
|------|---------|
| `pyproject.toml` | Central project configuration (dependencies, tools) |
| `uv.lock` | Dependency lock file for reproducibility |
| `.python-version` | Project Python version |
| `.pre-commit-config.yaml` | Pre-commit hooks configuration |
| `.gitignore` | Files to ignore by Git |
| `README.md` | Basic project documentation |
| `src/<package>/py.typed` | PEP 561 marker to indicate type support |
| `Justfile` | Project command definitions |

### 4.5 Recommended Files

| File | Purpose | When to use |
|------|---------|-------------|
| `.editorconfig` | Format consistency across editors | Always recommended |
| `CHANGELOG.md` | Change history | Projects with releases |
| `CONTRIBUTING.md` | Guide for contributors | Collaborative projects |
| `LICENSE` | Project license | Open source or shared projects |
| `.github/CODEOWNERS` | Owners by code area | Large teams |
| `.github/dependabot.yml` | Automatic dependency updates | Always recommended |
| `Dockerfile` | Docker image definition | Containerized projects |
| `.dockerignore` | Files to exclude from build context | Containerized projects |
| `compose.yaml` | Local service orchestration | Projects with multiple services |
| `noxfile.py` | Matrix testing | Libraries supporting multiple versions |
| `mkdocs.yml` | Documentation configuration | Projects with MkDocs documentation |

### 4.6 Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Modules and packages | snake_case | `user_service.py`, `data_processing/` |
| Classes | PascalCase | `UserService`, `DataProcessor` |
| Functions and methods | snake_case | `get_user()`, `process_data()` |
| Variables | snake_case | `user_count`, `max_retries` |
| Constants | UPPER_SNAKE_CASE | `MAX_CONNECTIONS`, `DEFAULT_TIMEOUT` |
| Type aliases | PascalCase | `UserId`, `JsonDict` |
| Private variables | `_` prefix | `_internal_cache` |
| "Very private" variables | `__` prefix | `__secret_key` |

---

## 5. Dependency Management

### 5.1 Principles

- All dependencies must be specified in `pyproject.toml`
- The `uv.lock` file must be versioned in Git (never in `.gitignore`)
- Development dependencies must be separated from production ones
- Specify minimum versions, not exact versions

### 5.2 Version Specification

| Format | Use | Example |
|--------|-----|---------|
| `>=X.Y.Z` | Minimum version (preferred) | `fastapi>=0.115.0` |
| `>=X.Y.Z,<A.0.0` | Bounded range | `pydantic>=2.0.0,<3.0.0` |
| `>=X.Y.Z` with extras | Dependency with extras | `uvicorn[standard]>=0.32.0` |

**Avoid**:

- Exact versions (`==X.Y.Z`) - make updates difficult
- No version - generates non-reproducible builds
- Very restrictive ranges - limit compatibility

### 5.3 Dependency Separation

| Group | Contents | Installation |
|-------|----------|--------------|
| `dependencies` | Required in production | `uv sync` |
| `dev` | Development and testing | `uv sync --all-extras` |
| `docs` | Documentation generation | `uv sync --extra docs` |

### 5.4 Lock File (uv.lock)

**Purpose**: Guarantee exact reproducibility between environments.

**Rules**:

- Never edit manually
- Always commit to repository
- Regenerate with `uv lock` when dependencies change
- In CI/CD use `uv sync --frozen` to fail if outdated

---

## 6. Code Quality

### 6.1 Linting with Ruff

Ruff should be configured to enable the following rule categories:

| Category | Code | Description |
|----------|------|-------------|
| Pyflakes | F | Logic errors and unused variables |
| pycodestyle | E, W | PEP 8 style |
| isort | I | Import sorting |
| pep8-naming | N | Naming conventions |
| pydocstyle | D | Docstring style |
| pyupgrade | UP | Syntax modernization |
| flake8-bugbear | B | Common bugs and bad practices |
| flake8-simplify | SIM | Code simplifications |
| flake8-comprehensions | C4 | Comprehension improvements |
| flake8-datetimez | DTZ | Correct datetime with timezone usage |
| flake8-print | T20 | Detect forgotten prints |
| flake8-bandit | S | Basic security rules |
| flake8-builtins | A | Avoid shadowing builtins |
| flake8-annotations | ANN | Type annotations |
| flake8-async | ASYNC | Async best practices |
| flake8-pytest-style | PT | Pytest test style |
| flake8-return | RET | Return simplification |
| flake8-use-pathlib | PTH | Prefer pathlib over os.path |
| Pylint | PL | Additional Pylint rules |
| tryceratops | TRY | Exception best practices |
| Perflint | PERF | Performance optimizations |
| Ruff-specific | RUF | Ruff-specific rules |

### 6.2 Formatting with Ruff Format

**Formatting style**:

- Line length: 88 characters (Black compatible)
- Quotes: Double quotes
- Indentation: 4 spaces
- Trailing commas: Preserved

### 6.3 Configuration by File Type

| File | Relaxed rules |
|------|---------------|
| Tests (`tests/**/*.py`) | Allow assert, no mandatory docstrings, no mandatory annotations |
| Scripts (`scripts/**/*.py`) | Allow print |

---

## 7. Type Checking

### 7.1 Strictness Level

Pyright should be configured in **strict** mode for all new projects. This mode requires:

- Type annotations on all public functions
- No implicit `Any` usage
- Type verification in all expressions
- Report unused variables
- Verify compatibility in method overrides

### 7.2 PEP 561 Marker

All packages must include the `py.typed` file in their root directory to indicate type checking support. This allows other tools and package consumers to benefit from type information.

### 7.3 Typing Practices

| Practice | Description |
|----------|-------------|
| Use native generic types | `list[str]` instead of `List[str]` from typing |
| Union with pipe | `str \| None` instead of `Optional[str]` |
| Type aliases for readability | Define aliases for repeated complex types |
| Protocol for duck typing | Use Protocol instead of ABCs when appropriate |
| TypedDict for structured dicts | When a dict has known structure |
| Literal for specific values | When a variable can only have certain values |

---

## 8. Testing

### 8.1 Test Structure

| Directory | Contents | Characteristics |
|-----------|----------|-----------------|
| `tests/unit/` | Unit tests | Fast, no I/O, no external dependencies |
| `tests/integration/` | Integration tests | Can use databases, mocked external APIs |
| `tests/e2e/` | End-to-end tests | Test the complete system |

### 8.2 conftest.py File

The `tests/conftest.py` file should contain fixtures shared among all tests. Module-specific fixtures should be in their own local conftest.py.

### 8.3 Markers

All tests should be appropriately marked:

| Marker | Use |
|--------|-----|
| `@pytest.mark.unit` | Unit tests |
| `@pytest.mark.integration` | Integration tests |
| `@pytest.mark.e2e` | End-to-end tests |
| `@pytest.mark.slow` | Tests taking more than 1 second |

### 8.4 Coverage

| Metric | Minimum required |
|--------|------------------|
| Line coverage | 80% |
| Branch coverage | 80% |

Coverage should be measured only on the `src/` directory, excluding configuration files and `__init__.py`.

### 8.5 Property-Based Testing

For complex business logic, **Hypothesis** is recommended for property-based testing. This is especially useful for:

- Data transformation functions
- Input validation
- Serialization/deserialization
- Mathematical or processing algorithms

### 8.6 Test Data

| Tool | Use |
|------|-----|
| **Faker** | Generate realistic fake data (names, emails, addresses) |
| **factory-boy** | Create factories for complex models with relationships |
| **Fixtures** | Static data shared between tests |

### 8.7 Mocking

| Tool | Use |
|------|-----|
| **pytest-mock** | unittest.mock wrapper integrated with pytest |
| **unittest.mock** | Stdlib mocking (patch, MagicMock) |
| **responses/respx** | HTTP request mocking |

### 8.8 Benchmarking

For measuring critical code performance:

| Tool | Use |
|------|-----|
| **pytest-benchmark** | Benchmarks integrated with pytest |
| **py-spy** | Sampling profiler without modifying code |
| **cProfile** | Stdlib profiler |

---

## 9. Security

### 9.1 Static Security Analysis

**Bandit** should run on every PR and in CI/CD to detect common security issues such as:

- Use of dangerous functions (eval, exec)
- Potential SQL injection
- Hardcoded secrets
- Insecure use of cryptographic functions
- Insecure deserialization (pickle)

**Semgrep** can be used as a complement for custom rules and more advanced analysis.

### 9.2 Dependency Scanning

**pip-audit** or **Safety** should run regularly to identify known vulnerabilities in dependencies. Should be integrated in:

- CI/CD pipeline
- Local execution before releases

### 9.3 Secret Detection

**gitleaks** or **detect-secrets** should be configured to prevent commits of:

- API keys
- Passwords
- Access tokens
- Private keys
- Connection strings with credentials

### 9.4 Container Scanning

For containerized projects, **Trivy** should be used to scan Docker images for:

- Vulnerabilities in OS packages
- Vulnerabilities in application dependencies
- Misconfigurations

### 9.5 Automatic Updates

**Dependabot** or **Renovate** should be configured to:

- Update Python dependencies weekly
- Update GitHub Actions weekly
- Update Docker base images weekly
- Group development updates to reduce PRs

### 9.6 Secure Coding Practices

| Practice | Description |
|----------|-------------|
| Don't use eval/exec | Never execute dynamic code from external input |
| Don't use assert for validation | assert is disabled with python -O |
| Parameterized queries | Never concatenate strings for SQL |
| Don't use pickle with external data | Pickle allows arbitrary code execution |
| Validate paths | Prevent path traversal attacks |
| Don't hardcode secrets | Use environment variables or secret managers |
| Principle of least privilege | Containers as non-root user |

---

## 10. Documentation

### 10.1 Docstring Style: Google Style

All projects should use Google-style docstrings. This style is:

- Readable both in code and when rendered
- Well supported by mkdocstrings
- Balance between completeness and brevity

### 10.2 What to Document

| Element | Required | Content |
|---------|----------|---------|
| Modules | Yes | General module description |
| Public classes | Yes | Purpose, main attributes |
| Public functions/methods | Yes | Description, Args, Returns, Raises |
| Private functions/methods | Recommended | At least brief description |
| Constants | Recommended | Purpose if not obvious |

### 10.3 Project Documentation

| Document | Content |
|----------|---------|
| README.md | Description, quick installation, basic usage |
| CONTRIBUTING.md | Guide for contributors |
| CHANGELOG.md | Change history by version |
| docs/ (MkDocs) | Extensive documentation, guides, API reference |

### 10.4 Documentation Generation

For projects requiring extensive documentation, use **MkDocs** with:

- **mkdocs-material**: Modern and responsive theme
- **mkdocstrings**: Automatic generation from docstrings

---

## 11. Git and Version Control

### 11.1 Conventional Commits

All commits should follow the Conventional Commits format:

| Type | Use |
|------|-----|
| `feat` | New user-facing functionality |
| `fix` | Bug fix |
| `docs` | Documentation-only changes |
| `style` | Format changes without affecting logic |
| `refactor` | Refactoring without functionality change |
| `perf` | Performance improvements |
| `test` | Adding or fixing tests |
| `build` | Changes in build system or dependencies |
| `ci` | Changes in CI/CD configuration |
| `chore` | Maintenance tasks |
| `revert` | Revert previous commit |

**Breaking changes**: Indicate with `!` after the type or in footer with `BREAKING CHANGE:`.

### 11.2 Semantic Versioning (SemVer)

All projects should follow SemVer: `MAJOR.MINOR.PATCH`

| Component | When to increment |
|-----------|-------------------|
| MAJOR | Backward-incompatible changes |
| MINOR | New backward-compatible functionality |
| PATCH | Backward-compatible bug fixes |

**Pre-releases**: Use suffixes `-alpha.N`, `-beta.N`, `-rc.N`.

### 11.3 Branching Strategy

Follow **GitHub Flow**:

- `main` should always be in deployable state
- Features in short branches (`feature/`, `fix/`, `hotfix/`)
- Pull Requests required for merge to main
- Squash merge as preferred method

### 11.4 Pre-commit Hooks

The following hooks should run before each commit:

| Hook | Purpose |
|------|---------|
| ruff check | Linting |
| ruff format | Verify format |
| pyright | Type checking |
| conventional-pre-commit | Validate commit message format |
| trailing-whitespace | Remove trailing spaces |
| end-of-file-fixer | Ensure newline at end |
| check-yaml | Validate YAML syntax |
| check-toml | Validate TOML syntax |
| detect-secrets | Detect secrets |
| no-commit-to-branch | Prevent direct commits to main |

---

## 12. CI/CD

### 12.1 Required Pipelines

Every project should have the following GitHub Actions workflows:

| Workflow | Trigger | Jobs |
|----------|---------|------|
| CI | Push to main, PRs | lint, typecheck, test, security |
| Release | Tags `v*` | build, publish |
| Security | Schedule (weekly) | vulnerability scan |

### 12.2 Job: Lint

- Run `ruff check` with GitHub format for annotations
- Run `ruff format --check` to verify format
- Fail if there are errors

### 12.3 Job: Type Check

- Run `pyright` in strict mode
- Fail if there are type errors

### 12.4 Job: Test

- Run tests with pytest
- Generate coverage report
- Upload coverage to Codecov or similar
- Matrix testing with multiple Python versions if library

### 12.5 Job: Security

- Run `bandit` for static analysis
- Run `pip-audit` for dependency vulnerabilities
- Run `gitleaks` for secret detection
- For containers: run `trivy`

### 12.6 CI/CD Practices

| Practice | Description |
|----------|-------------|
| Use `uv sync --frozen` | Fail if lock file is outdated |
| Dependency caching | Cache uv directory between runs |
| Concurrency groups | Cancel previous runs of same PR |
| Dependabot | Keep Actions and dependencies updated |
| Branch protection | Require green CI for merge |

---

## 13. Docker and Containers

### 13.1 Base Image

Use `python:3.12-slim-bookworm` as base image:

- Based on stable Debian
- Reduced size (slim)
- Regular security updates

### 13.2 Multi-stage Builds

All Dockerfiles should use multi-stage builds:

- **Builder stage**: Install dependencies with uv
- **Runtime stage**: Only copy virtualenv and necessary code

Benefits:

- Smaller final image
- No build tools in production
- Better security

### 13.3 Non-Root User

Applications should run as non-root user:

- Create dedicated user (appuser)
- Change ownership of necessary files
- Use `USER appuser` before CMD

### 13.4 Python Environment Variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `PYTHONDONTWRITEBYTECODE` | 1 | Don't create .pyc files |
| `PYTHONUNBUFFERED` | 1 | Unbuffered output for logs |
| `PYTHONFAULTHANDLER` | 1 | Better crash debugging |

### 13.5 Health Checks

Every image should include HEALTHCHECK. In Kubernetes there are three types:

| Type | Purpose | Failure = |
|------|---------|-----------|
| **Liveness** | Is the application alive? | Pod restart |
| **Readiness** | Can it receive traffic? | Removed from load balancer |
| **Startup** | Has it finished starting? | Wait before liveness/readiness |

**Recommended endpoints**:

| Endpoint | Type | What it verifies |
|----------|------|------------------|
| `/health/live` | Liveness | App responds (simple ping) |
| `/health/ready` | Readiness | DB connected, dependencies OK |
| `/health/startup` | Startup | Complete initialization |

**Best practices**:

| Practice | Description |
|----------|-------------|
| Simple liveness | Don't verify external dependencies |
| Complete readiness | Verify DB, cache, critical services |
| Appropriate timeouts | initialDelaySeconds, periodSeconds |
| Don't cache | Each check should be fresh |

### 13.6 Docker Compose

For local development, use `compose.yaml` (v2 format):

- Define all necessary services (app, db, cache)
- Configure healthchecks and dependencies
- Use volumes for persistence
- Limit resources (CPU, memory)

### 13.7 .dockerignore File

**Mandatory** in every containerized project. Improves:

- Build speed (smaller context size)
- Security (avoids copying sensitive files)
- Image size (excludes unnecessary files)

**Should exclude**:

| Category | Files/Directories |
|----------|-------------------|
| Git | `.git/`, `.gitignore` |
| Python | `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `*.egg-info/` |
| Testing | `.pytest_cache/`, `.coverage`, `htmlcov/`, `.nox/`, `.tox/` |
| IDEs | `.idea/`, `.vscode/`, `*.swp` |
| Cache | `.ruff_cache/`, `.mypy_cache/` |
| Docs | `docs/`, `*.md` (except README.md if needed) |
| CI/CD | `.github/`, `.gitlab-ci.yml` |
| Development | `.env.local`, `.env.development`, `Makefile`, `Justfile` |
| Tests | `tests/` (if not run in container) |
| Secrets | `*.pem`, `*.key`, `secrets/` |

### 13.8 Graceful Shutdown

Applications should handle termination signals correctly:

| Signal | Action |
|--------|--------|
| SIGTERM | Start graceful shutdown |
| SIGINT | Same as SIGTERM (Ctrl+C) |
| SIGKILL | Forced termination (not handleable) |

**Practices**:

| Practice | Description |
|----------|-------------|
| Finish in-flight requests | Wait for active requests to complete |
| Close DB connections | Release pool connections |
| Flush logs/metrics | Send pending data |
| Shutdown timeout | Time limit for graceful (e.g., 30s) |
| Failing health check | Mark unhealthy before shutdown |

**Uvicorn** and **FastAPI** handle SIGTERM automatically, but custom tasks must implement explicit handling.

---

## 14. Application Configuration

### 14.1 Configuration Source

Configuration should be loaded exclusively from **environment variables**, using **pydantic-settings** for:

- Strong configuration typing
- Automatic validation
- Default values
- Implicit documentation

### 14.2 Principles

| Principle | Description |
|-----------|-------------|
| 12-Factor App | Configuration in environment, not in code |
| Fail fast | Validate configuration at startup |
| Type safety | All configuration should be typed |
| Sensible defaults | Default values for development |
| Separate secrets | Never in code or versioned files |

### 14.3 Organization

| Config type | Example | Where to define |
|-------------|---------|-----------------|
| Required without default | DATABASE_URL | Only in environment |
| With dev default | DEBUG=false | With default in code |
| Secrets | API_KEY | Secret manager or env |
| Feature flags | ENABLE_FEATURE_X | Environment variables |

### 14.4 .env File

- `.env.example` should be versioned with example values
- `.env` should never be versioned (in .gitignore)
- Use only for local development

---

## 15. Logging and Observability

### 15.1 Structured Logging

Use **structlog** for structured logging:

- In development: readable format with colors
- In production: JSON for indexing and searching
- Automatic context (request_id, user_id, etc.)

### 15.2 Log Levels

| Level | Use |
|-------|-----|
| DEBUG | Detailed information for debugging |
| INFO | Normal operation events |
| WARNING | Unexpected but handled situations |
| ERROR | Errors preventing operation completion |
| CRITICAL | Errors requiring immediate attention |

### 15.3 What to Log

| Event | Level | Information |
|-------|-------|-------------|
| Request received | INFO | method, path, request_id |
| Request completed | INFO | status_code, duration |
| Business operation | INFO | entity, action, identifiers |
| Recoverable error | WARNING | error, context |
| Unrecoverable error | ERROR | error, stack trace, context |
| External service connection | DEBUG | service, operation |

### 15.4 Format for Cloud Logging (GCP)

When deploying to GCP, logging should format:

- `severity`: Log level in GCP format (Cloud Logging requirement)
- `message`: Main message
- `logging.googleapis.com/trace`: Trace ID for correlation
- `logging.googleapis.com/spanId`: Span ID

### 15.5 Distributed Tracing

For distributed systems, implement **OpenTelemetry**:

- Automatic context propagation between services
- End-to-end request traces
- Integration with backends (Cloud Trace, Jaeger, etc.)

### 15.6 Correlation IDs (Request Tracing)

Every request should have a unique ID for log correlation:

| Header | Description |
|--------|-------------|
| `X-Request-ID` | Unique request ID |
| `X-Correlation-ID` | Common alternative |
| `traceparent` | W3C Trace Context standard |

**Behavior**:

| Scenario | Action |
|----------|--------|
| Header present | Use received ID |
| Header absent | Generate new UUID |
| Calls to other services | Propagate the ID |
| Logs | Include ID in each log entry |
| Response | Return ID in header |

**Benefits**:

- Trace request across multiple services
- Correlate logs from different components
- Debug production issues

### 15.7 Metrics

Expose application metrics using **prometheus-client**:

- Request latency (histogram)
- Requests per second (counter)
- Errors by type (counter)
- Specific business metrics

### 15.8 Error Tracking

Use **Sentry** for error capture and analysis:

- Automatic capture of unhandled exceptions
- User and request context
- Alerts and notifications
- Performance monitoring (APM)

---

## 16. Patterns and Best Practices

### 16.1 Error Handling

| Practice | Description |
|----------|-------------|
| Typed exceptions | Create domain exception hierarchy |
| Don't silence errors | Never use `except: pass` |
| Logging on errors | Always log with context before re-raise |
| Appropriate HTTP status | Map exceptions to correct HTTP codes |
| Useful messages | Include debugging information |

### 16.2 Recommended Exception Hierarchy

| Exception | Use | HTTP Status |
|-----------|-----|-------------|
| AppError | Base for all application exceptions | 500 |
| NotFoundError | Resource not found | 404 |
| ValidationError | Invalid input data | 400 |
| AuthenticationError | Not authenticated | 401 |
| AuthorizationError | Not authorized | 403 |
| ConflictError | State conflict | 409 |

### 16.3 Resilience

| Pattern | When to use | Tool |
|---------|-------------|------|
| Retry with backoff | External service calls | tenacity |
| Circuit breaker | Services that can fail | tenacity |
| Timeout | Every external call | asyncio.timeout, httpx timeout |
| Bulkhead | Isolate failures | Semaphores |

### 16.4 Standard Timeouts

Every external operation should have a timeout. Recommended values:

| Operation | Timeout | Notes |
|-----------|---------|-------|
| HTTP to internal services | 5-10s | Adjust according to SLA |
| HTTP to external services | 30s | Third-party APIs |
| Database queries | 30s | Normal queries |
| Complex database queries | 60-120s | Reports, aggregations |
| Cache (Redis) | 1-3s | Should be fast |
| Pub/Sub publish | 10s | Message publishing |
| File upload/download | 60-300s | According to expected size |
| Total background job | 1h-24h | According to job type |

**Best practices**:

| Practice | Description |
|----------|-------------|
| Always define timeout | Never leave operations without limit |
| Timeout < caller timeout | Prevent caller from cutting first |
| Logging on timeout | Record timeouts for analysis |
| Retry with backoff | Don't retry immediately |

### 16.5 Async Best Practices

| Practice | Description |
|----------|-------------|
| TaskGroup | Use asyncio.TaskGroup for structured concurrency |
| Semaphores | Limit concurrency to external resources |
| Timeouts | Always set timeout on async operations |
| Don't block | Never use blocking code in async context |
| run_in_executor | For unavoidable blocking code |

### 16.6 Dependency Injection

Use FastAPI's dependency system for:

- Decoupling components
- Facilitating testing with mocks
- Managing lifecycle (DB sessions, connections)
- Service composition

### 16.7 Memory Management

**General principles**:

| Principle | Description |
|-----------|-------------|
| Avoid loading everything in memory | Use generators, streaming, lazy loading |
| Release resources explicitly | Context managers, del, gc.collect() |
| Know container limits | Cloud Run has configurable memory limit |
| Monitor usage | Memory metrics in production |

**Patterns for memory efficiency**:

| Pattern | When to use | Example |
|---------|-------------|---------|
| **Generators** | Process items one by one | `yield` instead of `return list` |
| **Streaming** | Large files | `iter_content()`, `aiter_bytes()` |
| **Chunking** | Batch processing | Process in batches of N items |
| **Memory-mapped files** | Very large files | `mmap` module |
| **Slots** | Many class instances | `__slots__` in dataclasses |

**Streaming large data**:

| Scenario | Solution |
|----------|----------|
| Read large file | `open()` with line-by-line iteration |
| Large HTTP response | `httpx` with `stream=True`, `iter_bytes()` |
| Large DB query | Server-side cursor, `yield_per()` in SQLAlchemy |
| Export large CSV | `StreamingResponse` in FastAPI |
| Process DataFrame | Polars lazy, `scan_*` instead of `read_*` |

**Memory limits in Cloud Run**:

| Memory | Recommended use |
|--------|-----------------|
| 256MB | Simple APIs, little processing |
| 512MB | APIs with ORM, light processing |
| 1GB | Moderate processing, multiple connections |
| 2GB+ | Data processing, ML inference |

**Memory leak detection**:

| Tool | Use |
|------|-----|
| **tracemalloc** | Stdlib, memory snapshots |
| **memory_profiler** | `@profile` decorator |
| **objgraph** | Visualize object references |
| **py-spy** | Profiling without modifying code |

**Best practices**:

| Practice | Description |
|----------|-------------|
| Context managers | Guarantee resource release |
| Weak references | For caches that shouldn't prevent GC |
| `__slots__` | Reduce overhead in classes with many instances |
| Avoid mutable globals | Can accumulate data between requests |
| Connection pooling | Reuse connections, don't create new ones |
| Batch processing | Don't load millions of records at once |

### 16.8 Parallelism and Concurrency

**Note on threading and multiprocessing**:

For CPU-bound parallelism, **prefer horizontal scaling** (more Cloud Run instances) or use **managed services** (Dataflow, Vertex AI) over threading/multiprocessing within the application.

| Need | Recommended GCP solution | Avoid |
|------|--------------------------|-------|
| More simultaneous requests | Cloud Run auto-scaling | Threads for requests |
| CPU-bound tasks | Cloud Run Jobs with more CPU | multiprocessing in API |
| Massive data processing | Dataflow (Apache Beam) | ThreadPoolExecutor |
| I/O-bound (APIs, DB) | async/await | threading |
| ML inference | Vertex AI, GPUs | Local processes |

**Why avoid threading/multiprocessing in containers?**

| Reason | Description |
|--------|-------------|
| Python GIL | Threads don't parallelize CPU-bound code |
| Memory overhead | Each process duplicates container memory |
| Container limits | Cloud Run has strict resource limits |
| Cold starts | More processes = slower startup |
| Complexity | Race conditions, deadlocks, difficult debugging |

**Valid exceptions**:

- `ProcessPoolExecutor` for **very specific** CPU-bound tasks in Cloud Run Jobs with sufficient memory
- `ThreadPoolExecutor` for **wrapping blocking code** (sync libraries) in async context
- `concurrent.futures` for local development/migration scripts

---

## 17. Databases

### 17.1 ORM: SQLAlchemy 2.0

Use SQLAlchemy 2.0 with modern declarative style:

- Mapped columns with types
- Async sessions by default
- Typed query expressions

### 17.2 Repository Pattern

Encapsulate data access in repositories:

- Separation of business logic and persistence
- Facilitates testing with mocks
- Single access point for each entity

### 17.3 Migrations

Use **Alembic** for migrations:

- Auto-generated migrations as starting point
- Always review before applying
- Versioned in Git
- Idempotent when possible

### 17.4 Connection Pooling

Configure connection pool appropriately:

| Parameter | Consideration |
|-----------|---------------|
| pool_size | Number of base connections |
| max_overflow | Additional allowed connections |
| pool_pre_ping | Verify connections before use |
| pool_recycle | Recycle connections periodically |

### 17.5 Data Practices

| Practice | Description |
|----------|-------------|
| Soft deletes | Mark as deleted instead of physically deleting |
| Auditing | Fields created_at, updated_at, created_by, updated_by |
| Timezone-aware timestamps | Always use timezone-aware datetimes (UTC) |
| UUIDs vs auto-increment | Consider UUIDs for distributed systems |

### 17.6 Indexes and Optimization

| Practice | Description |
|----------|-------------|
| Indexes on FKs | Always index foreign keys |
| Composite indexes | Column order matters (most selective first) |
| Covering indexes | Include SELECT columns to avoid table lookup |
| EXPLAIN ANALYZE | Analyze slow queries |
| N+1 prevention | Use joinedload/selectinload in SQLAlchemy |

### 17.7 N+1 Query Prevention

| Strategy | When to use |
|----------|-------------|
| `joinedload` | One-to-one relationships, few records |
| `selectinload` | One-to-many relationships, many records |
| `contains_eager` | When there's already a JOIN in the query |
| Batch loading | Load IDs first, then batch query |

---

## 18. APIs and Web Services

### 18.1 Framework: FastAPI

FastAPI is the standard framework for REST APIs because:

- High performance (based on Starlette)
- Automatic validation with Pydantic
- Automatic OpenAPI documentation
- Native async support
- Dependency injection

### 18.2 API Structure

| Component | Responsibility |
|-----------|----------------|
| Routers | Endpoint definition, input validation |
| Schemas | Pydantic models for request/response |
| Services | Business logic |
| Repositories | Data access |
| Dependencies | Component injection |

### 18.3 API Versioning

- Include version in path: `/api/v1/...`
- Maintain backward compatibility within a version
- Deprecate before removing

### 18.4 Pydantic Schemas

| Practice | Description |
|----------|-------------|
| Separate Create/Update/Response | Different schemas for each operation |
| Validation in schema | Use Pydantic validators |
| from_attributes | Enable to map from ORM |
| Examples | Include examples for documentation |

### 18.5 HTTP Client: httpx

Use **httpx** to consume external APIs:

- Consistent sync/async API
- HTTP/2 support
- Configurable timeouts
- Easy retry with tenacity

### 18.7 Pagination

| Strategy | Use | Pros/Cons |
|----------|-----|-----------|
| **Offset-based** | `?page=2&limit=20` | Simple, but inefficient on large datasets |
| **Cursor-based** | `?cursor=abc123&limit=20` | Efficient, recommended for public APIs |
| **Keyset** | `?after_id=100&limit=20` | Similar to cursor, based on column values |

**Standard pagination response**:

| Field | Description |
|-------|-------------|
| `data` | Results array |
| `meta.total` | Total items (if efficient to calculate) |
| `meta.page` / `meta.cursor` | Current page or cursor |
| `meta.limit` | Items per page |
| `links.next` | Next page URL |
| `links.prev` | Previous page URL |

### 18.8 Error Format (RFC 7807)

Use **Problem Details** (RFC 7807) for error responses:

| Field | Type | Description |
|-------|------|-------------|
| `type` | URI | Error type identifier |
| `title` | string | Human-readable error title |
| `status` | integer | HTTP status code |
| `detail` | string | Specific description of this instance |
| `instance` | URI | URI of the problem instance |

**Additional allowed fields**:

| Field | Use |
|-------|-----|
| `errors` | Array of validation errors |
| `trace_id` | ID for log correlation |
| `code` | Internal error code |

### 18.9 Idempotency Keys

For operations that mutate state (POST, PUT), support idempotency:

| Header | Description |
|--------|-------------|
| `Idempotency-Key` | Unique UUID per client operation |

**Behavior**:

| Scenario | Response |
|----------|----------|
| First time | Process and save result with key |
| Repeated key (previous success) | Return saved result |
| Repeated key (previous failure) | Retry operation |
| Expired key | Reject with 409 Conflict |

**Critical use cases**:

- Payment creation
- Money transfers
- Unique resource creation

### 18.10 REST Endpoint Naming

| Resource | Convention | Example |
|----------|------------|---------|
| Collection | Plural, noun | `/users`, `/orders` |
| Item | Collection + ID | `/users/{id}` |
| Action | Verb (avoid if possible) | `/orders/{id}/cancel` |
| Relationship | Nested resource | `/users/{id}/orders` |
| Filters | Query params | `/users?status=active` |

**HTTP Verbs**:

| Verb | Use | Idempotent |
|------|-----|------------|
| GET | Read resource(s) | Yes |
| POST | Create resource | No* |
| PUT | Replace complete resource | Yes |
| PATCH | Partially update | Yes |
| DELETE | Delete resource | Yes |

*POST can be idempotent with Idempotency-Key

### 18.11 Rate Limiting

| Strategy | Description |
|----------|-------------|
| Token bucket | Allows controlled bursts |
| Sliding window | Uniform limit over time |
| Per user/IP | Identify client to limit |

**Response headers**:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Total limit |
| `X-RateLimit-Remaining` | Remaining requests |
| `X-RateLimit-Reset` | Reset timestamp |
| `Retry-After` | Seconds to retry (on 429) |

### 18.12 CORS (Cross-Origin Resource Sharing)

CORS configuration for APIs consumed by web frontends.

**Configuration by environment**:

| Environment | Configuration |
|-------------|---------------|
| **Development** | Allow localhost with various ports |
| **Staging** | Specific staging domains |
| **Production** | Only production domains, explicit list |

**CORS Headers**:

| Header | Description |
|--------|-------------|
| `Access-Control-Allow-Origin` | Allowed origins (never `*` in production with credentials) |
| `Access-Control-Allow-Methods` | Allowed HTTP methods |
| `Access-Control-Allow-Headers` | Allowed custom headers |
| `Access-Control-Allow-Credentials` | Whether cookies/auth headers are allowed |
| `Access-Control-Max-Age` | Preflight cache (seconds) |

**Best practices**:

| Practice | Description |
|----------|-------------|
| Explicit origin list | Never use `*` with credentials |
| Validate Origin header | Verify against whitelist |
| Limit methods | Only necessary ones |
| Specific headers | Don't allow all headers |
| Reasonable Max-Age | 86400 (24h) is common |

**Common errors**:

| Error | Problem |
|-------|---------|
| `*` with credentials | Browsers reject it |
| Forgetting preflight | OPTIONS must respond correctly |
| Origin vs Referer | CORS uses Origin, don't confuse |

### 18.13 HTTP Security Headers

Headers that every API/web application should include:

| Header | Recommended value | Purpose |
|--------|-------------------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Force HTTPS |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevent clickjacking |
| `X-XSS-Protection` | `1; mode=block` | Browser XSS filter |
| `Content-Security-Policy` | According to needs | Control loadable resources |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referrer info |

---

## 19. Caching

### 19.1 Cache Strategies

| Strategy | Description | When to use |
|----------|-------------|-------------|
| **Cache-aside** | Application explicitly manages cache | Default, maximum control |
| **Read-through** | Cache manages backend reading | When cache library supports it |
| **Write-through** | Simultaneous write to cache and backend | Strong consistency required |
| **Write-behind** | Asynchronous write to backend | High write throughput |
| **Refresh-ahead** | Pre-load before expiration | Critical data, low latency |

### 19.2 Tools

| Tool | Use Case |
|------|----------|
| **Redis** | Distributed cache, sessions, pub/sub |
| **Memorystore (GCP)** | Managed Redis in GCP |
| **cachetools** | Local in-memory cache (LRU, TTL, LFU) |
| **aiocache** | Async cache with multiple backends |

### 19.3 Cache Levels

| Level | Tool | Latency | Capacity | Use |
|-------|------|---------|----------|-----|
| **L1 - In-process** | cachetools, dict | ~1μs | MB | Hot data, immutable |
| **L2 - Distributed** | Redis | ~1ms | GB | Sessions, shared data |
| **L3 - CDN** | Cloud CDN | ~10-50ms | TB | Static assets, HTTP responses |

### 19.4 Cache Key Patterns

| Pattern | Example | Use |
|---------|---------|-----|
| Namespace | `users:123:profile` | Logical separation |
| Version | `v2:users:123` | Invalidation by version |
| Hash | `query:sha256(params)` | Complex queries |
| Timestamp | `users:123:1699900000` | Temporal cache |

### 19.5 Invalidation Strategies

| Strategy | Description | Trade-off |
|----------|-------------|-----------|
| **TTL (Time-to-Live)** | Expires after N seconds | Simple, eventual consistency |
| **Event-based** | Invalidate on write/update | Consistency, more complex |
| **Version-based** | New version = new key | Simple, uses more memory |
| **Tag-based** | Group keys by tags | Group invalidation |

**Recommended TTLs**:

| Data type | TTL | Reason |
|-----------|-----|--------|
| User sessions | 24h - 7d | Security/UX balance |
| Configuration | 5-15 min | Infrequent changes |
| Catalog data | 1-24h | According to change frequency |
| Search queries | 1-5 min | Data can change |
| User data | 1-5 min | Sensitive to changes |
| Rate limit counters | 1 min - 1h | According to limit window |

### 19.6 Best Practices

| Practice | Description |
|----------|-------------|
| **Appropriate TTL** | Define expiration time according to data |
| **Cache stampede prevention** | Use locking or TTL jitter |
| **Graceful degradation** | App should work without cache |
| **Efficient serialization** | msgpack > JSON for high volume |
| **Monitoring** | Hit rate, miss rate, latency |
| **Memory limits** | Configure maxmemory in Redis |
| **Eviction policy** | allkeys-lru for general cache |

### 19.7 Cache Stampede Prevention

When many requests ask for the same expired data simultaneously:

| Technique | Description |
|-----------|-------------|
| **Locking** | Only one request regenerates, others wait |
| **Probabilistic refresh** | Refresh before TTL with probability |
| **Stale-while-revalidate** | Serve stale while regenerating in background |
| **Jitter** | Add random variation to TTL |

### 19.8 Cache in Multi-layer Architecture

| Layer | What to cache | Where |
|-------|---------------|-------|
| **HTTP** | Complete responses | CDN, reverse proxy |
| **Application** | Business objects | Redis, in-memory |
| **ORM** | Query results | SQLAlchemy cache, Redis |
| **Database** | Query plans, buffers | PostgreSQL internals |

---

## 20. Background Tasks and Scheduling

### 20.1 Task Types

| Type | Example | Tool |
|------|---------|------|
| Fire-and-forget | Send email | Celery, arq |
| Scheduled | Daily report | APScheduler, Cloud Scheduler |
| Long-running | Video processing | Cloud Run Jobs |
| Workflow | Multi-step pipeline | Cloud Workflows, Composer |

### 20.2 Tools by Case

| Tool | Use Case |
|------|----------|
| **Celery** | High scale, many workers, complex tasks |
| **arq** | Simple tasks, native async, Redis as broker |
| **dramatiq** | Simpler Celery alternative |
| **APScheduler** | Scheduling within application |
| **Cloud Scheduler** | Managed cron in GCP |
| **Cloud Tasks** | Managed task queues |

### 20.3 Best Practices

| Practice | Description |
|----------|-------------|
| Idempotency | Tasks should be re-executable |
| Timeouts | Define time limits |
| Dead letter queue | Handle permanent failures |
| Retries with backoff | Retry with exponential increase |
| Observability | Execution metrics and logs |

---

## 21. Data Processing

### 21.1 DataFrames: Polars

Prefer **Polars** over Pandas for:

- Better performance (Rust, multi-threading)
- More consistent API
- Lazy evaluation for optimization
- Better memory management

### 21.2 Lazy Evaluation

Use `scan_*` instead of `read_*` to:

- Process large files without loading in memory
- Automatic query optimization
- Filter and projection pushdown

### 21.3 DataFrame Validation

Use **Pandera** to validate DataFrame schemas:

- Column type validation
- Constraints (ranges, allowed values)
- Null validation
- Integration with Polars and Pandas

### 21.4 File Handling

| Practice | Use | Avoid |
|----------|-----|-------|
| Paths | pathlib.Path | os.path |
| In-memory data | io.BytesIO, io.StringIO | Unnecessary temporary files |
| Temporary files | tempfile with context manager | Creating in /tmp manually |
| Efficient formats | Parquet, Avro | CSV for large data |

### 21.5 Serialization

| Format | Use Case |
|--------|----------|
| JSON | APIs, configuration |
| Parquet | Columnar storage, analytics |
| Avro | Streaming, evolving schemas |
| MessagePack | Fast binary serialization |

---

## 22. Messaging and Events

### 22.1 Pub/Sub

For asynchronous messaging use the Pub/Sub client with:

- Async processing when possible
- Appropriate acknowledgment
- Dead letter queues for failed messages
- Idempotency in processing

### 22.2 Message Serialization

| Format | Use |
|--------|-----|
| JSON | Simple messages, easy debugging |
| Avro | Evolving schemas, validation |
| Protocol Buffers | High performance, strict schemas |

### 22.3 Pub/Sub Schemas

Define strict contracts directly in the GCP topic:

- Use Avro or Protobuf schemas
- Automatic validation in Pub/Sub
- Controlled schema evolution
- Use **fastavro** for efficient serialization/deserialization

### 22.4 Messaging Patterns

| Pattern | Description |
|---------|-------------|
| At-least-once | Guarantee delivery, handle duplicates |
| Idempotency | Process same message multiple times without effects |
| Outbox | Guarantee consistency between DB and messages |
| Saga | Distributed transactions |

### 22.5 FastStream

Modern framework for reactive microservices:

- FastAPI style for event consumers
- Native async/await support
- Automatic Pydantic validation
- Automatic AsyncAPI documentation
- Replaces raw google-cloud-pubsub usage for business logic

---

## 23. CLI and Scripts

### 23.1 Framework: Typer

Use **Typer** for CLIs:

- Based on type hints
- Shell autocompletion
- Automatic help
- Argument validation

### 23.2 Principles

| Principle | Description |
|-----------|-------------|
| Fail fast | Validate arguments at start |
| Exit codes | 0 for success, >0 for errors |
| Stderr for errors | Stdout only for useful output |
| Verbose/quiet flags | Control output level |
| Idempotency | Scripts should be re-executable |

### 23.3 Entry Points

Define entry points in pyproject.toml for:

- Automatic command installation
- Available in PATH after `uv sync`
- Documentation of available commands

### 23.4 Enriched Output

Use **Rich** to improve CLI UX:

- Formatted tables
- Progress bars
- Syntax highlighting
- Markdown rendering

---

## 24. Local Development

### 24.1 DevContainers

For teams using VS Code, provide DevContainer configuration:

- Consistent environment across developers
- Pre-installed tools
- Recommended extensions
- Project settings

### 24.2 Justfile

Define common commands in Justfile:

| Command | Action |
|---------|--------|
| `just install` | Initial project setup |
| `just dev` | Run in development mode |
| `just lint` | Run linting |
| `just fix` | Auto-fix issues |
| `just typecheck` | Type verification |
| `just test` | Run tests |
| `just test-cov` | Tests with coverage |
| `just security` | Security scanning |
| `just check` | All checks |
| `just build` | Docker build |
| `just up` / `just down` | Docker Compose |
| `just docs` | Serve local documentation |

### 24.3 Recommended Workflow

1. Clone repository
1. Run `just install` (or `uv sync --all-extras && uv run pre-commit install`)
1. Create branch for feature/fix
1. Develop with `just dev`
1. Verify with `just check`
1. Commit (pre-commit hooks run automatically)
1. Push and create PR

---

## 25. Environment Management

### 25.1 Standard Environments

| Environment | Purpose | Characteristics |
|-------------|---------|-----------------|
| **development** | Local development | Debug enabled, verbose logs |
| **staging** | Pre-production | Production replica, test data |
| **production** | Production | Optimized, complete monitoring |

### 25.2 Code Promotion

| Practice | Description |
|----------|-------------|
| Same artifact | Same Docker image passes through all environments |
| Config by environment | Only environment variables change |
| Feature flags | Gradually enable features |
| Canary releases | Deploy to percentage of users |

### 25.3 Feature Flags

| Approach | Use Case |
|----------|----------|
| Environment variables | Simple on/off flags |
| LaunchDarkly | Complex targeting, integrated metrics |
| Unleash | Open source, self-hosted |
| Database flags | Dynamic flags without redeploy |

### 25.4 Secrets Management

| Tool | Use Case |
|------|----------|
| Secret Manager (GCP) | Secrets in GCP |
| External Secrets Operator | Synchronization to Kubernetes |
| Berglas | Encrypted secrets in GCS |

---

## 26. Monorepo and Workspaces

### 26.1 Monorepo vs Polyrepo

| Aspect | Monorepo | Polyrepo |
|--------|----------|----------|
| **Definition** | Multiple projects in one repository | One repository per project |
| **Refactoring** | Easy, atomic changes | Difficult, multiple PRs |
| **Shared dependencies** | Simple, one version | Complex, independent versioning |
| **CI/CD** | More complex, selective builds | Simple, one pipeline per repo |
| **Ownership** | Less clear (CODEOWNERS helps) | Clear per repo |
| **Repo size** | Can grow large | Small repos |

**When to use Monorepo**:

| Scenario | Recommendation |
|----------|----------------|
| Multiple highly related services | Monorepo |
| Shared libraries between services | Monorepo |
| Small/medium teams | Monorepo |
| Independent services | Polyrepo |
| Very large autonomous teams | Polyrepo |
| Open source with external contributors | Polyrepo |

### 26.2 uv Workspaces

**uv** supports workspaces for managing multiple packages in a monorepo.

**Monorepo structure with uv**:

| File/Directory | Purpose |
|----------------|---------|
| `pyproject.toml` (root) | Workspace configuration |
| `uv.lock` (root) | Unified lock file |
| `packages/` | Directory with packages |
| `packages/core/` | Shared package |
| `packages/api/` | API service |
| `packages/worker/` | Worker service |

**Workspace configuration** (root pyproject.toml):

| Field | Description |
|-------|-------------|
| `tool.uv.workspace.members` | List of packages in workspace |
| `tool.uv.sources` | Local dependencies between packages |

**uv workspaces benefits**:

| Benefit | Description |
|---------|-------------|
| Single lock file | All dependencies synchronized |
| Local dependencies | Packages reference without publishing |
| Selective installation | Install only necessary packages |
| Package commands | `uv run --package api pytest` |

### 26.3 Recommended Monorepo Structure

| Directory | Contents |
|-----------|----------|
| `packages/` | Python packages in workspace |
| `packages/shared/` | Shared code (models, utils) |
| `packages/api/` | API service |
| `packages/worker/` | Background workers |
| `packages/cli/` | CLI tools |
| `infra/` | Terraform, Kubernetes manifests |
| `scripts/` | Shared development scripts |
| `.github/workflows/` | CI/CD (with path filters) |

### 26.4 CI/CD in Monorepo

| Practice | Description |
|----------|-------------|
| Path filters | Only run jobs if relevant files change |
| Selective build | Detect which packages changed |
| Aggressive caching | Cache shared dependencies |
| Affected tests | Only run tests for modified packages |

**Change detection**:

| Tool | Description |
|------|-------------|
| `dorny/paths-filter` | GitHub Action to detect changes by path |
| `tj-actions/changed-files` | Alternative with more features |
| Custom scripts | Compare against main branch |

### 26.5 CODEOWNERS in Monorepo

| Pattern | Owner | Example |
|---------|-------|---------|
| `/packages/api/` | API Team | `@org/api-team` |
| `/packages/shared/` | Core team | `@org/core-team` |
| `/infra/` | Platform | `@org/platform-team` |
| `*.lock` | Core team | Dependency changes |

---

## 27. Use Cases by Project Type

### 27.1 Case 1: Python App in Docker for Cloud Run

**Focus**: Serverless, Stateless, Auto-scaling.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Web framework | FastAPI | Validation, automatic docs |
| ASGI Server | **Granian** (recommended) | High performance, integrated workers |
| ASGI Server (fallback) | Uvicorn | If Granian incompatibilities exist |
| Configuration | pydantic-settings | Typed loading from env vars |
| Logging | structlog | JSON format mandatory |
| Tracing | OpenTelemetry + Cloud Trace | Distributed traces |
| Errors | Cloud Error Reporting | Exception capture |

**Why Granian over Uvicorn?**

| Benefit | Description |
|---------|-------------|
| No Gunicorn | Integrated workers, simpler image |
| Better performance | Rust core |
| Fewer dependencies | Cleaner Dockerfile |
| Same code | No app changes required |

**GCP logging requirements**:

| Requirement | Description |
|-------------|-------------|
| JSON format | Mandatory for Cloud Logging |
| severity field | Map log level (INFO, ERROR, etc.) |
| Trace context | Include `logging.googleapis.com/trace` |
| Span ID | Include `spanId` for correlation |

**Considerations**:

- Stateless: don't maintain state in memory between requests
- Cold starts: optimize startup time
- Concurrency: configure max_instances and min_instances
- Timeouts: maximum 60 minutes for HTTP, configure appropriately
- Memory: choose according to load (256MB - 32GB available)

**Note on Cloud Functions**: For very simple GCS or Pub/Sub triggers, Cloud Functions Gen 2 is a valid alternative. However, it internally uses Cloud Run, so we recommend standardizing on Cloud Run to maintain consistency in tooling, Docker, and development practices.

---

### 27.2 Case 2: Python App in Docker for GKE (Kubernetes)

**Focus**: Orchestration, Distributed Observability, High Availability.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Authentication | Workload Identity | Link KSA with GSA |
| Development | Skaffold | Iterative development in cluster |
| Secrets | External Secrets Operator | Synchronization from Secret Manager |
| Observability | OpenTelemetry Collector | Sidecar or daemonset |
| Metrics | prometheus-client | Metrics exposition |

**Considerations**:

| Aspect | Practice |
|--------|----------|
| Health checks | Separate liveness and readiness probes |
| Resources | Define requests and limits |
| HPA | Auto-scaling based on metrics |
| PodDisruptionBudget | Guarantee availability during updates |
| Network policies | Restrict traffic between pods |

---

### 27.3 Case 3: Python Package (.whl / Library)

**Focus**: Private Distribution, Matrix Compatibility, Stable API.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Matrix testing | nox | Multiple Python versions |
| Build backend | Hatch / Hatchling | Modern build |
| Repository | Artifact Registry | Private packages in GCP |
| Authentication | Keyring | Transparent local authentication |
| Versioning | Strict SemVer | Clear compatibility |

**Considerations**:

| Aspect | Practice |
|--------|----------|
| Compatibility | Support N and N-1 Python versions |
| Deprecation | Warnings before removing features |
| Changelog | Keep CHANGELOG.md updated |
| Public API | Clearly document what is public |
| Type stubs | Include py.typed and complete types |

---

### 27.4 Case 4: Data Processing & Batch Jobs (Cloud Run Jobs)

**Focus**: ETL, Scheduled Tasks, Long-running Scripts.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Execution | Cloud Run Jobs | Up to 24h timeout, automatic retries |
| CLI | Typer | Framework for robust CLIs |
| Resilience | tenacity | In-code retries |
| Data | Polars | Efficient processing |
| Validation | Pandera | DataFrame validation |
| Orchestration | Cloud Composer / Workflows | For complex flows (A → B → C) |

**When to use Cloud Run Jobs vs Services**:

| Use Jobs | Use Services |
|----------|--------------|
| Tasks without HTTP response | REST APIs |
| Batch processing | Webhooks |
| Migrations | Health endpoints |
| Scheduled ETL | Continuous services |

**Typical invocation**:

- Docker command: `docker run my-img process-data --date=today`
- Typer enables expressive CLIs with automatic validation

---

### 27.5 Case 5: Data Streaming & Event-Driven (Pub/Sub, Dataflow)

**Focus**: Real-time Processing, Event-Driven Architectures (EDA).

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Complex pipelines | Apache Beam (Dataflow) | Windowing, watermarks, exactly-once |
| Lightweight consumers | FastStream | FastAPI style for events |
| Schemas | Pub/Sub Schemas | Avro or Protobuf contracts |
| Serialization | fastavro | Efficient for Avro |

**Apache Beam (Dataflow)**:

| Capability | Description |
|------------|-------------|
| Windowing | Aggregations by time windows |
| Watermarks | Late data handling |
| Exactly-once | Processing guarantees |
| Type hints | PCollection validation |

**FastStream**:

| Advantage | Description |
|-----------|-------------|
| Native async | Full async/await support |
| Validation | Automatic Pydantic |
| Documentation | Generated AsyncAPI |
| Simplicity | Replaces raw google-cloud-pubsub usage |

**Pub/Sub Schemas**:

| Practice | Description |
|----------|-------------|
| Define in topic | Avro or Protobuf schema in GCP |
| Automatic validation | Pub/Sub validates before accepting |
| Evolution | Schema compatibility rules |
| fastavro | Serialization/deserialization in Python |

---

### 27.6 Case 6: API with Database (Typical CRUD)

**Focus**: Persistence, Transactions, Migrations.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Framework | FastAPI | Validation, automatic docs |
| ORM | SQLAlchemy 2.0 async | Typed queries |
| Driver | asyncpg | High-performance PostgreSQL |
| Migrations | Alembic | Schema versioning |
| Cache | Redis | Sessions, query cache |

**Recommended patterns**:

| Pattern | Description |
|---------|-------------|
| Repository | Encapsulate data access |
| Unit of Work | Consistent transactions |
| Service Layer | Separated business logic |
| DTO/Schema | Separate API models from DB |

---

### 27.7 Case 7: Event-Driven Microservice

**Focus**: Decoupling, Eventual Consistency, Resilience.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Consumer | FastStream | Pub/Sub events |
| Producer | google-cloud-pubsub | Event publishing |
| Outbox | Transactional outbox | DB + events consistency |
| Idempotency | Deduplication key | In DB or cache |

**Considerations**:

| Aspect | Practice |
|--------|----------|
| Idempotency | Store processed message_ids |
| Order | Don't assume message order |
| Dead letters | Configure DLQ for failures |
| Monitoring | Lag metrics, errors |

---

### 27.8 Case 8: Microservices with gRPC

**Focus**: High-Performance Communication, Strict Contracts, Streaming.

**Specific stack**:

| Component | Tool | Notes |
|-----------|------|-------|
| Framework | grpcio | gRPC server and client |
| Code generation | grpcio-tools or betterproto | From .proto files |
| Interceptors | grpc-interceptor | Logging, auth, metrics |
| Validation | protobuf | Strict schemas |
| Load balancing | Client-side or Envoy | gRPC requires L7 LB |

**When to use gRPC**:

| Use gRPC | Use REST |
|----------|----------|
| High-frequency internal communication | Public APIs |
| Need for bidirectional streaming | Web frontends |
| Strict contracts required | Flexibility in evolution |
| Critical performance | Simple debugging |

**Considerations**:

| Aspect | Practice |
|--------|----------|
| Proto files | Centralized or per-service repository |
| Versioning | Package versioning in .proto |
| Health checks | grpc-health-checking protocol |
| Reflection | Enable for debugging (not in prod) |
| Deadlines | Always propagate deadlines |
| Interceptors | Auth, logging, tracing as interceptors |

---

### 27.9 Tools Summary by Use Case

| Tool | Cloud Run | GKE | Package | Batch | Streaming | API+DB | Event-Driven | gRPC |
|------|-----------|-----|---------|-------|-----------|--------|--------------|------|
| FastAPI | ✅ | ✅ | - | - | - | ✅ | ✅ | - |
| grpcio | - | ✅ | - | - | - | - | - | ✅ |
| Typer | - | - | - | ✅ | - | - | - | - |
| SQLAlchemy | ✅ | ✅ | - | ✅ | - | ✅ | ✅ | ✅ |
| Polars | - | - | - | ✅ | ✅ | - | - | - |
| Apache Beam | - | - | - | ✅ | ✅ | - | - | - |
| FastStream | - | ✅ | - | - | ✅ | - | ✅ | - |
| structlog | ✅ | ✅ | - | ✅ | ✅ | ✅ | ✅ | ✅ |
| OpenTelemetry | ✅ | ✅ | - | - | ✅ | ✅ | ✅ | ✅ |
| Redis | ✅ | ✅ | - | - | - | ✅ | ✅ | ✅ |
| nox | - | - | ✅ | - | - | - | - | - |
| tenacity | ✅ | ✅ | - | ✅ | ✅ | ✅ | ✅ | ✅ |
| PyJWT | ✅ | ✅ | - | - | - | ✅ | - | ✅ |

---

## 28. New Project Checklist

### 28.1 Initial Structure

- [ ] Create with `uv init` or template
- [ ] Configure src-layout
- [ ] Create `py.typed` in package
- [ ] Configure `.python-version`

### 28.2 Tool Configuration

- [ ] Configure ruff in pyproject.toml
- [ ] Configure pyright in pyproject.toml
- [ ] Configure pytest in pyproject.toml
- [ ] Configure coverage in pyproject.toml
- [ ] Configure bandit in pyproject.toml

### 28.3 Git and Collaboration

- [ ] Create appropriate `.gitignore`
- [ ] Configure `.pre-commit-config.yaml`
- [ ] Configure `.github/workflows/ci.yml`
- [ ] Configure `.github/dependabot.yml`
- [ ] Create `CODEOWNERS` if applicable

### 28.4 Documentation

- [ ] Create `README.md`
- [ ] Create `CONTRIBUTING.md` if collaborative
- [ ] Configure `mkdocs.yml` if docs needed

### 28.5 Development

- [ ] Create `Justfile` with standard commands
- [ ] Configure `.devcontainer/` if using VS Code
- [ ] Create `.editorconfig`
- [ ] Create `.env.example`

### 28.6 Production

- [ ] Create multi-stage `Dockerfile`
- [ ] Create `.dockerignore`
- [ ] Create `compose.yaml` for local development
- [ ] Configure pydantic-settings
- [ ] Configure structlog
- [ ] Configure Sentry (if applicable)
- [ ] Implement graceful shutdown
- [ ] Configure health checks (liveness/readiness)

### 28.7 By Project Type

**Cloud Run**:

- [ ] Configure logging for GCP
- [ ] Configure Cloud Trace
- [ ] Configure healthcheck endpoint

**GKE**:

- [ ] Configure Workload Identity
- [ ] Create Kubernetes manifests
- [ ] Configure External Secrets

**Package**:

- [ ] Configure noxfile.py
- [ ] Configure Artifact Registry
- [ ] Configure Hatchling

**Batch Jobs**:

- [ ] Create CLI with Typer
- [ ] Configure Cloud Run Job
- [ ] Configure Cloud Scheduler if cron

**Streaming**:

- [ ] Define Pub/Sub schemas
- [ ] Configure FastStream or Beam
- [ ] Configure dead letter topics

---
