# TriageIQ — Changes 5: Demo & Presentation Hardening

These changes address six limitations identified during demo preparation. The goal was to make the system more convincing under live scrutiny — more patient variety, visible agent progress, live bed state, resilience against API slowdowns, and more complete free-text input.

---

## 1. Additional Demo Patients
**File:** `backend/tools.py`

**What changed:** Added three new patients to `MOCK_PATIENTS`:

- **PT-004 — Emma L., 8F** — Pediatric respiratory distress (barking cough, stridor, SpO₂ 93%, HR 138, Temp 39.8°C). Likely croup or epiglottitis. Expected ESI 2.
- **PT-005 — Dorothy W., 72F** — Elderly patient with vague presentation (general weakness, mild confusion, borderline vitals, GCS 13). Differential includes sepsis, DKA, or metabolic encephalopathy. Intentionally ambiguous to demonstrate how the system handles borderline ESI 3 cases.
- **PT-006 — Marcus J., 58M** — Septic shock following urinary catheterization (BP 82/50, HR 124, GCS 11, SpO₂ 91%). Expected ESI 1 with immediate Sepsis 3-Hour Bundle activation.

**Why:** The original three patients (chest pain, worst headache, ankle injury) covered only textbook presentations with clean vitals. During a demo or Q&A, any question about pediatric patients, unclear presentations, or sepsis could not be demonstrated live. The three new patients cover these gaps and show the system handles a wider range of acuity levels.

---

## 2. Per-Agent Progress Logging
**File:** `backend/orchestrator.py`

**What changed:** The orchestrator previously logged step 2 ("Running all specialists in parallel...") and then jumped directly to step 6 ("Synthesizing final triage decision..."). Steps 3–5 were never emitted. Now, as each specialist future resolves in `as_completed`, a progress callback fires with that agent's step number and a completion message:

- Step 2 — Vitals Analyzer complete
- Step 3 — Symptom Classifier complete
- Step 4 — Protocol Matcher complete
- Step 5 — Bed Allocator complete

**Why:** The UI's `AgentStatus` component was designed to show each of the four specialist agents completing individually. Without individual callbacks, the UI showed the coordinator agent active, then jumped directly to the synthesizer — making the parallel execution claim invisible. Now the agent grid produces the following visible sequence:

1. **Coordinator** spins alone briefly while patient data is retrieved
2. **All 4 specialists spin simultaneously** — Vitals Analyzer, Symptom Classifier, Protocol Matcher, and Bed Allocator all show active at the same time
3. **Each agent ticks off individually** as it completes, in whatever order the API returns them — the checkmark and done state appear as each future resolves
4. **Synthesizer** spins alone while the final ESI score and report are generated

This sequence makes the parallel architecture visible and self-explanatory to anyone watching the demo.

---

## 3. Live Bed Availability Decrement
**File:** `backend/tools.py`

**What changed:** Added a `_decrement_bed()` helper that parses the `bed_recommendation` string for known care area names (`trauma_bay`, `resus`, `fast_track`, `general`, `waiting`) and subtracts one from `MOCK_BEDS[area]["available"]` when a match is found. This helper is called at the end of `generate_triage_report()` each time a report is finalized.

**Why:** Previously, bed availability was hardcoded and never changed. Running the same patient twice (or running multiple patients) always returned identical bed counts. This undermined the credibility of the bed allocation feature during a demo. With the decrement, each triage run reflects prior assignments — trauma bay goes from 1 available to 0 after the first ESI 1 patient, which is a realistic and demonstrable behavior.

---

## 4. Specialist Retry on Timeout
**File:** `backend/orchestrator.py`

**What changed:** `run_specialist()` now accepts a `max_retries` parameter (default `1`). The inner agentic loop is now wrapped in an outer retry loop. If all 8 turns are exhausted without an `end_turn` response, the function retries once from a fresh message state before returning the timeout error string.

**Why:** A single API slowdown during a live demo could silently mark a specialist as unavailable, producing a degraded report. The orchestrator only aborts if 3 or more specialists fail, but even one failed agent looks bad in front of an audience. One automatic retry on timeout catches transient latency spikes without requiring the presenter to restart the triage.

---

## 5. Free-Text Vitals Input
**Files:** `backend/main.py`, `backend/orchestrator.py`, `frontend/src/App.jsx`, `frontend/src/App.css`

**What changed:** Free-text mode now has two separate labeled input fields — "Patient Description" and "Vitals (optional)". When vitals are provided, they are sent in the POST body as a `vitals` field alongside `description`. The backend passes them through to the orchestrator, which uses them as the `vitals` value in the patient data dict instead of the "NOT PROVIDED" placeholder.

Specifically:
- `FreeTextRequest` in `main.py` gained an optional `vitals: Optional[str]` field (max 500 chars)
- `run_triage_agent()` in `orchestrator.py` gained a `vitals_str: str = None` parameter
- The freetext POST endpoint passes `body.vitals` as the third argument to `run_triage_agent`
- `App.jsx` gained `freeTextVitals` state, a second textarea, and includes the value in the POST body
- `App.css` gained `.freetext-label` and `.freetext-label-optional` styles for the field labels

**Why:** Without vitals, agents were instructed not to invent values and to flag the ESI score as incomplete. This made every free-text triage look like a partial result. For a demo where someone types a patient description and wants to see a full report, the missing vitals were an obvious gap. The field is optional so it does not break existing free-text behavior.

---

## 6. Protocol Search Description Rename
**File:** `backend/tools.py`

**What changed:** The `search_protocols` tool description was changed from `"Search the clinical protocol knowledge base..."` to `"Search the clinical protocol library..."`.

**Why:** "Knowledge base" implies a vector database or semantic search infrastructure. The actual implementation is keyword scoring against four hardcoded protocols. The previous wording was a remnant of the ChromaDB config that was removed in Changes 2. "Protocol library" is accurate without overstating the capability — important if the tool schema is shown to a technical audience.

---

## 7. Chief Complaint Truncation Fix
**File:** `backend/main.py`

**What changed:** The `/patients` list endpoint was appending `"..."` to every chief complaint regardless of length (`data["chief_complaint"][:80] + "..."`). Changed to only append when the complaint exceeds 80 characters.

**Why:** With the new patients added in change 1, some complaints are under 80 characters and were displaying with a spurious trailing ellipsis in the patient selection list. Minor cosmetic fix.
