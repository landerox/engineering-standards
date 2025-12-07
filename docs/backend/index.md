# :material-server: Backend

> **Standards and guidelines for backend development, covering Python best practices, API design, and service architecture.**

---

## General

| Standard | Description |
|----------|-------------|
| :material-cog: [General / Core](python/general-core.md) | Base standards for all Python projects: tooling, code quality, testing, security, Git flow |

## By Project Type

| Standard | Description | Template |
|----------|-------------|----------|
| :material-api: [Web Services](python/web-services.md) | APIs, microservices, web applications (FastAPI, Docker, Cloud Run) | [template-python-web-service](https://github.com/landerox/template-python-web-service) |
| :material-package-variant: [Packages](python/packages.md) | Libraries, CLIs, distributable packages (Typer, Hatch, nox) | [template-python-package](https://github.com/landerox/template-python-package) |
| :material-database-sync: [Data Jobs](python/data-jobs.md) | ETL, batch processing, scheduled tasks (Polars, Cloud Run Jobs) | [template-python-data-job](https://github.com/landerox/template-python-data-job) |
| :material-file-document: [Documentation Sites](python/documentation-sites.md) | Documentation sites (MkDocs, mkdocs-material) | [template-mkdocs](https://github.com/landerox/template-mkdocs) |
