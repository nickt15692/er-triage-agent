# TriageIQ — Changes 4: ivin-edit Feature Integration

These changes merge five features from the `triageiq ivin-edit` build into v1 (`triageiq`). The integration preserved all of v1's existing improvements (parallel agents, input validation, animated progress, better error handling) while adding the new UI and backend capabilities on top.

---

## 1. Dark Mode
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

ivin-edit added a dark mode toggle. The feature was implemented using CSS custom properties (`--bg`, `--surface`, `--ink`, etc.) scoped under `[data-theme="dark"]`, so switching themes requires only setting a `data-theme` attribute on `<html>` — no class toggling or JS-driven style overrides.

**Integration:** v1's `App.css` used hardcoded color values (e.g. `#0F172A`, `#F8FAFC`) throughout. These were replaced with the CSS variable system from ivin-edit, which covers both light and dark values. A `darkMode` state and `useEffect` were added to `App` to sync the attribute on toggle. A `dark-toggle` button was added to the header inside a new `header-actions` wrapper, alongside the existing "New Patient" reset button.

---

## 2. Confidence Meter
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

Each triage report now shows an AI confidence percentage in the ESI banner. The `ConfidenceMeter` component reads `report.confidence` (a 0–1 float from the backend) and renders a labeled progress bar. Color codes: green ≥80%, yellow ≥60%, red below.

**Integration:** Added `ConfidenceMeter` as a new component and placed it in the `esi-right` section of the ESI banner inside `TriageReport`, alongside the existing "Nurse makes final call" badge. If the backend does not return a `confidence` field, the component defaults to 85% so it never renders broken.

---

## 3. Nurse Override
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

Nurses can now override the AI's ESI score directly in the UI. Clicking "Override ESI Score" opens an inline panel with 5 ESI buttons and a required free-text reason field. Once confirmed, the override is displayed persistently and flows through to the print report.

**Integration:** Added the `NurseOverride` component below the ESI banner in `TriageReport`. Override state is tracked locally in `TriageReport` via `useState`. The `displayESI` variable (used in the banner and print function) resolves to the override ESI when active, otherwise falls back to `report.esi_score`. No backend changes were needed — this is purely frontend state.

---

## 4. Vitals Trends with Sparklines
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`

The Vitals tab now renders structured vital sign cards with color-coded values and inline sparkline charts when historical trend data is available. Covered vitals: Blood Pressure, Heart Rate, SpO₂, Respiratory Rate, Temperature, GCS.

**Integration:** Added `Sparkline` (an SVG polyline renderer) and `VitalsTrends` components. In `TriageReport`, the Vitals tab now checks for `report.vitals_raw` — if present it renders `VitalsTrends`, otherwise it falls back to the existing plain-text `vitals_detail` display. This keeps the feature gracefully degraded for reports that don't include structured vitals data.

---

## 5. Chat ("Ask AI")
**Files:** `frontend/src/App.jsx`, `frontend/src/App.css`, `backend/main.py`

A new "💬 Ask AI" tab in the report view lets nurses ask follow-up questions about the triage decision. The chat is context-aware — the full triage report is passed to the backend on every message so the AI can reference specific findings, ESI reasoning, and protocol recommendations.

**Integration (frontend):** Added the `ConversationChat` component with scrolling message history, a typing indicator, and Enter-to-send support. A `useRef` was added to the import for auto-scroll. The "chat" tab was added to the `tabs` array in `TriageReport` and rendered at the bottom of the tab switcher.

**Integration (backend):** Added a `POST /chat` endpoint to `backend/main.py`. It accepts `message`, `report`, and `history`, builds a clinical system prompt from the report context, and calls `claude-haiku-4-5` (fast and cheap — appropriate for conversational follow-ups). The endpoint is self-contained and does not affect the existing triage routes.

CORS was also expanded to allow ports 3000 and 5173 in addition to the existing 3001, so the frontend works regardless of which dev server port Vite picks.

---

## 6. Markdown Rendering & Print Report
**Files:** `frontend/src/App.jsx`

Report tab content is now rendered as formatted markdown rather than plain `pre-wrap` text, improving readability of headers, bullet points, and bold text from AI-generated reports. A print function opens a clean print-ready view in a new tab, including the ESI score, AI confidence, nurse override (if applied), and all report sections.

**Integration:** Added `renderMarkdown` and `inlineMarkdown` helper functions. All `detail-text` divs in `TriageReport` were updated to use `renderMarkdown`. A `↓ Print` button was added to the tab bar. The corresponding `.detail-text.markdown` CSS block was added to `App.css`.
