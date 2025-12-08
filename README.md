# Engineering Standards

> **The single source of truth for development standards, architectural patterns, and engineering culture.**

## About This Project

This repository hosts the engineering framework used to ensure consistency, scalability, and maintainability across software projects. It serves as a living knowledge base connecting abstract principles with concrete implementation templates.

## Core Philosophy

- **Simplicity:** Straightforward solutions over unnecessary complexity.
- **Consistency:** Predictable patterns across all services and jobs.
- **Automation:** If it can be scripted, it is automated (CI/CD, Linting).
- **Reproducibility:** Deterministic environments using modern tooling.

## Live Site

- [engineering-standards](https://landerox.github.io/engineering-standards)

## Quick Start

### Option 1: Dev Container (recommended)

Requires [Docker](https://www.docker.com/) and [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

```bash
git clone https://github.com/landerox/engineering-standards.git
code engineering-standards
```

When VS Code opens, click **"Reopen in Container"** when prompted. The container will automatically install all dependencies and configure pre-commit hooks.

### Option 2: Local Setup

Requires [uv](https://docs.astral.sh/uv/getting-started/installation/) package manager.

```bash
git clone https://github.com/landerox/engineering-standards.git
cd engineering-standards
uv sync --all-groups
uv run pre-commit install
```

### Run Documentation Locally

```bash
uv run mkdocs serve
```

Then open [http://localhost:8000](http://localhost:8000) in your browser.

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) and our [Code of Conduct](CODE_OF_CONDUCT.md) before participating.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and releases.

## Security

To report vulnerabilities, see [SECURITY.md](SECURITY.md).

## License

MIT License. See [LICENSE](LICENSE).
