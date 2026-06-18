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
| `projects/brewery-app/` | Core application: FastAPI backend, Nginx, AI integration, infra configs & ADRs |
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
| **Code Quality** | pre-commit + commitizen (both repos) | ✅ Active |
| **AI Coding** | OpenCode CLI (OpenRouter free tier) + Ollama (local, batch/experiments) | ✅ Hybrid — see ADR-0003 |
| **Backend** | FastAPI + Uvicorn + Pydantic v2 | ✅ Implemented |
| **API Models** | Pydantic v2 — Recipe, Batch, FermentationSample | ✅ Implemented |
| **Testing** | pytest + httpx + pytest-asyncio (14 tests) | ✅ Implemented |
| **Containerization** | Docker 29.5.2 + Dockerfile + .dockerignore | ✅ Implemented |
| **Service Persistence** | Docker restart unless-stopped | ✅ Implemented |
| **Docker Networks** | brewery-network (API ↔ Nginx ↔ Cloudflared) | ✅ Implemented |
| **Reverse Proxy** | Nginx (alpine) | ✅ Implemented |
| **Static File Serving** | Volume-mounted, instant updates | ✅ Implemented |
| **External Access** | Cloudflare Tunnel (quick tunnel, free) | ✅ Implemented |
| **Secrets (pre-Vault)** | python-dotenv + `.env` (gitignored, 600) + `.env.example` | ⏳ Week 5 — see ADR-0004 |
| **Database** | PostgreSQL + SQLAlchemy 2 (async) + Alembic | ⏳ Planned (Week 5) |
| **Object Storage** | MinIO (S3-compatible API) | ⏳ Planned (Week 6) |
| **Secrets Mgmt** | HashiCorp Vault + Vaultwarden | ⏳ Planned (Week 7) |
| **Virtualization** | Proxmox VE | ⏳ Planned (Week 9) |
| **Orchestration** | Docker Compose → k3s (Kubernetes) | ⏳ Planned (Weeks 5–9) |
| **CI/CD** | GitHub Actions | ⏳ Planned (Week 8) |
| **Observability** | Prometheus + Grafana + Loki | ⏳ Planned (Week 11) |
| **Custom Domain** | trestigris.beer (Cloudflare) | ⏳ Deferred — until real content exists |
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

### Phase 1: Foundations ✅ Completed (Week 4/4)

- [x] Linux fundamentals: FHS, permissions, processes, networking
- [x] Security baseline: `audit-permissions.sh`, UFW, fail2ban config
- [x] AI Local Workflow: OpenCode CLI + Ollama (→ migrated to cloud, ADR-0003)
- [x] Backend Scaffold: FastAPI `/health` endpoint + Swagger UI
- [x] Python Environment: `venv` isolation + PEP 668 compliance
- [x] Documentation: ADR-0001 (superseded), ADR-0002, ADR-0003, ADR-0004
- [x] Pydantic models: Recipe, Batch, FermentationSample (Week 3)
- [x] API v1 endpoints: GET/POST recipes and batches (Week 3)
- [x] Professional src structure: api/, models/, core/ (Week 3)
- [x] pytest + httpx: 14 tests with fixtures and state isolation (Week 3)
- [x] pre-commit + commitizen: enforced on both repos (Week 3)
- [x] Docker: containerized API + Dockerfile + .dockerignore (Week 4)
- [x] Service persistence: restart unless-stopped verified post-reboot (Week 4)
- [x] requirements split: production vs development (Week 4)
- [x] Docker networks: brewery-network connecting containers (Week 4)
- [x] Nginx reverse proxy + static file serving (Week 4)
- [x] Cloudflare Tunnel: external HTTPS access without port forwarding (Week 4)
- [x] UTF-8 encoding bug fixed (HTML + Nginx charset) (Week 4)
- [x] Static IP fixed on Windows after DHCP change detected (Week 4)

### Phase 2: IaC, Storage & Secrets ⏳ Planned (Weeks 5–8)

**Week 5 — Data Layer**
- [ ] PostgreSQL setup + Docker Compose
- [ ] SQLAlchemy 2 async models
- [ ] Alembic: first migration
- [ ] python-dotenv + `.env.example` — pre-Vault secrets pattern (ADR-0004)
- [ ] Member (`Socio`) model — RGPD-compliant fields, quota type, renewal logic

**Week 6 — Storage**
- [ ] Docker Compose: multi-service orchestration (API + DB + MinIO)
- [ ] MinIO setup: S3-compatible storage for models & artifacts

**Week 7 — Secrets**
- [ ] HashiCorp Vault: secrets management & dynamic DB credentials
- [ ] Migrate `DATABASE_URL` from `.env` to Vault (see ADR-0004)

**Week 8 — CI/CD**
- [ ] GitHub Actions pipeline: lint, test, build, push
- [ ] PR-based workflow (feature branches → CI gates)

### Phase 3: Orchestration & Observability ⏳ Planned (Weeks 9–12)

- [ ] Proxmox VE: VM/LXC provisioning for isolated labs
- [ ] Kubernetes (k3s): pods, deployments, services, ingress
- [ ] Helm charts & GitOps basics
- [ ] Observability stack: Prometheus + Grafana dashboards
- [ ] Backup & DR: MinIO replication + Vault snapshots
- [ ] Admin panel with login (JPG export, member management)
- [ ] Member area with login (membership status, batch history)

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
| ~~No service persistence~~ | ~~Week 4~~ | ✅ Resolved — Docker unless-stopped |
| No CI/CD gates on merges | Week 8 (GitHub Actions) | Feature branch habit from Week 3 |
| Mock data in memory (no persistence) | Week 5 (PostgreSQL) | Acceptable for learning phase |
| Cloudflare Tunnel subdomain is temporary/random | Until `trestigris.beer` purchased | Re-check `docker logs brewery-cloudflared` after restarts |
| Router has no DHCP Reservation (Sercom firmware) | N/A | Static IP configured manually in Windows instead |

---

## Branching Strategy (Solo Dev)

- `main`: stable, tested code only — never leave it broken overnight
- `feature/<n>`: for anything that takes more than one session or could break `main`
- `fix/<n>`: bug fixes
- `experiment/<n>`: throwaway spikes and model/infra experiments

---

## Featured Project: brewery-app

**Gestión de cervecería artesana con IA** → [Ver repo](https://github.com/adriandelvalle/brewery-app)

| Estado | Stack | Propósito |
| --- | --- | --- |
| ✅ Fase 1 completa | FastAPI + Pydantic + pytest + Docker + Nginx + Cloudflare Tunnel | Vehículo de aprendizaje para DevOps/MLOps/LLMOps |

**Roadmap de features**:

- [x] API scaffold + health endpoint
- [x] Recipe & Batch management API (mock data)
- [x] pytest suite — 14 tests
- [x] pre-commit + commitizen
- [x] Docker containerization + service persistence
- [x] Nginx reverse proxy + static files
- [x] Cloudflare Tunnel — acceso público sin exponer IP doméstica
- [ ] Inventario con PostgreSQL (Semana 5)
- [ ] Formulario de registro de socios — RGPD, tipo de cuota, renovación (Semana 5)
- [ ] Panel admin con login (Fase 3)
- [ ] Área de socios con login (Fase 3)
- [ ] Predicción de fermentación con ML (Fase 4)
- [ ] Asistente de calendario con RAG + Agentes (Fase 5)
- [ ] Production hardening + beta cerrada (Fase 6)
- [ ] Dominio propio trestigris.beer (cuando haya contenido real)

---

## Documentation

### Learning Notes (Chronological)

| Fase | Semana | Tema | Estado | Notas |
| --- | --- | --- | --- | --- |
| **0** | Setup | Environment Setup & Hardening | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase0-environment-setup.md) |
| **1** | 1 | Linux Fundamentals & Security Hardening | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week1-linux-fundamentals.md) |
| **1** | 2 | AI Local Workflow & FastAPI Scaffold | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week2-opencode-fastapi.md) |
| **1** | 3 | Pydantic Models & API Structure | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week3-pydantic-models.md) |
| **1** | 3 | pytest + pre-commit | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week3-pytest-precommit.md) |
| **1** | 4 | Docker Fundamentals & Containerization | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week4-docker-fundamentals.md) |
| **1** | 4 | Nginx, Docker Networks & Cloudflare Tunnel | ✅ Done | [Ver](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/learning/phase1-week4-nginx-docker-networks-cloudflare.md) |
| **2** | 5–8 | IaC, MinIO Storage & HashiCorp Vault | ⏳ Planned | — |
| **3** | 9–12 | Kubernetes (k3s) & Observability | ⏳ Planned | — |

### Reference Cheatsheets & Docs

| Área | Documento | Descripción |
| --- | --- | --- |
| Git | [git-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/git-cheatsheet.md) | Commits, branching, remote sync & conventional patterns |
| FastAPI + Pydantic + AI | [fastapi-ai-dev-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/fastapi-ai-dev-cheatsheet.md) | venv, uvicorn, Pydantic patterns & project layout |
| pytest + pre-commit | [pytest-precommit-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/pytest-precommit-cheatsheet.md) | Testing patterns, fixtures, pre-commit hooks |
| Docker | [docker-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/docker-cheatsheet.md) | Build, run, logs, debug, cleanup |
| Nginx + Networks + Cloudflare | [nginx-docker-networks-cloudflare-cheatsheet.md](https://github.com/adriandelvalle/dev-ml-llm-ops/blob/main/docs/reference/nginx-docker-networks-cloudflare-cheatsheet.md) | Reverse proxy, volumes, tunnel, troubleshooting |
| Architecture | [ADR Index](https://github.com/adriandelvalle/brewery-app/tree/main/docs/decisions) | Decision records |

---

## Current Environment Status (Verified 2026-06-19)

| Component | Configuration | Status |
| --- | --- | --- |
| **Host** | `jotasrv` (ACEMAGIC Mini PC) | ✅ Active |
| **Kernel** | `6.8.0-117-generic` | ✅ Updated |
| **Static IP (jotasrv)** | `192.168.0.21/24` | ✅ Configured |
| **Static IP (Windows)** | `192.168.0.15` (fixed manually, router lacks DHCP reservation) | ✅ Configured |
| **SSH Access** | `ssh jota@jotasrv` | ✅ Working |
| **Docker** | 29.5.2 (official repo) | ✅ Active |
| **brewery-api** | brewery-app:v0.1, unless-stopped | ✅ Running |
| **brewery-nginx** | brewery-nginx:v0.2, unless-stopped | ✅ Running |
| **brewery-cloudflared** | quick tunnel, unless-stopped | ✅ Running |
| **External access** | via Cloudflare Tunnel (random subdomain) | ✅ Verified from mobile data |
| **Tests** | 14 tests — all passing | ✅ Green |
| **pre-commit** | Active on brewery-app and portfolio | ✅ Active |

---

## Useful Links

- [My GitHub Profile](https://github.com/adriandelvalle)
- [brewery-app Repo](https://github.com/adriandelvalle/brewery-app)
- [OpenCode Documentation](https://opencode.ai)
- [Ollama](https://ollama.com)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [AI Learning Roadmaps](https://github.com/bishwaghimire/ai-learning-roadmaps) — Reference for Phases 4–5

---

> *Last updated: 2026-06-19*
> *Philosophy: Learning-first. 100% free stack. Depth > speed.*
