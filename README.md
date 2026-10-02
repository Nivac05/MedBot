<div align="center">

<br />

# MedBot

### Intelligent insights. Thoughtful care.

A pastel clinical workspace for exploring 90-day deterioration risk, reviewing patient records, and preparing evidence-grounded clinical drafts.

<br />

[![Next.js](https://img.shields.io/badge/Next.js_15-FFFFFF?style=for-the-badge&logo=nextdotjs&logoColor=1f3446)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-EDF5FA?style=for-the-badge&logo=fastapi&logoColor=5D8DB5)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python_3.10+-E7F0F8?style=for-the-badge&logo=python&logoColor=4F7899)](https://www.python.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-EEF3F8?style=for-the-badge&logo=typescript&logoColor=5D8DB5)](https://www.typescriptlang.org/)

<br />

</div>

---

## What MedBot does

MedBot turns a chronic-care dataset into a focused clinical review workspace. It surfaces patient risk, supports cohort filtering, and provides four patient-level clinical tools while keeping the source and processing mode visible.

The repository ships with a small fictional demo dataset, so the complete application runs immediately after cloning. The larger research dataset remains outside Git to prevent patient-style records from being published accidentally.

<table>
  <tr>
    <td width="25%"><strong>61,445</strong><br /><sub>patient records</sub></td>
    <td width="25%"><strong>90 days</strong><br /><sub>prediction window</sub></td>
    <td width="25%"><strong>0.969</strong><br /><sub>reported model AUROC</sub></td>
    <td width="25%"><strong>4 tools</strong><br /><sub>patient-level workflows</sub></td>
  </tr>
</table>

<img width="1900" height="1078" alt="image" src="https://github.com/user-attachments/assets/6436151b-bf0d-4ff6-a38a-9aee607fd5c1" />
<img width="1897" height="1078" alt="image" src="https://github.com/user-attachments/assets/6684701d-7e23-45ce-9f5d-072e028658fa" />
<img width="1915" height="1073" alt="image" src="https://github.com/user-attachments/assets/33c39559-b666-49f9-a367-90184e2f8035" />

The interface includes:

- A responsive powder-blue clinical dashboard with accessible dialogs and minimal motion
- Risk distribution charts, patient search, combined filters, and server-side pagination
- Patient record windows containing conditions, vitals, labs, and stored risk scores
- Clinical summary, SOAP note, anomaly scan, and patient question workflows
- A live system-status window showing whether retrieval, a language model, and validation are active
- Honest offline behavior that extracts source facts without inventing confidence scores or clinical conclusions

> MedBot is a research and decision-support project. It does not diagnose conditions or replace qualified clinical judgment.

## System architecture

```text
Browser
  |
  v
Next.js dashboard (localhost:3000)
  |-- REST resources: /api/patients, /api/summary, /api/status
  |-- Clinical proxy: /api/ai/[task]
  |
  v
FastAPI service (localhost:8000)
  |
  +-- Dataset and stored model outputs
  |
  +-- Retrieval agent
  |     |-- exact patient-ID lookup
  |     `-- ChromaDB semantic retrieval
  |
  +-- Reasoning agent
  |     |-- Gemini when configured
  |     `-- deterministic record extraction offline
  |
  `-- Validation agent
        |-- source and numeric checks
        `-- LLM fact-checking when configured
```

### Clinical request flow

1. The orchestrator selects the requested patient and retrieves patient-specific evidence.
2. Semantic results are restricted to the selected patient. An exact source-record lookup guarantees coverage even when that patient is not in the vector index.
3. With a Gemini key, the reasoning agent prepares a structured draft from the retrieved evidence.
4. Without a key, MedBot enters **offline record mode** and returns deterministic source facts and configured range flags.
5. The response includes its mode, source path, and execution trace for review.

## Current operating modes

| Capability | Without Gemini key | With Gemini key |
|---|---|---|
| Patient dashboard | Available | Available |
| Stored risk scores | Available | Available |
| Exact patient lookup | Available | Available |
| Semantic retrieval | Available after indexing | Available after indexing |
| Clinical summary | Source-fact extraction | Generated draft |
| SOAP note | Objective record template | Generated draft |
| Anomaly scan | Configured numeric ranges | Model-assisted plus checks |
| Patient Q&A | Requested stored facts | Evidence-grounded response |
| Confidence score | Not claimed | Validation result returned |

The Chroma index is generated data and is not committed. Run ingestion locally to index the desired number of patients. The application still covers every loaded patient through exact record lookup.

## Quick start

### Requirements

- Python 3.10 or newer
- Node.js 20 or newer
- npm
- A Google Gemini API key only if you want generated clinical drafts

### 1. Clone and configure

```bash
git clone https://github.com/Nivac05/MedBot.git
cd MedBot
```

Windows:

```powershell
Copy-Item backend\.env.example backend\.env
```

macOS or Linux:

```bash
cp backend/.env.example backend/.env
```

To enable Gemini, edit `backend/.env`:

```env
GOOGLE_API_KEY=your_key_here
LLM_MODEL=gemini-2.0-flash
```

Leave `GOOGLE_API_KEY` empty to use offline record mode.

### 2. Start the API

```bash
cd backend
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 8000
```

macOS or Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 8000
```

API documentation is available at [http://localhost:8000/docs](http://localhost:8000/docs).

### 3. Start the dashboard

Open another terminal:

```bash
cd backend/dashboard
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Use the full research dataset

The included `dashboard_data.example.json` contains fictional records and is loaded automatically. To use the full dataset, place it at the project root as `dashboard_data.json`. That filename is ignored by Git so the data cannot be committed accidentally. Restart both services after changing datasets.

### 4. Build the semantic index

From `backend/`:

```bash
python -m rag.ingest
```

The default command indexes 1,000 patients into five chunks each and loads the bundled guideline documents. Change `max_patients` in the ingestion entry point, or call `run_full_ingestion(max_patients=None)`, to index the complete dataset. Full indexing uses more time and disk space.

## REST API

### Cohort resources

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/patients` | Paginated patients with risk, age, and gender filters |
| `GET` | `/patients/{patient_id}` | One complete patient record |
| `GET` | `/summary` | Risk distribution and cohort statistics |
| `GET` | `/metadata` | Dataset and stored model metadata |
| `GET` | `/analytics/risk-distribution` | Risk and age-group analytics |

### Clinical tools

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/ai/clinical-summary` | Prepare a longitudinal summary |
| `POST` | `/ai/soap-note` | Prepare a structured SOAP draft |
| `POST` | `/ai/anomaly-detection` | Compare recorded values with configured ranges |
| `POST` | `/ai/patient-query` | Ask a patient-specific question |
| `GET` | `/ai/status` | Inspect retrieval, index, and model status |

Example:

```bash
curl -X POST http://localhost:8000/ai/patient-query \
  -H "Content-Type: application/json" \
  -d '{"patient_id":"0","query":"What HbA1c value is recorded?"}'
```

## Project structure

```text
MedBot/
|-- backend/
|   |-- dashboard/              Next.js application
|   |   |-- src/app/api/        REST resources and API proxy
|   |   |-- src/components/     Dashboard, charts, dialogs, and patient tools
|   |   `-- src/lib/            Shared data access, types, and utilities
|   |-- rag/
|   |   |-- agents/             Retrieval, reasoning, validation, orchestration
|   |   |-- config.py           Models, thresholds, and paths
|   |   |-- ingest.py           Patient and guideline indexing
|   |   |-- offline.py          Deterministic no-key behavior
|   |   `-- vector_store.py     ChromaDB access
|   |-- main.py                 FastAPI application
|   `-- test_offline_pipeline.py
|-- model/                      Training, evaluation, and batch prediction code
|-- dashboard_data.example.json Fictional, safe-to-share demo records
`-- README.md
```

## Verification

Backend regression tests:

```bash
cd backend
python -m unittest test_offline_pipeline -v
```

Frontend production build:

```bash
cd backend/dashboard
npm run build
```

The regression suite verifies patient isolation, missing-value handling, deterministic SOAP behavior, question extraction, unknown-patient errors, and numeric contradiction detection.

## Data and model notes

- The public repository uses fictional demo records by default. A local `dashboard_data.json` overrides them when present.
- The full local project snapshot was generated on **7 September 2025** and is intentionally excluded from Git.
- Risk values displayed by the site are stored predictions from that snapshot; the web application does not run the XGBoost model live.
- Training and evaluation scripts live under `model/`. The serialized model is excluded from Git and can be regenerated from the pipeline.
- Semantic retrieval uses `all-MiniLM-L6-v2` embeddings and ChromaDB.
- Clinical reference material bundled with the project is demonstration content. Confirm current clinical guidance before real-world use.

## Responsible use

MedBot handles healthcare-style data and generated text. Before adapting it for real patient information, add authentication, authorization, encryption, audit logging, retention controls, deployment hardening, and a formal clinical validation process. Never expose the development servers directly to the public internet.

---

<div align="center">
  <sub>Designed as a calm, transparent clinical review experience.</sub>
</div>
