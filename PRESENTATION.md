# TriageIQ — PowerPoint Presentation Guide

---

## Slide 1 — Title Slide

**Title:** TriageIQ: AI-Assisted Emergency Triage
**Subtitle:** Clinical Decision Support for Emergency Nurses
**Footer:** BUAN 6v99.s01 · UT Dallas · April 2026

---

## Slide 2 — The Problem

**Headline:** ER Triage Is High-Stakes and Time-Pressured

**Body points:**
- Emergency departments see millions of patients annually — nurses must assign an ESI score within minutes of arrival
- Cognitive overload: nurses simultaneously assess vitals, chief complaint, history, and resource availability under pressure
- Inconsistent triage scoring increases risk: under-triaging delays critical care; over-triaging wastes scarce resources
- Protocol knowledge is vast — no single nurse can recall every guideline in real time
- Consequence of error: delayed intervention for STEMI, stroke, sepsis, or respiratory failure can cost lives

**Speaker note:** Set the stage with urgency. The ESI (Emergency Severity Index) is the 5-level scale used in most US emergency departments to prioritize patients. Getting it wrong has direct patient safety implications.

---

## Slide 3 — The Gap

**Headline:** Existing Tools Don't Go Far Enough

**Body points:**
- EHR systems record data but don't synthesize it into a recommendation
- Protocol lookup tools exist but require the nurse to know what to search for
- No widely adopted tool brings vitals analysis, symptom classification, protocol matching, and resource allocation together in one workflow
- Nurses are left to mentally integrate all inputs under time pressure — alone

**Visual suggestion:** Simple diagram showing a nurse as the single integration point between 4 separate data streams (vitals monitor, EHR, protocol binder, bed board)

---

## Slide 4 — The Solution

**Headline:** TriageIQ — An AI Triage Team in Your Browser

**Body points:**
- TriageIQ is a clinical decision-support tool that simulates a team of specialist AI agents analyzing a patient in parallel
- Input: patient vitals, chief complaint, and medical history (from an EHR or free-text entry)
- Output in seconds: ESI score (1–5), prioritized action checklist, matched clinical protocols, and bed/resource recommendation
- The nurse makes the final call — all recommendations are auditable and overridable with a documented reason

**Tagline on slide:** *"Not replacing the nurse. Backing them up."*

**Speaker note:** Emphasize decision-support framing. This is not autonomous diagnosis — it's a second opinion that arrives before the nurse even finishes the intake form.

---

## Slide 5 — Live Demo Preview (optional)

**Headline:** What the Nurse Sees

**Visual:** Screenshot or mockup of the TriageIQ UI showing:
- Patient selection cards on the left
- The 6-agent progress grid during analysis
- The final ESI banner (e.g., ESI 1 — Immediate, red)
- Tabbed report: Summary / Vitals / Symptoms / Protocol / Resources / Ask AI

**Speaker note:** If doing a live demo, walk through one patient card (e.g., PT-001 STEMI) and show the full pipeline running, then highlight the override panel and the Ask AI chat tab.

---

## Slide 6 — Tech Stack Overview

**Headline:** Built on Modern AI and Web Infrastructure

| Layer | Technology |
|---|---|
| Frontend | React 18 + Vite 5 (JSX, custom CSS) |
| Backend | Python 3.13 · FastAPI · Uvicorn |
| AI Agents | Anthropic Claude API (Tool Use) |
| Specialist Model | Claude Haiku 4.5 — fast, parallel agents |
| Synthesis Model | Claude Sonnet 4.6 — final scoring and chat |
| Streaming | Server-Sent Events (SSE) |
| Concurrency | Python ThreadPoolExecutor |
| Validation | Pydantic v2 |
| Data | In-memory mock (6 patients, bed state) |

**Speaker note:** No external database. The only third-party dependency is the Anthropic Claude API. Everything else is self-contained.

---

## Slide 7 — Two-Tier Model Strategy

**Headline:** Right Model for the Right Job

**Visual suggestion:** Two-column layout

| Haiku (Fast + Cheap) | Sonnet (Powerful) |
|---|---|
| Vitals Analyzer | Final Synthesizer |
| Symptom Classifier | ESI Score Decision |
| Protocol Matcher | Action Plan Generation |
| Bed Allocator | Conversational Chat |
| Runs in parallel | Runs after all 4 complete |

**Body:**
- Haiku runs all four specialist agents simultaneously — optimized for speed
- Sonnet takes the combined specialist findings and produces the authoritative ESI score and structured report
- Result: the accuracy of Sonnet where it matters, with Haiku's speed for parallel groundwork

**Speaker note:** This two-tier routing reduced cost significantly while maintaining quality where it counts — the final recommendation the nurse acts on.

---

## Slide 8 — System Architecture Diagram

**Headline:** Six Agents, One Pipeline

**Visual (build this as a flow diagram):**

```
[Nurse Input]
      |
      v
[FastAPI Backend]
      |
      |-- get_patient_data() --> [Mock Patient Record]
      |-- search_protocols() --> [Protocol Library]
      |
      v
[ThreadPoolExecutor — 4 agents in parallel]
  |           |           |           |
[Vitals]  [Symptom]  [Protocol]  [Bed Alloc]
 Haiku      Haiku      Haiku       Haiku
  |           |           |           |
  +-----+-----+-----------+
              |
              v
       [Synthesizer]
          Sonnet
              |
     generate_triage_report()
              |
              v
    [Structured Report JSON]
              |
              v
      [SSE Stream → React UI]
```

**Speaker note:** The parallel execution step is where the ~75% latency improvement came from. All four specialists analyze simultaneously rather than waiting for each other.

---

## Slide 9 — The Six AI Agents

**Headline:** Each Agent Has a Specialized Role

| Agent | Model | Role |
|---|---|---|
| Vitals Analyzer | Haiku | Scores vital abnormalities 1–5, flags hemodynamic instability |
| Symptom Classifier | Haiku | Identifies red-flag presentations: ACS, stroke, sepsis, respiratory failure |
| Protocol Matcher | Haiku | Maps presentation to evidence-based protocols and ordered interventions |
| Bed Allocator | Haiku | Recommends care area (trauma bay, resus, fast track) and required equipment |
| Synthesizer | Sonnet | Integrates all findings into a final ESI score and action checklist |
| Chat Agent | Haiku | Answers nurse follow-up questions with full report context |

**Speaker note:** Each specialist runs in its own mini agentic loop — it can call tools (like `check_bed_availability`) if needed. The synthesizer receives all four text outputs and has access to `generate_triage_report` as a structured tool call.

---

## Slide 10 — Anthropic Tool Use

**Headline:** Agents Don't Just Talk — They Call Functions

**Body:**
- Claude's **Tool Use** feature lets agents call structured functions mid-conversation
- TriageIQ exposes four tools to the agents:

| Tool | What It Does |
|---|---|
| `get_patient_data` | Fetches full patient record by ID |
| `search_protocols` | Keyword-matches clinical protocols to chief complaint |
| `check_bed_availability` | Returns real-time bed counts by care area |
| `generate_triage_report` | Assembles and returns the structured report dict |

- The Synthesizer *must* call `generate_triage_report` to complete the pipeline — this enforces structured output rather than free-form text
- Agents run in a loop: receive response → dispatch tool call → re-submit result → repeat until done

**Speaker note:** Tool use is what makes the pipeline reliable and structured. Without it, we'd be parsing LLM prose — with it, the final report is a clean JSON object the UI can render deterministically.

---

## Slide 11 — Streaming and Real-Time UX

**Headline:** The Nurse Sees Progress as It Happens

**Body:**
- Backend uses **Server-Sent Events (SSE)** — a persistent HTTP stream from server to browser
- As each specialist agent completes, a `status` event fires with the agent name and elapsed time
- The UI displays a 6-agent grid with live status: idle → active → done (with completion time badge)
- Final `complete` event carries the full report JSON
- No polling, no websockets — SSE is lightweight and browser-native

**Visual suggestion:** Screenshot of the agent status grid mid-analysis showing some agents done, some still active

**Speaker note:** This UX decision matters clinically. A nurse watching progress knows the system is working and can see which analysis finished first — if the synthesizer fires quickly, it means all specialists completed cleanly.

---

## Slide 12 — Iterative Development (6 Sprints)

**Headline:** Built and Improved in Six Documented Iterations

| Sprint | Key Change | Impact |
|---|---|---|
| 1 | Core pipeline, sequential agents | Baseline |
| 2 | Parallel agent execution | ~75% latency reduction |
| 3 | Two-tier model routing (Haiku/Sonnet) | Cost reduction, quality maintained |
| 4 | Pydantic validation + failure guards | Reliability, 3-failure abort |
| 5 | Specialist retry on timeout, thread pool offloading | Stability under load |
| 6 | Live bed decrement, per-agent timing badges | Demo realism, UX polish |

**Speaker note:** Each CHANGES_N.md file documents what was added and why. This iterative approach mirrors real agile development — shipping a working system first, then hardening it.

---

## Slide 13 — Key Engineering Decisions

**Headline:** Design Choices and Their Trade-offs

**Points:**

**Parallel > Sequential agents**
- Trade-off: complexity of thread management vs. latency. Parallel won decisively.

**Two-tier model routing**
- Trade-off: Haiku is fast and cheap but less capable. Used for data gathering; Sonnet used only for synthesis where accuracy is critical.

**SSE over WebSockets**
- Trade-off: SSE is server-to-client only, but that's all we need here. Simpler, browser-native, no library required.

**Tool use for structured output**
- Trade-off: adds round-trip latency vs. prose parsing. Tool use ensures the final report is always machine-readable JSON.

**Mock data only**
- Deliberate for this prototype: removes EHR compliance complexity. Production would replace `MOCK_PATIENTS` with a real FHIR integration.

---

## Slide 14 — Limitations and Future Work

**Headline:** What This Prototype Does Not Do (Yet)

**Current limitations:**
- No real patient data — 6 hardcoded mock records only
- No authentication or role-based access control
- Protocol search is keyword-based, not semantic (no vector DB)
- No audit log or persistence layer
- Not validated against real clinical outcomes

**Future work:**
- FHIR/HL7 integration with real EHR systems
- Vector-based semantic protocol search (ChromaDB or similar)
- HIPAA-compliant deployment with audit logging
- Outcome tracking to validate ESI accuracy over time
- Multi-patient dashboard for charge nurses
- Mobile-responsive UI for bedside tablet use

---

## Slide 15 — Closing

**Headline:** TriageIQ Puts an AI Team Behind Every Nurse

**Key takeaways:**
1. Emergency triage is high-stakes and cognitively demanding — AI can reduce that burden
2. A multi-agent architecture mirrors how real care teams operate: specialists in parallel, attending synthesizing
3. Tool use + structured output makes AI recommendations reliable and renderable
4. Decision-support framing keeps the nurse in command — the AI advises, the nurse decides

**Closing line:**
*"Faster triage. Better decisions. The nurse still leads."*

---

*Presentation guide for TriageIQ · BUAN 6v99.s01 · UT Dallas · April 2026*
