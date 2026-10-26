# Agentic-RAG-for-IT-Support-
# IT Helpdesk Copilot — Agentic RAG with LangGraph

An internal IT support assistant that answers employee questions from company documents first, falls back to web search only when those documents aren't enough, and shows every decision it made along the way.

Built with **LangGraph**, **FastAPI**, **Pinecone**, **Groq**, **Tavily**, and a plain **HTML/CSS/JavaScript** frontend.

---

## Why this exists

IT teams keep a lot of documentation: VPN setup, password and MFA policy, approved software, laptop fixes, service-desk runbooks. Employees still open tickets for questions those documents already answer, because:

- they can't find the right document,
- keyword search returns too much noise,
- a generic chatbot may confidently make things up,
- some answers depend on current vendor information that no internal doc has.

This project tackles that with an assistant that:

- **prefers trusted internal knowledge** over the public web,
- **checks whether retrieved evidence actually answers the question** before using it,
- **searches the web only as a fallback**, and labels those answers clearly,
- **rewrites vague questions and retries** (with a retry limit),
- **admits when it doesn't know** instead of guessing,
- **records its decision path** for transparency and debugging,
- lets authorized staff **add new documents** without touching code.

The example company used throughout is fictional.

---

## How it works

### Request flow

```
Browser (HTML/CSS/JS)
        │  POST /api/chat
        ▼
     FastAPI
        │
        ▼
 LangGraph workflow ──► Pinecone (internal docs)
        │          └──► Tavily (web fallback)
        ▼
   Groq LLM → grounded answer + decision trace
```

### Agent decision graph

```
                Question
                   │
              Route question
             ┌─────┴──────┐
         small talk     IT question
             │             │
       Direct reply   Search internal KB
                           │
                    Grade KB evidence
                   ┌───────┴───────┐
                 good             weak
                   │               │
          Answer from KB     Search the web
                                   │
                          Grade web evidence
                         ┌─────────┼──────────┐
                       good    weak, retries  weak, no
                         │       left         retries left
                         │         │              │
                Answer from web  Rewrite query   "Not enough
                                 → back to KB     evidence"
```

A standard RAG pipeline is *retrieve → generate*. Here, the agent inspects its own state at each step and decides what to do next — which source to trust, whether to retry, or whether to stop.

---

## Tech stack

| Layer | Tool | Role |
|---|---|---|
| Orchestration | LangGraph | Stateful graph with conditional routing |
| LLM | Groq (`openai/gpt-oss-20b` by default) | Routing, grading, rewriting, answering |
| Embeddings | HuggingFace `sentence-transformers/all-MiniLM-L6-v2` | Local, 384-dimensional, no API key needed |
| Vector store | Pinecone | Internal knowledge base |
| Web fallback | Tavily | Current/external information |
| Backend | FastAPI | REST API |
| Frontend | HTML, CSS, JavaScript | Chat UI and document upload |
| Audit log | SQLite | Stores questions and decision paths |
| Packaging | Docker | Reproducible deployment |

---

## Project layout

```
.
├── app/
│   ├── api/routes.py          # /health, /chat, /ingest endpoints
│   ├── core/config.py         # Settings loaded from .env
│   ├── core/logging.py        # Log setup
│   ├── rag/state.py           # Graph state and structured LLM outputs
│   ├── rag/vectorstore.py     # Embeddings + Pinecone connection
│   ├── rag/workflow.py        # The agentic RAG graph
│   ├── services/audit.py      # SQLite audit trail
│   ├── services/ingestion.py  # Load + chunk PDF/TXT/MD/DOCX
│   └── main.py                # FastAPI app
├── data/sample_kb/            # Example IT handbook and runbook
├── static/                    # CSS and JS
├── templates/index.html       # Chat page
├── tests/                     # Unit tests
├── uploads/                   # Uploaded documents
├── ingest_sample_kb.py        # Loads sample docs into Pinecone
├── run.py                     # Starts the server
├── Dockerfile
├── requirements.txt
└── .env.example
```

---

## Getting started

### Prerequisites

- Python **3.11 or 3.12**
- API keys for **Groq**, **Tavily**, and **Pinecone** (all have free tiers)

### 1. Create and activate a virtual environment

macOS / Linux:

```bash
python3 -m venv venv
source venv/bin/activate
```

Windows (PowerShell):

```powershell
python -m venv venv
venv\Scripts\activate
```

Your prompt should now start with `(venv)`.

> Using conda instead? `conda create -n itcopilot python=3.11 -y` then `conda activate itcopilot`.

### 2. Install dependencies

```bash
python -m pip install -r requirements.txt
```

Using `python -m pip` guarantees packages go into the active environment.

### 3. Add your keys

Copy the example file and fill it in:

```bash
cp .env.example .env
```

```dotenv
# Required
GROQ_API_KEY=gsk_...
TAVILY_API_KEY=tvly-...
PINECONE_API_KEY=...

# Optional (defaults shown)
PINECONE_INDEX_NAME=it-helpdesk-kb
PINECONE_NAMESPACE=company-it-docs
GROQ_MODEL=openai/gpt-oss-20b
ADMIN_API_KEY=replace-with-a-long-random-string
APP_ENV=development
```

| Variable | Purpose | Required |
|---|---|---|
| `GROQ_API_KEY` | LLM calls | Yes |
| `TAVILY_API_KEY` | Web search fallback | Yes |
| `PINECONE_API_KEY` | Vector database | Yes |
| `PINECONE_INDEX_NAME` | Index to create/use (dimension 384, cosine) | No |
| `PINECONE_NAMESPACE` | Keeps this project's vectors separate | No |
| `GROQ_MODEL` | Which Groq model to use | No |
| `ADMIN_API_KEY` | Protects the document upload endpoint | No, but change it |
| `APP_ENV` | `development` or `production` | No |

No OpenAI key is needed — embeddings run locally.

**Keep `.env` out of git.** Make sure it's listed in `.gitignore`.

### 4. Load the sample knowledge base

```bash
python ingest_sample_kb.py
```

This reads the files in `data/sample_kb/`, splits them into chunks, embeds them locally, and uploads them to Pinecone. Run it once; running it again adds duplicate vectors unless the script clears the namespace first.

The first run downloads the embedding model (~90 MB).

### 5. Start the app

```bash
python run.py
```

- Chat UI: http://127.0.0.1:8000
- Interactive API docs: http://127.0.0.1:8000/docs

---

## Try it out

| You ask | What should happen | What it shows |
|---|---|---|
| "How do I connect to the VPN from home?" | KB → graded good → answered from internal docs | Internal knowledge is preferred; no web call |
| "Is it okay to give my MFA code to someone from IT?" | KB → good → answer from policy | The model answers with company-specific rules it was never trained on |
| "Is Microsoft Teams having an outage right now?" | KB → weak → web search → good → answer with external-source label | The agent switches sources instead of forcing an answer from irrelevant chunks |
| "My chat app has been weird since the update, help?" | Weak results → query rewritten → retry | A failed search can be improved, with a retry cap to prevent loops |
| "Hi there!" | Routed straight to a direct reply | Not every message needs retrieval |

Each response includes a **trace** of the steps taken, shown in the UI.

---

## Adding documents

Use **Add Company Document** in the sidebar, enter your `ADMIN_API_KEY`, and upload a `.pdf`, `.txt`, `.md`, or `.docx` file.

The backend validates the file type, extracts the text, chunks it, embeds it, and indexes it in Pinecone. It's searchable as soon as the upload finishes.

---

## API reference

### `GET /api/health`

Returns service status.

### `POST /api/chat`

```json
{ "question": "How do I reset my password?" }
```

Example response:

```json
{
  "answer": "…",
  "source_used": "private_kb",
  "trace": [
    "Router → KB",
    "Retrieved 4 chunks from internal KB",
    "KB evidence → GOOD",
    "Answer generated from internal KB"
  ],
  "citations": []
}
```

`source_used` is one of `private_kb`, `web_search`, `direct`, or `insufficient_evidence`.

### `POST /api/ingest`

Multipart file upload. Requires the header `X-Admin-Key: <your ADMIN_API_KEY>`.

---

## Running with Docker

```bash
docker build -t it-helpdesk-copilot .
docker run --env-file .env -p 8000:8000 it-helpdesk-copilot
```

---

## Running tests

```bash
python -m pytest
```

---

## Troubleshooting

**`ModuleNotFoundError` even after installing.** Your editor or notebook is using a different Python. Run `python -c "import sys; print(sys.executable)"` — it should point inside `venv/`. In VS Code, use *Python: Select Interpreter* and choose `venv/bin/python` (or `venv\Scripts\python.exe` on Windows).

**Pinecone dimension mismatch.** The index must be 384-dimensional to match `all-MiniLM-L6-v2`. Delete and recreate it if it was created with a different size.

**Answers keep going to the web.** Check what's in the index. Noisy chunks (navigation menus, boilerplate) make the KB grader return "weak". Clean the source documents and re-ingest.

**Changes to the workflow don't take effect.** Restart the server — the graph is compiled at startup.

---

## Production notes

- Replace `ADMIN_API_KEY` with a strong secret and rotate it.
- Put the app behind your company's SSO before exposing it to employees.
- Review the SQLite audit log for questions that end in "insufficient evidence" — they show which documents are missing.
- Web-sourced answers come from public pages; keep the external-source label visible in the UI.

---

## License and attribution

Licensed under the **Apache License 2.0** — see [`LICENSE`](LICENSE).

This project is adapted from an educational Agentic RAG IT-support example released under Apache 2.0. Modifications include a rewritten README and setup instructions. Per the license, the original copyright and license notices are retained, and modified files are marked as changed.


