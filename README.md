# Engineering Standards

Personal knowledge base for engineering standards, architectural patterns, and technical guidelines.

## Overview

This repository contains curated technical standards and best practices for software development, covering:

- Python development standards (tooling, testing, security, Docker, observability)
- Architecture patterns for GCP
- Technology Radar and GCP services reference
- CI/CD, Infrastructure, Data Platform, and AI/ML guidelines (coming soon)

## Quick Start

### Option 1: Dev Container (recommended)

Requires [Docker](https://www.docker.com/) and [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

```bash
git clone https://github.com/landerox/engineering-standards.git
code engineering-standards
```

When VS Code opens, click **"Reopen in Container"** when prompted. The container will automatically install all dependencies and configure pre-commit hooks.

### Option 2: Local Setup

Requires [uv](https://docs.astral.sh/uv/) package manager.

```bash
git clone https://github.com/landerox/engineering-standards.git
cd engineering-standards
uv sync
```

### Run Documentation Locally

```bash
uv run mkdocs serve
```

Then open [http://localhost:8000](http://localhost:8000) in your browser.

## Documentation

📖 **[View published documentation](https://landerox.github.io/engineering-standards/)**

## Project Info

| Resource                  | Description                  |
| ------------------------- | ---------------------------- |
| [CHANGELOG](CHANGELOG.md) | Version history and releases |
| [LICENSE](LICENSE)        | MIT License                  |
| [CODEOWNERS](CODEOWNERS)  | Repository ownership         |

## Contributing

This is a personal knowledge base, but suggestions are welcome via issues.
Please open an issue for any additions or improvements you would like to see.
Contributions via pull requests are not accepted at this time.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
