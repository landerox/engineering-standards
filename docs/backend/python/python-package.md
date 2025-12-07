# Python Package Standards

> **Version:** 1.0.0
> **Last Updated:** December 2025
> **Extends:** [General Standards](general-standards.md)
> **Template:** [template-python-package](https://github.com/landerox/template-python-package)

Standards for building Python libraries, CLI tools, and distributable packages. Includes matrix testing, build configuration, and publishing to private registries.

---

## Use Cases

| Type | CLI Required | Examples |
|------|--------------|----------|
| Libraries | No | Shared utilities, SDKs, internal frameworks |
| CLI Tools | Yes | Command-line applications, developer tools |
| Library + CLI | Optional | Package usable as library AND as CLI |
| Plugins | No | Extensions for other packages |

> **Note:** Not all packages need a CLI. Libraries should work without CLI dependencies.

---

## Stack

### Build Backend

| Tool | Description |
|------|-------------|
| **Hatchling** | Modern, fast build backend (PEP 517/518 compliant) |
| **Hatch** | Project manager that uses Hatchling |

### Matrix Testing

| Tool | Description |
|------|-------------|
| **nox** | Test automation across multiple Python versions, pure Python config |

### CLI Framework

| Tool | Description |
|------|-------------|
| **Typer** | Modern CLI framework based on type hints |
| **Rich** | Beautiful terminal output, progress bars, tables |

### Distribution

| Tool | Description |
|------|-------------|
| **Artifact Registry** | Private Python repository in GCP |
| **Keyring** | Transparent authentication for private repos |

---

## Build Configuration

### pyproject.toml Structure

| Section | Purpose |
|---------|---------|
| `[project]` | Package metadata (name, version, dependencies) |
| `[build-system]` | Build backend (hatchling) |
| `[tool.hatch.build]` | Build configuration |
| `[project.scripts]` | CLI entry points |
| `[project.optional-dependencies]` | Extra dependencies (dev, docs) |

### Metadata

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Package name (lowercase, hyphens) |
| `version` | Yes | SemVer version |
| `description` | Yes | One-line description |
| `readme` | Yes | Path to README.md |
| `license` | Yes | License identifier |
| `requires-python` | Yes | Minimum Python version |
| `authors` | Yes | List of authors |
| `classifiers` | Recommended | PyPI classifiers |
| `keywords` | Recommended | Search keywords |

### Entry Points

| Type | Use | Example |
|------|-----|---------|
| `[project.scripts]` | CLI commands | `mycli = "mypackage.cli:app"` |
| `[project.gui-scripts]` | GUI applications | Rarely used |
| `[project.entry-points]` | Plugin systems | Framework extensions |

---

## Versioning

### Semantic Versioning (Strict)

| Component | When to Increment |
|-----------|-------------------|
| MAJOR | Breaking changes in public API |
| MINOR | New features, backward compatible |
| PATCH | Bug fixes, backward compatible |

### Version Sources

| Approach | Description |
|----------|-------------|
| Single source | `__version__` in `__init__.py` |
| Dynamic | Hatch reads from `__init__.py` or Git tags |
| Static | Version in `pyproject.toml` only |

> **Recommendation:** Use dynamic versioning from `src/package/__init__.py`

### Pre-release Versions

| Suffix | Use |
|--------|-----|
| `X.Y.Z-alpha.N` | Early development |
| `X.Y.Z-beta.N` | Feature complete, testing |
| `X.Y.Z-rc.N` | Release candidate |

---

## Public API

### What is Public

| Public | Private |
|--------|---------|
| Documented in API docs | Prefixed with `_` |
| Exported in `__all__` | Internal modules |
| Mentioned in README | Implementation details |

### `__all__` Best Practices

| Practice | Description |
|----------|-------------|
| Always define `__all__` | Explicit public API in every module |
| List only public names | Don't expose internals |
| Keep it sorted | Alphabetical for readability |
| Update with API changes | Keep in sync with actual exports |

### Module `__all__` Example

```python
# mypackage/core/models.py
__all__ = [
    "BaseModel",
    "Config",
    "ValidationError",
]

class BaseModel: ...
class Config: ...
class ValidationError(Exception): ...
class _InternalHelper: ...  # Not in __all__, private
```

### Package `__init__.py` Pattern

```python
# mypackage/__init__.py
"""MyPackage - A brief description."""

from mypackage.core.models import BaseModel, Config, ValidationError
from mypackage.core.processor import process

__all__ = [
    # Models
    "BaseModel",
    "Config",
    "ValidationError",
    # Functions
    "process",
]

__version__ = "1.0.0"
```

### Re-export Strategy

| Strategy | When to Use |
|----------|-------------|
| Flat re-export | Simple packages, few public items |
| Submodule access | Large packages, organized by domain |
| Mixed | Core items flat, specialized in submodules |

### Deprecation Process

| Step | Description |
|------|-------------|
| 1. Warn | Add `DeprecationWarning` in current MINOR |
| 2. Document | Note in CHANGELOG and docstring |
| 3. Remove | Remove in next MAJOR version |

### Deprecation Example

```python
import warnings

def old_function():
    """Deprecated: Use new_function instead."""
    warnings.warn(
        "old_function is deprecated, use new_function instead",
        DeprecationWarning,
        stacklevel=2,
    )
    return new_function()
```

---

## Type Hints

### Requirements

| Requirement | Description |
|-------------|-------------|
| `py.typed` | PEP 561 marker file in package root |
| Complete annotations | All public functions/methods typed |
| Strict Pyright | No implicit Any, full type coverage |

### Typing Patterns for Libraries

| Pattern | Use |
|---------|-----|
| `TypeVar` | Generic functions/classes |
| `Protocol` | Structural typing, duck typing |
| `overload` | Multiple signatures |
| `ParamSpec` | Decorator typing |

### Typing Exports

| Practice | Description |
|----------|-------------|
| Export types in `__init__.py` | Users can import types directly |
| Use `TYPE_CHECKING` | Avoid circular imports |
| Document type aliases | Explain complex types |

---

## Logging

### Golden Rule: Libraries Don't Configure Logging

Libraries must **never** configure logging handlers, formatters, or levels. Only obtain a logger and let the consuming application configure it.

### Correct Pattern

```python
# mypackage/core/processor.py
import logging

logger = logging.getLogger(__name__)

def process(data: str) -> str:
    logger.debug("Processing data: %s", data)
    # ... logic
    logger.info("Processing complete")
    return result
```

### What Libraries Should Do

| Do | Don't |
|----|-------|
| `logging.getLogger(__name__)` | Configure handlers |
| Use appropriate log levels | Set log level globally |
| Use lazy formatting `%s` | Configure formatters |
| Log useful debug info | Use print statements |
| Document what gets logged | Import structlog/loguru |

### Why This Matters

| Reason | Description |
|--------|-------------|
| Consumer controls | App decides format, level, destination |
| No conflicts | Multiple libraries don't fight over config |
| Flexibility | JSON in prod, pretty in dev - app's choice |
| Testing | Easy to capture/suppress logs in tests |

### Log Levels for Libraries

| Level | Use |
|-------|-----|
| DEBUG | Detailed internal operations |
| INFO | Significant events (use sparingly) |
| WARNING | Recoverable issues, deprecations |
| ERROR | Failures that don't crash |
| EXCEPTION | With traceback in except blocks |

### NullHandler Pattern

For libraries that want to avoid "No handler found" warnings:

```python
# mypackage/__init__.py
import logging
logging.getLogger(__name__).addHandler(logging.NullHandler())
```

---

## Package Architecture

### Core Separation Principle

The core library logic must be independent from CLI. This allows the package to be used as a library without installing CLI dependencies.

```bash
src/mypackage/
├── __init__.py          # Public API (re-exports from core)
├── core/                # Business logic (NO CLI dependencies)
│   ├── __init__.py
│   ├── models.py
│   └── processor.py
├── cli.py               # CLI layer (imports from core)
└── py.typed
```

### Why Separate Core from CLI

| Reason | Description |
|--------|-------------|
| Dependency isolation | Users who don't need CLI don't install Typer/Rich |
| Lighter installs | Smaller dependency footprint |
| Testability | Core testeable without CLI |
| Reusability | Same core usable as library AND CLI |
| Flexibility | Different CLIs can wrap the same core |

### Dependency Structure

| Layer | Dependencies | Installed by |
|-------|--------------|--------------|
| Core | Minimal (pydantic, etc.) | `pip install mypackage` |
| CLI | Typer, Rich | `pip install mypackage[cli]` |

### pyproject.toml Pattern

```toml
[project]
name = "mypackage"
dependencies = [
    "pydantic>=2.0",  # Core dependencies only
]

[project.optional-dependencies]
cli = [
    "typer>=0.9",
    "rich>=13.0",
]

[project.scripts]
# Entry point only works when [cli] extra is installed
mypackage = "mypackage.cli:app"
```

### Import Pattern

```python
# mypackage/__init__.py - Public API (core only)
from mypackage.core.models import MyModel
from mypackage.core.processor import process

__all__ = ["MyModel", "process"]

# mypackage/cli.py - CLI layer (imports core)
import typer
from mypackage.core.processor import process  # Import from core

app = typer.Typer()

@app.command()
def run():
    result = process()  # Use core functionality
    ...
```

### When to Include CLI

| Include CLI | Don't Include CLI |
|-------------|-------------------|
| Developer tools | Pure libraries/SDKs |
| Data processing utilities | Shared utilities |
| Build/automation tools | API clients |
| User-facing applications | Framework components |

---

## CLI Development

> **Note:** This section only applies if your package includes a CLI. CLI is optional for most libraries.

### Typer Structure

| Component | Purpose |
|-----------|---------|
| `app = typer.Typer()` | Main application |
| `@app.command()` | Subcommands |
| `@app.callback()` | Global options, help text |
| Type hints | Automatic argument parsing |

### CLI Best Practices

| Practice | Description |
|----------|-------------|
| Help text | `typer.Option(..., help="...")` for all options |
| Exit codes | Use `raise typer.Exit(code=1)` for errors |
| Progress bars | Use Rich for long operations |
| Confirmation | `typer.confirm()` for destructive actions |
| Stdin/stdout | Support piping when appropriate |

### Output

| Tool | Use |
|------|-----|
| **Rich** | Tables, progress bars, syntax highlighting |
| `rich.console.Console` | Styled output |
| `rich.table.Table` | Tabular data |
| `rich.progress` | Progress bars |

---

## Matrix Testing with nox

### Why nox

| Benefit | Description |
|---------|-------------|
| Pure Python | No INI files, Python configuration |
| uv integration | Fast environment creation |
| Multiple sessions | lint, typecheck, test, docs |
| Version matrix | Test Python 3.11, 3.12, 3.13 |

### Session Types

| Session | Purpose |
|---------|---------|
| `tests` | Run pytest across Python versions |
| `lint` | Run ruff check and format |
| `typecheck` | Run pyright |
| `docs` | Build documentation |
| `build` | Build wheel and sdist |

### Python Version Support

| Policy | Description |
|--------|-------------|
| N and N-1 | Support current and previous Python version |
| Example | Python 3.12 + 3.11 for 2024/2025 |
| EOL tracking | Drop versions at Python EOL |

---

## Testing

### Strategy

| Test Type | Purpose | Tool |
|-----------|---------|------|
| Unit | Test public API, edge cases | pytest |
| Property-based | Discover edge cases automatically | hypothesis |
| Doctest | Verify examples in docstrings | pytest --doctest-modules |
| Matrix | Test across Python versions | nox |

### Testing Tools

| Tool | Purpose |
|------|---------|
| **pytest** | Test framework |
| **pytest-cov** | Coverage reporting |
| **pytest-xdist** | Parallel test execution |
| **hypothesis** | Property-based testing |
| **nox** | Matrix testing across Python versions |

### What to Test

| Test | Description |
|------|-------------|
| Public API | All public functions/classes |
| Edge cases | Empty inputs, None, boundaries |
| Error handling | Exceptions raised correctly |
| Type coercion | Input types handled properly |
| Backward compatibility | No regressions |
| Docstring examples | All examples work |

### Test Structure

```bash
tests/
├── unit/
│   ├── test_models.py
│   └── test_processor.py
├── integration/          # If needed
├── conftest.py           # Shared fixtures
└── test_docstrings.py    # Doctest collection
```

### Coverage Requirements

| Metric | Minimum |
|--------|---------|
| Line coverage | 90% (higher than web services) |
| Branch coverage | 85% |

> **Note:** Libraries need higher coverage than applications because they're used by many consumers.

### Property-Based Testing with Hypothesis

Use Hypothesis for functions with complex input domains:

| Use Case | Description |
|----------|-------------|
| Data transformations | Verify invariants hold for any input |
| Serialization | `deserialize(serialize(x)) == x` |
| Validators | Test boundary conditions automatically |
| Parsers | Fuzz testing with generated inputs |

---

## Optional Dependencies (Extras)

### Common Extras Pattern

| Extra | Dependencies | Use Case |
|-------|--------------|----------|
| `cli` | typer, rich | Command-line interface |
| `dev` | pytest, ruff, pyright | Development tools |
| `docs` | mkdocs, mkdocstrings | Documentation |
| `all` | All optional deps | Everything |

### pyproject.toml Example

```toml
[project.optional-dependencies]
cli = [
    "typer>=0.9",
    "rich>=13.0",
]
dev = [
    "pytest>=8.0",
    "pytest-cov>=4.0",
    "hypothesis>=6.0",
    "ruff>=0.4",
    "pyright>=1.1",
]
docs = [
    "mkdocs>=1.5",
    "mkdocs-material>=9.0",
    "mkdocstrings[python]>=0.24",
]
all = [
    "mypackage[cli,docs]",
]
```

### Installation Commands

| Command | What Gets Installed |
|---------|---------------------|
| `pip install mypackage` | Core only |
| `pip install mypackage[cli]` | Core + CLI |
| `pip install mypackage[dev]` | Core + dev tools |
| `pip install mypackage[cli,docs]` | Core + CLI + docs |
| `uv sync --all-extras` | Everything (development) |

### Heavy Dependencies as Extras

If your library optionally integrates with heavy dependencies:

```toml
[project.optional-dependencies]
pandas = ["pandas>=2.0"]
polars = ["polars>=0.20"]
numpy = ["numpy>=1.24"]
```

This lets users install only what they need:

```bash
pip install mypackage[polars]  # Only if they need Polars integration
```

---

## Documentation

### Tools

| Tool | Purpose |
|------|---------|
| **MkDocs** | Documentation site |
| **mkdocs-material** | Material theme |
| **mkdocstrings** | Auto-generate API docs from docstrings |

### Documentation Structure

| Section | Content |
|---------|---------|
| Getting Started | Installation, quick start |
| User Guide | Tutorials, how-tos |
| API Reference | Auto-generated from docstrings |
| Changelog | Version history |
| Contributing | How to contribute |

### Docstring Requirements

| Element | Required |
|---------|----------|
| Description | What the function does |
| Args | All parameters with types |
| Returns | Return value description |
| Raises | Exceptions that can be raised |
| Examples | Usage examples (doctest format) |

---

## Building and Publishing

### Build Artifacts

| Artifact | Extension | Description |
|----------|-----------|-------------|
| **Wheel** | `.whl` | Binary distribution, fast install, no build step |
| **Source** | `.tar.gz` | Source distribution, requires build step |

> **Always publish both.** Wheel for fast installs, source for compatibility.

### What is a Wheel (.whl)

| Aspect | Description |
|--------|-------------|
| Format | ZIP archive with specific structure |
| Naming | `{name}-{version}-{python}-{abi}-{platform}.whl` |
| Pure Python | `py3-none-any.whl` (works everywhere) |
| Platform-specific | `cp312-cp312-linux_x86_64.whl` (compiled extensions) |

### Build Command

```bash
# Build both wheel and sdist
uv build

# Output in dist/
# mypackage-1.0.0-py3-none-any.whl
# mypackage-1.0.0.tar.gz
```

---

## Distribution Options

### Public vs Private

| Type | Use Case | Registry |
|------|----------|----------|
| **Public** | Open source, community packages | PyPI |
| **Private** | Internal tools, proprietary code | Artifact Registry, GitLab, etc. |

### Option 1: PyPI (Public)

| Aspect | Description |
|--------|-------------|
| URL | <https://pypi.org> |
| Cost | Free |
| Access | Public, anyone can install |
| Use case | Open source projects |

**Publishing to PyPI:**

```bash
# One-time: configure token
# Get token from https://pypi.org/manage/account/token/

# Publish
uv publish
# or
twine upload dist/*
```

### Option 2: TestPyPI (Testing)

| Aspect | Description |
|--------|-------------|
| URL | <https://test.pypi.org> |
| Use case | Test publishing before real PyPI |

```bash
uv publish --index-url https://test.pypi.org/simple/
```

### Option 3: Google Artifact Registry (Private - Recommended for GCP)

| Aspect | Description |
|--------|-------------|
| URL | `https://<region>-python.pkg.dev/<project>/<repo>` |
| Cost | Pay per storage/transfer |
| Access | IAM controlled |
| Use case | Internal packages in GCP |

**Setup:**

```bash
# 1. Create repository
gcloud artifacts repositories create python-packages \
    --repository-format=python \
    --location=us-central1 \
    --description="Internal Python packages"

# 2. Configure authentication
pip install keyrings.google-artifactregistry-auth
gcloud auth application-default login

# 3. Configure pip/uv to use it
```

**pyproject.toml configuration:**

```toml
# For publishing
[[tool.uv.index]]
name = "private"
url = "https://us-central1-python.pkg.dev/my-project/python-packages/simple/"
```

**Installing from Artifact Registry:**

```bash
pip install mypackage --index-url https://us-central1-python.pkg.dev/my-project/python-packages/simple/
```

### Option 4: GitLab Package Registry (Private)

| Aspect | Description |
|--------|-------------|
| URL | `https://gitlab.com/api/v4/projects/<id>/packages/pypi` |
| Cost | Included in GitLab |
| Access | Project/group based |
| Use case | Teams using GitLab |

### Option 5: GitHub Packages (Private)

| Aspect | Description |
|--------|-------------|
| URL | `https://npm.pkg.github.com` (limited Python support) |
| Note | Python support is limited, not recommended |

### Option 6: Self-hosted (Private)

| Tool | Description |
|------|-------------|
| **devpi** | Full-featured, caching proxy + private packages |
| **pypiserver** | Minimal, simple private PyPI |

### Comparison

| Registry | Public | Private | GCP Integration | Cost |
|----------|--------|---------|-----------------|------|
| PyPI | ✅ | ❌ | ❌ | Free |
| Artifact Registry | ❌ | ✅ | ✅ Native | Pay per use |
| GitLab Registry | ❌ | ✅ | ❌ | Included |
| devpi | ✅ | ✅ | ❌ | Self-host |

> **Recommendation:** Use **Artifact Registry** for private packages in GCP environments.

---

## Artifact Registry Setup (GCP)

### Repository Structure

| Strategy | Description |
|----------|-------------|
| Single repo | All packages in one repository |
| Per-team | Separate repos per team |
| Per-env | dev, staging, prod repositories |

### Authentication

| Method | Use Case |
|--------|----------|
| `gcloud auth` | Local development |
| Workload Identity | CI/CD, Cloud Run, GKE |
| Service Account Key | External CI (GitHub Actions) |

### GitHub Actions Publishing

```yaml
- name: Authenticate to Artifact Registry
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_SA_KEY }}

- name: Configure pip
  run: |
    pip install keyrings.google-artifactregistry-auth

- name: Publish
  run: uv publish --index-url https://...
```

### Consuming Private Packages

| Context | Configuration |
|---------|---------------|
| Local dev | `gcloud auth application-default login` |
| CI/CD | Service account + keyring |
| Docker | Multi-stage with auth in builder only |
| Cloud Run | Workload Identity |

### pip.conf / uv.toml for Teams

```toml
# uv.toml (project level)
[[index]]
name = "internal"
url = "https://us-central1-python.pkg.dev/my-project/python-packages/simple/"
default = true

[[index]]
name = "pypi"
url = "https://pypi.org/simple/"
```

---

## CI/CD

### Workflows

| Workflow | Trigger | Jobs |
|----------|---------|------|
| CI | Push, PRs | lint, typecheck, test (matrix) |
| Release | Tag `v*` | build, publish |

### Matrix Testing in CI

| Dimension | Values |
|-----------|--------|
| Python version | 3.11, 3.12, 3.13 |
| OS (optional) | ubuntu-latest, macos-latest, windows-latest |

### Release Process

| Step | Description |
|------|-------------|
| 1. Update version | Bump in `__init__.py` |
| 2. Update CHANGELOG | Add release notes |
| 3. Create tag | `git tag v1.2.3` |
| 4. Push tag | Triggers release workflow |
| 5. Build & publish | Automated by CI |

---

## Additional Files

Beyond [General Standards](general-standards.md), packages include:

| File | Purpose |
|------|---------|
| `noxfile.py` | Matrix testing configuration |
| `src/<package>/py.typed` | PEP 561 type marker |
| `src/<package>/__init__.py` | Package init with `__version__` and `__all__` |

---

## Best Practices

### API Design

| Practice | Description |
|----------|-------------|
| Minimal surface | Expose only what's necessary |
| Consistent naming | Follow Python conventions |
| Sensible defaults | Work out of the box |
| Explicit errors | Clear exception messages |
| No side effects on import | Don't run code at import time |

### Backward Compatibility

| Practice | Description |
|----------|-------------|
| Deprecation warnings | Warn before removing |
| Version bounds | Document supported versions |
| Migration guides | Help users upgrade |
| Changelog | Document all changes |

### Dependencies

| Practice | Description |
|----------|-------------|
| Minimal dependencies | Fewer deps = fewer conflicts |
| Loose version bounds | `>=1.0,<3` not `==1.2.3` |
| Optional dependencies | Heavy deps as extras |
| No upper bounds (usually) | Allow newer versions |

---

## Related Standards

| Standard | Description |
|----------|-------------|
| [General Standards](general-standards.md) | Base standards (required) |
| [Documentation Site](documentation-site.md) | For package documentation |
