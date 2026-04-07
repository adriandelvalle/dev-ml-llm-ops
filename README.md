# My DevOps • MLOps • LLMOps Journey



> Documentation of my learning path and projects on my personal server.

  

---

  

## About Me

- **GitHub**: [adriandelvalle](https://github.com/adriandelvalle)

- **Email**: adriandelvallecv@gmail.com

- **Focus**: DevOps infrastructure, MLOps pipelines, LLMOps local deployment

  

---

  

## Goal

Learn and document the deployment of local AI infrastructure, applying DevOps best practices to build a technical portfolio.

  

---

  

## Project Structure

| Folder | Purpose |
|:---|:---|
| `projects/brewery-app/` | Core application: FastAPI backend, AI integration, infra configs & ADRs |
| `/projects/portfolio/docs/learning/` | Chronological learning notes (by phase/week) |
| `/projects/portfolio/docs/reference/` | Reusable cheatsheets, architecture decisions & runbooks |
| `/projects/portfolio/"sandbox-folders"/` | Isolated experiments: `devops/`, `mlops/`, `llmops/` |

> **Note**: Deep technical documentation inside each folder is in Spanish.

---

## Tech Stack

| Component | Technology | Status |
|:---|:---|:---|
| **Hardware** | ACEMAGIC Mini PC (Ryzen 7 6800H, Radeon 680M, 32GB RAM) | ✅ Active |
| **OS / Access** | Ubuntu 24.04 LTS + VS Code Remote SSH / mRemoteNG | ✅ Active |
| **Version Control** | Git + GitHub (SSH) + Conventional Commits | ✅ Active |
| **AI Coding** | OpenCode CLI + Ollama + Qwen3:8b (100% local) | ✅ Active |
| **Backend** | FastAPI + Uvicorn + Pydantic | ✅ Implemented (v0.1) |
| **Database** | PostgreSQL + SQLAlchemy | ⏳ Planned (Week 3) |
| **Object Storage** | MinIO (S3-compatible API) | ⏳ Planned (Week 6) |
| **Secrets Mgmt** | HashiCorp Vault + Vaultwarden (team passwords) | ⏳ Planned (Week 7) |
| **Virtualization** | Proxmox VE | ⏳ Planned (Week 9) |
| **Orchestration** | Docker Compose → k3s (Kubernetes) | ⏳ Planned (Weeks 5-9) |
| **CI/CD** | GitHub Actions | ⏳ Planned (Week 8) |
| **Observability** | Prometheus + Grafana + Loki | ⏳ Planned (Week 11) |
| **Security** | UFW, fail2ban, Trivy, Vault policies | 🔄 In Progress |

---

## Learning Progress

### Phase 0: Environment Setup ✅ Completed
- [x] Remote server setup via SSH & hardening
- [x] Git configuration with GitHub keys & Conventional Commits
- [x] VS Code Remote SSH workflow & port forwarding
- [x] Portfolio & `brewery-app` repository structure

### Phase 1: Foundations 🟢 Completed (Week 2/4)
- [x] Linux fundamentals: FHS, permissions, processes, networking
- [x] Security baseline: `audit-permissions.sh`, UFW, fail2ban config
- [x] AI Local Workflow: OpenCode CLI + Ollama + Qwen3:8b integration
- [x] Backend Scaffold: FastAPI `/health` endpoint + Swagger UI
- [x] Python Environment: `venv` isolation + PEP 668 compliance
- [x] Documentation: Cheatsheet, ADR-0001 (AI Tooling), ADR-0002 (Infra Stack)
- [ ] Pydantic models & mock data (Week 3)
- [ ] Docker basics & containerization (Week 4)

### Phase 2: IaC, Storage & Secrets ⏳ Planned (Weeks 5-8)
- [ ] Docker fundamentals: images, containers, multi-stage builds
- [ ] Docker Compose: multi-service orchestration (API + DB + MinIO)
- [ ] MinIO setup: S3-compatible storage for models & artifacts
- [ ] HashiCorp Vault: secrets management & dynamic credentials
- [ ] CI/CD pipeline: GitHub Actions (lint, test, build, push)

### Phase 3: Orchestration & Observability ⏳ Planned (Weeks 9-12)
- [ ] Proxmox VE: VM/LXC provisioning for isolated labs
- [ ] Kubernetes (k3s): pods, deployments, services, ingress
- [ ] Helm charts & GitOps basics
- [ ] Observability stack: Prometheus + Grafana dashboards
- [ ] Backup & DR: MinIO replication + Vault snapshots

### Phase 4: MLOps Fundamentals ⏳ Planned
- [ ] Reproducible environments & experiment tracking (MLflow)
- [ ] Data validation with Great Expectations
- [ ] Model serving, monitoring & drift detection

### Phase 5: LLMOps Specialization ⏳ Planned
- [ ] RAG pipeline: embeddings, vector DB (Chroma/FAISS), retrieval
- [ ] Orchestration: LangChain/LlamaIndex
- [ ] Multi-agent systems (AutoGen/CrewAI)
- [ ] Evaluation frameworks: DeepEval, Ragas

### Phase 6: Integration & Production Readiness ⏳ Planned
- [ ] `brewery-app` v1.0: full stack deployment on k3s
- [ ] Security hardening, load testing, & documentation
- [ ] Portfolio polish: live demo, case study, & LinkedIn article

  

---

  

## Featured Project: brewery-app

  

**Gestión de cervecería artesana con IA** → [Ver repo](https://github.com/adriandelvalle/brewery-app)

  

| Estado | Stack | Propósito |
| :--- | :--- | :--- |
| 🟡 Fase 1 (scaffold) | FastAPI, PostgreSQL, Docker (próximamente) | Vehículo de aprendizaje para DevOps/MLOps/LLMOps |

  

**Roadmap de features**:

- [ ] Inventario read-only API (Fase 2)

- [ ] Auth básico + frontend PWA (Fase 3)

- [ ] Predicción de fermentación con ML (Fase 4)

- [ ] Asistente de calendario con RAG + Agentes (Fase 5)

- [ ] Production hardening + beta cerrada (Fase 6)

  

---

  

## Documentation

  

### Learning Notes (Chronological)

| Fase | Semana | Tema | Estado | Notas |
| :--- | :--- | :--- | :--- | :--- |
| **1** | 1 | Linux Fundamentals & Security Hardening | ✅ Done | [Ver](docs/learning/phase1-week1-linux-fundamentals.md) |
| **1** | 2 | AI Local Workflow & FastAPI Scaffold | ✅ Done | [Ver](docs/learning/phase1-week2-opencode-fastapi.md) |
| **1** | 3 | Pydantic Validation & Mock Data | ⏳ Pending | - |
| **1** | 4 | Docker Fundamentals & Containerization | ⏳ Pending | - |
| **2** | 5-8 | IaC, MinIO Storage & HashiCorp Vault | ⏳ Planned | - |
| **3** | 9-12 | Kubernetes (k3s) & Observability | ⏳ Planned | - |

---

### Reference Cheatsheets & Docs

| Área | Documento | Descripción |
| :--- | :--- | :--- |
| Git | [git-cheatsheet.md](docs/reference/git-cheatsheet.md) | Commits, branching, remote sync & conventional patterns |
| FastAPI + AI | [fastapi-ai-dev-cheatsheet.md](docs/reference/fastapi-ai-dev-cheatsheet.md) | venv, uvicorn, OpenCode prompts & project layout |
| Architecture | [ADR Index](https://github.com/adriandelvalle/brewery-app/tree/main/docs/decisions) | Decision records: AI Tooling & Infra Stack |

### Project Documentation

| Document | Language | Purpose |
| :--- | :--- | :--- |
| `brewery-app/README.md` | English | Technical docs for the brewery app |
| `brewery-app/docs/decisions/` | English | Architecture Decision Records (ADRs) |
| `portfolio/docs/*.md` | Español | Deep technical notes for learning |

  

---

  

## Useful Links

- [My GitHub Profile](https://github.com/adriandelvalle)

- [brewery-app Repo](https://github.com/adriandelvalle/brewery-app)

- [OpenCode Documentation](https://opencode.ai)

- [Ollama](https://ollama.com)

- [VS Code Remote SSH](https://code.visualstudio.com/docs/remote/ssh)

- [Conventional Commits](https://www.conventionalcommits.org/)

  

---

  

> *Last updated: 2026-04-06*

> *Philosophy: Learning-first. 100% free stack. Depth > speed.*
