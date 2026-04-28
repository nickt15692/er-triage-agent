# TriageIQ — ER Multi-Agent Decision Support
## Project 2/3 · BUAN 6v99.s01 · UT Dallas · April 2026

An AI-powered clinical triage assistant built on a 6-agent Claude pipeline. Specialists run in parallel, the synthesizer assigns an ESI 1–5 score, and nurses can override the decision directly in the UI.

> **Always say:** "Decision-support tool — the nurse makes the final call."
> **Never say:** "The AI diagnoses the patient."

---

## What's Running

| Component | What it does | Port |
|---|---|---|
| FastAPI backend | Runs the 6-agent pipeline, serves patient data | 8001 |
| React frontend | Clinical dashboard UI | 3001 |

---

## Setup

```bash
# 1. Add your API key
cd triageiq
cp .env.example .env
# Edit .env: ANTHROPIC_API_KEY=sk-ant-...

# 2. Backend (Terminal 1)
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
uvicorn backend.main:app --reload --port 8001

# 3. Frontend (Terminal 2)
cd frontend
npm install && npm run dev
# Opens at http://localhost:3001
```

**Windows:** Use `start.bat`. **Mac:** Use `start.command` or `start.sh`.

---

## Demo Flow

1. Open `http://localhost:3001`
2. The patient list is sorted by acuity — highest-severity first
3. Select **PT-006** (Marcus J., septic shock) or **PT-001** (65M, chest pain, BP 88/60) for maximum impact
4. Click the patient card — the agent grid activates, watch 4 specialists run in parallel
5. ESI Level appears with confidence score — walk through the action checklist
6. Click each tab: **Summary / Vitals / Symptoms / Protocol / Resources / Ask AI**
7. Try **Nurse Override** to log a manual ESI adjustment with a clinical reason
8. Click **↓ Print** for a clean print-ready report

**Free text mode:** Switch to the "Free Text Input" tab, enter a patient description and optional vitals.

---

## The 6 Agents

| Agent | File | What it does |
|---|---|---|
| Coordinator | `orchestrator.py` | Receives patient, dispatches specialists, calls `generate_triage_report()` |
| Vitals Analyzer | `agents.py` | Calls `calculate_news2_score()` → NEWS2 risk level + per-vital analysis |
| Symptom Classifier | `agents.py` | Chief complaint → red flags → urgency |
| Protocol Matcher | `agents.py` | Calls `search_protocols()` → matched protocol + time-sensitive steps |
| Bed Allocator | `agents.py` | Calls `check_bed_availability()` → care area based on live bed counts |
| Synthesizer | `orchestrator.py` | All findings → ESI 1–5 + action plan + confidence score |

Specialist agents use **Claude Haiku** (fast). The synthesizer uses **Claude Sonnet** (more capable). All four specialists run simultaneously via `ThreadPoolExecutor`.

---

## Demo Patients

| ID | Patient | Presentation | Expected ESI |
|---|---|---|---|
| PT-006 | Marcus J., 58M | Septic shock — BP 82/50, HR 124, GCS 11, SpO₂ 91% | ESI 1 |
| PT-001 | Robert M., 65M | Chest pain — BP 88/60, HR 112, diaphoresis | ESI 2 |
| PT-002 | Sarah K., 34F | Worst headache of life — sudden onset, neck stiffness | ESI 2 |
| PT-004 | Emma L., 8F | Pediatric respiratory distress — barking cough, SpO₂ 93%, Temp 39.8°C | ESI 2 |
| PT-005 | Dorothy W., 72F | Vague weakness, mild confusion, GCS 13 — borderline/ambiguous case | ESI 3 |
| PT-003 | James T., 28M | Ankle injury — mild swelling, no neurovascular compromise | ESI 4 |

The list is sorted by acuity so the most critical cases appear first. Each patient card shows an ESI badge before triage runs.

---

## Frontend Features

- **Live elapsed timer** — ticks up during analysis so the wait feels fast and intentional
- **Per-agent time badges** — each specialist card shows how long it took (e.g. `3.2s`)
- **Vitals sparkline charts** — trending vital signs with clinically meaningful directions (rising HR, declining BP, etc.)
- **AI confidence meter** — color-coded confidence bar in the ESI banner (green ≥80%, yellow ≥60%, red below)
- **Nurse override** — override ESI score inline with a required reason; flows through to the print report
- **Ask AI tab** — context-aware chat for follow-up questions about the triage decision
- **Markdown rendering** — formatted headers, bullets, and bold text in all report tabs
- **Print report** — clean print-ready view including ESI, confidence, override, and all report sections
- **Dark mode** — toggle in the header; persists via CSS custom properties
- **Cancel button** — abort an in-flight triage without refreshing the page

---

## Project Structure

```
triageiq/
├── config.py              ← Model tiers, ESI levels, settings
├── requirements.txt
├── .env                   ← Your API key (not committed)
├── backend/
│   ├── tools.py           ← 5 tool functions + schemas, mock patient data, bed state
│   ├── agents.py          ← Specialist system prompts
│   ├── orchestrator.py    ← Parallel multi-agent coordination + trend generation
│   └── main.py            ← FastAPI: /patients, /triage/stream/{id}, /chat
└── frontend/
    └── src/
        ├── App.jsx        ← Clinical dashboard + all components
        └── App.css        ← IBM Plex design system (light + dark)
```

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/patients` | Returns all demo patients sorted by acuity |
| GET | `/triage/stream/{patient_id}` | SSE stream — real-time agent progress + final report |
| POST | `/triage/freetext` | Free-text triage (JSON body: `description`, optional `vitals`) |
| POST | `/chat` | Context-aware AI chat about a completed triage report |

---

*TriageIQ · Project 2/3 · BUAN 6v99.s01 · UT Dallas · April 2026*
