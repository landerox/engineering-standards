# Contributing

Thank you for your interest in improving these engineering standards!

## How to Contribute

### Small Changes (typos, clarifications)

1. Fork the repository
2. Make your changes
3. Submit a PR

### New Standards or Major Changes

1. **Open an Issue first** to discuss the proposal
2. Wait for feedback before investing significant time
3. Submit a PR referencing the issue

## Local Development

```bash
git clone https://github.com/landerox/engineering-standards.git
cd engineering-standards
uv sync --group dev
uv run pre-commit install
uv run mkdocs serve
```

## Guidelines

- Follow [Conventional Commits](https://www.conventionalcommits.org/) for commit messages
- One focused change per PR
- Ensure pre-commit hooks pass before submitting

## License

By contributing, you agree that your contributions will be licensed under the MIT License. See [LICENSE](LICENSE) for details.
