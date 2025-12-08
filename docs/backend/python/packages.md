# :material-package-variant: Packages

> **Version:** 1.0.0
> **Extends:** [General / Core](general-core.md)
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
| [**Hatchling**](https://github.com/pypa/hatch) | Modern, fast build backend (PEP 517/518 compliant) |
| [**Hatch**](https://github.com/pypa/hatch) | Project manager that uses Hatchling |

### Matrix Testing

| Tool | Description |
|------|-------------|
| [**nox**](https://github.com/wntrblm/nox) | Test automation across multiple Python versions, pure Python config |

### CLI Framework

| Tool | Description |
|------|-------------|
| [**Typer**](https://github.com/fastapi/typer) | Modern CLI framework based on type hints |
| [**Rich**](https://github.com/Textualize/rich) | Beautiful terminal output, progress bars, tables |

### Distribution

| Tool | Description |
|------|-------------|
| [**Artifact Registry**](https://cloud.google.com/artifact-registry) | Private Python repository in GCP |
| [**Keyring**](https://github.com/jaraco/keyring) | Transparent authentication for private repos |

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
| Lighter installs | Users who only need library don't install Typer/Rich |
| Faster imports | No CLI framework loaded when using as library |
| Cleaner testing | Test core logic without CLI dependencies |
| Flexibility | CLI is just one interface to core functionality |

### Dependency Structure

```toml
[project]
dependencies = [
    # Core dependencies only - what the library needs
    "httpx>=0.27.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
cli = [
    "typer>=0.12.0",
    "rich>=13.0.0",
]
```

### Entry Point with Optional CLI

```toml
[project.scripts]
mycli = "mypackage.cli:app"
```

```python
# mypackage/cli.py
try:
    import typer
    from rich.console import Console
except ImportError:
    raise ImportError(
        "CLI dependencies not installed. "
        "Install with: pip install mypackage[cli]"
    )

from mypackage.core.processor import process

app = typer.Typer()
console = Console()

@app.command()
def run(input_file: str):
    """Process the input file."""
    result = process(input_file)
    console.print(result)
```

---

## Testing

### Test Structure

```bash
tests/
├── unit/
│   ├── test_models.py
│   └── test_processor.py
├── integration/
│   └── test_api.py
└── conftest.py
```

### Matrix Testing with nox

```python
# noxfile.py
import nox

@nox.session(python=["3.11", "3.12", "3.13"])
def tests(session):
    session.install(".[dev]")
    session.run("pytest", "tests/", "-v")

@nox.session(python="3.12")
def lint(session):
    session.install("ruff")
    session.run("ruff", "check", "src/")

@nox.session(python="3.12")
def typecheck(session):
    session.install(".[dev]")
    session.run("pyright", "src/")
```

### Running nox

```bash
# Run all sessions
nox

# Run specific session
nox -s tests

# Run specific Python version
nox -s tests-3.12

# List available sessions
nox -l
```

---

## Optional Dependencies Pattern

### When to Use Optional Dependencies

| Scenario | Make it Optional |
|----------|------------------|
| Heavy libraries (pandas, torch) | Yes |
| CLI frameworks (typer, click) | Yes |
| Alternative implementations | Yes |
| Dev/test tools | Yes (in dev group) |
| Core functionality | No |

### Example: Multiple Backends

```toml
[project.optional-dependencies]
pandas = ["pandas>=2.0.0"]
polars = ["polars>=0.20.0"]
all = ["pandas>=2.0.0", "polars>=0.20.0"]
```

```python
# mypackage/backends/__init__.py
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    import pandas as pd
    import polars as pl

# Cache imported modules to avoid repeated ImportError checking
# Note: Python's import system already caches modules after first import,
# but this pattern also caches the ImportError check itself, which is useful
# when functions are called thousands of times in tight loops.
_pandas_module = None
_polars_module = None


def _get_pandas():
    """Get pandas module with caching.

    Performance note:
    - Primarily caches the ImportError check/handling
    - Modest benefit (~1-5μs per call) but useful in high-frequency scenarios
    - Main value is cleaner separation of import logic from business logic
    """
    global _pandas_module
    if _pandas_module is None:
        try:
            import pandas as pd
            _pandas_module = pd
        except ImportError:
            raise ImportError("Install with: pip install mypackage[pandas]")
    return _pandas_module


def _get_polars():
    """Get polars module with caching.

    Performance note:
    - Primarily caches the ImportError check/handling
    """
    global _polars_module
    if _polars_module is None:
        try:
            import polars as pl
            _polars_module = pl
        except ImportError:
            raise ImportError("Install with: pip install mypackage[polars]")
    return _polars_module


def load_with_pandas(path: str) -> "pd.DataFrame":
    """Load CSV with pandas, using cached module import."""
    pd = _get_pandas()
    return pd.read_csv(path)


def load_with_polars(path: str) -> "pl.DataFrame":
    """Load CSV with polars, using cached module import."""
    pl = _get_polars()
    return pl.read_csv(path)
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
| [**MkDocs**](https://www.mkdocs.org/) | Documentation site |
| [**mkdocs-material**](https://squidfunk.github.io/mkdocs-material/) | Material theme |
| [**mkdocstrings**](https://github.com/mkdocstrings/mkdocstrings) | Auto-generate API docs from docstrings |

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
| [**devpi**](https://github.com/devpi/devpi) | Full-featured, caching proxy + private packages |
| [**pypiserver**](https://github.com/pypiserver/pypiserver) | Minimal, simple private PyPI |

### Comparison

| Registry | Public | Private | GCP Integration | Cost |
|----------|--------|---------|-----------------|------|
| [PyPI](https://pypi.org/) | ✅ | ❌ | ❌ | Free |
| [Artifact Registry](https://cloud.google.com/artifact-registry) | ❌ | ✅ | ✅ Native | Pay per use |
| [GitLab Registry](https://docs.gitlab.com/ee/user/packages/pypi_repository/) | ❌ | ✅ | ❌ | Included |
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

Beyond [General / Core](general-core.md), packages include:

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

### Performance Considerations

| Practice | Description | Impact |
|----------|-------------|--------|
| **Lazy imports** | Import heavy modules only when needed | Faster import time |
| **Cache expensive operations** | Use `@lru_cache` for repeated computations | Avoid redundant work |
| **Avoid I/O on import** | Don't read files at module level | Faster startup |
| **Profile before optimizing** | Use `cProfile` or `py-spy` to find bottlenecks | Target real issues |
| **Consider compiled extensions** | Use Cython/Rust for hot paths | 10-100x speedup |

- **Example: Lazy Import Pattern**

```python
# mypackage/utils.py
# Good: Heavy imports only when function is called
def process_with_pandas(data: str) -> dict:
    """Process data with pandas (lazy import)."""
    import pandas as pd  # Only imported if this function is used
    import numpy as np
    df = pd.DataFrame({"data": [data]})
    return df.to_dict()

# Bad: Heavy imports at module level
# import pandas as pd  # Imported even if never used
# import numpy as np
```

- **Example: Caching Expensive Operations**

```python
from functools import lru_cache
import hashlib

@lru_cache(maxsize=128)
def compute_expensive_hash(data: str) -> str:
    """Compute hash with caching.

    Performance: Repeated calls with same input return cached result.
    Useful for frequently called functions with repeated inputs.
    """
    # Simulate expensive operation
    for _ in range(1000):
        data = hashlib.sha256(data.encode()).hexdigest()
    return data

# First call: ~1ms
# Subsequent calls with same input: ~1μs (1000x faster)
result = compute_expensive_hash("same-input")
```
