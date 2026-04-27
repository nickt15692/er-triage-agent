# TriageIQ — Changes & Fixes

## 1. Hardcoded Timestamp
**File:** `backend/tools.py`

Every triage report was stamped with the same fixed date (`2026-04-13T10:00:00Z`) regardless of when it was actually run. In a clinical setting, audit trails require accurate timestamps — you need to be able to prove exactly when a triage decision was made for legal and medical records purposes.

**Fix:** Replaced with `datetime.now(timezone.utc).isoformat()` so each report gets the real current time.

---

## 2. Wrong CORS Port
**File:** `backend/main.py`

The backend was configured to accept requests from `localhost:3000`, but the frontend runs on `localhost:3001` (as documented in the README). This mismatch would block the browser from communicating with the API in certain configurations.

**Fix:** Updated the allowed origin to `localhost:3001` to match the actual frontend port.

---

## 3. Model Routing
**Files:** `config.py`, `backend/orchestrator.py`

Every agent — including trivial ones like checking bed availability — was using Claude Opus, the most powerful and expensive model. Simple specialist tasks don't require that level of capability and using Opus for everything adds unnecessary latency and cost.

**Fix:** Introduced two model tiers. Specialist agents (vitals, symptoms, protocols, beds) now use Claude Haiku (fast, cheap). The final synthesizer — which makes the actual triage decision — uses Claude Sonnet (more capable). Opus is no longer used.

---

## 4. Agent Failure Guards
**File:** `backend/orchestrator.py`

If a specialist agent timed out or crashed, its error message (e.g. `"Specialist timed out."`) was passed directly to the synthesizer as if it were real clinical data. The synthesizer would then try to make a triage decision based on that error string, producing nonsensical results with no warning.

**Fix:** After all specialists complete, the orchestrator checks each result for failure markers. If 3 or more agents fail, the pipeline aborts and returns a clear error instead of proceeding with bad data. Individual failures are labeled clearly so the synthesizer knows data is missing.

---

## 5. Input Validation
**File:** `backend/main.py`

The free-text triage endpoint accepted any input — including empty strings, single characters, or extremely long text — and passed it straight to the AI pipeline. There was no protection against garbage input, and missing vitals caused the AI to silently invent plausible-sounding values.

**Fix:** Replaced the raw `dict` body with a Pydantic model that enforces a minimum length of 10 characters, a maximum of 2000 characters, and rejects blank or whitespace-only input. FastAPI now returns a proper 422 validation error for bad requests before they reach the AI.

---

## 6. Sequential Agents (Parallel Fix)
**File:** `backend/orchestrator.py`

The four specialist agents (vitals, symptoms, protocols, beds) ran one after another despite being completely independent — none of them needed the others' results. The code even had a comment acknowledging this: `"simulated — sequential in demo"`. In an ER context where every second matters, this was a significant practical limitation adding unnecessary latency.

**Fix:** All four specialists now run simultaneously using `ThreadPoolExecutor` with 4 workers. Since they don't depend on each other, they can genuinely run in parallel, reducing the specialist phase from ~4x the single-agent time down to roughly the time of the slowest single agent (~75% faster).
