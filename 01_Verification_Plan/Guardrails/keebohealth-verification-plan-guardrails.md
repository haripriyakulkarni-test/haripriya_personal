# KeeboHealth — Verification Plan Guardrails (Maker / Non-Negotiable)

> **Product:** KeeboHealth (Tricog RPM / chronic care platform — patient mobile app + doctor web portal)
> **Role this governs:** HP (QA Engineer / Maker) — drafting a Verification Plan before it is ready to open a PR for Manager (QA Manager / Checker) review.
> **Regulatory frame:** ISO 13485 · IEC 62304 · ISO 14971 · IEC 62366
> **Adapted from:** the org-wide `Guardrails_QC/VerificationPlan_Guardrails` baseline, reworded around KeeboHealth's product and stack rather than copied verbatim.

**Purpose:** these are the checks a KeeboHealth Verification Plan must pass *before* it's sent for review — not guidance, not a style preference. A plan that fails any rule below isn't ready to send, however complete it looks otherwise.

---

## 1. Required structure

A KeeboHealth VP must contain all of the following, matching the approved baseline template — no section renamed, merged, or dropped:

1. **Document header** — prepared by, status (Draft / In Review / Approved), classification.
2. **Objective** — what this release verifies and why, in terms of the actual PRD change (e.g. adding Blood Sugar to the existing vitals-trend graph).
3. **Scope** — In-scope / Out-of-scope, each item pointing at a specific PRD section or requirement ID; every exclusion gets a one-line reason.
4. **References** — every source used, as a live link: PRD, SDS, Impact Doc, Figma, Jira/tracking IDs.
5. **Open Gaps** — anything ambiguous, missing, or blocked in the source docs. If genuinely nothing, write "None identified" — never leave the section out.
6. **Test Strategy & Approach** — which testing types apply to *this* release and why (not boilerplate reused from the last one).
7. **Module Overview** — table of every KeeboHealth module touched (patient app, doctor web portal, backend/API, database), requirement count, and platforms.
8. **Scenario Combinations** — real multi-condition interactions (e.g. "Blood Sugar entry while offline, reading-type = Post-meal, threshold breach on sync"), not a restatement of single-condition scenarios already listed elsewhere.
9. **Functional Requirements** — grouped by module; each with a tracking ID (live link), the requirement text, applicable platform(s), and verification scenarios.
10. **Non-Functional Requirements** — grouped by category: Performance, Reliability, Security, Auditability, Usability, Data Integrity, Scalability, Compatibility (+ any other that applies). A category with nothing applicable says "Not Applicable" with a reason — never silently dropped.
11. **Entry Criteria** — objectively checkable conditions before testing starts (e.g. staging cohort with thresholds configured).
12. **Exit Criteria** — objectively checkable conditions for testing to be considered complete.
13. **Test Deliverables** — the artifacts this effort produces (test cases, execution evidence, traceability matrix).
14. **Risks & Assumptions** — risks to the verification effort itself, and any assumption made to resolve ambiguity.

**Any section missing = automatic fail. No exceptions.**

---

## 2. Testing type & coverage

1. Every functional requirement has at least one scenario covering its documented happy path.
2. Every requirement touching patient input, access control, or vitals/PHI data handling is evaluated for Security — if genuinely not applicable, say so with a reason; never skip it silently.
3. Every NFR category is explicitly evaluated per module — populated where applicable, "Not Applicable + reason" where not. Never pad a category that doesn't apply; never quietly skip one that does.
4. Where more than one platform is in scope (patient mobile app / doctor web portal / backend), each requirement's scenarios state which platform(s) they cover.

---

## 3. Methodology soundness

1. Every verification approach names a concrete method — e.g. "manual boundary test at SpO₂ = 91%/92%/93%," "automated regression on the existing BP/Weight graph component," "clinician-persona usability walkthrough of the reading-type selector." Vague language ("will be tested") is an automatic fail for that line.
2. Every Entry/Exit Criterion is objectively checkable (yes/no) — not a subjective judgment call.
3. Test Strategy & Approach is justified against what this specific release actually touches — never generic language copied from a prior VP.

---

## 4. Regulatory & risk (ISO 13485 / IEC 62304 / ISO 14971 / IEC 62366)

1. Any requirement with patient-data, safety, or clinical-decision impact (e.g. a threshold that drives a doctor alert) is flagged and cross-referenced in Risks & Assumptions.
2. Any patient- or doctor-facing workflow considers usability/human-factors implications under the Usability NFR where it affects safe use (e.g. a patient misreading which Blood Sugar reading-type to select).
3. Status only moves Draft → In Review (PR open) → Approved (Manager's merge). HP never marks their own VP Approved.
4. Classification is set per org policy — never left blank.

---

## 5. Honesty / no-invention

1. Never fabricate a requirement, scenario, or detail that isn't actually present in the PRD, SDS, Impact Doc, or Figma. Anything unsupported goes under Open Gaps, explicitly — never quietly assumed.
2. Open Gaps is never empty by omission — write "None identified" only when that's genuinely true.
3. Any assumption made to resolve real ambiguity (e.g. "PRD doesn't state whether Blood Sugar unit is configurable per patient — assuming org-wide default per SDS §X") is stated as an assumption in Risks & Assumptions, never presented as settled fact.

---

*Companion: `keebohealth-verification-plan-skillset.md`. When a real review from Manager surfaces a gap this file didn't anticipate, that gap gets added here — not treated as a one-off.*
