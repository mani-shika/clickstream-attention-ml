# Clickstream & Attention Intelligence Platform

An end-to-end **Business Clickstream & Attention Intelligence SaaS** that captures granular browser interactions, engineers behavioral features with **Python (OOP), NumPy, and Pandas**, segments users and predicts drop-off risk with **Scikit-learn**, and answers natural-language UX/CRO questions via a **LangChain + FAISS RAG** assistant.

> **Status:** Active Development (Phase 1 — Core OOP Engine, ML Pipeline & FastAPI Backend). See [docs/PRD.md](https://www.google.com/search?q=docs/PRD.md) for the full specification.

---

## Architecture

```text
[ Browser / JS SDK / Chrome Extension ]
    │  Batches 13+ DOM interactions & HTML metadata (every 10s + sendBeacon)
    ▼
[ FastAPI Backend (Render) ] ──► [ SQLite (clickstream.db) ]
    │
    ├──► [ OOP FeatureEngineer (NumPy & Pandas) ]
    │       └── Attention proportion per section, engagement score, friction index
    │
    ├──► [ Scikit-learn ML Pipeline ]
    │       ├── KMeans + StandardScaler (Visitor Persona Segmentation)
    │       └── RandomForestClassifier (Conversion & Drop-Off Prediction)
    │
    └──► [ LangChain + FAISS RAG Analyst (Gemini API) ]
            └── Grounded Q&A over aggregated session & cluster summaries
    │
    ▼
[ React + Vite + Tailwind SaaS Dashboard (Vercel) ]
```

---

## Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Data & OOP Engine** | Python 3.11, Pydantic, NumPy, Pandas, SQLite |
| **Machine Learning** | Scikit-learn (`KMeans`, `RandomForestClassifier`, `StandardScaler`), `joblib` |
| **LLM & RAG** | LangChain, FAISS, Gemini API, structured prompt templates |
| **Backend API** | FastAPI, Uvicorn, Pytest |
| **Frontend Dashboard** | React, Vite, Tailwind CSS, Recharts |
| **Event Collector** | Vanilla JS SDK (`navigator.sendBeacon`), Chrome Extension (Manifest V3) |
| **Deployment** | Vercel (Frontend), Render (Backend) |

---

## Key Capabilities

- **13+ Tracked Interaction Signals:** Captures clicks, double-clicks, right-clicks, hover dwell time, mouse velocity, scroll depth, text selection, copy/paste, form focus, form abandonment, keystroke cadence, tab visibility, and rage clicks—scrubbing sensitive input values at the edge.
- **Visitor Segmentation (`KMeans`):** Groups sessions into behavioral personas (*High-Intent Buyer*, *Deep Reader*, *Friction / Confused*, *Quick Bounce*) evaluated via Silhouette Score.
- **Conversion & Drop-Off Prediction (`RandomForestClassifier`):** Predicts session conversion probability with ROC-AUC, precision/recall, and feature importance rankings per page section.
- **RAG AI Business Analyst:** Indexes aggregated session summaries, cluster profiles, and UX guidelines in FAISS so stakeholders can query drop-off causes in plain English.
- **Built-in Traffic Simulator:** Generates realistic multi-session e-commerce/SaaS clickstream data to test the entire pipeline locally or in live demos.

---

## Planned Repository Structure

```text
clickstream-attention-ml/
├── docs/
│   └── PRD.md
├── backend/
│   ├── app/
│   │   ├── core/          # models.py, database.py, simulator.py, feature_engineer.py
│   │   ├── ml/            # segmentation.py, predictor.py, pipeline.py
│   │   ├── rag/           # analyst.py (LangChain + FAISS + Gemini)
│   │   └── main.py        # FastAPI application
│   ├── tests/
│   │   └── test_pipeline.py
│   └── requirements.txt
├── frontend/              # React + Vite + Tailwind SaaS dashboard
├── tracker/               # JS clickstream SDK & Manifest V3 extension
├── .env.example
├── render.yaml
└── README.md
```

---

## Local Setup (Once Phase 1 Is Pushed)

```bash
# 1. Clone the repository
git clone https://github.com/mani-shika/clickstream-attention-ml.git
cd clickstream-attention-ml/backend

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment variables
cp ../.env.example .env

# 4. Run tests and start API
pytest
uvicorn app.main:app --reload
```
