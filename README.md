# My DevOps • MLOps • LLMOps Journey

> Documentation of my learning path and projects on my personal server.

---

## About Me

- **GitHub**: [adriandelvalle](https://github.com/adriandelvalle)
- **Email**: [adriandelvallecv@gmail.com](mailto:adriandelvallecv@gmail.com)
- **Focus**: DevOps infrastructure, MLOps pipelines, LLMOps local deployment

---

## Goal

Learn and document the deployment of local AI infrastructure, applying DevOps best practices to build a technical portfolio.

---

## Project Structure

| Folder | Purpose |
| --- | --- |
| `projects/brewery-app/` | Core application: FastAPI backend, AI integration, infra configs & ADRs |
| `/projects/portfolio/docs/learning/` | Chronological learning notes (by phase/week) |
| `/projects/portfolio/docs/reference/` | Reusable cheatsheets, architecture decisions & runbooks |
| `/projects/portfolio/"sandbox-folders"/` | Isolated experiments: `devops/`, `mlops/`, `llmops/` |

> **Note**: Deep technical documentation inside each folder is in Spanish.

---

## Tech Stack

| Component | Technology | Status |
| --- | --- | --- |
| **Hardware** | ACEMAGIC Mini PC (Ryzen 7 6800H, Radeon 680M iGPU, 32GB RAM DDR5) | ✅ Active |
| **OS / Access** | Ubuntu 24.04 LTS + VS Code Remote SSH / mRemoteNG | ✅ Active |
| **Version Control** | Git + GitHub (SSH) + Conventional Commits | ✅ Active |
| **AI Coding** | OpenCode CLI (OpenRouter free tier) + Ollama (local, batch/experiments) | ✅ Hybrid — see ADR-0003 |
| **Backend** | FastAPI + Uvicorn + Pydantic v2 | ✅ Implemented (v0.1) |
| **API Models** | Pydantic v2 — Recipe, Batch, FermentationSample | ✅ Implemented |
| **Secrets (pre-Vault)** | python-dotenv + `.env` (gitignored, 600) + `.env.example` | ⏳ Week 5 — see [ADR-0004](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0004-database-orm-migrations.md)|
| **Database** | PostgreSQL + SQLAlchemy 2 (async) + Alembic | ⏳ Planned (Week 5) |
| **Testing** | pytest + httpx | ⏳ Pending (Week 3) |
| **Pre-commit** | pre-commit + commitizen | ⏳ Pending (Week 3) |
| **Object Storage** | MinIO (S3-compatible API) | ⏳ Planned (Week 6) |
| **Secrets Mgmt** | HashiCorp Vault + Vaultwarden | ⏳ Planned (Week 7) |
| **Virtualization** | Proxmox VE | ⏳ Planned (Week 9) |
| **Orchestration** | Docker Compose → k3s (Kubernetes) | ⏳ Planned (Weeks 4–9) |
| **CI/CD** | GitHub Actions | ⏳ Planned (Week 8) |
| **Observability** | Prometheus + Grafana + Loki | ⏳ Planned (Week 11) |
| **Security** | UFW, fail2ban, Trivy, Vault policies | 🔄 In Progress |

---

## Local AI Infrastructure Notes

The ACEMAGIC's Radeon 680M (iGPU) shares memory bandwidth with the system (~50 GB/s
vs ~360 GB/s on a dedicated GPU). Tested inference with `qwen2.5-coder:7b` —
2–4 tok/s, 30–50s latency, 8 cores saturated. **Decision**: cloud-first for
interactive development, Ollama reserved for batch/MLOps experiments.
See [ADR-0001](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0001-ai-tooling-and-local-llm-strategy.md) (superseded) and [ADR-0003](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0003-ai-strategy.md).

---

## Learning Progress

### Phase 0: Environment Setup ✅ Completed

- [x] Remote server setup via SSH & hardening
- [x] Git configuration with GitHub keys & Conventional Commits
- [x] VS Code Remote SSH workflow & port forwarding
- [x] Portfolio & `brewery-app` repository structure

### Phase 1: Foundations 🔄 In Progress (Week 3/4)

- [x] Linux fundamentals: FHS, permissions, processes, networking
- [x] Security baseline: `audit-permissions.sh`, UFW, fail2ban config
- [x] AI Local Workflow: OpenCode CLI + Ollama (→ migrated to cloud, ADR-0003)
- [x] Backend Scaffold: FastAPI `/health` endpoint + Swagger UI
- [x] Python Environment: `venv` isolation + PEP 668 compliance
- [x] Documentation: ADR-0001 (superseded), ADR-0002, ADR-0003, ADR-0004
- [x] Pydantic models: Recipe, Batch, FermentationSample (Week 3)
- [x] API v1 endpoints: GET/POST recipes and batches (Week 3)
- [x] Professional src structure: api/, models/, core/ (Week 3)
- [ ] pytest + httpx: first unit tests (Week 3)
- [ ] pre-commit + commitizen (Week 3)
- [ ] Docker basics & containerization (Week 4)

### Phase 2: IaC, Storage & Secrets ⏳ Planned (Weeks 5–8)

**Week 5 — Data Layer**
- [ ] PostgreSQL setup + Docker Compose
- [ ] SQLAlchemy 2 async models
- [ ] Alembic: first migration
- [ ] python-dotenv + `.env.example` — pre-Vault secrets pattern ([ADR-0004](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0004-database-orm-migrations.md))

**Week 6 — Storage**
- [ ] Docker Compose: multi-service orchestration (API + DB + MinIO)
- [ ] MinIO setup: S3-compatible storage for models & artifacts

**Week 7 — Secrets**
- [ ] HashiCorp Vault: secrets management & dynamic DB credentials
- [ ] Migrate `DATABASE_URL` from `.env` to Vault (see [ADR-0004](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0004-database-orm-migrations.md))

**Week 8 — CI/CD**
- [ ] GitHub Actions pipeline: lint, test, build, push
- [ ] PR-based workflow (feature branches → CI gates)

### Phase 3: Orchestration & Observability ⏳ Planned (Weeks 9–12)

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
- [ ] Orchestration: LangChain / LlamaIndex
- [ ] Multi-agent systems (AutoGen / CrewAI)
- [ ] Evaluation frameworks: DeepEval, Ragas

### Phase 6: Integration & Production Readiness ⏳ Planned

- [ ] `brewery-app` v1.0: full stack deployment on k3s
- [ ] Security hardening, load testing & documentation
- [ ] Portfolio polish: live demo, case study & LinkedIn article

---

## Known Technical Debt

| Item | Accepted Until | Mitigation |
| --- | --- | --- |
| DB credentials in `.env` (plaintext) | Week 7 (Vault) | 600 permissions + gitignored + `.env.example` |
| No service persistence (FastAPI not auto-starting) | Week 4 (Docker) | Manual start during dev — app not yet in production |
| No CI/CD gates on merges | Week 8 (GitHub Actions) | Feature branch habit from Week 3 |
| Mock data in memory (no persistence) | Week 5 (PostgreSQL) | Acceptable for learning phase |

---

## Branching Strategy (Solo Dev)

- `main`: stable, tested code only — never leave it broken overnight
- `feature/<n>`: for anything that takes more than one session or could break `main`
- `fix/<n>`: bug fixes
- `experiment/<n>`: throwaway spikes and model/infra experiments

> Rationale: building the feature branch habit now means CI/CD in Week 8
> is a natural extension, not a workflow change.

---

## Featured Project: brewery-app

**Gestión de cervecería artesana con IA** → [Ver repo](https://github.com/adriandelvalle/brewery-app)

| Estado | Stack | Propósito |
| --- | --- | --- |
| 🔄 Fase 1 (Week 3) | FastAPI + Pydantic, PostgreSQL (próximo), Docker (próximo) | Vehículo de aprendizaje para DevOps/MLOps/LLMOps |

**Roadmap de features**:

- [x] API scaffold + health endpoint
- [x] Recipe & Batch management API (mock data)
- [ ] Tests + pre-commit (Semana 3)
- [ ] Containerización Docker (Semana 4)
- [ ] Inventario con PostgreSQL (Semana 5)
- [ ] Auth básico + frontend PWA (Fase 3)
- [ ] Predicción de fermentación con ML (Fase 4)
- [ ] Asistente de calendario con RAG + Agentes (Fase 5)
- [ ] Production hardening + beta cerrada (Fase 6)

---

## Documentation

### Learning Notes (Chronological)

| Fase | Semana | Tema | Estado | Notas |
| --- | --- | --- | --- | --- |
| **0** | Setup | Environment Setup & Hardening | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase0-environment-setup.md) |
| **1** | 1 | Linux Fundamentals & Security Hardening | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week1-linux-fundamentals.md) |
| **1** | 2 | AI Local Workflow & FastAPI Scaffold | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week2-opencode-fastapi.md) |
| **1** | 3 | Pydantic Models & API Structure | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week3-pydantic-models.md) |
| **1** | 3 | pytest + pre-commit | ⏳ Pending | — |
| **1** | 4 | Docker Fundamentals & Containerization | ⏳ Pending | — |
| **2** | 5–8 | IaC, MinIO Storage & HashiCorp Vault | ⏳ Planned | — |
| **3** | 9–12 | Kubernetes (k3s) & Observability | ⏳ Planned | — |

### Reference Cheatsheets & Docs

| Área | Documento | Descripción |
| --- | --- | --- |
| Git | [git-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/git-cheatsheet.md) | Commits, branching, remote sync & conventional patterns |
| FastAPI + Pydantic + AI | [fastapi-ai-dev-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/fastapi-ai-dev-cheatsheet.md) | venv, uvicorn, Pydantic patterns, OpenCode prompts & project layout |
| Architecture | [ADR Index](https://github.com/adriandelvalle/brewery-app/tree/main/docs/decisions) | Decision records |

---

## Current Environment Status (Verified 2026-04-13)

| Component | Configuration | Status |
| --- | --- | --- |
| **Host** | `jotasrv` (ACEMAGIC Mini PC) | ✅ Active |
| **Static IP** | `192.168.0.21/24` | ✅ Configured |
| **SSH Access** | `ssh jota@jotasrv` | ✅ Working |
| **SMB Mount** | `/mnt/Win_Projects` ← `//192.168.0.10/JotaSrv` | ✅ Mounted |
| **AI Tooling** | OpenCode CLI + OpenRouter (free tier) | ✅ Ready |
| **Project Root** | `~/projects/brewery-app` | ✅ Structured |
| **Python Env** | `backend/venv/` (FastAPI + Uvicorn + Pydantic v2) | ✅ Active |
| **API** | 7 endpoints under `/api/v1/` | ✅ Running |

---

## Useful Links

- [My GitHub Profile](https://github.com/adriandelvalle)
- [brewery-app Repo](https://github.com/adriandelvalle/brewery-app)
- [OpenCode Documentation](https://opencode.ai)
- [Ollama](https://ollama.com)
- [Conventional Commits](https://www.conventionalcommits.org/)

---

> *Last updated: 2026-04-13*
> *Philosophy: Learning-first. 100% free stack. Depth > speed.*