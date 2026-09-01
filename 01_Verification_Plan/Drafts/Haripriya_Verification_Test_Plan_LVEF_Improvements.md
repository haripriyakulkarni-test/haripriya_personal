# Verification Test Plan — `LVEF Improvements on Consolidated Report`

**Components under test**

- Tricog Admin (TA) — Server & Client — version not stated (OI-1) <!-- comment resolved -->
- Customer Portal — version not stated (OI-1)
- VCardia — Android & iOS — versions not stated; Impact Doc names the **oldest unsupported** builds (iOS v6.11.00, Android v9.5.2) but not the target "latest" version (OI-1)
- Cardionet V2 — version not stated (OI-1)
- Boron Server (VCardia backend) — version not stated (OI-1)
- Transformer (Cronjob / polling service) — version not stated (OI-1)
- Report Builder (RB templates) — version not stated (OI-1)

<!-- comment resolved -->

**Template ID:** L3_QC_VTE_310_VerificationTestPlan (shared artifact-type ID — not an instance version; see Version History below, added per Checker review G-03)
**Document Version:** v0.3
**Author:** HP (QA Engineer / Maker) · **Reviewer:** NR (QA Manager / Checker) · **Status:** Draft

**Version History**

| Version | Date | Author | Change |
| --- | --- | --- | --- |
| v0.1 | 2026-08-21 | HP | Initial draft opened as PR #1 |
| v0.2 | 2026-08-21 | HP | Addressed NR's Checker review: this version history, complete §18 References, Release Type & Coverage Floor declaration (§1), bidirectional App↔Backend + mixed-app-version scenarios (§6.5), Rollback Strategy (§14), per-endpoint API contract matrix (§6.14), Module Overview (§2.1), Auditability/Scalability NFR rows (§6.2), evidence-store commitment (§6.8), hazard-register traceability note (§3), thicker Out-of-Scope list (§2) |
| v0.3 | 2026-08-25 | HP | Self-review pass against the updated `verification-plan-guardrails-hp.md`/`verification-plan-skillset-hp.md` (not a new NR review): formalized OI-24's elevation to a Maker-external Blocker (§3, §13, §15), discovered via a cross-draft consistency check against the parallel VP's OG-24; added an explicit Maker-fixable/Externally-blocked classification to every Open Issue (§13) per the guardrails' new §6.2 rule |

> Tricog Confidential. All material contained in this section is the property of Tricog Singapore and is confidential and protected by copyright laws. © 2026 Tricog®. Tricog® is a registered trademark of Tricog Health Pvt. Ltd.

---

## 1. Introduction

This VP covers the **LVEF Improvements on Consolidated Report** change to InstaECG/CardioNet: consolidating the previously-separate LVEF Risk Assessment into the ECG report itself (page-1 one-liner + optional page-2 addendum), rationalising LVEF UI surfaces across TA/Customer Portal/VCardia, and unifying LVEF email communication — as specified in the PRD (`Testing/PRD/LVEF improvements.md`), the Impact Document, and the SDS, all read in full for this VP.

Objectives:
- Verify the page-1 one-liner and page-2 addendum render correctly across all output formats, gated correctly by the two toggles and the AI outcome.
- Verify the TA config block changes (rename, new toggle, Segment Type relocation/redefinition, Selective-Submission deprecation) behave and migrate correctly.
- Verify removal of standalone LVEF UI surfaces (Customer Portal, VCardia, TA case detail) with no regression to remaining actions.
- Verify the case-list/detail label logic across **all five** result states (Reduced / No reduced / Pending / Failed / Error — the fifth confirmed only by the SDS and design, not the PRD).
- Verify the report-delivery coordination timing (1-minute wait, 10-minute cap, subsequent consolidated delivery, stuck-ECG recovery) and the consolidated email.
- Verify the NFR targets given concretely in the SDS (load time, list latency, polling interval, generation latency).
- Verify data-storage separation (ECG vs LVEF interpretation fields) and backward compatibility for older app versions and non-RB templates.

Identified targets: TA, Customer Portal, VCardia (Android/iOS), Cardionet V2, Boron Server, Transformer, Report Builder templates, Bridge Health / generic LVEF-webhook customers.

**Approach:** functional + integration (post-diagnosis coordination across ECG/LVEF services) + regression (existing report/email/notification flows) + non-functional (NFR targets stated in the SDS) + data-migration testing (Segment Type, Selective Submission deprecation) + UAT/regulatory review (given the explicitly risk-accepted double-critical-alarm behaviour, SDS §11 Q4).

**Release Type & Coverage Floor** (added per Checker review M5 · VP-60/62a/66): this release is classified as a **mixed platform + backend + data-migration release** — it touches multi-platform clients (Customer Portal, VCardia Android/iOS), backend services (Boron Server, Transformer), TA configuration, and two data migrations (Segment Type, Selective-Submission deprecation). Per the mandatory-coverage-floor rule for this release type, both App↔Backend compatibility directions and device-level mixed-version coexistence are in scope — see the new rows in §6.5 — rather than only the single direction (old app, new backend) this VP originally covered.

---

## 2. Scope of the Release

| Source | Functional area | Brief |
| --- | --- | --- |
| PRD §3.1 / SDS Logical Design #5 | LVEF one-liner (page 1, portal, VCardia) | Fixed one-liner text by outcome, gated by Make-Results-Available toggle; pending text when processing |
| PRD §3.2 | LVEF Risk Assessment second page — trigger | Second page appended only when both toggles ON + successful prediction |
| PRD §3.3 / SDS Logical Design #8 | LVEF Risk Assessment second page — format | Logo, patient header, ECG image, result text, disclaimer, education, QR code |
| PRD §3.4 / SDS Logical Design #1–#2 | Customer Portal & VCardia — removal of standalone section/actions | Remove section + Download action; standalone status label added |
| PRD §3.5 / SDS Logical Design #4 | TA — LVEF config block update | Rename toggle, new toggle, Segment Type relocation/redefinition, Selective-Submission removal |
| PRD §3.6 / SDS Logical Design #1–#2 | TA ECG case detail — removal of section/button | Remove LVEF section + View LVEF button; other Case Actions unaffected |
| PRD §3.7 / SDS Logical Design #6 | Case list & detail — LVEF result label | 5-state label (incl. "LVEF Failed," confirmed only by SDS/design) |
| PRD §3.8 / SDS Logical Design #3 | Email consolidation | Single email replacing ECG + LVEF emails, gated by both toggles |
| PRD §3.9–§3.10 / SDS Logical Design #7 | Report delivery coordination + subsequent delivery | 1-min wait, TAT cap, subsequent consolidated dispatch, stuck-ECG recovery |
| PRD §3.11 / SDS §Logical Design (interpretation field) | LVEF interpretation storage separation | Dedicated field, never mixed with ECG interpretation field |
| PRD §3.12 / §3.13 | LVEF disclaimer append — Doctor & AI Reporting | Third conditional disclaimer append on page 1 |
| PRD §4.1–§4.5 / SDS NFR section | Non-functional requirements | Performance, availability/reliability, security, backward compatibility — SDS gives concrete targets and validation methods for each |
| SDS Logical Design #9–#13 (not in PRD at all) | Webhook LVEF field, RB-template-only support, deprecated API responses, stuck-ECG recovery script, Retry-Medical-AI bugfix | Real scope, absent from the PRD's own functional-requirements section — see OI list |

**Out of scope:**
- LVEF AI model / algorithm changes (PRD line 64 — "no AI changes are in scope"): the prediction logic itself is a black box to this VP; only its inputs/outputs are verified.
- Any device/UX redesign beyond the LVEF surfaces listed in the table above: e.g. unrelated Case Actions, non-LVEF report sections.
- Automation tooling procurement/selection (OI-21): this VP defines which scenarios are automation candidates (§6.9), not which framework or CI/CD pipeline gets adopted — that is an engineering/QA-infrastructure decision outside this VP's scope.
- Testing SOP creation, Risk Report Template population, or Secure Development Process review (§18 Appendix): referenced as existing org artifacts, not deliverables of this VP.

### 2.1 Module Overview

(Requirement counts derived from the Scope table above; some module boundaries are not distinctly specified in the PRD/SDS and are marked as such rather than guessed — added per Checker review, Required Structure §7)

| Module | Requirement rows touched (§2) | Platform(s) |
| --- | --- | --- |
| Tricog Admin (TA) — Server & Client | 5 (config block, case-detail removal, case-list label, NFRs, deprecated-API handling) | Web (Server & Client) |
| Customer Portal | 3 (one-liner, standalone-section removal, case-list label) | Web |
| VCardia | 3 (one-liner, standalone-section removal, case-list label) | Android & iOS |
| Cardionet V2 | Not distinctly specified — named in the Impact Doc's component list but no PRD/SDS requirement row maps to it separately from TA/Portal; tracked as **OI-25** | Web (per Impact Doc assumption, §13) |
| Boron Server (VCardia backend) | 4 (report delivery coordination, interpretation field separation, webhook/RB-only/deprecated-API/recovery-script scope, NFRs) | Backend |
| Transformer (Cronjob/polling) | 2 (report delivery coordination, email consolidation notify-trigger) | Backend |
| Report Builder (RB templates) | 4 (one-liner rendering, second-page trigger/format, disclaimer append, RB-only template support) | Backend (template engine) |

---

## 3. System Risk Profile

> **⛔ OI-24 is a Maker-external Blocker** (elevated to this framing per cross-draft consistency check, `verification-plan-guardrails-nr.md` §5 — the parallel VP for this same release treats the identical gap as its own gating blocker, OG-24): the HIGH/MEDIUM/LOW table below is a **qualitative risk-to-testing-priority ranking only**, not a hazard-ID-level ISO 14971 risk-control register. No ISO-14971 Risk Management Report or hazard register with per-hazard IDs, severity/probability scoring, or risk-control-measure references exists anywhere in the PRD/Impact Doc/SDS for this feature. Without hazard IDs, this VP **cannot** build the Hazard-ID → Test-Case-ID traceability a mandatory guardrail (VP-40a · M2) requires — it can only offer the coarser Module → test-row mapping shown in §2/§6.3. This is not something HP can resolve by adding more analysis to this VP; it requires **Product/QMS to supply the actual hazard register**. Until then, §3's table remains directional prioritisation input only, and this VP's own Draft → Approved progression is gated on OI-24 alongside any other open Blocker.

| Module | Risk | Rationale |
| --- | --- | --- |
| Report delivery coordination (queue/cron logic) | HIGH | A stuck or mis-timed dispatch is a direct path to a clinician never receiving the LVEF finding, or receiving it contradicted by a stale report. |
| TA LVEF config block (both toggles, Segment Type) | HIGH | The "Make LVEF Results Available" toggle is an explicit **regulatory gate** (Impact Doc: "surfacing AI-derived LVEF results in geographies where the AI is not approved for use is not permitted"). A config bug here is a compliance failure, not just a UI bug. |
| Report Builder template rendering (page 1 + page 2) | HIGH | Incorrect clinical text (wrong outcome, wrong disclaimer) reaching a clinician is a direct patient-safety risk. |
| LVEF interpretation field separation | HIGH | Contamination of the ECG interpretation field would corrupt the internal algorithm team's pipeline — an explicit non-negotiable in both PRD (§3.11) and SDS NFR (Backward Compatibility). |
| Case-list/detail label logic (5 states) | MEDIUM | Wrong or missing labels are a triage/usability risk rather than a direct safety miss, but "LVEF Error"/"LVEF Failed" ambiguity (OI-list) raises this toward the risk boundary. |
| Consolidated email / notifications | MEDIUM | Wrong or missing LVEF content in email is a workflow-continuity risk, mitigated by the report itself being the primary channel. |
| Stuck-ECG recovery script (SDS #12) | MEDIUM | Its entire purpose is to catch reports that would otherwise be silently lost — a failure here compounds an already-rare failure mode. |
| Segment Type / Selective Submission migration | MEDIUM | Incomplete data migration risks silent misclassification of existing clinics, but has no direct patient-facing effect. |
| Webhook LVEF field / Bridge Health integration | LOW | Opt-in, additive, currently affects a very small customer set per the Impact Doc's own investigation (one PNG-format customer found, LVEF not enabled for them). |
| Deprecated Selective-Submission removal | LOW | Feature removal for a deprecated, low-usage path; existing clinics unaffected until migration. |

---

## 4. Test Methodology & Approach

Applicable test types: **Functional** (all FRs above) · **Integration** (ECG↔LVEF service coordination via `wfm-to-sp` queue, Redis state, Medical AI polling) · **Regression** (existing report/disclaimer/email/notification flows, existing BP/Weight-equivalent unaffected surfaces) · **Non-Functional** (SDS gives concrete targets — see §6.2) · **Data Migration Testing** (Segment Type, Selective-Submission deprecation datafixes) · **Exploratory** (mid-flight toggle changes during Pending Diagnosis, race conditions around the 1-minute delay) · **UAT / Regulatory Review** (the toggle is a named regulatory gate; the double-critical-alarm behaviour is a named, risk-accepted decision — SDS §11 Q4 — that should be UAT-confirmed, not just functionally verified).

BBT techniques selected as relevant (techniques not grounded in this feature are not padded in):

| Technique | Applies to |
| --- | --- |
| Decision Table | Two-toggle × AI-outcome combinations (§3.2); cohort-independent config combinations |
| Boundary Value Analysis | 1-minute wait boundary, 10-minute TAT cap, 20-second poll interval, 5-attempt retry limit, 600-second minimum billing SLA |
| State Transition | Case/report lifecycle (waiting → available/timeout → subsequent delivery); `isWaitingForLVEF` flag transitions |
| Equivalence Partitioning | `assessmentStatus` × `riskIndicator` combinations driving every label/one-liner state |
| Configuration Testing | Segment Type migration (5 old values → 3 new), Selective-Submission removal, template inclusion/exclusion list |
| Audit / Compliance Testing | Every named audit action (§6.2/SDS) — config changes, force-generate, re-generation, held/repushed cases |
| Regression Testing | Existing disclaimer text, existing single ECG/LVEF emails for toggle-OFF clinics, existing Case Actions |
| Negative / Error-Guessing | Deprecated Selective-Submission API hit by an old mobile build; malformed/null LVEF result; Medical AI down mid-poll |
| Security / RBAC | Webhook opt-in field scoped correctly; audit logs capture actor without exposing PHI |
| Database / Storage-Separation | ECG vs LVEF interpretation field isolation |
| Localization Testing | LVEF content English-only even inside a non-English Custom Language Template |
| Cross-Template Compatibility | Consolidated report supported **only** on RB templates — explicit exclusion list (DEFAULT, LISTER, IMAGE_NO_BORDER, REPORT_WITH_VITALS, MEDICARD_LIFESTYLE) must be verified, not assumed |
| End-to-End | Full acquisition → diagnosis → LVEF wait → dispatch → (subsequent dispatch) flow |
| Email / Notification Channel | Consolidated email, web/push notification title append, Slack alert channel for stuck-ECG recovery |

---

## 5. Testing Process

Standard flow — analyze PRD/Impact/SDS/design (done for this VP; see the Open Issues in §13 for what remains genuinely unresolved across them) → author test cases mapped to PRD requirement IDs, using the org's Test Case YAML template → sanity check on build receipt → **defects tracked in Jira** (`tricog1.atlassian.net`, per the SDS's own referenced tickets — e.g. `OLPD-5848`, `ADSRQ-162`, `ADSRQ-298` — this product's actual tracker, unlike KeeboHealth's GitHub-Issues convention) → regression after fixes → summary report → **sign-off = NR's merge/approval**, per the maker-checker gate (HP drafts and self-reviews against `test-execution-guardrails-hp.md`/`test-execution-skillset-hp.md` at the execution stage; this VP itself is reviewed against the org's `VerificationPlan_Guardrails`).

---

## 6. Testing Strategy

### 6.1 Feature Testing

**Feature testing scope** (includes the design-only master radio discovered in the wireframe — not documented in PRD/Impact/SDS, see OI-10)

| LVEF Risk Assessment | Make Results Available | Include Addendum | AI Outcome | Test Type |
| --- | --- | --- | --- | --- |
| Medical AI | ON | ON | High risk | Feature Testing |
| Medical AI | ON | ON | Low risk | Feature Testing |
| Medical AI | ON | OFF | High/Low risk | Feature Testing |
| Medical AI | OFF | (n/a — addendum disabled per design, page 44) | any | Feature Testing (negative path) |
| **None** | n/a | n/a | n/a | Feature Testing (negative path — service fully off for the clinic) |

**Config/flag behaviour matrix**

| Configuration | Flag | Condition | Page 1 | Page 2 | Portal/VCardia | Expected |
| --- | --- | --- | --- | --- | --- | --- |
| Make Results Available | OFF | any AI outcome | no one-liner | no page | no label | Single-page report, no LVEF content anywhere (PRD §3.2 AC3) |
| Make Results Available | ON, Addendum OFF | successful prediction | one-liner shown | **not** appended | label shown | Single page + one-liner + label, no page 2 (PRD §3.2 AC4) |
| Make Results Available | ON, Addendum ON | successful prediction | one-liner shown | appended | label shown | Full 2-page consolidated report |
| Make Results Available | ON (either addendum state) | pending | "LVEF Risk Assessment is pending." | not appended | "Pending LVEF" | Confirm **exact capitalisation** — the design deck shows one instance rendered lowercase (OI-12); this must be tested as a literal string match, not eyeballed |

**Outcome × state matrix** (5 states — the 5th, "Failed," confirmed only by SDS `LVEF_ONE_LINERS` config and the design deck's "Failed LVEF" label, absent from the PRD's own Definitions §1.7)

| Outcome | Case list | Case/ECG detail | Report/one-liner |
| --- | --- | --- | --- |
| High risk of LVSD | "Reduced LVEF" (red) | "Reduced LVEF" (red) | High-risk one-liner, red on page 2 |
| Low risk of LVSD | "No reduced LVEF" (green) | "No reduced LVEF" (green) | Low-risk one-liner, green on page 2 |
| Pending (IN_PROGRESS/SHEET_ID_GENERATED, null indicator) | "Pending LVEF" | "Pending LVEF" | "LVEF Risk Assessment is pending." |
| Errored (Medical AI rejected — quality) | not shown | "LVEF Error" | no one-liner, no fallback text |
| **Failed** (trigger condition undefined — OI-9) | not shown | "LVEF Failed" | no one-liner, no fallback text |
| Not eligible (diagnosis doesn't match trigger config) | not shown | not shown | not shown — explicitly expected, not an error |

**Timing / coordination matrix**

| Scenario | Input | Expected | Test type |
| --- | --- | --- | --- |
| LVEF result within 1 minute | Result arrives at t=59s | Consolidated report dispatched | Feature |
| LVEF result exactly at 1-minute boundary | Result arrives at t=60s / 61s | Boundary behaviour confirmed explicitly (PRD doesn't specify inclusive/exclusive) | Edge Case — OI |
| LVEF not available, TAT not expired | t=60s no result | ECG-only dispatched + pending notification; monitoring continues | Feature |
| TAT already expired at diagnosis time | diagnosis completes after 10-min cap | No wait applied; ECG-only dispatched immediately | Feature |
| LVEF becomes available after ECG-only dispatch | subsequent arrival | Consolidated report dispatched automatically, notification 2 fires | Feature |
| Polling exhausts 5 attempts (SDS-confirmed count, not the PRD Conclusion's "4" — OI-2) | 5 failed polls | Fallback triggers per §4.2.1/§6.2 NFR-Availability | Boundary |
| Message "lost" after 1-min delivery delay | simulated queue drop | Stuck-ECG recovery script detects via `LVEF_AWAITING_ECGS`, re-pushes, Slack-alerts on repeated failure | Edge Case — new scope, not in PRD (OI-6) |
| TA Force Generate, LVEF available | manual trigger | Consolidated report generated immediately | Feature |
| TA Force Generate, LVEF not available | manual trigger | ECG-only report with pending notice generated immediately | Feature |
| STEMI-critical case gets two critical alarms (initial + subsequent) | accepted risk per SDS §11 Q4 | **Confirm** the double alarm occurs — this is accepted behaviour, not a defect to suppress | Feature — risk-acceptance confirmation |

**Segmentation / targeting matrix**

| Segment | Applies to | Expected |
| --- | --- | --- |
| Segment Type = Pilot / Test-Internal User / Paid | new clinics | Single-select, first field after the master LVEF Risk Assessment control |
| Existing clinics on old Segment H/M/L/Test/Pilot values (SDS: 5 old values, not the PRD's stated 3 — OI-7) | migration | Old→new mapping **not specified anywhere** — must be confirmed before this can be tested meaningfully (OI-7) |
| Perform LVEF Risk Assessment for = All Cases | any clinic | LVEF runs without waiting for doctor diagnosis |
| Perform LVEF Risk Assessment for = Diagnosis-Based Selective (Normal/Abnormal/Critical checkboxes — confirmed by design, not detailed in PRD text) | any clinic | Only checked diagnosis types forwarded to LVEF AI |
| Perform LVEF Risk Assessment for = User Selected Cases (deprecated) | existing clinics only | Option removed from TA UI; existing clinics continue functioning until migration; old mobile app calling the deprecated endpoint gets an explicit deprecation response |

**Notification / output-channel matrix**

| Situation | Expected |
| --- | --- |
| Consolidated email, both toggles ON | Single email; body includes Center Name/Patient ID/Age/Acquired-on **and**, per the design mockups (not the PRD's literal field list — OI-13), Gender/Status/Interpretation; one Download Report button |
| Either toggle OFF | Standard ECG email only, no LVEF email |
| AI produced no result | Email body: "LVEF Risk Assessment result unavailable." — reconcile against PRD §3.1 ACs 8/9, which say **no fallback text** should appear elsewhere for the same underlying no-result case (OI-3) |
| Push/web notification title | Appends "reduced LVEF" / "no reduced LVEF" per SDS; desktop notification format confirmed in design mockup |
| Stuck-ECG recovery failure after max retries | Slack alert to `lvef-ecg-monitoring` with case ID, attempt count, held-at timestamp |

**Performance matrix** (SDS gives real targets — unlike a spec-free feature, these are not invented)

| Scenario | Target (SDS) | Validation method (SDS-stated) |
| --- | --- | --- |
| Consolidated report load, Portal/VCardia | ≤ 6 seconds | Performance testing |
| Case list load overhead from LVEF label | ≤ +200ms at P95 | Performance testing & Query Execution Plan |
| Polling interval / attempts | ≥20s interval, ≤5 attempts | Functional, Load & Performance testing |
| Report generation overhead from LVEF logic | ≤ +500ms vs baseline | Functional, Load & Performance testing |

### 6.2 NFR Verification Matrix

(Populated directly from the SDS's own NFR section, including its stated validation methods — grounded, not inferred)

| NFR | Requirement | Target | Validation method (per SDS) | Risk (§3) | Automation Candidate |
| --- | --- | --- | --- | --- | --- |
| Performance | Consolidated report load time | ≤6s | Performance testing | MEDIUM | Auto — scripted load measurement, repeated runs |
| Performance | Case list latency overhead | ≤200ms P95 | Performance testing & Query Execution Plan | MEDIUM | Auto — scripted P95 measurement + query plan capture |
| Performance | Polling interval/attempts | ≥20s / ≤5 attempts | Functional, Load & Performance testing | HIGH (report delivery coordination) | Auto — mocked Medical AI, attempt/interval assertion |
| Performance | Report generation latency overhead | ≤500ms | Functional, Load & Performance testing | HIGH (report delivery coordination) | Auto — scripted timing delta vs. baseline |
| Availability/Reliability | Polling fallback on degradation | Stops calling Medical AI on degradation | Functional testing of the caching mechanism + "ECG sent alone" path | HIGH (report delivery coordination) | Auto — mocked degradation, call-count assertion |
| Availability/Reliability | Report dispatch guarantee | No missed/silently dropped reports | Audit Logs, Functional & Load Testing | HIGH (report delivery coordination) | Auto — audit-log/DB completeness query |
| Availability/Reliability | Graceful handling of missing/null/malformed LVEF result | No error state on any surface | Functional Testing | HIGH (report delivery coordination) | Auto — data-driven negative-input regression |
| Security | AuthN/AuthZ on all API communication | Enforced | Not detailed beyond this statement in the SDS — OI | HIGH (TA LVEF config block is a named regulatory gate) | Partially auto — baseline 401/403 assertions scriptable now; full coverage blocked on the OI until concrete auth rules are specified |
| Security | Audit logs capture admin/workflow events without exposing PHI | — | Not detailed beyond this statement in the SDS — OI | HIGH (TA LVEF config block is a named regulatory gate) | Auto — automated PHI-pattern scan of captured audit-log payloads, once the OI defines what "admin/workflow events" concretely means |
| Usability | Case-list/detail label & one-liner copy-exactness (5-state) | Not defined as a formal NFR by the SDS — OI-20 | OI-9 (Failed vs Errored trigger undefined) and OI-12 (copy-exactness defects in design deck) are usability-relevant findings with no NFR category to trace back to; surfaced here by Checker review, not invented as if the SDS specified it | MEDIUM (case-list/detail label logic, §3) | Manual (visual/copy) — automatable only with visual-regression tooling, not confirmed in stack (OI-21) |
| Backward Compatibility | ECG interpretation field integrity | Unmodified by LVEF append | Functional Testing | HIGH (interpretation field separation, §3) | Auto — DB field-content regression assertion |
| Backward Compatibility | Graceful handling of missing LVEF config | No crash, features hidden | Functional Testing | MEDIUM | Auto — config-absent regression scenario |
| Auditability | Every named admin/workflow action (§6.8's audit-log action-name list) produces a corresponding, correctly-attributed audit log entry | 100% of named actions logged | Audit Logs (grounded in §6.8's action-name list, not invented — added per Checker review, Required Structure §10) | HIGH (TA LVEF config block is a named regulatory gate) | Auto — DB/audit-log query per action name |
| Scalability | Behavior under production-representative case volume / concurrent clinics | Not stated anywhere in the PRD/Impact Doc/SDS beyond the specific latency targets above — tracked as **OI-26** | Not detailed — OI-26 (added per Checker review, Required Structure §10) | MEDIUM | Not automatable until OI-26 defines a target |

### 6.3 Test Combination Matrix — Black-Box Test Design

| # | Technique | Combination | Expected | Decision | Automation Candidate |
| --- | --- | --- | --- | --- | --- |
| 1 | Decision Table | Make-Available × Addendum × AI-outcome (full cross) | Per §6.1 config/flag matrix | TEST | Auto — API/DB, data-driven |
| 2 | Boundary Value Analysis | LVEF arrival at 59s / 60s / 61s relative to the 1-min wait | Exact boundary behaviour confirmed (currently unspecified — OI) | TEST — GAP-FILLER | Auto once boundary confirmed — time-mocked, blocked on OI |
| 3 | Boundary Value Analysis | Diagnosis completion at TAT 9:59 / 10:00 / 10:01 | Exact cutoff behaviour confirmed against the SDS's own unclear TAT formula (OI-8) | TEST — GAP-FILLER | Auto once boundary confirmed — time-mocked, blocked on OI-8 |
| 4 | Boundary Value Analysis | Poll attempt 4 vs 5 vs 6 | Confirms 5 is the real cutoff (SDS), not 4 (PRD Conclusion) | TEST | Auto — pollable via mocked Medical AI |
| 5 | State Transition | `isWaitingForLVEF` flag: unset → set → resolved/timeout | Matches SDS Logical Design #7 flow | TEST | Auto — Redis-state check, no UI needed |
| 6 | Equivalence Partitioning | assessmentStatus × riskIndicator → each of the 5 label states | Correct label per combination | TEST | Auto — data-driven, pure combination |
| 7 | Configuration | Existing clinic with Segment = "Test" (pre-existing value per SDS, absent from PRD) migrated to new options | Mapping undefined — TEST cannot be designed until OI-7 resolved | TEST — GAP-FILLER (blocked) | Not automatable until OI-7 resolved |
| 8 | Configuration | Old mobile app version calls deprecated Selective-Submission endpoint | HTTP 410 + one of two conflicting response bodies (SDS shows both — OI-14) | TEST — GAP-FILLER | Auto — HTTP status/body assertion |
| 9 | Regression | Existing disclaimer flow before this release's disclaimer replacement | Impact Doc frames PRD's "current" text as new — verify actual pre-release production text first (OI-11) | TEST — GAP-FILLER | Manual first pass (OI-11 baseline capture), then automatable as regression |
| 10 | Negative | Webhook customer with Webhook Report Format = PNG | Only page 1 converts; LVEF page 2 lost — confirm this is accepted, not silently broken | TEST | Auto — webhook payload/API check |
| 11 | Cross-Template | Consolidated report requested on an excluded template (e.g. DEFAULT) | Not supported — confirm graceful fallback, not a crash | TEST | Auto — API/report-generation response check |
| 12 | Localization | LVEF content on a non-English Custom Language Template | Displays in English only, rest of report in configured language | TEST | Manual (visual) — automatable only with visual-regression tooling, not confirmed in stack (OI-21) |
| 13 | Audit/Compliance | Toggle changed, Force Generate used, report re-generated | All three produce distinct, correctly-populated audit log entries (action names per SDS) | TEST | Auto — DB/audit-log query assertion |
| 14 | Reliability | Simulated stuck ECG (queue message dropped after 1-min delay) | Recovery script detects, re-pushes; on repeated failure, Slack alert fires | TEST | Auto — scriptable queue-drop simulation |
| 15 | Risk-acceptance | STEMI-critical case, ECG-only then subsequent consolidated dispatch | Two critical alarms fire — confirm as accepted (SDS §11 Q4), not suppressed | TEST | Manual — clinical-alarm confirmation, UAT-style |
| 16 | Regression | "Failed LVEF" vs "LVEF Error" — both surfaced identically? | Trigger condition for each undefined (OI-9) — TEST cannot fully distinguish causes until resolved | TEST — GAP-FILLER (blocked) | Not automatable until OI-9 resolved |
| 17 | Security/RBAC | Webhook `sendLvefInWebhook` key present/absent, LVEF enabled/disabled | Field appears only when both conditions true, only on `/api/v1/external/report` | TEST | Auto — API field-presence assertion |

Combination totals: **17 combinations defined · 13 TEST · 4 TEST-GAP-FILLER (of which 2 are currently blocked on unresolved Open Issues, not just newly added coverage).** Automation totals: **11 Auto-ready · 2 Auto-once-unblocked · 2 Manual (visual/clinical judgment) · 2 not automatable until blocking OIs resolve** — see §6.9.

### 6.4 Coverage Analysis

| Coverage Dimension | Total Partitions | Covered | % | Backed by |
| --- | --- | --- | --- | --- |
| Toggle combinations | 4 (2×2) + master radio negative path | 5 | 100% | Rows 1, §6.1 |
| Label/outcome states | 5 (High/Low/Pending/Errored/Failed — NotEligible is a 6th "no label" state) | 6 | 100% surfaced, but **Failed vs Errored trigger distinction unresolved** | Row 6, 16 |
| Timing boundaries | 1-min wait, 10-min TAT, poll interval/count | 3 | 100% surfaced, **exact boundary semantics unconfirmed** | Rows 2–4 |
| Segment Type migration | 5 old → 3 new values | Cannot be scored — mapping undefined | 0% design-ready | Row 7 (blocked) |
| Template compatibility | RB-supported vs 5 named excluded templates | 2 | 100% | Row 11 |
| NFR categories | 4 SDS-defined + 2 SDS-acknowledged-but-undetailed (security) + 1 Checker-surfaced gap (usability, OI-20) | 4 fully, 2 partially, 1 not scoreable | 57% fully specified | §6.2 |
| **Weighted Total** | — | — | **High on documented, testable dimensions; two dimensions are explicitly blocked, not silently passed** | — |

Residual/out-of-scope (per PRD line 64): LVEF AI model/algorithm itself.

### 6.5 Regression Testing

| Flow | Expectation |
| --- | --- |
| Existing ECG report generation for non-LVEF clinics | Fully unaffected |
| Existing disclaimer text (pre-this-release) | Must be captured from actual current production before asserting what "append" means — see OI-11 |
| Existing single ECG email for toggle-OFF clinics | Unaffected, no LVEF email sent |
| Existing Case Actions (Resend, Resend SMS/EMAIL/APP, Prioritize, Force Diagnose, Clone, View Report) | All unaffected by the two removed elements |
| Existing clinics on User Selected Cases (pre-migration) | Continue functioning exactly as before until migration completes |
| Old app versions (iOS ≤6.11.00, Android ≤9.5.2) | Consolidated report **not available**; must degrade to single-page report without crashing, per Impact Doc |
| New (target-release) VCardia build talking to a not-yet-upgraded Boron Server, during staged rollout | Added per Checker review M5/VP-62a — must degrade gracefully (consolidated-report-dependent calls fail closed to ECG-only behavior), no crash; this is the reverse direction of the Old-App/New-Backend row above |
| Device-level mixed-app-version coexistence: legacy and target-release VCardia users active concurrently in the same clinic, against shared clinic-level config/case state | Added per Checker review VP-66 — each user's app version behaves consistently with its own compatibility tier; no shared-state corruption or crash from the version mismatch during staged rollout |

### 6.6 Platform / browser versions

Impact Doc names the versions that will **not** get the consolidated report (iOS v6.11.00, Android v9.5.2) but no target "latest supported" version is stated anywhere — **Open Issue (OI-1)**.

### 6.7 Test data

- Clinics spanning: LVEF Risk Assessment = None / Medical AI; both toggle states; All-Cases vs Selective (each of Normal/Abnormal/Critical); each of the 3 new + (if resolvable) old Segment Type values.
- Cases spanning all 6 label/outcome states, including at least one genuinely "Failed" (not "Errored") case once that distinction is resolved.
- A webhook-enabled clinic with `sendLvefInWebhook` set, and one with Webhook Report Format = PNG.
- A clinic on an excluded (non-RB) template, and one on a Custom Language (non-English) template.
- Simulated old app-version clients for the deprecated Selective-Submission endpoint and for pre-consolidated-report compatibility.
- All patient data synthetic — no real PHI.

### 6.8 Test Results

Captured with objective evidence per the org's Test Case YAML `execution`/`evidence` fields (see `test-execution-guardrails-hp.md`), stored per this product's actual practice — Jira (`tricog1.atlassian.net`) for defects, linked audit-log entries (action names per SDS: `ECG_LVEF_WAITING`, `ECG_LVEF_NO_WAIT`, `ECG_LVEF_RESULT_AVAILABLE`, `ECG_LVEF_WAIT_TIMEOUT`, `ECG_LVEF_RESULT_AVAILABLE_AFTER_TIMEOUT`, `ECG_LVEF_HELD_REPUSHED`, `ECG_LVEF_HELD_REPUSH_FAILED`, `ECG_REPORT_FORCE_GENERATED`, `CONSOLIDATED_REPORT_GENERATED`, `LVEF_CONSOLIDATED_EMAIL`) as part of the evidence trail.

### 6.9 Automation Strategy

No test automation framework, CI/CD pipeline, or automation tooling is named anywhere in the PRD, Impact Document, or SDS for this feature — the SDS's only tooling reference (Postman, for updating Report Builder templates) is a content-update mechanism, not a test-execution framework. This is tracked as **OI-21**; the automation scope below is provisional against the Test Combination Matrix (§6.3) and must be confirmed against the QA team's actual automation stack before Stage ② test cases are authored as automated.

**Risk-based automation priority (per §3 System Risk Profile):** automation build-out is sequenced by risk, not by ease of scripting —
1. **Report delivery coordination (HIGH)** — the four Performance/Availability rows in §6.2 tied to this module (polling fallback, dispatch guarantee, malformed-result handling, generation latency) are all Auto-ready and should be automated first: this is the module where a missed or mistimed dispatch is a direct clinician-facing miss.
2. **TA LVEF config block / regulatory gate (HIGH)** — the two Security NFR rows (§6.2) are only *partially* automatable today (baseline AuthN/AuthZ assertions, yes; full PHI-exposure and admin-event coverage, no) because the SDS never states concrete auth rules or what "admin/workflow events" covers (OI). Automating only the partial slice risks a false sense of coverage on a regulatory-gated feature — the OI must be resolved before this is reported as "automated" rather than "partially automated."
3. **Interpretation field separation (HIGH)** — the Backward Compatibility field-integrity row (§6.2) is Auto-ready (DB assertion) and should be in the first automation batch alongside (1), since a silent regression here corrupts the algorithm team's pipeline.
4. **Report Builder template rendering (HIGH)** — covered indirectly by §6.3 rows 1, 11 (decision-table, cross-template); both are Auto-ready.
5. **MEDIUM-risk items** (case-list/detail labels, consolidated email, stuck-ECG recovery, Segment Type migration) are automated after the HIGH-risk batch above, in the order listed in §3 — except the Usability NFR row, which stays manual regardless of risk ranking until visual-regression tooling is confirmed (OI-21).
6. **LOW-risk items** (webhook/Bridge Health, deprecated Selective-Submission removal) are automated last, opportunistically, once the HIGH/MEDIUM batches are stable.

**Good automation candidates** (deterministic, backend/API/DB-driven, no clinical judgment required):
- Decision-table / equivalence-partitioning rows (§6.3 rows 1, 6) — pure config × outcome combinations, ideal for data-driven automated regression.
- Boundary-value rows (§6.3 rows 2–4) — once the OIs blocking exact boundary semantics (rows 2–3) are resolved, these suit time-mocked automated tests rather than real-clock waits.
- State-transition row (§6.3 row 5) — `isWaitingForLVEF` flag transitions are backend/Redis-state-driven and scriptable without UI interaction.
- API/webhook/audit rows (§6.3 rows 8, 10, 11, 13, 17) — status codes, response bodies, webhook field presence, and audit log entries are all API/DB-checkable without a human reading a screen.
- NFR/performance rows (§6.2) — load time, latency overhead, and generation-time targets are inherently suited to scripted, repeated measurement rather than one-off manual timing.
- Reliability/recovery row (§6.3 row 14) — a simulated dropped queue message and recovery-script detection is scriptable end-to-end.

**Not automation candidates for this cycle** (require human judgment, visual confirmation, or are currently blocked):
- UAT/regulatory-review row (§6.3 row 15, the risk-accepted double alarm) — confirming a real alarm fires as clinically intended is a human confirmation step, not a scripted assertion.
- Copy-exactness/localization checks (OI-12, §6.3 row 12) — automatable only with a visual-regression tool, not confirmed to exist in this org's stack; treat as manual until OI-21 is resolved.
- Exploratory scenarios (mid-flight toggle changes, race conditions around the 1-minute delay) — by definition not scripted in advance.
- Currently-blocked rows (§6.3 rows 7, 16 — undefined migration mapping, undefined Failed/Errored trigger) — cannot be automated, or even manually test-designed, until OI-7/OI-9 resolve.

**Carried into Stage ② (Test Case Writing):** each Test Case authored from this VP should carry an explicit `automatable: yes/no` field (extending the org's Test Case YAML baseline template) so automation scope is decided once, here, rather than re-litigated per case at execution time.

### 6.10 Performance Testing Approach

**Manual:** an initial exploratory timing pass (qualitative spot-check of report load, case-list load, and generation time) during functional testing, before any scripted measurement exists — per `test-execution-guardrails-hp.md` §4.4/§4.5, this qualitative result is recorded and never substituted for a real measurement against the SDS's numeric targets (§6.2).

**Automated:** scripted, repeated measurement against each §6.2 Performance target (report load ≤6s, case-list overhead ≤200ms P95, poll interval ≥20s/≤5 attempts, generation overhead ≤500ms) at production-representative load — per guardrail §4.5, never at a smaller data volume/concurrency than the real system sees. Tooling TBD (OI-21).

**Open gap:** the SDS states each numeric target but never states a minimum run count or sample size for statistical validity (e.g. "median & P95 over ≥N runs") — tracked as **OI-22**. Neither the manual nor the automated approach above invents a number to fill this; both are blocked from being called "complete" against guardrail §4.3 until QA/Performance defines one.

### 6.11 Security Testing Approach

**Manual:** role-based walkthrough confirming only authorized roles can toggle "Make Results Available" / "Include LVEF Report Addendum" (the named regulatory gate, §3); manual spot-check of captured audit-log payloads for PHI leakage; manual verification that the webhook `sendLvefInWebhook` field (§6.3 row 17) is scoped only to opted-in, LVEF-enabled clinics.

**Automated:** baseline AuthN/AuthZ assertions (401/403 on unauthenticated/unauthorized calls) and the webhook field-presence check (§6.3 row 17) are scriptable now. Full audit-log PHI-exposure coverage and complete AuthZ-rule coverage are **not** — the SDS states only "Authentication and authorization shall be enforced for all API communications" with no concrete rule set (existing OI, §6.2), so automated coverage here is partial by necessity, not by choice. Reporting this as "automated" without that caveat would overstate coverage on a regulatory-gated feature.

### 6.12 Database Testing Approach

**Manual:** direct before/after inspection of the LVEF interpretation field vs. the ECG interpretation field (§3 HIGH risk item) — per guardrail §4.2, never inferred from an app-level success message alone; and manual inspection of the Segment Type migration's actual stored values once OI-7's mapping is resolved (cannot be meaningfully tested before then).

**Automated:** scripted DB assertions for field-integrity regression (§6.2 Backward Compatibility row), audit-log completeness/action-name correctness (§6.3 row 13), and report-dispatch-guarantee queries (§6.2 Availability/Reliability row) — all state changes, not just the UI outcome they produce.

### 6.13 Risk-Based Testing Approach (Manual + Automated)

Every **HIGH**-risk item in §3 gets both a manual confirmation step and automated regression — neither alone is treated as sufficient for a HIGH item:

| Risk item (§3) | Manual step | Automated step |
| --- | --- | --- |
| Report delivery coordination | Exploratory mid-flight/race-condition testing (timing can't be fully scripted in advance) | §6.2/§6.9 scripted timing, fallback, and dispatch-guarantee checks |
| TA LVEF config block (regulatory gate) | Role-based manual walkthrough (§6.11); UAT/regulatory sign-off per §4's stated approach | Partial — baseline AuthN/AuthZ + webhook field checks only (§6.11) |
| Report Builder template rendering | Manual visual confirmation of page 1/page 2 clinical text and disclaimers | §6.3 rows 1, 11 (decision-table, cross-template) |
| LVEF interpretation field separation | Manual DB inspection (§6.12) on first execution cycle | Scripted regression assertion thereafter (§6.12) |

MEDIUM/LOW items follow the same manual-plus-automated pattern per §6.9's build-out order, except the Usability NFR row (OI-20) and the risk-accepted double-alarm scenario (§6.3 row 15), which stay manual-only regardless of risk ranking — see §6.9/§6.11 for why.

### 6.14 Per-Endpoint API Contract Matrix

Added per Checker review VP-68. Endpoints named explicitly in the SDS, tested individually (schema, status codes, auth) rather than folded only into feature-level assertions. **HTTP method is not stated for any of these in the SDS** — tracked as **OI-23** — so none is invented here.

| Endpoint | Owning component | Change under this release | Contract check |
| --- | --- | --- | --- |
| `/v1/ecg/:id/pdf/:pdftype` | TA Server | Deprecate standalone LVEF report functionality | Response no longer includes the deprecated LVEF report path; status code for the deprecated path per OI-14's two-response ambiguity |
| `/ecg/:ecgId` (ECG Details API) | TA Server | Deprecate LVEF Report URL field | Response schema no longer includes the deprecated field; existing fields unaffected |
| `extras/custom/fields/:name` | TA Server | Deprecate LVEF Selective Submission config (`LVEF_AI_PREDICTION`) | Config field removed/deprecated cleanly; old clients reading this config degrade per OI-14 |
| `/ecg/:ecgId/lvef/status` | TA Server | Deprecated Selective-Submission status API — old mobile builds must get an explicit deprecation response | **This is the OI-14 endpoint** — SDS shows two conflicting response bodies for old-mobile-app calls; contract test blocked until OI-14 resolves |
| `/ecg/${ecgId}/diagnosis/notify/email?emailType=lvef` | Transformer (Cronjob) | Reference endpoint for LVEF email notification trigger | Fires only under the consolidated-email gating rules (§6.1 notification matrix) |
| `/ecg/all` | TA Server | Updated to fetch/send LVEF standalone status for the ECG list | Response includes correct 5-state label (§6.1 outcome × state matrix) per case |
| `ecg/retry/lvef/:caseId` | TA Server / Boron Server | Retry-Medical-AI bugfix (SDS scope, OI-6 — no PRD-level spec) | Retry only succeeds for eligible cases; known gap for FAILED assessmentStatus + Diagnosis-Based Selective config (per SDS's own stated limitation) |
| `/api/v1/external/report` | Boron Server (webhook) | Add optional `sendLvefInWebhook` field to webhook payload | Field appears **only** when both the webhook opt-in key and LVEF are enabled for the clinic (§6.3 row 17) |

---

## 7. Roles & Responsibilities

| Role | Name | Responsibility |
| --- | --- | --- |
| QA Engineer / Maker | HP | Authoring & executing test cases, logging defects, self-review against `test-execution-guardrails-hp.md` before handoff |
| QA Manager / Checker | NR | Independent review of this VP and downstream test cases/execution; approval = merge |
| Development (per Impact Doc's Code/Repository table) | Prasad Baviskar | Owns all listed repos (TA Server/Client, Customer Portal, Boron Server, Transformer) — per SDS §Code/Repository, all "Todo" as of SDS authoring; **must be re-confirmed current before test execution begins** |
| Impact Document preparers | Sheik Sibgathulla, Pooja C S (with Prasad Baviskar) | Named in the Impact Document's own header |
| Impact Document reviewers/approvers | Yureka M R, Nandha Kumar, Priyanka Ajith | Named in the Impact Document's own header — real names, grounded, not invented |
| Product Manager / Delivery Manager | Not named anywhere in the source material | Open Issue (OI-15) |

---

## 8. Entry Criteria

- This VP merged (NR's approval).
- Test cases authored and reviewed against `Guardrails_QC/TestCaseReview_Guardrails`.
- Build deployed with all named repos (TA Server/Client, Customer Portal, Boron Server, Transformer) at a version past "Todo" — confirm actual current dev status, since the SDS's own tracking table showed no branches/PRs open at time of writing.
- Exact component version/build numbers confirmed against the build under test (resolving OI-1) — per `test-execution-skillset-hp.md` §2.2, execution cannot validly start against "a version past Todo" alone; the actual version/commit for each of TA, Customer Portal, VCardia, Cardionet V2, Boron Server, Transformer, and Report Builder must be known before this entry criterion is considered met.
- Segment Type and "Include LVEF Report Addendum" datafixes completed for existing customers (per Impact Doc — the Addendum toggle datafix in particular must default existing LVEF customers to ON, or they silently lose their second page).
- RB templates updated with the new LVEF design; confirmed which of the 5 named legacy templates remain correctly excluded.
- Medical AI sandbox/mock reachable from QA with failure-injection capability (for polling/fallback testing).
- Slack test channel available for stuck-ECG recovery alert verification.

---

## 9. Configuration of Test Environment

| Environment | Description |
| --- | --- |
| DEV | Smoke checks, mocked Medical AI dependency. |
| QA | Full functional/integration scenarios; primary environment for this VP. |
| STAGING/UAT | End-to-end on production-like data; a real UAT environment is confirmed to exist — its download-report link (`uat-apps.tricogdev.net`) appears directly in the design deck's email mockups. |
| PERF | Reserved for the SDS's stated performance NFRs (§6.2). |
| PROD (canary) | Reserved for post-release smoke validation. |

---

## 10. Test Data Planning & Management

Fixtures and rewritten test cases stored per the org's Test Case YAML baseline template convention (`lvef_test_cases.yaml`), evidence and execution records per `test-execution-guardrails-hp.md`/`test-execution-skillset-hp.md`.

---

## 11. Metrics and Reporting

| Name | Content | Frequency | Method |
| --- | --- | --- | --- |
| Status Report | Progress, new issues, next steps | Per cycle | Written summary |
| Issues/Risks | Open item, mitigation, follow-up | As needed | This VP's §12/§13 |
| Defects | Filed in Jira (`tricog1.atlassian.net`) | As found | Jira |

---

## 12. Test Risks & Mitigation

| Risk | Mitigation |
| --- | --- |
| The PRD's own Conclusion (§5) contradicts its body on two points (no-new-toggle claim; "4 retries"/"result unavailable" mechanism) | Treat the SDS as authoritative where it conflicts with the PRD Conclusion; escalate the PRD itself for correction (OI-2, OI-4) |
| The Impact Document states a "2-minute wait" that contradicts both the PRD and the SDS's actual queue logic (both say 1 minute) | Treat as a drafting error in the Impact Document; verify 1 minute against the live system, not the Impact Doc's header text |
| "Failed" vs "Errored" trigger condition is undefined in any source document | Escalate to Product/Engineering before finalizing the affected test cases (row 16, §6.3); do not guess a trigger condition |
| Segment Type migration mapping (5 old values → 3 new) is not specified anywhere | Escalate before finalizing migration test cases (row 7, §6.3) |
| SDS's TAT-window description is unclear and never explicitly restates "10 minutes" in its technical flow | Confirm the actual implemented cap directly against the build, not assumed from the PRD/Impact Doc text alone |
| Two different error-response bodies given in the SDS for the same deprecated endpoint | Confirm the actual implemented response before finalizing the backward-compatibility test case (row 8, §6.3) |
| Design deck mixes mockup vintages (some pages show the pre-this-release disclaimer with no LVEF append at all) | Do not treat every page in the design PDF as current-state; cross-check disclaimer-related test cases against the SDS/PRD text, not the earliest mockup pages |

---

## 13. Assumptions & Open Issues (Open Gaps)

*Section renamed to include "Open Gaps" alongside its original title per Checker review G-01 — a template-naming alignment, not a content change; the section's substance was already complete.*

**Assumptions:**
- Where the SDS and PRD conflict, the SDS (later, more technically detailed, with explicit "Done" resolutions in its own Q&A) is treated as the more current source — stated explicitly here per the honesty/no-invention guardrail, not silently applied.
- "Cardionet V2" in the Impact Document's scope and the "Shubhra Spoke 4" web UI seen in the design deck are the same application.

**Open Issues** (carried forward, never silently resolved):

Classification added in v0.3 per `verification-plan-guardrails-hp.md` §6.2 (self-review, not an NR finding) — **Maker-fixable** means HP can close it directly from existing source material or its own testing action; **Externally-blocked** means it requires Product, Engineering, or QMS input HP cannot supply alone. NR should independently confirm these, not accept them at face value (guardrails-nr.md §6).

| ID | Open Issue | Classification |
| --- | --- | --- |
| OI-1 | No component version numbers stated anywhere except the two *unsupported* app versions named in the Impact Doc; no target "latest" version given. | Externally-blocked — needs Engineering/DevOps to confirm actual build/version numbers under test. |
| OI-2 | PRD Conclusion (§5, point 10) states "4 retries" and a "result unavailable" mechanism that contradicts the SDS's "5 attempts" and the PRD's own §3.9/§4.1.3 — three inconsistent tellings of the same mechanism. | Externally-blocked — needs Product to correct the PRD Conclusion; test design already proceeds on the SDS as authoritative (§13 Assumptions). |
| OI-3 | PRD §3.1 ACs 8–9 say no fallback text is shown when the AI produced no result / case not eligible, while §3.8 AC5 and the Conclusion both describe specific fallback text ("result unavailable") shown in that same scenario in the email/report — direct contradiction not resolved by the SDS. | Externally-blocked — needs Product to reconcile which behavior is actually intended. |
| OI-4 | PRD Conclusion (§5, point 5) states no new toggle is added, directly contradicted by the PRD's own §1.2, §2.4, §3.5 (the "Include LVEF Report Addendum" toggle). | Externally-blocked — needs Product to correct the PRD Conclusion itself. |
| OI-5 | Deleted — merged into OI-2 (kept numbering intentionally non-sequential here to mirror how the source PRD itself was found, not renumbered to look tidier than the source). | N/A — deleted. |
| OI-6 | The stuck-ECG recovery script, the webhook `sendLvefInWebhook` field, RB-template-only support, and the Retry-Medical-AI bugfix (all from the SDS) have **no corresponding functional requirement or acceptance criteria in the PRD at all** — real scope with no PRD-level spec to trace test cases back to. | Externally-blocked — needs Product to add PRD-level spec/acceptance criteria for these SDS-only items. |
| OI-7 | Segment Type: PRD states existing values are H/M/L (3); the SDS states existing values are H/M/L/Test/Pilot (5). No old→new value mapping is given anywhere. | Externally-blocked — needs Product/Engineering to define the old→new migration mapping. |
| OI-8 | The SDS's own description of the TAT/10-minute-window calculation ("timeRead... deviceAcquisition + 60 sec") is unclear and never explicitly restates the 10-minute figure used elsewhere. | Externally-blocked — needs Engineering to clarify the actual implemented calculation. |
| OI-9 | "Failed" vs "Errored" ("LVEF Error" vs "LVEF Failed") are confirmed as two distinct, separately-labelled states by the SDS config and the design deck, but no document defines what data condition causes one versus the other. | Externally-blocked — needs Product/Engineering to define the distinguishing trigger condition. |
| OI-10 | The design wireframe reveals a master **"LVEF Risk Assessment: None / Medical AI"** radio control that gates the entire service per clinic — not mentioned in the PRD, Impact Document, or SDS at all. | Externally-blocked — needs Product to document this control's behavior. |
| OI-11 | The Impact Document frames the PRD's assumed "current" (short) disclaimer text as **new** replacement text, with a different, longer "existing" text being retired — meaning the PRD's assumed baseline may not reflect actual current production. | Maker-fixable — HP can capture the actual current-production disclaimer text directly as the real baseline. |
| OI-12 | Copy-exactness defects found only in the design deck: one Android mockup renders "LVEF risk assessment is pending" in lowercase against the specified capitalised string; one TA mockup shows "**Filed** LVEF" instead of "Failed LVEF." | Maker-fixable — HP tests against the specified (correct) string regardless of the mockup typo; no external input needed. |
| OI-13 | The consolidated email design mockups show Gender, Status, and Interpretation as body fields, beyond the PRD §3.8 point 2's stated field list (Center Name, Patient ID, Age, Acquired on only). | Externally-blocked — needs Product to confirm the authoritative field list (PRD text vs. design mockups). |
| OI-14 | The SDS shows two different JSON error-response bodies for the same deprecated Selective-Submission endpoint hit by an old mobile app. | Externally-blocked — needs Engineering to confirm the actual implemented response body. |
| OI-15 | No Product Manager, Delivery Manager, or DevOps/SRE named anywhere in the source material. | Externally-blocked — needs the org/Product to name these owners. |
| OI-16 | `wireframe-ta-lvef-config.html`, referenced as a design link in PRD §3.5, does not exist under that name anywhere locally — the actual content was later found embedded in the design PDF instead. | Maker-fixable — already resolved by HP locating the actual content; kept as a record, not a live blocker. |
| OI-17 | PRD §1.2/§1.5/§2.1/§3.1 are explicitly tagged **"Status: TBD"** — the single most central requirement (the page-1/portal/VCardia one-liner) is, by the PRD's own admission, not finalized. | Externally-blocked — needs Product to finalize these PRD sections. |
| OI-18 | §3.2's stated assumption ("the LVEF result will be available at render time") contradicts §3.9's own timeout path, where rendering explicitly happens *before* LVEF is available. | Externally-blocked — needs Engineering to reconcile the SDS's own internal contradiction. |
| OI-19 | §4.1.3's "polling stops after 5 attempts" and §3.10's "system continues to monitor for LVEF indefinitely" are not reconciled — unclear how a late LVEF result would be detected once polling has stopped, absent the stuck-ECG recovery script filling that gap (which itself is undocumented in the PRD — see OI-6). | Externally-blocked — needs Engineering to reconcile polling-stop vs. indefinite-monitoring behavior. |
| OI-20 | The SDS's NFR section defines no Usability category, despite two already-identified findings (OI-9: undefined Failed-vs-Errored trigger; OI-12: copy-exactness defects in the design deck) being usability-relevant in nature — surfaced during Checker review against `test-execution-guardrails-hp.md`/`test-execution-skillset-hp.md` (the only guardrail/skillset pair available for this feature, since no VP-stage-specific pair exists for LVEF — see the newly authored `verification-plan-guardrails-hp.md`/`verification-plan-skillset-hp.md`). | Maker-fixable — the gap is already named honestly in §6.2; no further external input needed to keep the VP accurate about it. |
| OI-21 | No test automation framework, CI/CD pipeline, or automation tooling is named anywhere in the PRD, Impact Document, or SDS — the SDS's only tooling mention (Postman) is for updating Report Builder templates, not for running or asserting test results. The Automation Strategy (§6.9) and the Automation Candidate column (§6.3) are scoped provisionally against this gap and must be re-confirmed against QA's actual stack. | Externally-blocked — needs QA/Engineering leadership to select the automation framework/CI-CD tooling. |
| OI-22 | The SDS states numeric performance targets (§6.2) but never states a minimum run count or sample size for statistical validity (e.g. "median & P95 over ≥N runs") — per `test-execution-guardrails-hp.md` §4.3, a performance case cannot be validly marked Pass on a single anecdotal run, but no source document defines what a sufficient number of runs actually is. | Externally-blocked — needs QA/Performance to define a minimum run count/sample size. |
| OI-23 | None of the endpoints named in the SDS (§6.14 API contract matrix) states its HTTP method — contract tests can name the endpoint path but not assert the correct verb until this is confirmed against the actual implementation. | Externally-blocked — needs Engineering to confirm each endpoint's HTTP method. |
| OI-24 | **🔴 Maker-external Blocker.** No ISO 14971 hazard register is named anywhere in the PRD/Impact Doc/SDS; §3's HIGH/MEDIUM/LOW risk table is a qualitative testing-priority ranking only and cannot, by itself, support hazard-ID-level risk-control traceability. Surfaced during Checker review (VP-40a · M2); severity elevated to gating-Blocker per cross-draft consistency check against the parallel VP's own OG-24. **Escalate to Product/QMS** for the actual ISO-14971 Risk Management Report/hazard register — this VP's Draft → Approved progression is gated on this alongside any other open Blocker; it is not resolvable by HP alone. | Externally-blocked — needs Product/QMS to supply the ISO-14971 hazard register. |
| OI-25 | Cardionet V2 is named in the Impact Document's component list, but no PRD/SDS requirement row maps to it distinctly from TA/Customer Portal — its actual LVEF-related scope (if any beyond report rendering it inherits generically) is unconfirmed. | Externally-blocked — needs Product to clarify Cardionet V2's distinct LVEF scope. |
| OI-26 | No scalability requirement (behavior under production-representative case volume or concurrent-clinic load) is stated anywhere beyond the specific latency targets in §6.2 — a distinct scalability target is undefined. | Externally-blocked — needs Product/Engineering to define a scalability target. |
| OI-27 | The PRD, Impact Document, and SDS are all cited in §18 without a stated version/revision identifier of their own — only the Web Development/disclaimer-tracking Jira tickets have a stable identifier. Surfaced during Checker review (G-05). | Externally-blocked — needs Product/QMS to assign version/revision identifiers to the source documents themselves. |
| OI-28 | No immutable, dedicated evidence-store location is committed to anywhere for raw screenshots, logs, or API/DB payload captures — §6.8/§16 describe defect tracking (Jira) and audit-log linkage, but not a durable evidence repository distinct from those. Surfaced during Checker review (VP-54). | Externally-blocked — needs QA/Engineering leadership to designate a durable evidence-store location. |
| OI-29 | No rollback runbook (who executes it, in what order, with what verification steps) is defined anywhere in the PRD/Impact Doc/SDS — §14's Rollback Strategy states the minimum scope a runbook must cover, derived from this VP's own content, but the runbook itself does not yet exist. Surfaced during Checker review (VP-94). | Externally-blocked — needs Engineering/DevOps to author the actual rollback runbook. |

---

## 14. Rollback Strategy

Added per Checker review VP-94. No rollback runbook is defined anywhere in the PRD, Impact Document, or SDS for this release — this section states what a rollback would need to cover, synthesized from this VP's own regression/entry-criteria content (§6.5, §8), not a verified procedure. The exact runbook (who executes it, in what order, with what verification steps) is tracked as **OI-29** and must be authored before production sign-off — it is not satisfied by this section alone.

| Component | What would need to roll back | Basis |
| --- | --- | --- |
| Segment Type datafix | Revert migrated clinics to their pre-migration Segment values | §8 Entry Criteria names this datafix; exact old→new mapping is itself undefined (OI-7), which also blocks defining its reverse |
| "Include LVEF Report Addendum" default-ON datafix | Revert the default flag for existing LVEF customers | §8 Entry Criteria names this datafix as customer-impacting if mishandled forward; the same is true in reverse |
| Boron Server / Transformer (report delivery coordination) | Revert to pre-release single-report (ECG-only) dispatch behavior, no LVEF wait/coordination | §3 HIGH risk item; §6.5 regression baseline is the target state to roll back to |
| Report Builder templates | Revert RB templates to their pre-release versions | SDS states RB templates are updated via Postman — the same mechanism would apply in reverse, but no rollback template set is named |
| VCardia rollout | Halt/pause staged app-store rollout percentage | Impact Doc's own two-stage delivery framing implies a stageable rollout; old app versions already degrade gracefully (§6.5), so a rollback does not require forced app downgrade |

**Open gap:** none of the above is a tested, confirmed-safe procedure — it is the minimum scope a rollback runbook must address, derived from what this release actually touches. OI-29 tracks the missing runbook itself.

---

## 15. Exit Criteria

- All Test Cases passed; all defects closed or explicitly deferred with sign-off from NR and Product.
- 100% of P0 scenarios pass, ≥95% of P1.
- 100% of the Test Combination Matrix's TEST rows executed — the two currently-**blocked** GAP-FILLER rows (Segment Type migration, Failed-vs-Errored) are explicitly still blocked at exit, not silently dropped, unless OI-7/OI-9 are resolved first.
- The double-critical-alarm behaviour (row 15, §6.3) is confirmed present and functioning as the accepted risk describes — not suppressed as if it were a bug.
- Coverage Analysis (§6.4) gaps are explicitly accepted or resolved, never silently closed.
- Rollback Strategy (§14) scope is confirmed and its runbook (OI-29) is authored and reviewed — not just scoped — before production sign-off.
- **OI-24 (ISO-14971 hazard register) is resolved by Product/QMS** before this VP can move past Draft — a Maker-external Blocker, not something this VP's own content can close on its own (see §3, §13).
- Sign-off = NR's merge on this VP and its downstream test cases/execution records.

---

## 16. Test Artifacts

| Verification Requirement | Deliverable | Location |
| --- | --- | --- |
| Verification approach | This VP | `Change-Management_InstaECG/LVF/Testing/Verification Plan/` |
| Test case authoring | Test Case YAML set | Per org Test Case baseline template |
| Test execution | Execution records + evidence | Per `test-execution-guardrails-hp.md` |
| Defects | Jira issues | `tricog1.atlassian.net` |
| Traceability | Requirement → VP → Test Case → Evidence matrix | Companion to this VP |
| Automation suite | Scripted cases for the Auto-ready rows in §6.3/§6.9 | Location/tooling not yet defined — blocked on OI-21 |
| Evidence store (added per Checker review VP-54) | Raw screenshots, logs, API/DB payload captures — an **immutable, dedicated** location, distinct from ad hoc local storage or the Jira defect record alone | Exact location/service not committed anywhere in the source material — tracked as **OI-28**; "stored per the org's actual practice" (§6.8) is not sufficient on its own for audit purposes |

---

## 17. Training

The team needs a walkthrough of the SDS's Redis-based coordination logic (`LVEF_AWAITING_ECGS`, `isWaitingForLVEF`, the Medical AI health-status cache) and the stuck-ECG recovery script before authoring test cases for §6.1's timing/coordination matrix — this is unusually deep backend logic for a QA-authored case to get right by guesswork.

---

## 18. Referenced Documents

Completed per Checker review G-05: PRD/SDS/Impact Document version identifiers are stated where known; where the source material itself gives no version, that is flagged rather than left implicit (**OI-27**), not silently treated as "no version needed."

| Document | Link | Version/Revision |
| --- | --- | --- |
| PRD | Outline doc, linked from the SDS References table | Not stated in the source — OI-27 |
| Impact Document | Google Doc, linked from the SDS References table | Not stated in the source — OI-27 |
| SDS | This document's own source | Not stated in the source — OI-27 |
| Web Development Ticket | Jira `OLPD-5848` | Jira ticket ID is its own identifier |
| Medical AI V2 Audit Logs | Google Sheet, linked from the SDS | Not versioned (live sheet) |
| Doctor Reporting base disclaimer tracking | Jira `ADSRQ-162` | Jira ticket ID is its own identifier |
| AI Reporting base disclaimer tracking | Jira `ADSRQ-298` | Jira ticket ID is its own identifier |
| Design | `LVEF Imporvements.pdf` (46 pages) + Figma (`e3Wu4LM1z3V2LhPNxpVBZP/LVEF-Imporvements`) | PDF page count stated; Figma file has no stated revision — OI-27 |

**Appendix**

| Document | Status |
| --- | --- |
| Testing SOP | Not located for this feature — Open Issue |
| Risk Report | SDS references a Risk Report Template (Google Sheet) — not yet populated for this feature |
| Secure Development Process | Referenced by the SDS; not reviewed as part of this VP |

**Sign-off:** Author — HP (Maker), Draft. Reviewer — NR (Checker), pending review. Approval is recorded as NR's merge, per the maker-checker gate.
