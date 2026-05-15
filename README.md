# DevMind 🧠

> **An open-source agentic AI system that acts as an intelligent productivity co-pilot for developers.**

DevMind is a multi-agent backend powered by LangGraph that monitors your GitHub, analyzes your codebase, reviews PRs, breaks down issues into actionable steps, and auto-generates documentation — all through a single FastAPI interface.

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-green.svg)](https://fastapi.tiangolo.com)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.1+-purple.svg)](https://langchain-ai.github.io/langgraph/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](docs/contributing.md)

---

## The Problem

Developers context-switch constantly — GitHub, Jira, Slack, docs — just to understand what to work on next. Code reviews are inconsistent. Documentation is always an afterthought. Planning from a vague issue is painful.

**DevMind solves this with four specialized AI agents working in concert.**

---

## Agents

| Agent | What It Does |
|---|---|
| 🔍 **Code Analyst** | Scans your repo for code smells, high complexity, dead code, and anti-patterns. Produces a severity-tagged report. |
| 📋 **PR Reviewer** | Fetches PR diffs from GitHub, runs structured LLM analysis, and posts actionable review comments with a weighted score. |
| 🗂 **Task Planner** | Takes a GitHub issue (even a vague one) and breaks it into ordered, concrete coding steps with file-level context. |
| 📝 **Doc Writer** | Auto-generates or updates README files, module docstrings, and changelogs based on your actual code. |

All agents are orchestrated by a **master Orchestrator Agent** built with LangGraph that plans, delegates, tracks state, and aggregates results.

---

## Architecture

```
Developer / CLI / API
        │
        ▼
 Orchestrator Agent (LangGraph)
        │
   ┌────┼────────────┬──────────────┐
   ▼    ▼            ▼              ▼
Code  PR           Task           Doc
Analyst Reviewer  Planner        Writer
   │    │            │              │
   └────┴────────────┴──────────────┘
                 │
        Memory + Context Layer
        (ChromaDB · Redis · PostgreSQL)
                 │
   ┌─────────────┼──────────────────┐
   ▼             ▼                  ▼
GitHub API    FastAPI            Celery Workers
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Agent orchestration | LangGraph + LangChain |
| Backend API | FastAPI + Pydantic v2 |
| Async task queue | Celery + Redis |
| Relational database | PostgreSQL + SQLAlchemy (async) |
| Vector memory | ChromaDB |
| GitHub integration | GitHub REST API + Webhooks |
| Auth | JWT + OAuth 2.0 |
| Infra | Docker + Docker Compose |
| CI/CD | GitHub Actions |
| LLM | OpenAI / Groq / Ollama (pluggable) |

---

## Project Structure

```
devmind/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                   # Run tests + lint on every PR
│   │   └── lint.yml                 # Standalone code quality gate
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       └── feature_request.md
│
├── devmind/                         # Main application package
│   ├── agents/                      # LangGraph agent definitions
│   │   ├── orchestrator.py          # Master planner & task router
│   │   ├── code_analyst.py          # Code smell & complexity agent
│   │   ├── pr_reviewer.py           # PR diff review agent
│   │   ├── task_planner.py          # Issue-to-steps planner agent
│   │   └── doc_writer.py            # README & docstring generator
│   │
│   ├── api/                         # FastAPI HTTP layer
│   │   ├── main.py                  # App entry point
│   │   ├── dependencies.py          # Auth, DB, shared deps
│   │   └── routers/
│   │       ├── analyze.py           # POST /analyze/repo
│   │       ├── review.py            # POST /review/pr
│   │       ├── plan.py              # POST /plan/issue
│   │       ├── docs.py              # POST /docs/generate
│   │       └── health.py            # GET /health
│   │
│   ├── core/                        # Config, logging, exceptions
│   │   ├── config.py                # Pydantic BaseSettings
│   │   ├── logging.py               # Structured logging
│   │   └── exceptions.py            # Custom exception classes
│   │
│   ├── services/                    # Business logic layer
│   │   ├── github_service.py        # GitHub API interactions
│   │   ├── llm_service.py           # Pluggable LLM provider
│   │   └── embedding_service.py     # Embedding + vector search
│   │
│   ├── memory/                      # Memory & context management
│   │   ├── vector_store.py          # ChromaDB operations
│   │   ├── redis_cache.py           # Redis short-term cache
│   │   └── conversation.py          # Multi-turn history manager
│   │
│   ├── db/                          # Database layer
│   │   ├── models.py                # SQLAlchemy ORM models
│   │   ├── session.py               # Async DB session factory
│   │   └── migrations/              # Alembic migrations
│   │
│   ├── tasks/                       # Celery background jobs
│   │   ├── celery_app.py            # Celery + Redis config
│   │   ├── analyze_tasks.py         # Async code analysis jobs
│   │   └── review_tasks.py          # Async PR review jobs
│   │
│   └── schemas/                     # Pydantic request/response models
│       ├── analyze.py
│       ├── review.py
│       ├── plan.py
│       └── docs.py
│
├── tests/
│   ├── conftest.py                  # Shared pytest fixtures
│   ├── unit/
│   │   ├── test_agents.py
│   │   └── test_services.py
│   └── integration/
│       ├── test_api.py
│       └── test_github.py
│
├── docker/
│   ├── Dockerfile                   # API server image
│   ├── Dockerfile.worker            # Celery worker image
│   └── nginx.conf
│
├── docs/
│   ├── architecture.md
│   ├── agents.md
│   └── contributing.md
│
├── .env.example
├── docker-compose.yml
├── pyproject.toml
└── README.md
```

---

## Quickstart

### Prerequisites

- Python 3.11+
- Docker + Docker Compose
- GitHub Personal Access Token
- OpenAI API key (or Groq / Ollama)

### 1. Clone the repo

```bash
git clone https://github.com/ROSHAN-KHANDAGALE/devmind.git
cd devmind
```

### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env`:

```env
OPENAI_API_KEY={OPENAI_API_KEY}
GITHUB_TOKEN={GITHUB_TOKEN}
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/devmind
REDIS_URL=redis://localhost:6379
CHROMA_HOST=localhost
CHROMA_PORT=8000
JWT_SECRET_KEY=your-secret-key
LLM_PROVIDER=openai
```

### 3. Start the full stack

```bash
docker compose up --build
```

### 4. Run migrations

```bash
docker compose exec api alembic upgrade head
```

### 5. Explore the API

```
Swagger UI:  http://localhost:8000/docs
Health:      http://localhost:8000/health
```

---

## API Usage

### Analyze a repository

```bash
curl -X POST http://localhost:8000/analyze/repo \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"repo_url": "https://github.com/owner/repo", "branch": "main"}'
```

### Review a Pull Request

```bash
curl -X POST http://localhost:8000/review/pr \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"repo": "owner/repo", "pr_number": 42}'
```

### Plan an Issue

```bash
curl -X POST http://localhost:8000/plan/issue \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"repo": "owner/repo", "issue_number": 17}'
```

### Generate Documentation

```bash
curl -X POST http://localhost:8000/docs/generate \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"repo": "owner/repo", "target": "README"}'
```

---

## Local Development

```bash
pip install -r requirements-dev.txt

# Start backing services only
docker compose up db redis chromadb -d

# Run API
uvicorn devmind.api.main:app --reload --port 8000

# Run Celery worker
celery -A devmind.tasks.celery_app worker --loglevel=info

# Tests
pytest tests/ -v --cov=devmind

# Lint
ruff check . && black --check .
```

---

## Roadmap

- [x] Orchestrator agent with LangGraph state machine
- [x] PR Reviewer agent (structured 0–10000 scoring)
- [x] Code Analyst agent (complexity + smell detection)
- [ ] Task Planner agent
- [ ] Doc Writer agent
- [ ] GitHub Webhook listener (auto-trigger on PR open)
- [ ] Slack integration
- [ ] CLI tool (`devmind review --pr 42`)
- [ ] Web dashboard (React)
- [ ] GitLab + Bitbucket support

---

## Contributing

The cleanest contribution is a **new agent** — find a developer pain point, build a LangGraph node, wire it to the orchestrator, and open a PR.

See [docs/contributing.md](docs/contributing.md) for the full guide.

```bash
pytest tests/ -v --cov=devmind
ruff check . && black .
```

Good first issues are tagged `good-first-issue` in the GitHub Issues tab.

---

## Why DevMind?

Most AI coding tools are chat interfaces. DevMind is **infrastructure** — it runs in your CI, responds to webhooks, stores context about your codebase over time, and gets smarter the more you use it. Self-hosted, open, and built to be extended by the community.

---

## License

MIT © [Roshan Khandagale](https://github.com/ROSHAN-KHANDAGALE)

---

<p align="center">Built with 🧠 by <a href="https://github.com/ROSHAN-KHANDAGALE">Roshan Khandagale</a></p>
