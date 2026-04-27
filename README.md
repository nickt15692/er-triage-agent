# TriageIQ — Run Guide
## ER Multi-Agent Decision Support · Project 2/3

---

## What's Running

| Component | What it does | Port |
|---|---|---|
| FastAPI backend | Runs the 5-agent pipeline, serves patient data | 8001 |
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

---

## Demo Flow

1. Open `http://localhost:3001`
2. Select **PT-001** (65M, chest pain, BP 88/60) — designed for maximum impact
3. Click **Run Triage** — agent grid activates, watch 4 specialists run
4. ESI Level 1 appears — walk through action checklist
5. Click each tab: Vitals / Symptoms / Protocol / Resources

**Free text mode:** Switch to the "Free Text Input" tab and type any patient description.

---

## The 6 Agents

| Agent | File | What it does |
|---|---|---|
| Coordinator | `orchestrator.py` | Receives patient, dispatches specialists, calls generate_triage_report() |
| Vitals Analyzer | `agents.py` | Calls calculate_news2_score() → NEWS2 risk level + per-vital analysis |
| Symptom Classifier | `agents.py` | Chief complaint → red flags → urgency |
| Protocol Matcher | `agents.py` | Calls search_protocols() → matched protocol + time-sensitive steps |
| Bed Allocator | `agents.py` | Calls check_bed_availability() → care area based on actual bed counts |
| Synthesizer | `orchestrator.py` | All findings → ESI 1-5 + action plan |

---

## Project Structure

```
triageiq/
├── config.py              ← Model, ESI levels, settings
├── requirements.txt
├── .env                   ← Your API key
├── backend/
│   ├── tools.py           ← 5 tool functions + schemas
│   ├── agents.py          ← 5 system prompts
│   ├── orchestrator.py    ← Multi-agent coordination loop
│   └── main.py            ← FastAPI: /patients + /triage/stream/{id}
└── frontend/
    └── src/
        ├── App.jsx        ← Clinical dashboard
        └── App.css        ← IBM Plex design system
```

---

## Key framing for presentation

**Always say:** "Decision-support tool — the nurse makes the final call."
**Never say:** "The AI diagnoses the patient."

---

*TriageIQ · Project 2/3 · BUAN 6v99.s01 · UT Dallas · April 2026*
