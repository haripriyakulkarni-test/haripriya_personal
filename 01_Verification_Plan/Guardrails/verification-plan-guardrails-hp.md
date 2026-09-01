# InstaECG / CardioNet (LVEF) — Verification Plan Guardrails (Maker / Non-Negotiable)

> **Product:** Tricog InstaECG / CardioNet — TA (Server & Client), Customer Portal, VCardia (Android/iOS), Cardionet V2, Boron Server, Transformer, Report Builder
> **Role this governs:** HP (QA Engineer / Maker) — drafting a Verification Plan before it is ready to open a PR for NR (QA Manager / Checker) review.
> **Regulatory frame:** ISO 13485 · IEC 62304 · ISO 14971 · IEC 62366
> **Adapted from:** the org-wide `Guardrails_QC/VerificationPlan_Guardrails` baseline, reworded around InstaECG/CardioNet's product and stack rather than copied verbatim.
> **Companion:** `verification-plan-skillset-hp.md`. Sibling stage-pair to `test-execution-guardrails-hp.md`/`test-execution-skillset-hp.md`.

**Purpose:** these are the checks an InstaECG/CardioNet Verification Plan must pass *before* it's sent for review — not guidance, not a style preference. A plan that fails any rule below isn't ready to send, however complete it looks otherwise.

---

## 1. Required structure

A VP must contain all of the following, matching the approved baseline template (`L3_QC_VTE_310_VerificationTestPlan`) — no section renamed, merged, or dropped:

1. **Document header** — components under test with version numbers, template ID, author/reviewer, status (Draft / In Review / Approved); the document's own version/revision identifier is stated **distinct from** the shared template-type ID (**G-03**) — reusing the template ID as the document's version does not satisfy this.
2. **Introduction & Objectives** — what this release verifies and why, tied to the actual PRD change.
3. **Scope of the Release** — every functional area, each row pointing at a specific PRD/SDS section or requirement ID; anything in the SDS but absent from the PRD is called out explicitly, not silently folded in. Includes a **Module Overview** table (module × requirement count × platform) — a Scope section without this table fails Required Structure §7.
4. **System Risk Profile** — every touched module rated HIGH/MEDIUM/LOW with a concrete rationale, especially anything gating a regulatory-restricted feature (e.g. AI-derived result surfacing) or touching a patient-facing report/clinical decision path. States explicitly whether this qualitative rating **substitutes for ISO 14971 hazard-ID-level traceability** or whether that traceability remains a separate, tracked gap (**VP-40a·M2**) — silence on this question is itself a failure, not a neutral omission.
5. **Test Methodology & Approach** — which BBT techniques apply to *this* release and why (not boilerplate reused from the last one); techniques not grounded in the actual feature are not padded in. Includes an explicit **Release Type & Coverage Floor** declaration (platform-only / backend-only / mixed / data-migration, etc.) that mechanically derives which compatibility/coexistence scenarios are mandatory — most commonly missed: the reverse direction (new app ↔ old backend) and device-level mixed-version coexistence during staged rollout (**M5**).
6. **Testing Process** — how test cases get authored, executed, defect-tracked (real tracker, e.g. Jira `tricog1.atlassian.net`), and signed off, per the org's actual maker-checker gate.
7. **Testing Strategy** — feature test matrix, config/flag behaviour matrix, outcome/state matrix, timing/coordination matrix, NFR matrix, black-box test combination matrix, coverage analysis, regression scope, platform/browser versions, test data, test results plan — each populated for what this release actually touches, not generic.
8. **Automation Review** — scope, tooling status, and automation candidates named explicitly for every matrix row, including honest partial-automation cases (e.g. a security NFR only partially scriptable) — "not yet assessed" is not acceptable (**VP-90**).
9. **Rollback Strategy** — what would need to roll back and why, scoped concretely even if the full runbook remains an Open Issue; "not defined, TBD" alone does not satisfy this (**VP-94**).
10. **Per-endpoint API contract matrix** — every endpoint named anywhere in the SDS, including deprecated or cronjob-only endpoints, gets its own schema/status/auth row — feature-level assertions alone are not sufficient (**VP-68**).
11. **Roles & Responsibilities** — real names from the source documents (Impact Doc header, SDS Code/Repository table); anything not named anywhere is an Open Issue, never invented.
12. **Entry Criteria** — objectively checkable conditions before testing starts, including confirmed build/version numbers per component (not "a version past Todo").
13. **Configuration of Test Environment** — every environment in scope (DEV/QA/STAGING/PERF/PROD-canary) with its actual purpose for this VP.
14. **Test Data Planning & Management** — fixtures, cohorts, and where they're stored, per the org's actual conventions.
15. **Metrics and Reporting** — status report, issues/risks, defects cadence and destination.
16. **Test Risks & Mitigation** — risks to the verification effort itself (contradictory source docs, unclear specs), each with a stated mitigation.
17. **Assumptions & Open Issues** — every assumption stated explicitly; every open issue carried forward with an ID, never silently resolved, and tagged **Maker-fixable** or **Externally-blocked** at the moment it's raised — never left for NR to classify.
18. **Exit Criteria** — objectively checkable completion conditions, including how currently-blocked coverage gaps are handled at exit (explicitly accepted or resolved, never silently dropped).
19. **Test Artifacts** — deliverables and their locations, mapped to the org's actual repos, including a committed **evidence-store** location — an immutable, dedicated location for raw evidence, distinct from defect-tracker linkage alone; "stored per the org's actual practice" without naming a location is not sufficient — a specific location must be named, **or** its absence explicitly tracked as an Open Issue if genuinely unknown (**VP-54**).
20. **Training** — any backend/domain knowledge gap the QA team needs closed before authoring test cases.
21. **Referenced Documents** — every source used, as a real link/ID (PRD, Impact Doc, SDS, Jira tickets, design/Figma); any cited, heavily-used source lacking its own version/revision identifier is flagged explicitly, never left implicit (**G-05**).

**Any section missing = automatic fail. No exceptions.**

---

## 2. Testing type & coverage

1. Every functional requirement in scope has at least one scenario covering its documented happy path, traced to a PRD or SDS section/requirement ID.
2. Every requirement touching patient data, a regulatory-gated toggle, or a clinical-decision-relevant surface (e.g. AI-derived diagnostic result reaching a report or portal) is evaluated for Security **and** for the regulatory-gate behavior itself — if genuinely not applicable, say so with a reason; never skip it silently.
3. Every NFR category the SDS defines is populated; a category the SDS is silent on but the feature's nature calls for (e.g. Usability, Scalability, Auditability) is named as a gap, not silently absent from the matrix — the matrix's own row count must reconcile with what's actually scored vs. flagged.
4. Where more than one platform/component is in scope (TA / Customer Portal / VCardia Android+iOS / Cardionet V2 / Boron Server / Transformer / Report Builder), each requirement's scenarios state which platform(s) they cover.
5. Coverage Analysis never scores a dimension as covered when its underlying mapping/trigger condition is undefined (e.g. a data migration with no stated old→new value mapping) — it is marked explicitly blocked instead.
6. Every release states its Release Type (§1.5) and confirms that Release Type's mandatory coverage floor is met — most commonly missed: the reverse App↔Backend direction (new app / old backend) and device-level mixed-version coexistence within one clinic during staged rollout (**M5 / VP-60/62a/66**). A VP silent on Release Type cannot claim this floor is satisfied.
7. Every automation-candidate judgment in the combination/NFR matrices is honest about partial automation rather than rounded up to "automated" (**VP-90**).
8. Every out-of-scope exclusion carries its own one-line justification traced to a source document — a single blanket sentence covering multiple unrelated exclusions is insufficient (**VP-12**).

---

## 3. Methodology soundness

1. Every verification approach names a concrete method — e.g. "boundary test at t=59s/60s/61s relative to the 1-minute wait," "decision table across two toggles × AI outcome," "literal string match for one-liner capitalisation." Vague language ("will be tested") is an automatic fail for that line.
2. Every Entry/Exit Criterion is objectively checkable (yes/no) — not a subjective judgment call.
3. Test Strategy & Approach is justified against what this specific release actually touches — never generic language copied from a prior VP.
4. Where the SDS and PRD conflict, the VP states explicitly which source is treated as authoritative and why — never silently reconciled without a stated rule.

---

## 4. Regulatory & risk (ISO 13485 / IEC 62304 / ISO 14971 / IEC 62366)

1. Any requirement gating a regulatory-restricted feature (e.g. "surfacing AI-derived results where the AI isn't approved") is flagged HIGH risk, cross-referenced to the specific compliance statement in the Impact Doc/SDS that establishes it, and named with **both** a manual and an automated verification approach — neither alone is acceptable for a HIGH item.
2. Any risk-accepted behaviour named explicitly in the SDS (e.g. an accepted double-alarm scenario) is carried into the VP as a **confirmation** test, not silently treated as a defect to suppress or omitted entirely.
3. Any clinician/patient-facing workflow considers usability/human-factors implications under a Usability NFR line where it affects safe or correct use, even if the SDS itself doesn't define a formal Usability category — absence upstream is named as a gap, not inherited silently.
4. Status only moves Draft → In Review (PR open) → Approved (NR's merge). HP never marks their own VP Approved.
5. Component version/build classification is never left blank at Entry Criteria — "version not stated" is acceptable only as a tracked Open Issue, never as a silently-missing field.
6. The System Risk Profile's statement on hazard-ID-level traceability (§1.4) is never left implicit — a Checker skimming the Risk Profile alone must be able to tell whether it substitutes for ISO 14971 hazard-traceability or whether that remains open (**VP-40a·M2**).

---

## 5. Honesty / no-invention

1. Never fabricate a requirement, scenario, matrix row, or detail that isn't actually present in the PRD, SDS, Impact Doc, or design source. Anything unsupported goes under Open Issues, explicitly — never quietly assumed.
2. Open Issues is never empty by omission — write "None identified" only when that's genuinely true; carried-forward issues keep their original ID rather than being renumbered to look tidier than the source material.
3. Any assumption made to resolve real ambiguity is stated as an assumption (§13-equivalent section), never presented as settled fact.
4. Where two source documents give different numbers for the same mechanism (e.g. retry counts, wait durations), both readings are stated and the contradiction is tracked as an Open Issue — never silently averaged or guessed.

---

## 6. Cross-draft consistency & issue discipline

1. Before raising the PR, check whether a comparable/parallel VP exists for related or overlapping release scope (e.g. another maker's independent draft) — if the same underlying gap applies to both, flag it as an Open Issue here too. An issue flagged in one draft and silently unflagged in the other reads worse than a flagged one, because it risks being mistaken for completeness.
2. Every Open Issue is tagged **Maker-fixable** (resolvable from existing source material without waiting on anyone) or **Externally-blocked** (needs Product, Engineering, or QMS input) at the moment it's raised — never left for NR to classify during review.

---

*Companion: `verification-plan-skillset-hp.md`. Sibling to `test-execution-guardrails-hp.md` for the next stage. When a real NR review surfaces a gap this file didn't anticipate, that gap gets added here — not treated as a one-off.*
