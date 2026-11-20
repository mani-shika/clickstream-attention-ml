# Product Requirements Document (PRD): Business Clickstream & Attention Intelligence SaaS

## 1. Overview & Business Goal
An end-to-end **Clickstream & Attention Intelligence Platform** that captures granular user interactions on web pages, computes page-section attention and friction metrics using **Python (OOP), NumPy, and Pandas**, segments visitors and predicts drop-off risk using **Scikit-learn**, and enables non-technical business teams to query insights via a **LangChain + FAISS RAG** assistant inside a **SaaS Analytics Dashboard**.

### Core Objectives
- **For Businesses:** Turn raw clickstream noise into actionable conversion funnels, behavioral user segments, and plain-English UX recommendations.
- **For Portfolio / Interviews:** Provide a live, deployable SaaS product with transparent ML evaluation metrics (ROC-AUC, Silhouette Score, Feature Importance) and a 1-click traffic simulator so reviewers can test every feature immediately.

---

## 2. Deployment Architecture

| Layer | Technology | Hosting Platform | Why |
| :--- | :--- | :--- | :--- |
| **SaaS Frontend** | React (Vite) + Tailwind CSS + Recharts | **Vercel** | Instant global CDN, zero cold starts, auto-deploys from `frontend/`. |
| **Backend API & ML** | Python 3.11 + FastAPI + NumPy + Pandas + Scikit-learn | **Render** | Supports heavy Python ML/FAISS libraries without Vercel's 250 MB serverless limit. |
| **RAG & Vector Store** | LangChain + FAISS (local index) + Gemini API | **Render** (bundled in backend) | Zero external DB setup; fast in-memory/disk retrieval for aggregated summaries. |
| **Database** | SQLite (`clickstream.db`) | **Render** (local/disk) | Zero-config relational storage for events, sessions, and model metrics. |
| **Event Collector** | Vanilla JS `<script>` SDK + Manifest V3 Extension | **Vercel / Chrome Web Store** | 1-line embed for external sites or local browser demo. |

---

## 3. Functional Requirements

### 3.1 Event Capture Layer (`tracker/`)
- **13 Tracked Interaction Types:**
  1. `click` (x, y, target element)
  2. `dblclick`
  3. `contextmenu` (right-click)
  4. `hover_dwell` (mouse enter/leave duration in ms)
  5. `mouse_velocity` (pointer movement speed / idle detection)
  6. `scroll_depth` (max % scrolled + scroll velocity)
  7. `text_select` (highlighted text length, not raw PII)
  8. `copy_paste` (copy/paste event flags)
  9. `form_focus` (field focus duration)
  10. `form_abandon` (focused form without submission)
  11. `key_activity` (keystroke cadence/count; never log actual values)
  12. `tab_visibility` (`visibilitychange` / window focus-blur)
  13. `rage_click` (3+ rapid clicks on the same element within 1s)
- **HTML Element Metadata:** Capture `tag`, `css_selector`, `section` (e.g., `hero`, `pricing`, `checkout`, `faq`), and `text_snippet` (truncated to 60 chars).
- **Edge Privacy & Batching:**
  - Strip values from `<input type="password">`, email, and payment fields before transmission.
  - Buffer events in memory and flush every **10 seconds** via `fetch` and on page unload via `navigator.sendBeacon`.

### 3.2 Python OOP Data Engine (`backend/app/core/`)
- **`models.py`:** Pydantic schemas for `ClickstreamEvent`, `EventBatch`, `UserSession`, and `AskRequest`.
- **`database.py`:** SQLite manager with tables for `raw_events`, `sessions`, and `model_runs`.
- **`simulator.py` (`ClickstreamSimulator`):** Generates 500–1,000 realistic multi-session e-commerce/SaaS clickstream records across distinct user behaviors so the dashboard is populated on first load.
- **`feature_engineer.py` (`FeatureEngineer`):** Vectorized **NumPy** & **Pandas** pipeline computing per-session features:
  - `attention_proportion` per page section (`section_dwell_ms / total_session_ms`)
  - `engagement_score` (weighted composite of dwell time, scroll depth, and meaningful clicks)
  - `friction_index` (rage clicks + form abandonments + rapid tab switches)
  - `converted` target label (0 or 1)

### 3.3 ML Fundamentals Pipeline (`backend/app/ml/`)
- **`segmentation.py` (`UserSegmenter`):**
  - Pipeline: `StandardScaler` + `KMeans` (`n_clusters=4`).
  - Evaluates clustering quality with **Silhouette Score**.
  - Assigns human-readable persona labels based on cluster centroids: `High-Intent Buyer`, `Deep Reader`, `Friction / Confused`, `Quick Bounce`.
- **`predictor.py` (`ConversionPredictor`):**
  - Pipeline: `RandomForestClassifier` with stratified `train_test_split` (80/20).
  - Outputs **ROC-AUC**, **Precision**, **Recall**, **F1-Score**, **Confusion Matrix**, and **Feature Importances** (ranking which page sections/interactions drive conversion).
- **Persistence:** Save and load trained models + scalers via `joblib` (`backend/models/`).

### 3.4 LangChain RAG AI Analyst (`backend/app/rag/`)
- **`analyst.py` (`ClickstreamRAGAnalyst`):**
  - Indexes **aggregated** documents into **FAISS** (never raw individual click rows):
    1. Page-section attention & drop-off summaries
    2. ML cluster persona profiles and counts
    3. Random Forest feature importance rankings
    4. Built-in UX/CRO optimization guidelines
  - Uses **LangChain** prompt templates enforcing strict grounding: cite exact numbers from retrieved context and state if data is insufficient rather than guessing.

### 3.5 FastAPI REST API (`backend/app/main.py`)
| Method | Endpoint | Purpose |
| :--- | :--- | :--- |
| `GET` | `/health` | Health check for Render wake-up status |
| `POST` | `/api/ingest` | Receive batched clickstream events from JS SDK / Extension |
| `POST` | `/api/simulate` | Generate synthetic sessions, run FeatureEngineer, and retrain ML + FAISS |
| `GET` | `/api/overview` | Return KPIs, section attention breakdown, and funnel metrics |
| `GET` | `/api/segments` | Return K-Means cluster distribution and persona stats |
| `GET` | `/api/model-metrics` | Return ROC-AUC, precision/recall, silhouette score, and feature importances |
| `POST` | `/api/ask` | Query `ClickstreamRAGAnalyst` with a business question |

### 3.6 SaaS Frontend (`frontend/` — React + Vite + Tailwind CSS)
- **Navigation Layout:** Clean SaaS sidebar with 4 views:
  1. **Overview:** 4 KPI cards (Total Sessions, Conversion Rate, Avg Engagement Score, Friction Alerts), Page-Section Attention Bar Chart, and Conversion Funnel (`recharts`).
  2. **ML & Evaluation:** Cluster Persona breakdown cards, Random Forest ROC-AUC / Precision / Recall / Silhouette Score badges, and Feature Importance horizontal bar chart.
  3. **AI Analyst (RAG):** Chat interface with clickable starter questions (*"Which page section has the highest drop-off risk?"*, *"What separates High-Intent Buyers from Friction users?"*) and cited metric responses.
  4. **Live Tracker & Setup:** Copyable `<script>` embed snippet, Chrome Extension load instructions, and a **"Simulate 500 Sessions"** trigger button.
- **Cold-Start Handling:** Display a clean "Waking up analytics engine..." banner if Render's free tier is spinning up on the first request.

---

## 4. Non-Functional & Code Quality Requirements
- **Natural Code Style:** Write clean, idiomatic Python and TypeScript/React. **Do not** add AI-style narrating comments (`# Step 1:`, `# Initialize...`) or emojis in code, logs, or UI copy.
- **Testing:** Include `pytest` unit tests in `backend/tests/test_pipeline.py` verifying `FeatureEngineer` output shapes, `UserSegmenter` cluster assignments, and `ConversionPredictor` metric bounds.
- **Configuration:** Manage secrets (`GEMINI_API_KEY`, `VITE_API_BASE_URL`, `ALLOWED_ORIGINS`) via `.env` and document them in `.env.example`.

---

## 5. Acceptance Criteria (Definition of Done)
1. Running `pytest` in `backend/` passes all feature-engineering and ML tests.
2. Starting FastAPI locally (`uvicorn app.main:app --reload`) and clicking **"Simulate 500 Sessions"** in the React UI populates all charts, ML metrics, and FAISS index with zero console errors.
3. Asking a question in the **AI Analyst** tab returns a grounded answer citing actual session/cluster numbers from the current dataset.
4. Both `backend/` (Render) and `frontend/` (Vercel) deploy cleanly from the single GitHub repository.
