Here is an **updated, cleaner, more polished README** tailored to *your actual repo structure*, your *demo video*, and the *expense_manager_agent* layout you showed in the screenshot.

It is **shorter, clearer, and more focused** while still looking like a professional ADK project README.

---

# 🧾 Expense Manager Agent (Google ADK)

A **multimodal personal finance agent** built with **Google ADK**.
It ingests **images, PDFs, and text**, extracts structured expense data, stores it, and answers natural-language financial queries using **RAG** with citations.

📺 **Demo Video:** [https://youtu.be/7CjIFm7-YNE](https://youtu.be/7CjIFm7-YNE)

---

## ✨ Features

* **OCR + Parsing** for receipts, invoices, bank statements
* **Schema-grounded extraction** (Pydantic schema + post-processing)
* **RAG pipeline** for answering finance questions with citations
* **Persistent storage** of parsed expenses (SQLite/Postgres)
* **Modular ADK agent** with tools, callbacks, and a task prompt
* **Frontend CLI/Web interface** for quick testing
* **Containerized deployment** via Docker & supervisord

---

## 📦 Repository Structure

```
expense_manager_agent/
├── agent.py                 # Core ADK agent
├── callbacks.py             # Tool callbacks / logging hooks
├── tools.py                 # OCR, parsing, RAG, DB tools
├── schema.py                # Pydantic schemas for expenses & documents
├── backend.py               # API service exposing agent endpoints
├── frontend.py              # Simple UI / CLI interface
├── logger.py                # Structured logging
├── utils.py                 # Helpers
├── settings.py              # Runtime config
├── settings.yaml            # Default configuration
├── Dockerfile               # Containerization
├── supervisord.conf         # Process manager (backend + agent)
├── task_prompt.md           # Agent system prompt & tool descriptions
├── README.md                # ← You are here
└── LICENSE
```

---

## 🧰 Tech Stack

* **Google ADK** (agents, graphs, tools)
* **Python 3.10+**, FastAPI-style backend
* **Gemini 2.x models** for OCR + reasoning
* **Pydantic** for structured extraction
* **SQLite/Postgres** for persistent expense storage
* **Docker + Supervisord** for deployment

---

## 🚀 Quickstart

### 1️⃣ Install Dependencies

```bash
pip install -r requirements.txt
```

### 2️⃣ Run the Agent Backend

```bash
python backend.py
```

### 3️⃣ Run the Frontend/CLI

```bash
python frontend.py
```

### 4️⃣ Upload a Receipt

Try:

* an image
* a screenshot
* a PDF

Then ask questions like:

> “What did I spend on dining last month?”
> “Summarize my expenses by category.”
> “Find anomalies in November.”

---

## 🧬 How It Works

### 🔹 1. Ingest

User uploads an image/PDF → routed into the ADK graph.

### 🔹 2. Parse

OCR + multimodal Gemini model extract:

* vendor
* date
* amount
* line items
* category (predicted or inferred)

### 🔹 3. Normalize

Mapped to canonical schema (`schema.py`).

### 🔹 4. Store

Inserted into persistent storage.

### 🔹 5. Retrieval

RAG retrieves relevant entries using:

* keyword search
* semantic retrieval (embeddings)

### 🔹 6. Grounded Answering

Gemini answers with **citations** to parsed data.

---

## 🖥️ Demo (Video Walkthrough)

The demo covers:

1. Architecture overview
2. Code structure (tools, callbacks, agent graph)
3. Uploading receipts
4. Extracted structured JSON
5. Asking financial questions
6. Grounded answers with citations

📺 **Watch the Demo:** [https://youtu.be/7CjIFm7-YNE](https://youtu.be/7CjIFm7-YNE)

---

## 🛠️ Configuration

Copy the example:

```bash
cp settings.yaml.example settings.yaml
```

Edit keys:

```yaml
gemini_model: "gemini-2.0-pro-exp"
database_url: "sqlite:///expenses.db"
enable_embeddings: true
```

---

## 🐳 Docker Deployment

Build & run:

```bash
docker build -t expense-agent .
docker run -p 8000:8000 expense-agent
```

Supervisord launches:

* agent service
* backend API

---

## 📑 API Endpoints (Backend)

```
POST /ingest       # Upload + parse
POST /chat         # Ask a question
GET  /expenses     # List stored expenses
GET  /healthz      # Health check
```

---

## 📌 Roadmap

* [ ] Add voice → text ingestion
* [ ] Fine-grained category prediction
* [ ] Export as CSV / PDF
* [ ] Monthly/weekly insights dashboards
* [ ] Cloud Run deployment with managed DB

---

## 🙏 Credits

Built using **Google ADK** multimodal agent patterns and Gemini 2.x capabilities.

---

If you want, I can also:

✅ Add screenshots
✅ Add architecture diagrams
✅ Add a badge section (Python version, license, etc.)
✅ Format this markdown perfectly for GitHub
✅ Tailor it exactly to your professor or submission requirements

Would you like a **more compact**, **more aesthetic**, or **more academic** version?
