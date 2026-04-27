# TriageIQ — Changes & Fixes 3

## 1. Free Text Animation Out of Sync
**File:** `frontend/src/App.jsx`

**Problem:** The progress animation played three steps with 400ms delays each, then fired the actual POST request to the backend. This meant the animation finished in about 1.2 seconds, then the screen went blank and sat there for up to 60 seconds waiting for the real response. To anyone watching, it looked like the app had frozen.

**Fix:** The POST request now fires immediately when the user clicks Run Triage. The animation runs in parallel with the request — the coordinator step shows right away, the specialists step appears after 2 seconds while the request is already in flight, and the synthesizer step shows briefly as a transition once the response arrives. The animation is now tied to what the system is actually doing.

---

## 2. No Cancel Button During Analysis
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

**Problem:** Once a triage started there was no way to stop it. If the API was slow or something went wrong mid-run, the user was stuck staring at the spinner with no escape except refreshing the entire page — which looks bad in a demo and loses any context.

**Fix:** A small "Cancel" button now appears below the progress message during analysis. Clicking it immediately resets the UI back to the patient selection screen. It's styled subtly so it doesn't distract during a normal run, but it's visible and accessible if needed.

---

## 3. Free Text Endpoint Blocks the Server
**File:** `backend/main.py`

**Problem:** The free text triage endpoint called the AI pipeline directly inside an async function. FastAPI is built on a single-threaded async event loop — when you call slow, blocking code directly in an async function, the entire server freezes until it finishes. This meant no other requests (including health checks or a second user's request) could be handled while a free text triage was running.

**Fix:** The triage function now runs in a thread pool via `run_in_executor`, exactly the same pattern already used by the streaming endpoint. The event loop hands the blocking work off to a background thread and stays free to handle other requests while it runs.
