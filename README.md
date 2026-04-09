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
| **AI Coding** | OpenCode CLI (OpenRouter free tier) + Ollama (local, batch/experiments) | ✅ Hybrid — [see ADR-0003](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0003-ai-strategy.md) |
| **Backend** | FastAPI + Uvicorn + Pydantic | ✅ Implemented (v0.1) |
| **Secrets (pre-Vault)** | python-dotenv + `.env` (gitignored, 600) + `.env.example` | ⏳ Week 3 — see [see ADR-0004](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0004-database-orm-migrations.md) |
| **Database** | PostgreSQL + SQLAlchemy 2 (async) + Alembic | ⏳ Planned (Week 3) |
| **Object Storage** | MinIO (S3-compatible API) | ⏳ Planned (Week 6) |
| **Secrets Mgmt** | HashiCorp Vault + Vaultwarden (team passwords) | ⏳ Planned (Week 7) |
| **Virtualization** | Proxmox VE | ⏳ Planned (Week 9) |
| **Orchestration** | Docker Compose → k3s (Kubernetes) | ⏳ Planned (Weeks 5–9) |
| **CI/CD** | GitHub Actions | ⏳ Planned (Week 8) |
| **Observability** | Prometheus + Grafana + Loki | ⏳ Planned (Week 11) |
| **Security** | UFW, fail2ban, Trivy, Vault policies | 🔄 In Progress |
| **Testing** | pytest + httpx (async) | ⏳ Week 3 |
| **Pre-commit** | pre-commit + commitizen | ⏳ Week 3 |

---

## Local AI Infrastructure Notes

The ACEMAGIC's Radeon 680M (iGPU) shares memory bandwidth with the system (~50 GB/s vs ~360 GB/s on a dedicated GPU).
Tested inference with `qwen2.5-coder:7b` — 2–4 tok/s, 30–50s latency, 8 cores saturated.
**Decision**: cloud-first for interactive development, Ollama reserved for batch/MLOps experiments.
See [ADR-0001](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0001-ai-tooling-and-local-llm-strategy.md) (superseded) and [ADR-0003](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0003-ai-strategy.md).

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
- [x] AI Local Workflow: OpenCode CLI + Ollama + Qwen3:8b integration (→ migrated to cloud, ADR-0003)
- [x] Backend Scaffold: FastAPI `/health` endpoint + Swagger UI
- [x] Python Environment: `venv` isolation + PEP 668 compliance
- [x] Documentation: Cheatsheet, ADR-0001 (superseded), ADR-0002, ADR-0003, ADR-0004
- [ ] Pydantic models & mock data (Week 3)
- [ ] Docker basics & containerization (Week 4)

### Phase 2: IaC, Storage & Secrets ⏳ In Progress (Weeks 3–8)

**Week 3 — Data Layer & Dev Tooling**
- [ ] PostgreSQL setup (local socket for dev)
- [ ] SQLAlchemy 2 async models + Pydantic schemas
- [ ] Alembic: first migration, conventions established
- [ ] `python-dotenv` + `.env.example` — pre-Vault secrets pattern (ADR-0004)
- [ ] pytest + httpx: first unit tests for models and endpoints
- [ ] pre-commit + commitizen: enforce Conventional Commits automatically
- [ ] Feature branch habit: branch for anything that takes >1 session
- [ ] ADR-0004: Database, ORM & Migrations

**Week 4 — Docker Fundamentals**
- [ ] Docker: images, containers, multi-stage builds
- [ ] Containerize FastAPI + PostgreSQL with Docker Compose
- [ ] systemd or Docker `restart: always` for service persistence (resolves known tech debt)

**Weeks 5–6 — Docker Compose & Storage**
- [ ] Docker Compose: multi-service orchestration (API + DB + MinIO)
- [ ] MinIO setup: S3-compatible storage for models & artifacts

**Week 7 — Secrets**
- [ ] HashiCorp Vault: secrets management & dynamic DB credentials
- [ ] Migrate `DATABASE_URL` from `.env` to Vault (see ADR-0004 migration path)

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
| No service persistence (FastAPI not auto-starting) | Week 4 (Docker) | Manual start during dev — app not yet functional |
| No CI/CD gates on merges | Week 8 (GitHub Actions) | Feature branch habit from Week 3 |

---

## Branching Strategy (Solo Dev)

- `main`: stable, tested code only — never leave it broken overnight
- `feature/<name>`: for anything that takes more than one work session or could leave `main` in a broken state
- `fix/<name>`: bug fixes
- `experiment/<name>`: throwaway spikes and model/infra experiments

> Rationale: building the feature branch habit now means CI/CD in Week 8 is a natural extension, not a workflow change.

---

## Featured Project: brewery-app

**Gestión de cervecería artesana con IA** → [Ver repo](https://github.com/adriandelvalle/brewery-app)

| Estado | Stack | Propósito |
| --- | --- | --- |
| 🟡 Fase 1 (scaffold) | FastAPI, PostgreSQL, Docker (próximamente) | Vehículo de aprendizaje para DevOps/MLOps/LLMOps |

**Roadmap de features**:

- Inventario read-only API (Fase 2)
- Auth básico + frontend PWA (Fase 3)
- Predicción de fermentación con ML (Fase 4)
- Asistente de calendario con RAG + Agentes (Fase 5)
- Production hardening + beta cerrada (Fase 6)

---

## Documentation

### Learning Notes (Chronological)

| Fase | Semana | Tema | Estado | Notas |
| --- | --- | --- | --- | --- |
| **0** | Setup | Environment Setup & Hardening | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase0-environment-setup.md) |
| **1** | 1 | Linux Fundamentals & Security Hardening | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week1-linux-fundamentals.md) |
| **1** | 2 | AI Local Workflow & FastAPI Scaffold | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week2-opencode-fastapi.md) |
| **1** | 3 | PostgreSQL + SQLAlchemy + Alembic + Testing | ⏳ Pending | - |
| **1** | 4 | Docker Fundamentals & Containerization | ⏳ Pending | - |
| **2** | 5–8 | IaC, MinIO Storage & HashiCorp Vault | ⏳ Planned | - |
| **3** | 9–12 | Kubernetes (k3s) & Observability | ⏳ Planned | - |

### Reference Cheatsheets & Docs

| Área | Documento | Descripción |
| --- | --- | --- |
| Git | [git-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/git-cheatsheet.md) | Commits, branching, remote sync & conventional patterns |
| FastAPI + AI | [fastapi-ai-dev-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/fastapi-ai-dev-cheatsheet.md) | venv, uvicorn, OpenCode prompts & project layout |
| Architecture | [ADR Index](https://github.com/adriandelvalle/brewery-app/tree/main/docs/decisions) | Decision records |

### Project Documentation

| Document | Language | Purpose |
| --- | --- | --- |
| `brewery-app/README.md` | English | Technical docs for the brewery app |
| `brewery-app/docs/decisions/` | English | Architecture Decision Records (ADRs) |
| `portfolio/docs/*.md` | Español | Deep technical notes for learning |

---

## Current Environment Status (Verified 2026-04-08)

| Component | Configuration | Status | Verification Command |
| --- | --- | --- | --- |
| **Host** | `jotasrv` (ACEMAGIC Mini PC) | ✅ Active | `hostname` |
| **Network Interface** | `eno1` (confirmed, not eth0) | ✅ Detected | `ip link show` |
| **Static IP** | `192.168.0.21/24` | ✅ Configured | `ip -4 addr show eno1` |
| **SSH Access** | `ssh jota@jotasrv` or `ssh jota@192.168.0.21` | ✅ Working | Key-based auth, no password prompt |
| **SMB Mount** | `/mnt/Win_Projects` ← `//192.168.0.10/JotaSrv` | ✅ Mounted | `df -h /mnt/Win_Projects` |
| **Mount Persistence** | `/etc/fstab` entry (CIFS v3.0, creds in `/root/.smbcredentials`) | ✅ Persistent | `grep Win_Projects /etc/fstab` |
| **AI Tooling** | OpenCode CLI free tier) | ✅ Ready | `opencode` → user selects model at runtime |
| **Project Root** | `~/projects/brewery-app` | ✅ Structured | `ls -la` shows `backend/`, `docs/`, `scripts/` |
| **Python Env** | `backend/venv/` (FastAPI + Uvicorn + Pydantic v2) | ✅ Active | `source backend/venv/bin/activate` |
| **AI Strategy** | Hybrid: Cloud-first for dev, local for batch/MLOps experiments | ✅ Documented | See [docs/decisions/0003-ai-strategy.md](https://github.com/adriandelvalle/brewery-app/blob/main/docs/decisions/0003-ai-strategy.md) |

---

## Useful Links

- [My GitHub Profile](https://github.com/adriandelvalle)
- [brewery-app Repo](https://github.com/adriandelvalle/brewery-app)
- [OpenCode Documentation](https://opencode.ai)
- [Ollama](https://ollama.com)
- [VS Code Remote SSH](https://code.visualstudio.com/docs/remote/ssh)
- [Conventional Commits](https://www.conventionalcommits.org/)

---

> *Last updated: 2026-04-09*
> *Philosophy: Learning-first. 100% free stack. Depth > speed.*