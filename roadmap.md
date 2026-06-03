# CareerOS — Development Roadmap

## Vision
Build the #1 open-source AI career platform for students and job seekers in India and globally.

---

## Phase 1 — Foundation & MVP (Week 1–4)
**Goal**: Working product with core features. First 100 users.

### Week 1 — Project Setup
- [ ] GitHub org, repos, CI/CD pipeline
- [ ] Docker Compose with PostgreSQL, Redis, RabbitMQ
- [ ] Spring Boot skeleton — gateway + user service
- [ ] Next.js project with auth pages (login/register UI)
- [ ] Flyway migrations: users table

### Week 2 — Auth + Resume Upload
- [ ] JWT authentication (register, login, refresh, logout)
- [ ] User profile API
- [ ] Resume upload endpoint (Spring Boot multipart)
- [ ] MinIO integration for file storage
- [ ] Apache Tika text extraction
- [ ] Basic dashboard UI

### Week 3 — ATS Analyzer
- [ ] LangGraph ResumeAnalysisGraph agent
- [ ] ATS scoring algorithm (keyword, sections, verbs, quantification)
- [ ] Async job processing via RabbitMQ
- [ ] Analysis results API (poll endpoint)
- [ ] ATS score UI — scorecard with breakdown

### Week 4 — Job Tracker
- [ ] Job applications CRUD API
- [ ] Kanban board UI (React Beautiful DnD)
- [ ] Application status updates
- [ ] Basic analytics (response rate, rejection count)
- [ ] Follow-up reminder system

**Phase 1 milestone**: Deploy to Railway/Render. Share on LinkedIn. Target: 100 users, 50 GitHub stars.

---

## Phase 2 — AI Core (Week 5–10)
**Goal**: Ship AI-powered features. Attract power users. 500 users.

### Week 5 — Vector Infrastructure
- [ ] Qdrant setup and Docker integration
- [ ] Embedding pipeline (sentence-transformers)
- [ ] Resume embedding on upload
- [ ] JD embedding on paste
- [ ] Semantic similarity scoring

### Week 6 — Recruiter Simulator
- [ ] RecruiterSimGraph LangGraph agent
- [ ] JD parser + requirement extractor
- [ ] Resume vs JD alignment score
- [ ] Red flag detector (gaps, missing requirements)
- [ ] Shortlist probability UI with reasoning

### Week 7–8 — AI Interview Coach (Part 1)
- [ ] InterviewGraph agent — question generation
- [ ] Session state machine (Spring Boot)
- [ ] Interview session UI — question display
- [ ] Text-based answer submission
- [ ] Real-time answer evaluation

### Week 9 — AI Interview Coach (Part 2)
- [ ] STAR format checker for behavioral answers
- [ ] Hint system (nudge without revealing answer)
- [ ] Session report generation
- [ ] Score history and trends UI

### Week 10 — Skill Gap Analyzer
- [ ] Skill extraction from resume
- [ ] Role-based skill requirements database (500+ roles)
- [ ] Gap matrix calculation
- [ ] Priority matrix (high-impact, low-effort skills first)
- [ ] Resource recommendations per skill

**Phase 2 milestone**: Feature on DEV.to and IndiaHacks communities. Target: 500 users, 300 GitHub stars.

---

## Phase 3 — Growth (Week 11–16)
**Goal**: Complete platform. 2,000 users. Launch Pro tier.

### Week 11 — Learning Roadmap
- [ ] RoadmapGraph LangGraph agent
- [ ] Day-by-day plan generation
- [ ] Progress tracker (checkboxes, completion %)
- [ ] Adaptive adjustments if user falls behind
- [ ] Calendar export

### Week 12–13 — LinkedIn Optimizer
- [ ] LinkedIn profile URL import / manual paste
- [ ] Profile section scorer
- [ ] Headline and About section rewriter
- [ ] Post generator (5 post ideas from user's projects)
- [ ] Recruiter keyword ranking

### Week 14 — Company-Specific Prep
- [ ] Company profiles database (Google, Amazon, Microsoft, Atlassian, Flipkart)
- [ ] Company-specific question sets
- [ ] Culture fit guidance per company
- [ ] Role-specific prep tracks

### Week 15 — Chrome Extension
- [ ] Extension scaffold
- [ ] Save job from LinkedIn/Naukri to Job Tracker in one click
- [ ] Quick ATS check overlay on job pages

### Week 16 — Polish & Launch
- [ ] Mobile responsiveness audit
- [ ] Onboarding flow for new users
- [ ] Product Hunt launch preparation
- [ ] Freemium tier enforcement (Pro tier)
- [ ] Analytics dashboard (user metrics)

**Phase 3 milestone**: Product Hunt launch. Target: 2,000 users, 1,000 GitHub stars, 100 Pro subscribers.

---

## Future (Post Phase 3)
- Voice-based interview practice (Web Speech API)
- Mobile app (React Native)
- Placement cell dashboard for colleges
- Integration with Naukri, LinkedIn Easy Apply
- Resume builder (create from scratch)
- Referral network features
