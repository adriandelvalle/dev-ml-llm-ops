My DevOps • MLOps • LLMOps Journey



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
| `devops/` | Infrastructure, automation, CI/CD sandbox exercises |
| `mlops/` | ML model lifecycle, training, deployment sandbox |
| `llmops/` | Local LLMs, RAG, inference sandbox experiments |
| `docs/learning/` | Chronological learning notes (by phase/week) |
| `docs/reference/` | Reusable cheatsheets (Git, Docker, K8s, etc.) |

  
> **Note**: Deep technical documentation inside each folder is in Spanish.

  

---

  

## Tech Stack

  

| Component | Technology |
|:---|:---|
| **Hardware** | ACEMAGIC Mini PC (Ryzen 7 6800H, Radeon 680M, 32GB RAM) |
| **OS** | Linux (Ubuntu) |
| **Remote Access** | VS Code Remote SSH | mRemote SSH
| **Version Control** | Git + GitHub (SSH) |
| **AI Tools** | OpenCode CLI + Ollama + Qwen (local, 100% free) |
| **Backend** | FastAPI (planned), PostgreSQL (planned) |
| **Infrastructure** | Docker, Docker Compose, Kubernetes (planned) |
| **CI/CD** | GitHub Actions, Terraform, Ansible (planned) |
| **MLOps** | MLflow, Great Expectations (planned) |
| **LLMOps** | LangChain, RAG (Chroma/FAISS), Agentes (planned) |
| **Monitoring** | Prometheus + Grafana (planned) |
| **Security** | Trivy, UFW, fail2ban (planned) |

  

---

  

## Learning Progress

  

### Phase 0: Setup ✅ Completed

- [x] Remote server setup via SSH

- [x] Git configuration with GitHub keys

- [x] VS Code Remote SSH workflow

- [x] Portfolio repository structure

  

### Phase 1: Foundations 🔄 In Progress (Week 1/4 completed)

- [x] Linux fundamentals applied: FHS, permissions (700/644/755), processes

- [x] Project scaffold: `brewery-app/` structure with backend/, docs/, scripts/

- [x] Security script: `audit-permissions.sh` for automated permission checks

- [x] Git conventions: Conventional Commits, branch naming, remote sync

- [x] Reference documentation: Git cheatsheet in `docs/reference/`

- [x] OpenCode CLI + Web UI deployment (Week 2)

- [x] FastAPI backend scaffold with /health endpoint (Week 2)

  

### Phase 2: Docker + Kubernetes ⏳ Planned

- [ ] Docker fundamentals: images, containers, Dockerfile, multi-stage builds

- [ ] Docker Compose: multi-service orchestration for local development

- [ ] Kubernetes basics with k3s: pods, deployments, services, ingress

- [ ] Deploy brewery-app backend in k3s with resource limits

  

### Phase 3: CI/CD + IaC ⏳ Planned

- [ ] GitHub Actions: lint, test, build workflows

- [ ] Terraform basics: infrastructure as code for server provisioning

- [ ] Ansible: configuration management for ACEMAGIC setup

- [ ] GitOps concepts with ArgoCD (optional)

  

### Phase 4: MLOps Fundamentals ⏳ Planned

- [ ] Reproducible environments: venv, requirements, Docker for ML

- [ ] Experiment tracking with MLflow

- [ ] Data validation with Great Expectations

- [ ] Model serving with FastAPI + monitoring basics

  

### Phase 5: LLMOps Specialization ⏳ Planned

- [ ] Local LLMs with Ollama: model selection, quantization, performance tuning

- [ ] RAG fundamentals: embeddings, vector stores (Chroma/FAISS), retrieval

- [ ] LangChain/LlamaIndex for orchestration

- [ ] Agentes (AutoGen/CrewAI) for multi-step tasks

- [ ] Evaluation of LLM outputs with DeepEval/Ragas

  

### Phase 6: Integration Project ⏳ Planned

- [ ] brewery-app v1.0: production-like deployment with all components

- [ ] Documentation, testing, security hardening

- [ ] Portfolio polish: README, demo, LinkedIn post

  

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

| Fase | Semana | Tema | Notas |
| :--- | :--- | :--- | :--- |
| 1 | 1 | Linux Fundamentals Applied | [Ver](docs/learning/phase1-week1-linux-fundamentals.md) |
| 1 | 2 | OpenCode + FastAPI Scaffold | [Ver](docs/learning/phase1-week2-opencode-fastapi.md) |
  

### Reference Cheatsheets

| Área | Cheatsheet |
| :--- | :--- |
| Git | [git-cheatsheet.md](docs/reference/git-cheatsheet.md) |

  

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
