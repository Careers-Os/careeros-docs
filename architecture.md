# CareerOS — System Architecture

## Overview

CareerOS is built as a microservices system with three main layers:
1. **Frontend** — Next.js app (careeros-web)
2. **Backend** — Spring Boot services (careeros-api)
3. **AI Layer** — LangGraph agents (careeros-ai)

---

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     User (Browser)                           │
└──────────────────────────┬───────────────────────────────────┘
                           │ HTTPS
┌──────────────────────────▼───────────────────────────────────┐
│                  Next.js Frontend                            │
│           careeros-web (port 3000)                           │
│   TypeScript + Tailwind CSS + shadcn/ui + TanStack Query     │
└──────────────────────────┬───────────────────────────────────┘
                           │ REST / WebSocket
┌──────────────────────────▼───────────────────────────────────┐
│               Spring Cloud Gateway                           │
│           careeros-api gateway (port 8080)                   │
│          JWT Validation + Rate Limiting + Routing            │
└──────┬──────────┬───────────┬──────────┬────────────┬────────┘
       │          │           │          │            │
  ┌────▼───┐ ┌───▼────┐ ┌────▼───┐ ┌────▼───┐ ┌─────▼───┐
  │  User  │ │Resume  │ │Analysis│ │Intervw │ │  Job    │
  │Service │ │Service │ │Service │ │Service │ │Tracker  │
  │ :8081  │ │ :8082  │ │ :8083  │ │ :8084  │ │ :8085   │
  └────┬───┘ └───┬────┘ └────┬───┘ └────┬───┘ └─────┬───┘
       │         │           │          │            │
       └─────────┴─────┬─────┴──────────┴────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
┌───────▼────────┐           ┌────────▼───────┐
│  PostgreSQL 16 │           │     Redis       │
│  (port 5432)   │           │  (port 6379)    │
│  Primary DB    │           │  Cache+Sessions │
└────────────────┘           └────────────────┘

        ┌────────────────────────────────┐
        │        AI Layer                │
        │   careeros-ai (port 8090)      │
        │   FastAPI + LangGraph          │
        │                                │
        │  ┌──────────┐  ┌────────────┐  │
        │  │  OpenAI  │  │   Groq     │  │
        │  │  GPT-4o  │  │  Llama 3   │  │
        │  └──────────┘  └────────────┘  │
        │                                │
        │  ┌──────────┐  ┌────────────┐  │
        │  │  Qdrant  │  │ RabbitMQ   │  │
        │  │ :6333    │  │  :5672     │  │
        │  └──────────┘  └────────────┘  │
        └────────────────────────────────┘

        ┌────────────────────────────────┐
        │        File Storage            │
        │   MinIO (dev) / S3 (prod)      │
        │   Resumes + Documents          │
        └────────────────────────────────┘
```

---

## Service Responsibilities

| Service | Port | Tech | Responsibility |
|---------|------|------|----------------|
| Gateway | 8080 | Spring Cloud Gateway | Routing, JWT validation, rate limiting |
| User Service | 8081 | Spring Boot + PostgreSQL | Auth, profiles, preferences |
| Resume Service | 8082 | Spring Boot + Tika + MinIO | Upload, parse, store resumes |
| Analysis Service | 8083 | Spring Boot + LangGraph | ATS scoring, recruiter simulation |
| Interview Service | 8084 | Spring Boot + LangGraph | Sessions, questions, evaluation |
| Job Tracker | 8085 | Spring Boot + PostgreSQL | Applications, Kanban, reminders |
| AI Service | 8090 | FastAPI + LangGraph | All LLM agent orchestration |

---

## AI Agent Flows

### ResumeAnalysisGraph
```
Upload PDF/DOCX
      ↓
  Apache Tika extraction
      ↓
  Section Parser Agent   → extracts: summary, skills, experience, education, projects
      ↓
  Keyword Extractor      → extracts all technical/domain keywords
      ↓
  JD Matcher Agent       → semantic similarity via Qdrant embeddings
      ↓
  ATS Scorer Agent       → calculates 0-100 score with breakdown
      ↓
  Feedback Generator     → produces actionable improvement suggestions
      ↓
  Structured JSON output → returned to frontend
```

### InterviewGraph
```
User selects: role + company + type + difficulty
      ↓
  Context Loader         → fetches resume + company context from DB
      ↓
  Question Generator     → produces tailored question set (LangGraph node)
      ↓
  [For each question]
  User submits answer
      ↓
  Answer Evaluator       → scores on accuracy, clarity, depth, STAR format
      ↓
  Hint Generator         → optional nudge without revealing answer
      ↓
  [Session complete]
  Report Generator       → overall score + per-question feedback + improvement tips
```

---

## Database Overview

```
users
  └── resumes (one user, many resumes)
  └── interview_sessions (one user, many sessions)
  └── job_applications (one user, many applications)
  └── skill_profiles (one user, one profile)
  └── learning_roadmaps (one user, many roadmaps)

resumes
  └── analysis_results (one resume, many analyses)

interview_sessions
  └── interview_questions (one session, many questions)
  └── interview_answers (one question, one answer)
```

Full schema → [database-schema.md](database-schema.md)

---

## Local Development

All services run via Docker Compose:

```bash
git clone https://github.com/career-os/careeros-api.git
cd careeros-api
docker-compose up -d
```

This starts: PostgreSQL, Redis, RabbitMQ, MinIO, Qdrant

---

## Deployment (Production)

- **Container orchestration**: Kubernetes
- **CI/CD**: GitHub Actions → Docker build → push to registry → K8s rollout
- **Frontend**: Vercel (Next.js)
- **Backend**: AWS EKS or Railway
- **AI Service**: AWS ECS (Python containers)
- **Database**: AWS RDS (PostgreSQL)
- **File storage**: AWS S3
