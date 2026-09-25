# Intent Friction Analysis — Framework Design & Agentic Handoff
### Brainstorming session record

**Date:** 24 September 2026
**Participant:** bharath.dasari@one.verizon.com
**Source input:** `intent x friction matrix.md` (Intent × Friction matrix + Travel Pass friction analysis executive summary)
**Companion artifact:** `friction-analysis-architecture.svg` — end-to-end architecture diagram

---

## 0. Session objective

Two goals, pursued in parallel:

1. **Define a repeatable analysis framework** for friction analysis across the Intent × Friction matrix — so that every intent produces a comparable, auditable artifact instead of a bespoke deck.
2. **Identify the human / agent handoff line** — what can be safely automated, and what must stay with an analyst.

Decisions taken during the session:

| Question | Decision |
|---|---|
| Framework first, or agent first? | **Both in parallel**, with a visual architecture as the anchor |
| How much to automate on day one? | **Everything up to and including hypothesis generation** |
| Is ConvoIQ transcript data available? | **Yes — covers these intents** (changes the design materially) |

---

## 1. Starting point — what already exists

### 1.1 The Intent × Friction matrix
Identified friction areas across domains. **International** used as the worked example.

### 1.2 Top International intents by call volume

| Intent | Monthly calls | Notes |
|---|---:|---|
| Travel Pass Issue | 30,000 | Largest — likely post-success contact |
| **Add Travel Pass** | **21,000** | Primary deflection target in the original analysis |
| Travel Pass Inquiry | 16,000 | |
| Roaming Charges | 13,000 | Post-action surprise pattern |

### 1.3 The Travel Pass analysis (already completed, 5 phases)

**Goal:** understand why users intending to activate Travel Pass hit friction in digital channels and call instead. Scope limited to digital journeys; bill shock and eligibility excluded due to telemetry constraints.

**Phases 1–2 — Data mapping and journey definition**
- Digital session metrics: `cwlsdo-0`
- Intent signals: `vzdsdw-0`
- Conversation records: `cdwldo-0`
- Field corrections discovered during validation: `page_name` → `page_nm`; use `omni_hit_join_vcg`
- Standard journey flow established:
  ```
  international_dashboard
    → itp_destinationselection
    → itp_traveldates
    → itp_deviceandplansselection
    → itp_planselection
    → itp_effectivedate
    → itp_skipnewplan
    → itp_confirmation
  ```

**Phase 3 — Impact measurement**
- Call volume baseline established (see table above)
- **Major funnel drop:** `itp_deviceandplansselection` → `itp_planselection`, falling 121K → 55K = **55% loss**
- L3/L4 breakdowns evaluated for roaming fees, error types, virtual assistant gaps, user cohorts

**Phase 4 — Segmentation**
- Caller behaviours, repeat contact trends, plan distribution

**Phase 5 — Additional angles + gap closure**
1. **Plan changes prior to session** — narrowed to 30 days prior on stakeholder feedback (Prashi Mittal). Trend moved 72% → 45% (June–August), indicating automated account updates may still skew the metric.
2. **Three-month longitudinal (June–August)** — sessions +14%, unique users −25%, confirming a persistent 50–55% drop-off. Mid-summer CIR fluctuations attributed to data coverage limits, not process change.
3. **Historical international usage** — scoped to trailing 12 months before interaction.
4. **Account tenure** — 74% of intent from accounts with 3+ years tenure; newer users complete self-service at **half the rate (4% vs 11%)**.

**Status:** all five phases complete and aligned with stakeholder input.

---

## 2. Key insight — the analysis is already a pipeline

Stripped of Travel Pass specifics, the completed work follows a fixed sequence:

```
Intent selection → Data binding → Journey spec → Funnel quant →
Call linkage → Friction hypothesis → Cohort/segment proof →
Root cause → Recommendation → SCR narrative
```

- Everything **before** "Friction hypothesis" is **mechanical and parameterizable**.
- Everything **after** is **judgment**.

**That line is the natural human / agent handoff.**

---

## 3. The framework: FRICTION-7

The 7-step problem-solving method, instantiated for the friction domain so every intent yields the same artifact shape.

| # | 7-step PS | Friction-specific instantiation | Output artifact |
|---|---|---|---|
| 1 | Define problem | "Intent X is digitally self-servable, yet drives N calls/month" | Problem statement + deflectable volume estimate |
| 2 | Break down | Decompose intent into journey stages + friction surfaces | **Journey Spec** (page/event sequence) |
| 3 | Prioritize | Rank stages by `drop-off volume × call correlation × fixability` | Friction heatmap |
| 4 | Workplan | Data sources, tables, joins, filters | **Data Binding Manifest** |
| 5 | Analyze | Funnel, error/L3-L4, repeat contact, cohort, longitudinal | Evidence pack |
| 6 | Synthesize | Root cause classified into friction taxonomy | Root cause statement + confidence |
| 7 | Recommend | Fix, owner, effort, deflection estimate | SCR one-pager |

### SCR wrapper (exec communication layer)

Worked example, Travel Pass:

- **S** — Travel Pass activation is fully self-servable; digital sessions growing 14% YoY
- **C** — 55% abandon at plan selection → 21K calls/month; 74% from tenured accounts; new users complete at half the rate
- **R** — Three fixes, ranked by deflectable calls per engineering week

---

## 4. The missing reusable layer

> Travel Pass knowledge currently lives in a document. To scale across the matrix, what is needed is **four config assets**, not more analyses.

### 4.1 Intent Registry
One row per cell in the Intent × Friction matrix. This is the work-queue and the status tracker.

```yaml
intent_id: itp_add_travel_pass
domain: International
call_volume_monthly: 21000
self_servable: true
journey_spec_ref: journeys/travel_pass.yaml
status: analyzed | in_progress | backlog
last_run: 2026-09-24
deflection_target: 21000
deflection_actual: null      # filled by the feedback loop post-fix
```

### 4.2 Journey Spec — **highest-value artifact**
The page/event DAG per intent. Once this exists, funnel construction, drop-off calculation and call linkage become 100% automatable.

```yaml
intent_id: itp_add_travel_pass
entry_points: [international_dashboard]
steps:
  - id: itp_destinationselection
    required: true
  - id: itp_traveldates
    required: true
  - id: itp_deviceandplansselection
    required: true
  - id: itp_planselection
    required: true
    known_friction: [F2, F6]     # price/plan reveal
  - id: itp_effectivedate
    required: true
  - id: itp_skipnewplan
    required: false              # branch, NOT a drop
  - id: itp_confirmation
    required: true
    is_success: true
attribution:
  call_window_days: 7            # locked centrally, not per-analysis
  call_intents: [add_travel_pass, travel_pass_issue]
```

**Why `required: false` matters:** Travel Pass has `itp_skipnewplan` as an optional branch. A naive funnel reads it as a drop-off and manufactures a phantom friction point. The flag prevents an entire class of false findings.

### 4.3 Data Binding Manifest
Encodes hard-won schema knowledge **once**, so no analyst re-discovers it.

| Concern | Binding |
|---|---|
| Digital session metrics | `cwlsdo-0` |
| Intent signals | `vzdsdw-0` |
| Conversation records | `cdwldo-0` |
| Page field | `page_nm` (**not** `page_name`) |
| Session join | `omni_hit_join_vcg` |

Currently tribal knowledge. Codifying it is one of the cheapest, highest-return moves available.

### 4.4 Friction Taxonomy — closed vocabulary
Makes findings comparable across intents, and gives the agent a scoring target instead of free-association.

| Code | Friction type | Telemetry signature | Transcript signature |
|---|---|---|---|
| F1 | Discovery | intent volume ≫ journey entries | "couldn't find where to…" |
| F2 | Comprehension / choice overload | mid-funnel drop, long dwell, back-navigation | "didn't know which one to pick" |
| F3 | Eligibility / validation block | hard stop at a gate, low retry | "it said I'm not eligible" |
| F4 | Technical | error events, timeouts, repeat attempts | "the page wouldn't load" |
| F5 | Auth / identity | drop at login / OTP | "couldn't log in" |
| F6 | Trust / price anxiety | drop at price-reveal, high dwell, **no error** | "wanted to confirm the charge" |
| F7 | Confirmation gap | **completes digitally, still calls** | "did my order go through?" |
| F8 | Post-action surprise | call 2–30 days after success | "why was I charged this" |

---

## 5. Critical finding — F7 / F8 are structurally invisible to a funnel

The Travel Pass 55% drop at `deviceandplansselection → planselection` is most likely **F2 + F6**.

But look at the surrounding volumes:

| Intent | Calls | Likely code | Fix class |
|---|---:|---|---|
| Travel Pass Issue | 30,000 | F7 / F8 | Proactive comms, notifications |
| Roaming Charges | 13,000 | F8 | Billing transparency, alerts |
| **Combined** | **43,000** | | **Not a UI fix at all** |
| Add Travel Pass | 21,000 | F2 / F6 | UI / journey fix |

**43K in post-success contact volume exceeds the 21K originally targeted.** A funnel-only framework cannot see these, because the customer *succeeded* digitally and called anyway.

**Design implication:** F7/F8 detection must be a **first-class subagent query**, not an afterthought. The Call Linker must run a "completed digitally, contacted anyway" cohort as standard, alongside the abandonment cohort.

---

## 6. Automation triage

| Pipeline step | Automatable? | Rationale |
|---|---|---|
| Data binding / schema resolution | **Full** | Deterministic once the manifest exists |
| Funnel construction + drop-off calc | **Full** | Pure SQL generated from the journey spec |
| Call volume by intent / L3-L4 breakdown | **Full** | Templated SQL |
| Digital → call linkage (session → contact within N days) | **Full** | Templated, single locked attribution rule |
| Repeat contact, tenure, plan-change, longitudinal | **Full** | Phases 4 and 5 are all reusable templates |
| Anomaly flagging ("drop > 30% at any step") | **Full** | Thresholded |
| Transcript retrieval + theming (ConvoIQ) | **Full** | Scoped to intent + drop step |
| Evidence pack assembly + charts | **Full** | |
| **Friction hypothesis generation** | **Assisted** | Agent proposes ranked candidates from taxonomy + evidence; human selects |
| **Root cause confirmation** | **Human** | Requires product / UX / roadmap context the data cannot see |
| **Fix recommendation + effort / owner** | **Assisted** | Agent drafts, human owns |
| **Data-validity judgment** | **Human** | Exactly where two false signals were already caught (CIR coverage; automated account updates) |
| SCR narrative drafting | **Full** (human edits) | Template is fixed |

> **Rule of thumb:** automate everything up to and including *"here are the top 3 statistically-significant friction points with evidence."* Humans own *why* and *what to build*.

---

## 7. Proposed agentic architecture

Orchestrator plus focused subagents, reusing existing workspace skills (`vz-bigquery-execute`, `convoiq-*`, `project-overview-generator`).

```
Friction Analysis Orchestrator
├─ Journey Mapper        → discovers/validates page sequence from telemetry; flags spec drift
├─ Funnel Quantifier     → step counts, drop-off %, MoM trend, anomaly flags
├─ Call Linker           → intent→call volume, L3/L4, session→contact join, F7/F8 cohort
├─ Voice Analyzer        → ConvoIQ transcripts scoped to that intent + drop step
├─ Cohort Segmenter      → tenure, repeat contact, plan mix, prior usage, longitudinal
└─ SCR Narrator          → assembles evidence pack → exec one-pager
```

### 7.1 Critical design choice — Voice Analyzer
**Telemetry tells you *where* people drop; transcripts tell you *why*.**

The Travel Pass analysis was telemetry-only. Because ConvoIQ covers these intents, the agent can attach verbatim evidence directly to the drop step. This collapses what was weeks of hypothesis work into a retrieval step, and turns hypothesis ranking from guesswork into evidence-weighted scoring.

### 7.2 Trigger modes

| Mode | Behaviour | Value |
|---|---|---|
| **On-demand** | "run friction analysis for intent `add_travel_pass`" | Analyst acceleration |
| **Scheduled monitor** | Weekly re-run of analyzed intents; alert on funnel step degradation > X% or call volume spike | Converts one-time analysis into a standing early-warning system |
| **Batch backlog burn-down** | Agent works the Intent Registry, producing draft evidence packs for review | Scales coverage across the whole matrix |

### 7.3 The evidence pack (the handoff artifact)

Contents:
- Funnel chart + drop-off table
- Call linkage + deflectable volume estimate
- Cohort cuts (tenure, repeat, plan mix)
- Verbatim themes from ConvoIQ
- Anomaly flags
- **Candidate F-codes with confidence scores**
- Draft SCR skeleton
- **Mandatory: Known Limitations section**

Two non-negotiable rules:
1. **The agent proposes hypotheses. It never asserts root cause.**
2. **Known Limitations is a required output field.** Without it the agent silently over-claims.

---

## 8. SCR template (fixed, so intents stay comparable)

```
S — [Intent] is fully self-servable. [N] digital sessions/month, [trend].

C — [X]% abandon at [step] → [N] calls/month ([$] cost).
    Concentrated in [cohort]. Verbatims say: "[theme]".
    Classified [F-code], confidence [High/Med/Low].

R — [Fix 1]: [N] deflectable calls, [effort], owner [X]
    [Fix 2]: ...
    [Fix 3]: ...

    Known limitations: [telemetry blind spots, coverage gaps,
                        attribution caveats]
```

---

## 9. Crawl / walk / run roadmap

### Crawl (2–3 weeks) — codify, don't build
Build the **skill**, not the agent:
- Data binding manifest
- Journey spec schema
- SQL template library
- Output contract (evidence pack + limitations block)

**Validation gate:** re-run Travel Pass end-to-end and reproduce the original numbers —
`121K → 55K` funnel drop · `74%` tenure concentration · `4% vs 11%` completion gap.

> If it cannot reproduce work done by hand, it is not ready for intents that have not been done by hand.

### Walk — build the orchestrator
- Stand up orchestrator + subagents
- Run against 2–3 new International intents: Travel Pass Issue, Roaming Charges, Inquiry
- Human reviews every output
- **Measure:** analyst-days from ~3 weeks → target 1–2 days

### Run — scale and close the loop
- Open to other domains in the matrix
- Add the scheduled degradation monitor
- Add the **deflection-tracking loop**: after a fix ships, did calls actually drop?

> Without the deflection loop the programme cannot prove ROI.

---

## 10. Risks and failure modes

| Risk | Detail | Mitigation |
|---|---|---|
| **Journey spec drift** | Site releases rename pages; specs silently rot | Validation step flags "expected step `itp_planselection` had 0 sessions this month" |
| **Telemetry blind spots** | Already hit — CIR coverage, excluded bill shock / eligibility | Mandatory Known Limitations section in every output |
| **Correlation ≠ deflection** | A 55% drop-off does not mean 55% call | Explicit, defensible session→call attribution rule, locked centrally, versioned |
| **Taxonomy drift** | Analysts inventing labels destroys comparability | Closed vocabulary F1–F8, enforced by the agent |
| **Recommendation credibility** | Agents produce plausible-but-generic UX advice | Agent restricted to evidence + ranked hypotheses; humans write the fix |
| **Phantom drop-offs** | Optional branches read as abandonment | `required: false` flag in journey spec |
| **Silent over-claiming** | Agent states findings without caveats | Limitations block is a schema-required field, not optional prose |

---

## 11. The sharpest version of this

Do **not** build "a friction analysis agent." Build:

1. **A skill** — encodes the method + schema knowledge. Portable, reviewable, versioned.
2. **A registry** — intents and journey specs. The scaling asset.
3. **A scheduled agent** — keeps the registry warm: monitoring, flagging, drafting.

> **The analysis framework is the product. The agent is just the thing that runs it 50 times instead of 3.**

Corollary on sequencing: codify the skill **before** building the agent. Building the agent first bakes the method into a prompt where nobody can audit it.

---

## 12. Open items / next steps

| # | Item | Owner | Status |
|---|---|---|---|
| 1 | Draft the FRICTION-7 skill — taxonomy, journey spec schema, SQL template set, output contract with mandatory limitations block | — | **Proposed, awaiting go-ahead** |
| 2 | Lock the session→call attribution rule (window length, eligible call intents) | Analytics + stakeholders | Open |
| 3 | Populate Intent Registry for the International domain | — | Open |
| 4 | Write journey specs for Travel Pass Issue, Roaming Charges, Inquiry | — | Open |
| 5 | Run the reproduction test against Travel Pass | — | Blocked on #1 |
| 6 | Decide F7/F8 cohort query definition (post-success contact window) | — | Open |
| 7 | Define degradation alert thresholds for the scheduled monitor | — | Open |

---

## Appendix A — Reference numbers (Travel Pass)

| Metric | Value |
|---|---|
| Add Travel Pass calls/month | 21,000 |
| Travel Pass Issue calls/month | 30,000 |
| Travel Pass Inquiry calls/month | 16,000 |
| Roaming Charges calls/month | 13,000 |
| Funnel: `deviceandplansselection` | 121,000 |
| Funnel: `planselection` | 55,000 |
| Drop-off at that step | 55% |
| Sessions, Jun–Aug trend | +14% |
| Unique users, Jun–Aug trend | −25% |
| Plan change prior to session (30d, Jun→Aug) | 72% → 45% |
| Intent from 3+ year tenure accounts | 74% |
| Self-service completion — established accounts | 11% |
| Self-service completion — new accounts | 4% |

## Appendix B — Artifacts produced in this session

| File | Description |
|---|---|
| `friction-analysis-architecture.svg` | End-to-end layered architecture: config assets → automated orchestrator + 5 subagents → evidence pack → automation boundary → human judgment layer → deflection feedback loop |
| `friction-analysis-framework-brainstorm.md` | This document |
