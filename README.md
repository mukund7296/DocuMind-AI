# DocsAI — Chat With Your Documents

A RAG-based document Q&A assistant. Upload PDFs or text files, ask questions, get answers grounded in your documents.

## Quick Start

**Prerequisites:** Docker + Docker Compose, a free [Groq API key](https://console.groq.com)

```bash
# 1. Clone and enter project
git clone https://github.com/mukund7296/DocuMind-AI.git && cd DocuMind-AI

# 2. Set your API key
cp backend/.env.example backend/.env
# Edit backend/.env and set OPENAI_API_KEY=gsk_your_groq_key

# 3. Run
docker compose up --build

# 4. Open http://localhost:3000
```

That's it. No local Python/Node setup needed.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Browser                              │
│              React + Tailwind (port 3000)                │
└────────────────────┬────────────────────────────────────┘
                     │ HTTP (nginx proxy)
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  FastAPI Backend (port 8000)             │
│                                                         │
│   POST /documents/upload    POST /chat/                 │
│          │                       │                      │
│          ▼                       ▼                      │
│   ┌─────────────┐      ┌──────────────────┐            │
│   │  Ingestion  │      │   RAG Pipeline   │            │
│   │  Service    │      │                  │            │
│   │             │      │ 1. Embed query   │            │
│   │ 1. Extract  │      │ 2. Search Chroma │            │
│   │ 2. Chunk    │      │ 3. Build context │            │
│   │ 3. Embed    │      │ 4. Call LLM      │            │
│   │ 4. Store    │      └──────┬───────────┘            │
│   └──────┬──────┘             │                        │
│          │                    │                        │
│          ▼                    ▼                        │
│   ┌─────────────┐      ┌──────────────┐               │
│   │  ChromaDB   │      │  Groq API    │               │
│   │ (local vec) │      │ Llama3-8b    │               │
│   │ all-MiniLM  │      │  (free tier) │               │
│   │ embeddings  │      └──────────────┘               │
│   └─────────────┘                                      │
└─────────────────────────────────────────────────────────┘
```

### RAG Flow

```
User question
     │
     ▼
Embed question ──────────────────────────┐
     │                                   │
     ▼                                   ▼
ChromaDB similarity search          [top-5 chunks]
     │                                   │
     └───────────────────────────────────┘
                      │
                      ▼
           Build context string
           (with source labels)
                      │
                      ▼
           LLM prompt assembly:
           [System] + [History] + [Context + Question]
                      │
                      ▼
                Groq / LLM API
                      │
                      ▼
              Answer + Sources → UI
```

---
---

## Step-by-Step: Run Without Docker (Local Dev Setup)

### Step 1 — Clone the repo

```bash
git clone https://github.com/mukund7296/DocuMind-AI.git
cd DocuMind-AI
```

---

### Step 2 — Set up Python virtual environment (Backend)

```bash
# Go into backend folder
cd backend

# Create virtual environment
python -m venv venv

# Activate it
# On Mac/Linux:
source venv/bin/activate

# On Windows (Command Prompt):
venv\Scripts\activate

# On Windows (PowerShell):
venv\Scripts\Activate.ps1
```

You'll see `(venv)` appear at the start of your terminal line — that means it's active.

---

### Step 3 — Install Python dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

This takes 2–5 minutes the first time (downloads sentence-transformers model etc).

---

### Step 4 — Configure the Groq API key

```bash
# Still inside the parent folder
cp .env.example .env
```


<img width="358" height="64" alt="image" src="https://github.com/user-attachments/assets/ab689f89-c899-4f90-a4ac-785efee7488a" />


Now open `.env` in any text editor (VS Code, Notepad, nano) and edit this line:

```
OPENAI_API_KEY=gsk_your_groq_key_here   ← replace with your real key
OPENAI_BASE_URL=https://api.groq.com/openai/v1
LLM_MODEL=llama3-8b-8192
```

**Where to get the Groq key:**
1. Go to [console.groq.com](https://console.groq.com)
2. Sign up free (Google/GitHub login works)

   <img width="1267" height="443" alt="image" src="https://github.com/user-attachments/assets/f34bafae-9e9e-44a6-bda5-43d84325ba40" />

4. Click **API Keys** → **Create API Key**

   <img width="1032" height="639" alt="image" src="https://github.com/user-attachments/assets/6dbd4fcc-05e4-453b-bd84-b2c2d07d5259" />

6. Copy the key (starts with `gsk_...`)
7. Paste it into your `.env` file

   <img width="1135" height="438" alt="image" src="https://github.com/user-attachments/assets/9a79fcc3-2dac-4a93-a057-f215fdce4911" />


---

### Step 5 — Start the backend

```bash
# Make sure you're in backend/ with (venv) active
uvicorn app.main:app --reload --port 8000
```

You should see:
```
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000
```

Test it works: open [http://localhost:8000/health](http://localhost:8000/health) in your browser — you should see `{"status":"ok"}`.

---

### Step 6 — Set up the Frontend (new terminal window)

Open a **second terminal**, keep the backend running in the first one.

```bash
# Go to the frontend folder
cd DocuMind-AI/frontend

# Install Node dependencies
npm install

# Start the dev server
npm run dev
```

You should see:
```
  ➜  Local:   http://localhost:3000/
```

Open [http://localhost:3000](http://localhost:3000) in your browser — the app is live!

---

### Summary: Two terminals running

| Terminal 1 | Terminal 2 |
|---|---|
| `cd backend` | `cd frontend` |
| `source venv/bin/activate` | `npm install` |
| `uvicorn app.main:app --reload` | `npm run dev` |
| Runs on port **8000** | Runs on port **3000** |

---

### Stopping and resuming later

```bash
# Stop: Ctrl+C in both terminals

# Next time — you DON'T need to reinstall anything, just:
# Terminal 1:
cd backend && source venv/bin/activate && uvicorn app.main:app --reload

# Terminal 2:
cd frontend && npm run dev
```

---

### Common problems

**`python: command not found`** → try `python3` instead of `python`

**`venv\Scripts\Activate.ps1 cannot be loaded`** (Windows PowerShell) → run this first:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**`npm: command not found`** → install Node.js from [nodejs.org](https://nodejs.org) (pick the LTS version)

**Chat gives "invalid api key" error** → your `.env` file isn't saved correctly, double-check the key has no extra spacesbecause they'll pick whatever sounds popular, not what fits the constraints.

The README is written by me. The code comments are mine. The LLM wrote the brackets.

---

## What I'd Do Differently with More Time

1. **Async ingestion** — background task + WebSocket progress for large files
2. **Better chunking** — semantic chunking (split on meaning, not char count)
3. **Hybrid search** — BM25 + vector (keyword + semantic) for better recall
4. **Persistent doc registry** — SQLite so docs survive container restart  
5. **Re-ranking** — a cross-encoder pass after initial retrieval
6. **Eval harness** — a test set of (question, expected_answer) pairs to catch regressions

---

## Running Tests

```bash
cd backend
pip install -r requirements.txt
pip install pytest
pytest tests/ -v
```
