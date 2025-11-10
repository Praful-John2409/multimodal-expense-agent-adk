# multimodal-expense-agent-adk

Full-stack **multimodal agent** built with Google **ADK**: web frontend, Python backend, RAG pipeline, and a production database (Postgres/AlloyDB with pgvector). The app ingests **images/PDFs/voice** (receipts, bank statements), extracts and grounds facts, and answers questions with citations—end-to-end, deployable on Cloud Run.

> **Submission**: push this repo to GitHub and include a video walkthrough showing code + live demo.

---

## ✨ Capabilities

* **Multimodal intake**: images (receipts), PDFs, text, and microphone capture
* **RAG** over user transactions & uploaded docs with **source citations**
* **Structured DB** (Postgres/AlloyDB) + **pgvector** for embeddings
* **Personal finance skills**: categorize expenses, monthly summaries, anomaly flags
* **ADK graph**: tools for OCR, parsing, vector search, SQL, and summarization
* **Prod-ready**: Docker, health checks, logging, tests, Makefile, IaC stubs

---

## 🗂️ Repository Structure

```
multimodal-expense-agent-adk/
├─ README.md
├─ LICENSE
├─ Makefile
├─ .env.sample
├─ devcontainer.json
├─ scripts/
│  ├─ bootstrap.sh              # install deps, enable APIs
│  ├─ run_local.sh              # local dev
│  ├─ deploy_cloudrun.sh        # container build + deploy
│  ├─ seed_db.sql               # schema + seed data
│  └─ create_pgvector.sql       # pgvector extension
├─ infra/
│  ├─ gcloud/                   # optional gcloud helpers
│  └─ terraform/                # optional IaC (Cloud Run, DB, buckets)
├─ frontend/                    # Next.js (App Router) + Tailwind + shadcn/ui
│  ├─ app/
│  ├─ components/
│  ├─ lib/
│  ├─ public/
│  └─ README.md
├─ backend/                     # FastAPI + ADK graph + tools
│  ├─ adk_graph/                # nodes, tools, policies
│  ├─ apis/                     # REST endpoints
│  ├─ rag/                      # chunking, embeddings, retriever
│  ├─ db/                       # SQLAlchemy models, migrations
│  ├─ tests/
│  ├─ main.py
│  ├─ pyproject.toml
│  └─ README.md
├─ docs/
│  ├─ ARCHITECTURE.md
│  ├─ VIDEOS.md
│  ├─ API.md
│  ├─ DB_SCHEMA.md
│  └─ RAG_PIPELINE.md
└─ out/                         # sample artifacts (sanitized)
```

---

## 🧰 Tech Stack

* **Frontend**: Next.js 14, React 18, Tailwind, shadcn/ui, file & audio upload
* **Backend**: FastAPI, Google **ADK**, Pydantic, SQLAlchemy
* **RAG**: text/image parsers, chunker, **pgvector** embeddings, retriever with citations
* **DB**: Postgres / **AlloyDB** (managed), migration ready
* **Models**: Gemini 2.0/2.5 (vision+text) via ADK tools
* **Deploy**: Docker + Cloud Run, optional Terraform modules

---

## 🔐 Environment Variables

Copy `.env.sample` → `.env` and fill:

```
GOOGLE_CLOUD_PROJECT=your-gcp-project
GOOGLE_CLOUD_REGION=us-central1
GEMINI_MODEL=gemini-2.0-pro-exp   # or 1.5-pro, 2.5-flash (per access)
GEMINI_API_KEY=...

DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=expenses
DB_USER=postgres
DB_PASSWORD=postgres

# Storage (for raw uploads)
BUCKET_NAME=mm-expense-uploads
```

---

## 🚀 Quickstart

```bash
# 1) Clone
git clone https://github.com/<you>/multimodal-expense-agent-adk.git
cd multimodal-expense-agent-adk

# 2) Bootstrap
bash scripts/bootstrap.sh            # installs, enables gcloud APIs (optional)

# 3) Setup DB (local Postgres)
createdb expenses || true
psql -d expenses -f scripts/create_pgvector.sql
psql -d expenses -f scripts/seed_db.sql

# 4) Backend (FastAPI + ADK)
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -e .
uvicorn main:app --reload --port 8001

# 5) Frontend (Next.js)
cd ../frontend
pnpm install
pnpm dev  # http://localhost:3000
```

---

## 🧪 Smoke Test

* Open `http://localhost:3000`
* Upload a **receipt image** → see parsed line-items and total
* Ask: “What did I spend on dining in October?” → RAG answer with citations
* Check backend health: `GET http://localhost:8001/healthz`

---

## 🧬 Architecture (High-Level)

**Frontend**

* File/image/voice capture → `/api/upload`
* Chat UI streams ADK responses (SSE)

**Backend**

* **ADK Graph**:

  1. **Ingest Node** (detect type: image/PDF/text)
  2. **Parse/OCR Tool** (vision model → JSON)
  3. **Normalizer** (schema: `transactions`, `merchants`, `documents`)
  4. **Embed & Upsert** (pgvector, chunk + metadata)
  5. **Retriever** (hybrid keyword + vector)
  6. **Grounded Answer** (Gemini with tool context, return citations)

**Database**

* Tables: `users`, `transactions`, `merchants`, `documents`, `chunks`, `embeddings`
* Views: `v_monthly_spend`, `v_anomalies`

See `docs/ARCHITECTURE.md` and `docs/DB_SCHEMA.md`.

---

## 🗃️ Database Schema (excerpt)

```sql
CREATE TABLE IF NOT EXISTS documents(
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  filename TEXT,
  media_type TEXT,
  uploaded_at TIMESTAMP DEFAULT now(),
  source TEXT
);

CREATE TABLE IF NOT EXISTS transactions(
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  merchant TEXT,
  category TEXT,
  amount NUMERIC(12,2),
  currency TEXT DEFAULT 'USD',
  ts TIMESTAMP,
  doc_id UUID REFERENCES documents(id)
);

-- Vector store
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE IF NOT EXISTS chunks(
  id UUID PRIMARY KEY,
  doc_id UUID REFERENCES documents(id),
  content TEXT,
  metadata JSONB,
  embedding VECTOR(1536)
);
CREATE INDEX ON chunks USING ivfflat (embedding vector_cosine_ops);
```

Full schema: `docs/DB_SCHEMA.md`.

---

## 📚 RAG Pipeline

1. **Parse**: OCR/vision → clean JSON lines (`vendor`, `date`, `amount`)
2. **Normalize**: map to canonical categories (Food, Travel, Utilities…)
3. **Chunk & Embed**: invoice line-items, bank txt → `chunks` + `embedding`
4. **Retrieve**: vector + keyword; filter by user/time/category
5. **Answer**: prompt Gemini with retrieved context → **citations** back to `documents/chunks`

Details & prompts: `docs/RAG_PIPELINE.md`.

---

## 🧩 Key API Endpoints

See `docs/API.md` for full spec.

```
POST /api/upload              # multipart: image/pdf
POST /api/chat                # {message, mode} → SSE stream
GET  /api/transactions        # query params: month, category
GET  /healthz                 # liveness
GET  /readyz                  # readiness
```

---

## 🖥️ Frontend UX

* **Upload Panel**: drag-drop files, preview
* **Chat**: streaming responses, source chips (click to preview doc snippet)
* **Insights**: monthly spend, category charts, anomalies
* **Settings**: DB status, reindex, export CSV

Run scripts and component notes in `frontend/README.md`.

---

## ☁️ Deploy to Cloud Run

```bash
# build images
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/mm-expense-backend ./backend
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/mm-expense-frontend ./frontend

# create Cloud SQL / AlloyDB (optional via Terraform)
# set env vars & deploy
bash scripts/deploy_cloudrun.sh
```

* Use **Serverless VPC Access** or a connector for DB
* Configure **CORS** for frontend ↔ backend
* Set **BUCKET_NAME** for uploads if using GCS

---

## ✅ Grading Checklist

* [ ] Repo builds locally; **frontend** + **backend** run
* [ ] **DB schema + seed** executed; pgvector enabled
* [ ] Upload → parse → DB upsert → embed → retrieve → answer (citations)
* [ ] Chat works with **images** and **text**
* [ ] Clear **RAG** docs, prompts, and evaluation examples
* [ ] Cloud Run deploy for both services (URLs shown)
* [ ] **Video walkthrough** links in `docs/VIDEOS.md`
* [ ] Tests pass (`backend/tests`, basic e2e smoke)

---

## 🧪 Tests

```bash
# Backend unit tests
cd backend && pytest -q

# Minimal e2e: starts API, posts a sample receipt, queries monthly summary
make e2e
```

* Golden answers for fixed inputs in `backend/tests/golden/`
* Deterministic stubs for OCR and embeddings in CI

---

## 🎥 Video Walkthrough (what to show)

1. **Architecture tour** (diagram, ADK graph nodes, DB schema)
2. **Code walkthrough**:

   * ADK tool nodes (OCR, RAG, SQL)
   * Retriver/embedding path
   * FastAPI endpoints
   * Frontend upload & chat components
3. **Run locally**: upload a receipt → show parsed rows → chat summary with citations
4. **DB demo**: `SELECT * FROM transactions` and `chunks`
5. **Cloud Run**: open deployed URLs, repeat a query
6. **Gotchas**: MIME handling, timezones, currency parsing, PII considerations

Put the link(s) in `docs/VIDEOS.md`.

---

## 🔒 Privacy & Safety

* Do not commit raw user receipts or PII; store redacted samples in `out/`
* Enable HTTPS on Cloud Run; restrict public access if needed
* Consider row-level security per `user_id`

---

## 🙏 Credits

* Codelab: *Personal Expense Assistant (Multimodal ADK)*
* Related articles on Gemini 2.5 + ADK multimodal workflows
* Thanks to the ADK community for reference graphs & tools

---

> **Repo name ideas**: `multimodal-expense-agent-adk`, `adk-multimodal-finance-assistant`, or `gemini-expense-agent-e2e`.
