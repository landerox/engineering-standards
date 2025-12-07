# Backend

## Overview

Standards and guidelines for backend development, covering Python best practices, API design, and service architecture.

---

## Python Standards

### General

| Standard | Description |
|----------|-------------|
| [General Standards](python/general-standards.md) | Base standards for all Python projects: tooling, code quality, testing, security, Git flow |

### By Project Type

| Standard | Description | Template |
|----------|-------------|----------|
| [Web Service](python/web-service.md) | APIs, microservices, web applications (FastAPI, Docker, Cloud Run) | [template-python-web-service](https://github.com/landerox/template-python-web-service) |
| [Python Package](python/python-package.md) | Libraries, CLIs, distributable packages (Typer, Hatch, nox) | [template-python-package](https://github.com/landerox/template-python-package) |
| [Data Job](python/data-job.md) | ETL, batch processing, scheduled tasks (Polars, Cloud Run Jobs) | [template-python-data-job](https://github.com/landerox/template-python-data-job) |
| [Documentation Site](python/documentation-site.md) | Documentation sites (MkDocs, mkdocs-material) | [template-mkdocs](https://github.com/landerox/template-mkdocs) |
