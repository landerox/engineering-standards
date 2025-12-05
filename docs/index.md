# Engineering Standards & Guidelines

Welcome to my personal engineering knowledge base.

This portal consolidates the standards, "Golden Paths", and best practices I advocate for building robust, scalable, and maintainable software. It serves as a living document of technical decisions and preferred stacks.

---

## 🚀 How to Navigate

The content is organized by technical domain:

### Backend & Languages

| Topic | Tools | Document |
|-------|-------|----------|
| **Python Development** | `uv`, `Ruff`, `Pyright`, `Granian`, `Polars` | [**Python Standards**](backend/python-development-standards.md) |
| **API Design** | REST, gRPC, GraphQL, OpenAPI | *(Coming Soon)* |

### Data Platform

#### Databases

| Type | Tools | Document |
|------|-------|----------|
| **Relational (OLTP)** | PostgreSQL, AlloyDB, Cloud SQL | *(Coming Soon)* |
| **Graph** | Neo4j, Memgraph, Dgraph | *(Coming Soon)* |
| **Document** | Firestore, MongoDB | *(Coming Soon)* |
| **Vector** | Qdrant, Weaviate, Pinecone, Milvus, pgvector | *(Coming Soon)* |
| **Key-Value / Cache** | Redis, Memorystore | *(Coming Soon)* |
| **Embedded OLAP** | DuckDB | *(Coming Soon)* |

#### Analytics & Data Warehouse

| Topic | Tools | Document |
|-------|-------|----------|
| **Data Warehouse** | BigQuery | *(Coming Soon)* |
| **Lakehouse** | Delta Lake, Apache Iceberg, BigLake | *(Coming Soon)* |
| **Transformation** | dbt, Dataform | *(Coming Soon)* |

#### Data Orchestration & Pipelines

| Tool | Use Case | Document |
|------|----------|----------|
| **Dagster** | Asset-centric orchestration (recommended) | *(Coming Soon)* |
| **Cloud Composer** | Managed Airflow for complex workflows | *(Coming Soon)* |
| **Dataflow** | Apache Beam — Batch & Streaming at scale | *(Coming Soon)* |
| **Pub/Sub** | Event streaming & messaging | *(Coming Soon)* |
| **Cloud Workflows** | Lightweight serverless orchestration | *(Coming Soon)* |

#### Data Transformation

| Tool | Use Case | Document |
|------|----------|----------|
| **dbt** | SQL-first transformations in warehouse | *(Coming Soon)* |
| **Dataform** | dbt alternative (GCP native) | *(Coming Soon)* |
| **Polars** | DataFrame transformations (Python) | Covered in Python Standards |

### AI & Machine Learning

#### ML Platform

| Topic | Tools | Document |
|-------|-------|----------|
| **Training & Serving** | Vertex AI Training, Vertex AI Endpoints | *(Coming Soon)* |
| **Pipelines** | Vertex AI Pipelines, Kubeflow | *(Coming Soon)* |
| **Feature Store** | Vertex AI Feature Store, Feast | *(Coming Soon)* |
| **MLOps** | Experiment tracking, Model registry, Monitoring | *(Coming Soon)* |

#### Generative AI

| Topic | Tools | Document |
|-------|-------|----------|
| **LLM Integration** | Gemini, Claude, OpenAI APIs | *(Coming Soon)* |
| **RAG Systems** | LangChain, LlamaIndex, Vector DBs | *(Coming Soon)* |
| **Agentic AI** | LangGraph, CrewAI, AutoGen, Google ADK | *(Coming Soon)* |
| **Embeddings** | Vertex AI Embeddings, Sentence Transformers | *(Coming Soon)* |
| **Prompt Engineering** | Best practices, evaluation | *(Coming Soon)* |

### Infrastructure

| Topic | Tools | Document |
|-------|-------|----------|
| **Infrastructure as Code** | Terraform, Pulumi | *(Coming Soon)* |
| **Containers (Serverless)** | Cloud Run, Cloud Run Jobs | *(Coming Soon)* |
| **Containers (Orchestrated)** | GKE, Kubernetes | *(Coming Soon)* |
| **Networking** | VPC, Cloud NAT, Load Balancing, Cloud Armor | *(Coming Soon)* |
| **Secrets Management** | Secret Manager, External Secrets Operator | *(Coming Soon)* |

### CI/CD & DevOps

| Topic | Tools | Document |
|-------|-------|----------|
| **CI/CD Pipelines** | GitHub Actions, GitLab CI | *(Coming Soon)* |
| **Artifact Management** | Artifact Registry, Container Registry | *(Coming Soon)* |
| **Security Scanning** | SAST, DAST, SCA, Trivy, Bandit | *(Coming Soon)* |

---

## 🛠 Portal Stack

This documentation follows the **Docs as Code** philosophy:

| Component | Technology |
|-----------|------------|
| Engine | MkDocs Material |
| Manager | `uv` (Rust-based Python tooling) |
| Hosting | GitHub Pages via GitHub Actions |

---

## 📊 Document Status

| Status | Meaning |
|--------|---------|
| ✅ | Complete |
| 🚧 | In Progress |
| 📋 | Planned |

### Current Progress

| Document | Status | Priority |
|----------|--------|----------|
| Python Development Standards | ✅ | — |
| Terraform Standards | 📋 | High |
| GitHub Actions Standards | 📋 | High |
| BigQuery Standards | 📋 | High |
| Dagster + dbt Standards | 📋 | High |
| Vertex AI Standards | 📋 | Medium |
| GenAI / RAG Standards | 📋 | Medium |
| Agentic AI Patterns | 📋 | Medium |
| Vector Database Standards | 📋 | Medium |
| Graph Database Standards | 📋 | Low |
| GKE Standards | 📋 | Low |

---

## 🔧 Technology Radar (2025)

### Adopt (Use in Production)

| Category | Technology |
|----------|------------|
| Python Tooling | `uv`, `Ruff`, `Pyright`, `pytest` |
| Web Framework | FastAPI + Granian |
| DataFrames | Polars |
| Data Orchestration | Dagster |
| Data Transformation | dbt / Dataform |
| Data Warehouse | BigQuery |
| Vector Database | Qdrant (self-host), Vertex AI Vector Search (managed) |
| LLM Framework | LangChain + LangGraph |
| ML Platform | Vertex AI |
| IaC | Terraform |
| CI/CD | GitHub Actions |
| Containers | Cloud Run |

### Trial (Evaluate for Production)

| Category | Technology |
|----------|------------|
| Embedded OLAP | DuckDB |
| Graph Database | Memgraph, Neo4j |
| Lakehouse | Apache Iceberg on BigQuery |
| Agentic AI | Google ADK, CrewAI |
| Streaming | Apache Beam (Dataflow) |

### Assess (Watch & Learn)

| Category | Technology |
|----------|------------|
| Multi-agent | AutoGen, OpenAgents |
| Alternative Vector DBs | Weaviate, Milvus, Turbopuffer |
| Alternative Orchestration | Prefect, Kestra |

### Hold (Avoid in New Projects)

| Category | Technology | Replacement |
|----------|------------|-------------|
| Python Tooling | pip, poetry, Black, Flake8 | uv, Ruff |
| DataFrames | Pandas (for large data) | Polars |
| Orchestration | Airflow (self-managed) | Dagster, Cloud Composer |
| ML | Custom training infra | Vertex AI |

---

## 📚 Quick Reference

### GCP Services Mapping

| Need | GCP Service | Open Source Alternative |
|------|-------------|------------------------|
| Data Warehouse | BigQuery | — |
| Data Lake | Cloud Storage + BigLake | — |
| Batch Processing | Dataflow | Apache Beam |
| Stream Processing | Dataflow, Pub/Sub | Apache Beam, Kafka |
| Orchestration | Cloud Composer | Dagster, Airflow |
| ML Training | Vertex AI Training | — |
| ML Serving | Vertex AI Endpoints | — |
| Feature Store | Vertex AI Feature Store | Feast |
| Vector Search | Vertex AI Vector Search | Qdrant, Weaviate |
| Serverless Compute | Cloud Run | — |
| Kubernetes | GKE | — |
| Secrets | Secret Manager | HashiCorp Vault |
| IaC | — | Terraform |
| CI/CD | Cloud Build | GitHub Actions |

---

**Last updated:** December 2025
