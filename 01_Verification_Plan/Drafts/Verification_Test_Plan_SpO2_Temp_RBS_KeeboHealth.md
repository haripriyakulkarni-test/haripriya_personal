# Verification Test Plan — `SpO2, Temperature & Blood Sugar (RBS) Vitals`

**Components under test**

- Patient Mobile App (`tricare-mobile-flutter`) — version not stated in source PRD (see Open Issue OI-1)
- Doctor Web Portal ("Doctor Panel") — version not stated in source PRD (OI-1)
- Backend / API layer (vitals ingestion, thresholds, alerts) — inferred from Impact reasoning below; not separately named in the PRD; version not stated (OI-1)
- Database (vitals storage) — inferred; version not stated (OI-1)

**Template ID:** L3_QC_VTE_310_VerificationTestPlan
**Author:** HP (QA Engineer / Maker) · **Reviewer:** Manager (QA Manager / Checker) · **Status:** Draft

> Tricog Confidential. All material contained in this section is the property of Tricog Singapore and is confidential and protected by copyright laws. © 2026 Tricog®. Tricog® is a registered trademark of Tricog Health Pvt. Ltd.

---

## 1. Introduction

This Verification Plan (VP) describes the verification of the **SpO2, Temperature and Blood Sugar (RBS) vitals** feature in KeeboHealth (the Tricog RPM / chronic-care platform), as defined in the source document `Including SpO2, Temp & RBS - Draft PRD.pdf` (Draft, no version/date stated — Open Issue OI-2). The release introduces these three vitals as first-class citizens across the Patient Mobile App and the Doctor Web Portal: capture via tasks/manual entry, visualization in Summary cards and trend graphs, doctor-configurable thresholds, and — conditionally, once the Smart Alerts engine is live — threshold-based alerting.

Objectives of this plan:
- Verify SpO2, Temperature and Blood Sugar can be captured as patient tasks and via manual entry, with correct validation (PRD §2.1, §2.2).
- Verify Doctor Panel treatment-plan/cohort bundling, baseline capture, and threshold configuration (PRD §4.4, §2.5, §2.6).
- Verify the three vitals surface correctly in Patient Summary cards and trend graphs (PRD §2.7, §2.8).
- Verify conditional alerting behaves correctly whether or not the Smart Alerts engine is live, without blocking the rest of the release (PRD §2.9, especially AC4).
- Verify reminders and compliance visibility for these vitals (PRD FR-10).
- Confirm the change does not regress the existing BP / Weight / HR task, manual-entry, and trend-graph behavior these new vitals are believed to extend (see Assumptions, §13).
- Identify targeted items: Patient Mobile App, Doctor Web Portal, Backend/API, Database.
- Outline approach: functional, integration (patient app ↔ backend ↔ doctor portal data flow), regression on reused components, and NFR evaluation (§6.2).

---

## 2. Scope of the Release

| Source PRD | Functional area | Brief |
| --- | --- | --- |
| §2.1 | Vitals as Patient Tasks | SpO2/Temperature/Blood Sugar appear as patient tasks; completed tasks sink to bottom; disabled tasks don't show; Glucose readings carry a reading-type tag |
| §2.2 | Manual Entry Flow | Manual entry form with range validation, provenance=Manual, and a Fasting/Random/Post-meal selector for Blood Sugar |
| §3.3 | "Add Manually" speed path | Skip-timer manual entry shortcut — PRD states this is **for BP, HR & Weight only**, not the three new vitals (see OI-9) |
| §4.4 | Treatment Plans (Doctor Panel) | Cohort-based mandatory-vitals bundling (e.g. Pulmonary: ECG+SpO2+BP+Weight+RBS) with admin-override-gated disable |
| §2.5 | Baseline Capture at Enrollment | Doctor-entered baseline SpO2/Temperature/RBS at onboarding, surfaced with date in Patient Summary |
| §2.6 | Doctor-Configurable Thresholds | Org-default + per-patient threshold overrides, audit-logged |
| §2.7 | Patient Summary Cards | Clinician-facing latest-value/timestamp/flag-badge cards for the three vitals |
| §2.8 | Vitals Trend Graphs | Trend-graph extension to the three vitals; 7/30/90-day windows, tooltips with source |
| §2.9 | Alerts (conditional) | Threshold-breach alerting via Smart Alerts, gated on engine availability; must not block §2.1–2.8 if engine isn't live |
| FR-10 | Reminders & Compliance | In-app reminders; missed tasks surfaced in Doctor Panel compliance views |

**Out of scope** (per PRD §1.3, reasons as stated in the source):
- New device/Bluetooth integrations beyond what already exists (may be planned separately).
- Major UX redesign — the existing task-based flow is retained.
- Automated treatment recommendations.

---

## 3. System Risk Profile

| Module | Risk | Rationale |
| --- | --- | --- |
| Backend/API — vitals ingestion & threshold evaluation | HIGH | A dropped reading or a miscalculated threshold is a direct path to a missed clinical deterioration signal. |
| Doctor Web Portal — Thresholds & Alerts (§2.6, §2.9) | HIGH | A misconfigured or non-firing alert is a direct patient-safety risk under ISO 14971. |
| Patient Mobile App — Tasks & Manual Entry (§2.1, §2.2) | HIGH | This is the *only* capture path in scope for these three vitals (no new device integration) — a broken entry flow means zero data, not degraded data. |
| Database — vitals schema/storage | HIGH | Governs data integrity of both new and existing (BP/Weight) vitals records; backward compatibility risk. |
| Doctor Web Portal — Baseline Capture & Summary Cards (§2.5, §2.7) | MEDIUM | Incorrect display is a clinical-context/usability risk, less acute than a missed real-time alert. |
| Patient App / Doctor Portal — Trend Graphs (§2.8) | MEDIUM | Visualization correctness matters for trend-based judgment; a point failure is less severe than a missed alert. |
| Treatment Plans / cohort bundling (§4.4) | MEDIUM | Wrong mandatory-vital bundling affects program compliance more than immediate safety. |
| Reminders & Compliance (FR-10) | LOW | Adherence-support feature; failure degrades data completeness over time rather than causing an acute miss. |
| "Add Manually" speed path (§3.3, pre-existing, BP/HR/Weight) | LOW | Pre-existing feature; regression risk only for this release. |

---

## 4. Test Methodology & Approach

Applicable test types:
- **Functional Testing** — new flows (§2.1, §2.2, §2.5–§2.9, FR-10) plus regression of the existing behavior these are believed to extend.
- **Integration Testing** — patient app ↔ backend ↔ doctor web portal data flow; Smart Alerts engine integration when live.
- **Regression Testing** — existing BP/Weight/HR task, manual-entry, and trend-graph components (see §6.5).
- **Exploratory Testing** — mid-entry app kill, offline entry then sync, threshold changed mid-cycle.
- **Non-Functional Testing** — see NFR matrix (§6.2); Performance/Scalability targets are **not specified in the PRD** and are explicitly deferred rather than invented (OI-3).
- **UAT & Sign-off** — recommended given the HIGH risk items in §3; no clinical/product SME is named in the PRD (OI-7).

Formal Black-Box Test (BBT) techniques applied (selected for relevance to this PRD — not every technique in the org catalogue applies, and techniques with no grounding here are explicitly marked Not Applicable):

| Technique | Applies to this release | Applicable? |
| --- | --- | --- |
| Boundary Value Analysis | Threshold boundaries (SpO2 <92%, RBS >180 mg/dL) and manual-entry range limits | Yes |
| Equivalence Partitioning | Manual-entry values: below-min / in-range / above-max per vital | Yes |
| Decision Table Testing | Cohort × vital × disable-attempt × admin-override-granted combinations (§4.4) | Yes |
| State Transition Testing | Task lifecycle (pending → completed → missed); Alert lifecycle (fired → new-critical-not-blocked, §2.9 AC2) | Yes |
| Configuration Testing | Per-patient/program enable-disable of vitals; threshold override vs org default | Yes |
| Audit / Compliance Testing | Threshold-change audit log (§2.6 AC3) | Yes |
| Regression Testing | Existing BP/Weight/HR flows and shared components | Yes |
| Negative / Error-Guessing | Invalid manual entry, missing reading-type selection (§2.2 AC4, see OI-8), network drop mid-submit, alert engine down | Yes |
| Security / RBAC Testing | Doctor sees only their own patients' vitals | Yes |
| Pairwise (All-Pairs) | Platform × vital-type combinations, kept lightweight given the limited dimension count | Yes (light use) |
| PDF/Document Output Testing | No document-generation behavior described in this PRD | Not Applicable |
| Email/Notification Channel Testing | Only relevant conditionally under §2.9 (Smart Alerts email notify) | Applicable, conditional |
| Localization Testing | Not addressed anywhere in the PRD | Not Applicable — flagged, not assumed absent (see OI-6) |
| Responsive/Orientation Testing | Applicable to Patient App UI generally, not called out as changed by this PRD | Applicable, low priority |
| Cross-Browser/Compatibility | Doctor Web Portal — target browsers not specified (OI-6) | Applicable, target list pending |

---

## 5. Testing Process

1. Analyze the PRD (the sole source document currently available — no SDS or Impact Document exists yet; see OI-13) and clarify open items with Product before test-case authoring finalizes.
2. Perform Impact Analysis as part of this VP (§3, §6.5) in the absence of a separate Impact Document.
3. Design and author test cases mapped to the PRD's own requirement IDs (§2.1, §2.2, §3.3, §4.4, §2.5–§2.9, FR-10).
4. On receiving the build, run a sanity check before full execution begins.
5. Log defects as **GitHub Issues on `Tricog-Engineering/keebo-health-monorepo`** (KeeboHealth's actual defect-tracking mechanism, per current QA practice) — this is a deliberate deviation from the generic template's JIRA/Zephyr default, made because that's where KeeboHealth defects are actually tracked today.
6. Regression-test after defect fixes land.
7. Summary report to stakeholders with execution/defect metrics (§11).
8. Sign-off in this project's model is **Manager's PR merge** (the maker-checker gate) rather than a separate testing-team sign-off step — see [[keebohealth-qa-maker-checker]] architecture.

---

## 6. Testing Strategy

### 6.1 Feature Testing

**Feature testing scope**

| Cohort / Program | Vital | Platform | Task enabled | Test Type |
| --- | --- | --- | --- | --- |
| Pulmonary cohort (mandatory bundle, §4.4 AC1) | SpO2 | Patient App | ON (locked) | Feature Testing |
| Pulmonary cohort | RBS | Patient App | ON (locked) | Feature Testing |
| Standard/other program | Blood Sugar | Patient App | ON (doctor-enabled) | Feature Testing |
| Any program | Temperature | Patient App | OFF (disabled at patient level) | Feature Testing (negative path) |

**Configuration / feature-flag behavior matrix**

| Configuration | Flag | Condition | Patient App | Doctor Panel | Expected output |
| --- | --- | --- | --- | --- | --- |
| Vital task enabled | ON | Patient enrolled, vital enabled | Task appears in list | Vital shown active in Treatment Plan | Task completable, value recorded |
| Vital task disabled | OFF | Disabled at patient level (§2.1 AC3) | Task absent | Vital shown inactive | No task, no entry possible |
| Mandatory cohort vital, no override | ON (locked) | Pulmonary cohort (§4.4 AC2) | Task appears; patient cannot disable | Doctor blocked from disabling without admin override | Disable attempt blocked/requires override |

**Outcome × state matrix**

| Outcome | State | Patient App | Doctor Panel | Pass criteria | Test type |
| --- | --- | --- | --- | --- | --- |
| Reading within range | Completed | Task → completed section, timestamped | Summary card: normal flag | Value/time/flag correct | Feature |
| Reading breaches threshold, Smart Alerts live | Completed | Task completed | Flag shown + Alert generated (§2.9 AC1) | Correct severity, no duplicate | Feature |
| Reading breaches threshold, Smart Alerts **not** live | Completed | Task completed | Flag shown, **no** Alert (§2.9 AC4) | §2.1–2.8 unaffected, Alert absent | Edge Case |
| Task left past due | Missed | Reminder fires, rolls to pending (FR-10) | Compliance view shows missed | Missed task visible to care team | Edge Case |

**Timing / coordination matrix**

| Scenario | Input | Expected behaviour | Pass criteria | Test type |
| --- | --- | --- | --- | --- |
| Doctor edits threshold concurrently with an incoming reading | Threshold change + concurrent reading | Reading evaluated against the **updated** threshold "immediately" (§2.6 AC2) | No stale-threshold evaluation | Edge Case |
| Patient submits reading offline, syncs later | Offline entry, delayed sync | Correct capture timestamp preserved; syncs without loss | No data loss, correct timestamp | Edge Case |

**Segmentation matrix**

| Segment | Applies to | Cases triggered | Expected |
| --- | --- | --- | --- |
| Pulmonary cohort | Patients tagged pulmonary | Mandatory bundle: ECG+SpO2+BP+Weight+RBS (§4.4 AC1) | All five auto-enabled and locked |
| Standard/other cohort | All other patients | Vitals enabled individually by doctor | Only doctor-enabled vitals appear |

**Notification / output-channel matrix** (conditional on Smart Alerts engine being live)

| Situation | Count | Expected |
| --- | --- | --- |
| Threshold breach, engine live, email configured | 1 | Notified via Doctor Panel + email; no duplicate for the same event (§2.9 AC2) |
| Repeated breach, same episode | Multiple readings | Deduplicated per §2.9 AC2 |
| New critical breach while a prior one is unaddressed | 2 alerts | New critical alert is **not** blocked by the unaddressed prior one (§2.9 AC2) |

**Performance matrix**

| Scenario | Expected behaviour | Pass criteria | Notes |
| --- | --- | --- | --- |
| Trend graph render, 90-day window, realistic reading volume | Loads without visible hang/error | No numeric target in PRD — Open Issue OI-3; qualitative pass only until SDS defines a threshold | Deferred formal PERF sign-off |

### 6.2 NFR Verification Matrix

| NFR | Requirement | Target / Threshold | Mapped scenario | Env | Measurement method |
| --- | --- | --- | --- | --- | --- |
| Auditability | Threshold changes logged with who/what/when (§2.6 AC3) | 100% of threshold-change events logged | Config/flag matrix | QA | Audit-log inspection |
| Data Integrity | Range validation rejects impossible values (§2.2 AC1); provenance always recorded (§2.2 AC2) | 100% invalid values rejected; provenance populated on every save | Manual-entry functional cases | QA | DB record inspection |
| Security / PHI | Vitals are patient health data; standard authZ + injection/XSS protection expected on entry forms | Not explicitly specified in PRD — Open Issue OI-4; verified against the product's general security baseline | Manual entry, Doctor Panel data access | QA/STAGING | Manual security check + existing security-test suite if any |
| Performance | Vitals ingestion / graph render responsiveness | Not specified in PRD — Open Issue OI-3 | Trend graph load | PERF | Deferred pending SDS |
| Reliability | Alert dedup/rate-limiting (§2.9 AC2) | No duplicate alerts per event; new critical alerts never blocked | Notification matrix | QA | Alert-log inspection |
| Scalability | — | Not addressed in PRD — Open Issue OI-3 | — | — | Deferred |

### 6.3 Test Combination Matrix — Black-Box Test Design

| # | Technique | Combination | Expected behaviour | Decision |
| --- | --- | --- | --- | --- |
| 1 | Boundary Value Analysis | SpO2 = 91% / 92% / 93% vs. threshold "<92%" | 91% flags; 92%/93% do not (exact operator semantics to confirm against SDS — OI-5) | TEST |
| 2 | Boundary Value Analysis | RBS = 179 / 180 / 181 mg/dL vs. threshold ">180" | 180 does not flag; 181 flags | TEST |
| 3 | Equivalence Partitioning | Manual entry: below-min / in-range / above-max, each of SpO2/Temp/RBS | Below-min & above-max rejected; in-range accepted | TEST |
| 4 | Decision Table | Cohort=Pulmonary × Vital=RBS × patient-disable-attempt × admin-override granted/not | Not granted → blocked; granted → allowed | TEST |
| 5 | State Transition | Task: not-started → completed → missed (if past due) | Matches §2.1/FR-10 | TEST |
| 6 | Negative/Error-Guessing | Blood Sugar entry with no reading-type selected | PRD doesn't state whether selection is mandatory (OI-8) | TEST — GAP-FILLER (ambiguity closed by QA, flagged for Product) |
| 7 | Regression | Existing BP/Weight trend graph, unaffected by this change | No regression | TEST |
| 8 | Regression | "Add manually" speed path (§3.3) stays absent for SpO2/Temp/RBS tasks | Speed path shown only for BP/HR/Weight | TEST — GAP-FILLER (PRD's own §3.3 title implies this, but never states the 3 new vitals are excluded from it explicitly — OI-9) |
| 9 | Security/RBAC | Doctor A attempts to view Doctor B's patient's vitals | Access denied | TEST |
| 10 | Audit/Compliance | Threshold changed, then reverted | Both changes logged with who/when | TEST |

Combination totals: **10 combinations defined · 10 TEST · 0 IGNORE.** This is a representative seed set; the full Test Case authoring stage (next artifact in the maker-checker pipeline) expands this into the complete test-case bank.

### 6.4 Coverage Analysis

| Coverage Dimension | Total Partitions | Covered | % | Backed by |
| --- | --- | --- | --- | --- |
| Vitals (SpO2 / Temperature / RBS) | 3 | 3 | 100% | Rows 1–3, 6 |
| Platforms (Patient App, Doctor Web Portal) | 2 | 2 | 100% | §6.1 Feature testing scope |
| Cohort paths (Pulmonary-mandatory vs. standard-optional) | 2 | 2 | 100% | Segmentation matrix, row 4 |
| Alert-engine state (live / not live) | 2 | 2 | 100% | Outcome × state matrix |
| Threshold boundary conditions | 3 (one per vital) | 2 (SpO2, RBS) | 67% | Rows 1–2; Temperature has no boundary example in the PRD (OI-5) |
| **Weighted Total** | — | — | **High on documented dimensions; one explicit residual gap (Temperature boundary) rather than a padded 100%** | — |

Residual/out-of-scope (acknowledged, per PRD §1.3): new device/Bluetooth integrations, major UX redesign, automated treatment recommendations.

### 6.5 Regression Testing

The three new vitals are believed to extend the existing BP/Weight/HR task list, manual-entry form pattern, and trend-graph component (per Haripriya's verbal input, 2026-08-20 — **not stated in the PRD itself**, see Assumptions §13). Regression flows:

| Flow | Expectation |
| --- | --- |
| Existing BP/Weight/HR task list and manual entry, feature disabled for new vitals | Full existing flow unaffected |
| Existing patients without the new vitals enabled | No change to their task list or graphs |
| Existing "Add manually" speed path (§3.3) | Continues to work for BP/HR/Weight exactly as before |

### 6.6 Platform / browser versions

Not specified in the PRD — **Open Issue OI-6.** Needs an SDS or the org's current device/browser support policy before this row can be filled in.

### 6.7 Test data

- Patients across both cohort types (Pulmonary-mandatory, standard-optional).
- Values spanning below / at / above threshold for each of the three vitals.
- At least one patient with a baseline value only (§2.5) and no logged readings yet, to check the graph/summary empty-state.
- Mixed manual + device-provenance data for BP/Weight regression checks.
- All patient-like data synthetic/de-identified — no real PHI (per Data Integrity/Security NFR, §6.2).

### 6.8 Test Results

Captured with evidence (screenshots, API responses, DB snapshots) in `haripriyakulkarni-test/KeeboHealth_QA` (`reports/`, `test-data/`) per KeeboHealth's current QA repo structure — and, once the Agentic SDLC pipeline described in [[keebohealth-qa-maker-checker]] is live, in its Evidence Store. This is a deliberate deviation from the generic template's JIRA+Zephyr default, since that's not how KeeboHealth actually tracks QA today.

---

## 7. Roles & Responsibilities

| Role | Name | Responsibility |
| --- | --- | --- |
| QA Manager / Checker | Manager | Independent review; merge = approval of this VP and downstream artifacts |
| QA Engineer / Maker | HP | Authoring & executing this VP and its test cases; defect logging |
| Product Manager | Not named in source material — Open Issue OI-7 | Resolve open PM questions (OI-5, OI-8, OI-9, OI-10, OI-11); scope sign-off |
| Delivery Manager | Not named — OI-7 | Review/answer open items |
| Developers | Not named — OI-7 | Review test-case combinations |
| DevOps / SRE | Not named — OI-7 | Monitoring, rollback readiness |

---

## 8. Entry Criteria

- This Verification Plan is merged (Manager's approval per the maker-checker gate).
- Test cases authored and reviewed.
- Build deployed to the test environment with the SpO2/Temp/RBS feature enabled across patient app + doctor web portal + backend.
- Build sanity check passed.
- Test data fixtures ready per §6.7.
- Per-patient/cohort vitals-enable flags configurable and reachable from QA.
- Smart Alerts engine's live/not-live status is known before FR 2.9 scenarios are executed (conditional per §2.9 AC4).

---

## 9. Configuration of Test Environment

| Environment | Description |
| --- | --- |
| DEV | Smoke checks, mocked dependencies. |
| QA | Full functional + integration scenarios; primary environment for this VP's execution. |
| STAGING/UAT | End-to-end on production-like data, pending confirmation this environment exists for KeeboHealth today (Open Issue). |
| PERF | Reserved for the deferred Performance NFR (§6.2) once targets exist. |
| PROD (canary) | Reserved for post-release smoke validation (§14). |

Confirmation of which of these environments actually exist for KeeboHealth (beyond DEV/QA) is an open item — not assumed.

---

## 10. Test Data Planning & Management

Fixtures and rewritten test cases are stored in `haripriyakulkarni-test/KeeboHealth_QA` under `test-data/` and `test-cases/` (this project's actual QA repo structure), not the generic template's Google Drive/JIRA CSV convention — deliberate deviation, same rationale as §6.8.

---

## 11. Metrics and Reporting

| Name | Content | Frequency | Method |
| --- | --- | --- | --- |
| Status Report | Progress, new issues, next steps | Per release cycle | Written summary to Haripriya |
| Issues/Risks | Open item, mitigation, follow-up | As needed | Tracked in this VP's Open Issues + Risks sections |
| Defects | Filed as GitHub Issues on `keebo-health-monorepo` | As found | GitHub Issues |

---

## 12. Test Risks & Mitigation

| Risk | Mitigation |
| --- | --- |
| No SDS/Impact Document exists yet; this VP is drafted from the PRD alone | Escalate OI-2, OI-3, OI-4, OI-5, OI-13 to Product before finalizing detailed test cases; track as Open Issues, not silent assumptions |
| Smart Alerts engine readiness is unknown | Scope §2.9 execution as conditional; do not block §2.1–2.8/FR-10 sign-off on it, per PRD §2.9 AC4 |
| Temperature unit inconsistency (°F in §2.2 vs °C in §2.6/§2.8) | Raise to Product before finalizing Temperature threshold/graph test cases (OI-5) |
| PRD's own requirement numbering is non-sequential | Retained verbatim in traceability rather than renumbered, to avoid introducing a second numbering scheme (OI-12) |
| Reused BP/Weight/graph/task components could regress under this change | Explicit regression flows, §6.5 |

---

## 13. Assumptions

- "Doctor Panel" refers to the KeeboHealth doctor-facing web portal (per Haripriya's verbal description, 2026-08-20) — not stated in the PRD itself.
- The existing BP/Weight vitals-trend graph component is being extended/reused for SpO2/Temperature/Blood Sugar (per Haripriya's verbal input, 2026-08-20), rather than built net-new — not stated in the PRD.
- Unit/dev-level testing is complete before the build reaches QA.
- No major UX redesign accompanies this change, per PRD §1.3.2.

### Open Issues (never silently resolved — carried into test-case design as explicit gaps)

| ID | Open Issue |
| --- | --- |
| OI-1 | No component/app version numbers stated in the PRD. |
| OI-2 | No release/version identifier stated for this change. |
| OI-3 | No explicit Performance/Scalability/Reliability numeric targets anywhere in the PRD. |
| OI-4 | No explicit Security/PHI-handling requirement stated for the new vitals fields beyond general product expectations. |
| OI-5 | No Temperature threshold boundary example given (only SpO2 <92% and RBS >180 examples exist); Temperature's own unit is inconsistent — °F in the manual-entry range (§2.2 AC1) vs °C in the threshold example (§2.6). |
| OI-6 | No target Android/iOS versions or supported web browsers specified. |
| OI-7 | No names given for Product Manager, Delivery Manager, Developer, or DevOps/SRE roles. |
| OI-8 | §2.2 AC4 requires a reading-type tag for Blood Sugar entries but never states whether selecting one is mandatory before save. |
| OI-9 | §3.3 ("Add Manually" speed path) is written for BP/HR/Weight only, inside a PRD titled for SpO2/Temp/RBS — unclear whether this is deliberate bundling or a drafting artifact; verified per its literal wording (excluded from the three new vitals). |
| OI-10 | §4.4 AC2's "admin override" mechanism (who grants it, how it's audited) is undefined. |
| OI-11 | §2.8's trend-graph tooltip spec (timestamp/value/source) doesn't say whether the Blood Sugar reading-type tag from §2.2 AC4 should also surface on the graph/tooltip. |
| OI-12 | The PRD's own section numbering is non-sequential (2.1 → 2.2 → 3.3 → 4.4 → 2.5 → 2.6 → 2.7 → 2.8 → 2.9 → FR-10); retained verbatim here rather than renumbered. |
| OI-13 | No SDS or Impact Document exists yet; the Impact reasoning in §3 and §6.5 was derived directly from the PRD by the QA/Maker agent, not from a separate Impact Analysis artifact. |
| OI-14 | The PRD defines no rollback strategy for this release; one must be defined at SDS stage before production sign-off (see §14). |

---

## 14. Exit Criteria

- All Test Cases passed.
- All defects closed, or explicitly deferred with sign-off from Manager and Product.
- 100% of P0 scenarios pass · ≥ 95% of P1 pass.
- 100% of Test Combination Matrix TEST rows (§6.3) executed and passing.
- Coverage Analysis (§6.4) shows 100% of documented in-scope dimensions covered; the one acknowledged residual gap (Temperature boundary, OI-5) is explicitly accepted, not silently closed.
- FR 2.9 (Alerts) is either fully verified (engine live) or explicitly marked not-yet-verifiable-pending-engine-availability, without blocking sign-off of FR 2.1–2.8/FR-10 (per §2.9 AC4).
- 0 open P0/P1 defects; ≤ 3 P2 with a documented workaround.
- Rollback approach reviewed — currently undefined in the PRD (OI-14); must exist before production sign-off.
- Legacy vitals data (existing BP/Weight/HR history) confirmed unaffected — backward compatibility check passed.
- Sign-off = Manager's PR merge on this VP and its downstream artifacts, per the maker-checker gate.

---

## 15. Test Artifacts

| Verification Requirement | Deliverable | Location |
| --- | --- | --- |
| Verification approach & strategy | This Verification Plan | `haripriyakulkarni-test/KeeboHealth_QA/test-plans/` |
| Test case authoring | System test cases mapped to §2.1–FR-10 | `.../test-cases/` |
| Traceability | Requirement → scenario → test case → evidence matrix | `.../test-plans/` (companion to this VP) |
| Test execution | Execution report + evidence | `.../reports/` |
| Defects | GitHub Issues | `Tricog-Engineering/keebo-health-monorepo` |

---

## 16. Training

The team's current context is limited to the Draft PRD — no SDS or Impact Document exists yet. A re-walkthrough is needed once those land, particularly to resolve OI-5, OI-8, OI-9, OI-10, and OI-11 before test-case authoring is finalized for the affected areas.

---

## 17. Referenced Documents

| Document | Version |
| --- | --- |
| `Including SpO2, Temp & RBS - Draft PRD.pdf` | Draft — no version/date stated in the document itself |
| SDS | Not yet available — OI-13 |
| Impact Analysis | Not yet available — OI-13 |

**Appendix**

| Document | Link / Status |
| --- | --- |
| Testing SOP | Not located for KeeboHealth — Open Issue |
| Risk Report | Not located for KeeboHealth — Open Issue |
| KeeboHealth Verification Plan Guardrails (Maker) | `keebohealth-verification-plan-guardrails.md`, same folder |
| KeeboHealth Verification Plan Skillset (Maker) | `keebohealth-verification-plan-skillset.md`, same folder |

**Sign-off:** Author — HP (Maker), Draft. Reviewer — Manager (Checker), pending review. In this project's model, approval is recorded as Manager's PR merge, not a separate signature block.
