# TriageIQ — Changes 6: Visual & Demo Polish

These changes improve the visual quality and demo impact of the UI. None of them affect backend logic, agent behavior, or report output — all changes are confined to the frontend and the patient list endpoint.

---

## 1. Live Elapsed Timer
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

**What changed:** A large teal `0:00` counter now displays on the analyzing screen and ticks up every second while the agents are running. It stops and freezes when the report arrives (or clears on cancel/error). The timer is driven by a `setInterval` stored in a `useRef` so it can be reliably cleared from any code path — SSE complete, SSE error, freetext complete, freetext error, and the cancel button.

**Why:** The system typically completes in 20–45 seconds. Without a timer, that wait feels undefined and long. With a visible counter, the same wait feels fast and intentional — and gives the presenter something to point to when making the case for speed.

---

## 2. Per-Agent Completion Time Badge
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

**What changed:** When each specialist agent finishes, its card now shows a small green time badge (e.g. `3.2s`) in the top-right corner. Start times are recorded in a `useRef` map when the `active_agents` SSE event fires (all 4 agents starting in parallel), and the elapsed time for each is calculated when the corresponding `agent_done` event arrives. Time badges only appear for the SSE (patient mode) path where real per-agent timing is available — the freetext animation path does not show times since it uses simulated progress.

**Why:** Showing that agents complete in 3–8 seconds individually — despite running in parallel — makes the architecture claim tangible. The presenter can point to the fact that all 4 finished in roughly the same wall-clock time, which would have taken 3–4× longer sequentially.

---

## 3. Patient List Sorted by Acuity
**Files:** `backend/tools.py`, `backend/main.py`

**What changed:** Added a `PATIENT_ACUITY` dict to `tools.py` mapping each patient ID to their expected ESI level (1–5). The `/patients` endpoint in `main.py` now sorts patients by this value before returning the list, so the most critical cases appear first. Order: PT-006 (ESI 1), PT-001 (ESI 2), PT-002 (ESI 2), PT-004 (ESI 2), PT-005 (ESI 3), PT-003 (ESI 4). The `PATIENT_ACUITY` dict lives separately from `MOCK_PATIENTS` so the acuity hint is never passed to agents.

**Why:** The original list appeared in arbitrary insertion order, burying the most dramatic case (Marcus, septic shock) at the bottom. For a demo, you want to open with the highest-acuity patient — ESI 1 cases are the clearest illustration of why triage speed matters.

---

## 4. ESI Acuity Badge on Patient Cards
**Files:** `backend/main.py`, `frontend/src/App.jsx`, `frontend/src/App.css`

**What changed:** The `/patients` endpoint now includes `acuity_hint` in each patient object. The patient card renders a small colored pill badge (e.g. "ESI 1" in red, "ESI 2" in orange) in the top-right of the card header, using the same color palette as the ESI banner in the report. A new `.patient-card-header` flex row was added to hold the name and badge side by side.

**Why:** Previously the patient list gave no indication of severity before running triage. The acuity badges let an audience immediately see the range of cases available and understand why certain patients need faster attention — before a single click.

---

## 5. Summary Tab Shows Full Summary
**Files:** `frontend/src/App.jsx`

**What changed:** The Summary tab previously only showed Bed Assignment and Protocol Match — two of the four synthesizer outputs. It now shows all four sections in clinical priority order: Vitals Assessment, Symptom Assessment, Protocol Match, and Bed Assignment. The individual detail tabs (Vitals, Symptoms, Protocol, Resources) still show the full specialist agent output; the Summary tab now shows the synthesizer's concise version of all four findings together.

**Why:** The first thing a user sees after triage completes is the Summary tab. Showing only two of four findings made it feel incomplete — someone had to click through multiple tabs to get the full picture. Now the Summary tab lives up to its name.

---

## 6. Tab Label Consistency
**Files:** `frontend/src/App.jsx`

**What changed:** The "Ask AI" tab previously rendered as "💬 Ask AI" with an emoji, while all other tabs used plain text. The emoji was removed and the tab now reads "Ask AI" to match the style of the other tabs. Tab labels are now rendered from a lookup object rather than a conditional expression, which also makes future label changes easier.

**Why:** The emoji was visually inconsistent with the clinical design language of the rest of the UI. Small details like this stand out under a projector.
