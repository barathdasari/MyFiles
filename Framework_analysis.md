# Digital Friction Analysis Framework
**Target Application:** Self-Service Deflection & Journey Optimization
**Input Source:** `intent x friction matrix.md`

*Instruction for Analyst/Agent: Use this template to evaluate any L1 Intent identified in the Intent x Friction Matrix. Replace bracketed text `[ ]` with specific data queries and analysis results.*

---

## 1. Executive Summary (SCR Framework)
*A high-level synthesis of the problem and actionable insights, formatted for executive review.*

*   **Situation:** Customers intend to achieve a specific goal digitally (e.g., `[Insert L1 Intent, e.g., Activate Travel Pass]`). The journey is designed to be fully self-servable.
*   **Complication:** Users are experiencing friction at specific digital nodes, preventing self-service completion and driving unwanted call volume (`[Insert Call Volume Impact, e.g., 21K monthly calls]`). 
*   **Resolution:** By addressing root causes at `[Insert Primary Drop-off Node]`, specifically regarding `[Insert Top Issue, e.g., Plan Selection Confusion / Tenure-based eligibility]`, we project a deflection of `[X]%` of related call volume.

---

## 2. Problem Definition & Prioritization (7-Step: Steps 1, 2 & 3)
*Defining the scope, exclusions, and business impact based on the intent matrix.*

*   **L1 Intent Analyzed:** `[Extract from intent x friction matrix.md]`
*   **Analysis Goal:** Uncover why users intending to execute `[Intent]` experience friction digitally and pivot to assisted channels.
*   **Exclusions:** `[e.g., Focus solely on digital journeys, excluding bill shock or eligibility factors due to telemetry constraints]`
*   **Impact Sizing (Call Volume Metrics):**
    *   Primary Intent Calls: `[e.g., Add Travel Pass - 21K]`
    *   Secondary/Related Calls: `[e.g., Issue - 30K, Inquiry - 16K, Charges - 13K]`

---

## 3. Data Strategy & Journey Mapping (7-Step: Step 4 / Phases 1-2)
*Establishing the data repositories and outlining the ideal digital funnel.*

**A. Telemetry & Repository Mapping:**
*   **Digital Session Metrics:** `[e.g., cwlsdo-0]`
*   **Intent Signals:** `[e.g., vzdsdw-0]`
*   **Conversation Records:** `[e.g., cdwldo-0]`
*   **Attribute Adjustments/Joins Required:** `[e.g., swap page_name to page_nm, join via omni_hit_join_vcg]`

**B. Standard Journey Flow (Node Mapping):**
*Map the step-by-step digital pathway the user is expected to take.*
1. `[Node 1: e.g., international_dashboard]` 
2. `[Node 2: e.g., itp_destinationselection]`
3. `[Node 3: e.g., itp_traveldates]`
4. `[Node 4: e.g., itp_deviceandplansselection]`
5. `[Node 5: e.g., itp_planselection]` -> *Primary Drop-off Risk*
6. `[Node 6: e.g., itp_effectivedate]`
7. `[Node N: e.g., itp_confirmation]`

---

## 4. Friction Analysis & Impact Measurement (7-Step: Step 5 / Phase 3)
*Identifying exactly where and how the journey breaks.*

**A. Funnel Drop-Off Analysis:**
*   **Node-to-Node Drop-off:** Identify the largest session loss. 
    *   *Finding:* Major drop-off moving from `[Node A]` to `[Node B]`. 
    *   *Metrics:* Dropped from `[Volume X]` to `[Volume Y]` (`[Z]%` loss).

**B. L3/L4 Friction Taxonomy:**
*Break down the specific friction drivers at the drop-off point.*
*   **Friction Type 1 (e.g., Fees/Pricing):** `[Data points]`
*   **Friction Type 2 (e.g., Technical Errors):** `[Data points]`
*   **Friction Type 3 (e.g., UI/UX Gaps, VA dead-ends):** `[Data points]`

---

## 5. Cohort Segmentation & Deep Dives (7-Step: Step 5 Cont. / Phases 4-5)
*Analyzing user behaviors, historical context, and longitudinal data to find the "Why".*

**A. Segmentation & Behavioral Trends (Phase 4):**
*   **Caller Behaviors:** `[Identify repeat contact trends, e.g., X% call within 24 hours of digital drop-off]`
*   **Distribution:** `[e.g., Plan distribution among callers]`

**B. Multi-Angle Analytical Deep Dives (Phase 5):**
*(Customize these based on the specific L1 intent being analyzed)*
1.  **Prior Account Activity (Trailing 30 Days):**
    *   *Query:* Did recent automated or manual account updates skew the journey?
    *   *Insight:* `[e.g., Plan changes prior to session shifted from 72% to 45%, indicating automation interference]`
2.  **Longitudinal Trends (Trailing 3 Months):**
    *   *Query:* Are we looking at a seasonal spike, a systemic issue, or a reporting artifact?
    *   *Insight:* `[e.g., Sessions rose 14% but unique users fell 25%, confirming ongoing 50-55% drop-off issue. CIR fluctuations stem from data coverage limitations, not process fixes.]`
3.  **Historical Usage Patterns (Trailing 12 Months):**
    *   *Query:* How does past usage correlate to current friction?
    *   *Insight:* `[Insert findings]`
4.  **Account Tenure Breakdown:**
    *   *Query:* Do new users experience different friction than legacy users?
    *   *Insight:* `[e.g., 74% of intent comes from 3+ year accounts, but new users complete self-service at half the rate (4% vs 11%)]`

---

## 6. Synthesis & Recommendations (7-Step: Steps 6 & 7)
*Translating data into actionable fixes for Engineering, UX, and Product teams.*

**Root Cause Synthesis:**
*   Based on the data, the primary reason users fail to digitally complete `[L1 Intent]` is `[Insert Synthesized Reason, e.g., new users are unable to navigate plan selection due to confusing UI regarding tenure requirements]`.

**Recommended Fixes:**
1.  **UX/UI Adjustment:** `[Actionable recommendation, e.g., Redesign Node 5 to simplify plan comparisons]`
2.  **Logic/Process Fix:** `[Actionable recommendation, e.g., Suppress automated account update errors during the Travel Pass flow]`
3.  **Telemetry/Data Fix:** `[Actionable recommendation, e.g., Implement tracking for mid-summer CIR data coverage limitations]`

**Review Status:** `[e.g., Finalized and aligned with stakeholder input for upcoming review]`
