# Engineering Standards

> **The central hub for development standards, architectural patterns, and technical guidelines.**

---

## Architecture & Domains

| Domain | Focus |
|--------|-------|
| [**Backend**](backend/index.md) | Python ecosystem, API design, Data processing, and Cloud infrastructure. |

---

## About This Project

This repository serves as the single source of truth for building software in my projects. It connects abstract **Standards** (the "What" and "Why") with concrete **Templates** (the "How").

### Core Philosophy

The standards defined here are opinionated to ensure speed and maintainability.

| Principle | Description |
|-----------|-------------|
| **Simplicity** | Straightforward solutions over unnecessary abstractions. |
| **Consistency** | Using the same patterns across all projects (CLI, API, or Jobs). |
| **Automation** | If it can be scripted, it is automated (CI/CD, Linting). |
| **Security** | Security is configured by default, not added later. |
| **Reproducibility** | Environments are deterministic (using `uv`, Docker). |
