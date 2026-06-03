<div align="center">

<h1>📚 CareerOS — Docs</h1>
<p><strong>Architecture, roadmap, and contribution guides for CareerOS</strong></p>

</div>

---

## 📖 Contents

| Document | Description |
|----------|-------------|
| [vision.md](vision.md) | Why we're building CareerOS and who it's for |
| [architecture.md](architecture.md) | Full system design with diagrams |
| [roadmap.md](roadmap.md) | Phase-by-phase development plan |
| [database-schema.md](database-schema.md) | PostgreSQL schema reference |
| [api-contracts.md](api-contracts.md) | REST API endpoint specifications |
| [ai-agents.md](ai-agents.md) | LangGraph agent design and flows |
| [contributing.md](contributing.md) | How to contribute to any CareerOS repo |

---

## 🏗️ System Overview

```
careeros-web (Next.js)
    ↓ REST
careeros-api (Spring Boot)
    ↓ HTTP
careeros-ai (FastAPI + LangGraph)
    ↓
  Qdrant (vector search)
  PostgreSQL (relational data)
  Redis (cache)
  MinIO / S3 (files)
```

---

## 🗺️ Roadmap Summary

| Phase | Timeline | Focus |
|-------|----------|-------|
| Phase 1 — MVP | Week 1–4 | Auth, Resume Analyzer, ATS Scorer, Job Tracker |
| Phase 2 — AI Core | Week 5–10 | Recruiter Simulator, Interview Coach, Skill Gap |
| Phase 3 — Growth | Week 11–16 | Roadmap Generator, LinkedIn Optimizer, Chrome Extension |

Full roadmap → [roadmap.md](roadmap.md)

---

## 🤝 Contributing

New to CareerOS? Start here:

1. Read [vision.md](vision.md) to understand what we're building
2. Read [architecture.md](architecture.md) to understand the system
3. Pick a repo: [careeros-web](https://github.com/career-os/careeros-web) | [careeros-api](https://github.com/career-os/careeros-api) | [careeros-ai](https://github.com/career-os/careeros-ai)
4. Filter issues by `good-first-issue` and pick one

---

## 📄 License

MIT License
