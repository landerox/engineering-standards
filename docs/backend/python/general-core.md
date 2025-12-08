# :material-cog: General / Core

> **Version:** 1.0.0
> **Applies to:** Python 3.12+

Base standards that apply to **all** Python project types. Each project type (Web Service, Package, Data Job, Documentation Site) extends these standards with specific tools.

---

## Mandatory Base Stack

### Runtime

| Tool | Version | Description |
|------|---------|-------------|
| [**Python**](https://www.python.org/) | 3.12+ | Performance improvements, better generic type support, descriptive error messages |

### Dependency and Environment Management

| Tool | Description | Replaces |
|------|-------------|----------|
| [**uv**](https://github.com/astral-sh/uv) | All-in-one management: dependencies, virtual environments, Python versions. Written in Rust, 10-100x faster | pip, poetry, pipenv, pip-tools, pyenv, venv |

### Code Quality

| Tool | Description | Replaces |
|------|-------------|----------|
| [**Ruff**](https://github.com/astral-sh/ruff) | Extremely fast linter and formatter (Rust). 800+ rules, auto-fix, unified configuration in pyproject.toml | Flake8, Black, isort, autopep8, pylint, pyupgrade |
| [**Pyright**](https://github.com/microsoft/pyright) | Static type checker (Microsoft). Pylance engine in VS Code, better inference than mypy | mypy |

### Testing

| Tool | Description |
|------|-------------|
| [**pytest**](https://github.com/pytest-dev/pytest) | Testing framework with extensible plugin architecture |
| [**pytest-cov**](https://github.com/pytest-dev/pytest-cov) | Coverage integration with pytest |
| [**pytest-mock**](https://github.com/pytest-dev/pytest-mock) | unittest.mock wrapper integrated with pytest |

### Validation and Serialization

| Tool | Description |
|------|-------------|
| [**Pydantic v2**](https://github.com/pydantic/pydantic) | Data validation using type hints. Rust core for maximum performance |
| [**pydantic-settings**](https://github.com/pydantic/pydantic-settings) | Typed configuration from environment variables |

### Task Runner and Automation

| Tool | Description | Replaces |
|------|-------------|----------|
| [**just**](https://github.com/casey/just) | Modern command runner, clean syntax, cross-platform | Makefile |
| [**pre-commit**](https://github.com/pre-commit/pre-commit) | Framework for managing Git hooks consistently | Scattered bash scripts |

---

## Project Structure

### Layout: src-layout (Mandatory)

All projects must use src-layout where source code resides within a `src/` directory. This avoids accidental imports of local code during development and is officially recommended by PyPA.

### Mandatory Files

| File | Purpose |
|------|---------|
| `pyproject.toml` | Central configuration (dependencies, tools) |
| `uv.lock` | Lock file for exact reproducibility |
| `.python-version` | Project Python version |
| `.pre-commit-config.yaml` | Git hooks configuration |
| `.gitignore` | Files ignored by Git |
| `.gitattributes` | Line ending normalization, LFS, merge strategies |
| `src/<package>/py.typed` | PEP 561 marker for type support |
| `Justfile` | Standardized project commands |
| `README.md` | Basic project documentation |
| `LICENSE` | Project license |

### Configuration Files

| File | Purpose |
|------|---------|
| `.editorconfig` | Format consistency (indentation, charset, EOL) across editors |
| `.markdownlint.yaml` | Linting rules for Markdown files |
| `.secrets.baseline` | Baseline of known secrets (detect-secrets) |
| `cspell.json` | Dictionary and spell checking configuration |

### Community/Governance Files

| File | Purpose |
|------|---------|
| `LICENSE` | Project license (MIT, Apache 2.0, etc.) |
| `CODE_OF_CONDUCT.md` | Code of conduct for contributors |
| `CONTRIBUTING.md` | Project contribution guide |
| `CHANGELOG.md` | Change history by version ([Keep a Changelog](https://keepachangelog.com/) + [SemVer](https://semver.org/)) |
| `SECURITY.md` | Security policy and vulnerability reporting process |

### GitHub Files

| File | Purpose |
|------|---------|
| `.github/CODEOWNERS` | Defines code owners by area for automatic review |
| `.github/pull_request_template.md` | PR template |
| `.github/workflows/` | GitHub Actions workflows |
| `.github/dependabot.yml` | Automatic update configuration |

### Directories

| Directory | Purpose |
|-----------|---------|
| `.devcontainer/` | DevContainers configuration (VS Code) |
| `.github/workflows/` | CI/CD pipelines |
| `docs/` | Project documentation (MkDocs, guides) |
| `src/<package_name>/` | Application source code |
| `tests/` | Automated tests (unit/, integration/, e2e/) |

---

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Modules/Packages | snake_case | `user_service.py` |
| Classes | PascalCase | `UserService` |
| Functions/Methods | snake_case | `get_user()` |
| Variables | snake_case | `user_count` |
| Constants | UPPER_SNAKE_CASE | `MAX_CONNECTIONS` |
| Type aliases | PascalCase | `UserId` |
| Private | `_` prefix | `_internal_cache` |
| "Very private" | `__` prefix | `__secret_key` |

---

## Python Code Standards

### Type Hints (PEP 484, PEP 604)

All functions must have type annotations:

| Practice | Standard | Example |
|----------|----------|---------|
| Native generics | Use built-in types | `list[str]` not `List[str]` |
| Union syntax | Use pipe operator | `str \| None` not `Optional[str]` |
| Return type | Always specify | `def get_user() -> User:` |
| Complex types | Use TypeAlias | `UserId: TypeAlias = int` |

### Function Definitions

| Practice | Description |
|----------|-------------|
| Type all parameters | Every parameter has a type annotation |
| Type return values | Every function has a return type |
| Default values | Type before default: `name: str = "default"` |
| `*args` / `**kwargs` | Type the contents: `*args: str`, `**kwargs: int` |

### Decorators

| Practice | Description |
|----------|-------------|
| Order matters | Outermost decorator executes first |
| Preserve metadata | Use `@functools.wraps` in custom decorators |
| Type decorators | Use `ParamSpec` and `TypeVar` for typed decorators |

### Module Organization

| Section | Order | Description |
|---------|-------|-------------|
| 1. Module docstring | Top | Module description |
| 2. `__future__` imports | First imports | Future annotations if needed |
| 3. Standard library | Second | `import os`, `import sys` |
| 4. Third-party | Third | `import fastapi`, `import pydantic` |
| 5. Local imports | Fourth | `from .models import User` |
| 6. Constants | After imports | `MAX_RETRIES = 3` |
| 7. Type aliases | After constants | `UserId: TypeAlias = int` |
| 8. Classes/Functions | Main content | Implementation |
| 9. `if __name__` | Bottom | Entry point guard |

### Code Patterns

| Pattern | Use | Avoid |
|---------|-----|-------|
| `pathlib.Path` | File system operations | `os.path` |
| `datetime.UTC` | Timezone-aware datetimes | Naive datetimes |
| Context managers | Resource management | Manual open/close |
| Dataclasses/Pydantic | Data structures | Plain dicts for structured data |
| Enums | Fixed set of values | String constants |

### Typing Patterns

| Pattern | When to Use |
|---------|-------------|
| `Protocol` | Duck typing, structural subtyping |
| `TypedDict` | Dicts with known structure |
| `Literal` | Variable with specific allowed values |
| `Final` | Constants that shouldn't be reassigned |
| `ClassVar` | Class-level variables |

---

## Dependency Management

### Version Specification

| Format | Use | Example |
|--------|-----|---------|
| `>=X.Y.Z` | Minimum version (preferred) | `fastapi>=0.115.0` |
| `>=X.Y.Z,<A.0.0` | Bounded range | `pydantic>=2.0.0,<3.0.0` |

**Avoid:** Exact versions (`==`), no version, overly restrictive ranges.

### Dependency Separation

| Group | Contents | Installation |
|-------|----------|--------------|
| `dependencies` | Production | `uv sync` |
| `dev` | Development and testing | `uv sync --all-extras` |
| `docs` | Documentation | `uv sync --extra docs` |

---

## Linting and Static Analysis

### Ruff - Enabled Rule Categories

| Category | Code | Description |
|----------|------|-------------|
| Pyflakes | F | Logic errors, unused variables |
| pycodestyle | E, W | PEP 8 style |
| isort | I | Import sorting |
| pep8-naming | N | Naming conventions |
| pyupgrade | UP | Syntax modernization |
| flake8-bugbear | B | Common bugs |
| flake8-simplify | SIM | Simplifications |
| flake8-bandit | S | Basic security rules |
| Ruff-specific | RUF | Ruff-specific rules |

### Pyright - Strict Mode

- Type annotations on all public functions
- No implicit `Any` usage
- Type verification in all expressions

---

## Testing Strategy

### Test Structure

| Directory | Contents | Characteristics |
|-----------|----------|-----------------|
| `tests/unit/` | Unit tests | Fast, no I/O, no external dependencies |
| `tests/integration/` | Integration tests | Can use DBs, mocked APIs |
| `tests/e2e/` | End-to-end tests | Test the complete system |

### Markers

| Marker | Use |
|--------|-----|
| `@pytest.mark.unit` | Unit tests |
| `@pytest.mark.integration` | Integration tests |
| `@pytest.mark.slow` | Tests > 1 second |

### Minimum Coverage

| Metric | Minimum |
|--------|---------|
| Line coverage | 80% |
| Branch coverage | 80% |

---

## Security

### Tools

| Tool | Description | When |
|------|-------------|------|
| [**Bandit**](https://github.com/PyCQA/bandit) | Static security analysis | CI/CD, pre-commit |
| [**pip-audit**](https://github.com/pypa/pip-audit) | Dependency vulnerability scanning | CI/CD |
| [**detect-secrets**](https://github.com/Yelp/detect-secrets) | Secret detection with baseline | pre-commit |
| [**gitleaks**](https://github.com/gitleaks/gitleaks) | Secret detection in code | CI/CD |

### Practices

| Practice | Description |
|----------|-------------|
| Don't use eval/exec | Never execute dynamic code from external input |
| Parameterized queries | Never concatenate strings for SQL |
| Don't use pickle with external data | Pickle allows arbitrary code execution |
| Don't hardcode secrets | Use environment variables or secret managers |

---

## Git and Version Control

### Conventional Commits

| Type | Use |
|------|-----|
| `feat` | New functionality |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Refactoring without functional change |
| `test` | Add or fix tests |
| `chore` | Maintenance tasks |

### Semantic Versioning (SemVer)

All projects follow [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`

| Component | When to Increment |
|-----------|-------------------|
| MAJOR | Backward-incompatible changes (breaking changes) |
| MINOR | New backward-compatible functionality |
| PATCH | Backward-compatible bug fixes |

**Pre-releases:** Use suffixes `-alpha.N`, `-beta.N`, `-rc.N`

### Changelog Standard

Follow [Keep a Changelog](https://keepachangelog.com/) format:

| Section | Description |
|---------|-------------|
| `Added` | New features |
| `Changed` | Changes in existing functionality |
| `Deprecated` | Soon-to-be removed features |
| `Removed` | Removed features |
| `Fixed` | Bug fixes |
| `Security` | Vulnerability fixes |

### Pre-commit Hooks

| Hook | Purpose |
|------|---------|
| ruff check | Linting |
| ruff format | Verify format |
| pyright | Type checking |
| [conventional-pre-commit](https://github.com/compilerla/conventional-pre-commit) | Validate commit message format |
| detect-secrets | Detect secrets |
| no-commit-to-branch | Prevent direct commits to main |

### Branching: GitHub Flow

- `main` always in deployable state
- Features in short branches (`feature/`, `fix/`)
- Pull Requests required for merge to main
- Squash merge as preferred method

---

## Base CI/CD

### Required Workflows

| Workflow | Trigger | Jobs |
|----------|---------|------|
| CI | Push to main, PRs | lint, typecheck, test, security |
| Security | Schedule (weekly) | vulnerability scan |

### Jobs

| Job | Tools |
|-----|-------|
| Lint | `ruff check`, `ruff format --check` |
| Type Check | `pyright` (strict mode) |
| Test | `pytest` with coverage |
| Security | `bandit`, `pip-audit`, `gitleaks` |

### CI/CD Practices

| Practice | Description |
|----------|-------------|
| `uv sync --frozen` | Fail if lock file is outdated |
| Dependency caching | Cache uv directory between runs |
| Branch protection | Require green CI for merge |

---

## Documentation

### Docstring Style: Google Style

### What to Document

| Element | Required |
|---------|----------|
| Modules | Yes |
| Public classes | Yes |
| Public functions/methods | Yes |
| Private functions | Recommended |

### Documentation Linting Tools

| Tool | Purpose | Configuration File |
|------|---------|-------------------|
| [**markdownlint**](https://github.com/DavidAnson/markdownlint) | Markdown file linting | `.markdownlint.yaml` |
| [**cspell**](https://github.com/streetsidesoftware/cspell) | Spell checking in code and docs | `cspell.json` |

---

## Prohibited Tools in New Projects

| Legacy Tool | Replaced by |
|-------------|-------------|
| pip, poetry, pipenv | [uv](https://github.com/astral-sh/uv) |
| Flake8, Black, isort | [Ruff](https://github.com/astral-sh/ruff) |
| mypy | [Pyright](https://github.com/microsoft/pyright) |
| Makefile | [just](https://github.com/casey/just) |
| argparse | [Typer](https://github.com/fastapi/typer) |
| tox | [nox](https://github.com/wntrblm/nox) |

---

## Local Development

### DevContainers

| File | Purpose |
|------|---------|
| `.devcontainer/devcontainer.json` | Containerized development environment configuration |

**Benefits:**

- Consistent environment for the entire team
- Instant onboarding (just open in VS Code)
- Pre-installed extensions and configurations
- Host system isolation
