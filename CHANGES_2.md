# TriageIQ — Changes & Fixes 2

## 1. Spam-Click Bug
**File:** `frontend/src/App.jsx`

**Problem:** There was nothing stopping a user from clicking "Run Triage" multiple times in quick succession. Each click fired a separate request to the AI pipeline, meaning the same patient could get 3 different ESI scores back at once with no indication of which one to trust. In a clinical context this is directly dangerous.

**Fix:** Added an `isLoading` flag that is set to `true` the moment a triage starts and only cleared when it completes, errors, or is reset. All triage triggers — the patient card click, the free text button, and the button's disabled state — now check this flag before doing anything.

---

## 2. No Request Timeout
**File:** `backend/orchestrator.py`

**Problem:** There was no timeout set on any of the Anthropic API calls. If the API was slow or the network dropped mid-request, the call would hang indefinitely. The UI would show the spinner forever with no way to recover except refreshing the page.

**Fix:** The Anthropic client is now initialized with a 45-second timeout. If any API call exceeds that, it raises an exception which the orchestrator catches and returns as a clean error to the frontend.

---

## 3. Error Messages Being Swallowed
**File:** `frontend/src/App.jsx`

**Problem:** Any failed request in the free text flow just threw a generic `"Triage failed"` error, discarding the actual reason from the server. If a user typed something too short or the backend rejected the input for any reason, they had no way of knowing why.

**Fix:** The frontend now reads the response body on failure and extracts the actual error detail. Pydantic validation errors (e.g. "description too short") are parsed and shown directly. Server errors include the HTTP status code. The user always sees a meaningful message.

---

## 4. Missing Vitals in Free Text Path
**File:** `backend/orchestrator.py`

**Problem:** When a patient was entered via free text, the vitals field was set to an empty object `{}`. The vitals agent received this with no instruction on how to handle it, so it would silently invent plausible-sounding values — potentially producing an ESI score based on made-up data.

**Fix:** The vitals field is now explicitly set to a string telling the agent that vitals were not provided, instructing it not to invent values, and flagging that a full vitals set should be obtained before the ESI score is finalized.

---

## 5. Hardcoded API URL
**Files:** `frontend/src/App.jsx`, `frontend/.env`

**Problem:** The backend URL was hardcoded as `http://localhost:8001` directly in the React source code. If the app were ever demoed from a different machine, deployed to a server, or the backend moved to a different port, it would silently fail to connect with no easy way to change it.

**Fix:** The URL now reads from a `VITE_API_URL` environment variable, with `http://localhost:8001` as a fallback. A `.env` file was added to the frontend directory so the default still works out of the box, but it can be changed without touching source code.

---

## 6. Pydantic Missing from Requirements
**File:** `requirements.txt`

**Problem:** The new input validation added in the previous round of fixes uses Pydantic, but Pydantic was not listed in `requirements.txt`. It was being installed silently as a dependency of FastAPI, meaning its version was unpinned and could change unexpectedly in a fresh install.

**Fix:** Added `pydantic>=2.0.0` explicitly to `requirements.txt` so it is a declared, versioned dependency.

---

## 7. Dead ChromaDB Config
**File:** `config.py`

**Problem:** Three config variables — `EMBEDDING_MODEL`, `CHROMA_PATH`, and `COLLECTION_NAME` — referenced a vector database search feature that was never actually built. They implied the system had real semantic protocol search when it did not, and added confusion about what the system actually does.

**Fix:** Removed all three variables from `config.py`. The keyword-based protocol search in `tools.py` remains unchanged — this just removes the misleading references to infrastructure that doesn't exist.

---

## 8. Free Text Progress Animation
**File:** `frontend/src/App.jsx`

**Problem:** When using free text mode, the UI jumped directly from showing the spinner to showing the results with no intermediate steps. The agent grid sat frozen because free text uses a regular POST request rather than a streaming SSE connection, so no progress events were ever received.

**Fix:** The free text flow now simulates three progress steps — coordinator, specialists, synthesizer — with short delays between them before the POST fires. This keeps the UI consistent between the two input modes and gives the user a sense of what is happening while they wait.
