# InstaECG / CardioNet (LVEF) — Verification Plan Drafting Skillset (Maker / QA Engineer)

> **Product:** Tricog InstaECG / CardioNet — TA (Server & Client), Customer Portal, VCardia (Android/iOS), Cardionet V2, Boron Server, Transformer, Report Builder
> **Role:** HP (QA Engineer / Maker) — drafts and hardens a Verification Plan before raising a PR for NR (Checker) to review.
> **Stage:** ① Verification Plan · Agentic SDLC — first stage, before ② Test Case Writing.
> **Regulatory frame:** ISO 13485 · IEC 62304 · ISO 14971 · IEC 62366
> **Companion:** `verification-plan-guardrails-hp.md`
> **Derived from:** the org-wide `Guardrails_QC/VerificationPlan_Guardrails` non-negotiables, turned into a drafting skill for InstaECG/CardioNet specifically — not copied wording.

---

## 1. Role & objective

**Role:** QA Engineer acting as the drafting agent for an InstaECG/CardioNet Verification Plan — pulling the release's PRD, Impact Document, and SDS in full, the approved baseline template (`L3_QC_VTE_310_VerificationTestPlan`), and this project's guardrails, and producing a plan genuinely ready for independent review, not a first pass that offloads the real thinking onto NR.

**Objective:** by the time the PR opens, the plan should already satisfy every rule in `verification-plan-guardrails-hp.md` — structure complete, every requirement traced, every method concrete, every gap named instead of hidden. NR's job is to independently verify that, not to do the first read.

**Why draft quality matters here specifically:** InstaECG/CardioNet reports feed clinical decisions in real time. A verification gap in report-delivery timing, a regulatory-gated toggle, or interpretation-field separation isn't just a missed test case — it's a path for a wrong or missing diagnostic finding to reach a clinician. A weak first draft costs real review cycles on a product where those cycles matter.

---

## 2. Drafting workflow

1. **Pull inputs** — the release's PRD, Impact Document, and SDS in full, the current baseline VP template, and the current guardrails file. Never draft from memory of a past release's PRD.
2. **Check for a comparable/parallel VP** covering related or overlapping release scope (e.g. another maker's independent draft) — if one exists, plan to cross-check consistency against it before raising the PR (guardrails §6), not after NR finds the mismatch.
3. **Work the Impact Document first** — before writing requirements, work out the actual blast radius: which of TA / Customer Portal / VCardia / Cardionet V2 / Boron Server / Transformer / Report Builder this release touches, since that becomes the Components-Under-Test list and the Module scope.
4. **Cross-read the SDS against the PRD deliberately** — the SDS is typically later and more technically detailed; note every place they conflict (counts, timings, terminology) rather than picking a reading silently. Treat the SDS as authoritative only where you say so explicitly.
5. **Draft section by section, in template order** — don't skip to the Testing Strategy matrices before Scope and the Risk Profile are solid; downstream sections depend on scope and risk being right.
6. **Name every open question as an Open Issue** the moment it's found, with a stable ID, tagged **Maker-fixable** or **Externally-blocked** — don't resolve it silently and move on, and don't let it pile up to fix "later."
7. **Self-review against every guardrail rule** (§1–6 of `verification-plan-guardrails-hp.md`) before raising the PR — treat this as if NR were already reading it, because effectively they are.
8. **Raise the PR only once the self-review pass is clean** — a plan raised with known gaps still open should have those gaps flagged in the PR description, not discovered by NR for the first time.

---

## 3. Self-review checklist (map to guardrails)

Before opening the PR, confirm each of these against the actual draft — not from memory of having "covered it":

- [ ] All required sections present, in template order, none renamed or merged — including Module Overview, Automation Review, Rollback Strategy, and the per-endpoint API contract matrix as their own identifiable content, not folded silently into other sections.
- [ ] Document version/revision identifier is stated distinct from the shared template-type ID (G-03).
- [ ] Introduction/Objectives tie directly to what the PRD actually changes this release.
- [ ] Every Scope row points at a specific PRD/SDS section or requirement ID; every SDS-only item (not in the PRD) is called out as such, not silently merged in; Scope includes a Module Overview table (module × requirement count × platform).
- [ ] Risk Profile rates every touched module with a concrete rationale — regulatory-gated toggles and clinical-decision-relevant paths rated HIGH, not defaulted to MEDIUM — and states explicitly whether the rating substitutes for ISO 14971 hazard-ID-level traceability or names that as a separate open gap (VP-40a).
- [ ] Test Strategy is written for *this* release's actual scope, not reused boilerplate, and states an explicit Release Type & Coverage Floor — including reverse-direction and mixed-version-coexistence scenarios where the release type calls for them (M5).
- [ ] Every matrix (config/flag, outcome/state, timing/coordination, NFR, combination) is grounded in the actual PRD/SDS text — no invented numbers or thresholds.
- [ ] NFR categories reconcile: every category the SDS defines is scored; a category the SDS is silent on is named as a gap, not omitted from the matrix.
- [ ] An Automation Review is present with an honest per-row automation-candidate judgment, including partial-automation cases (VP-90).
- [ ] A Rollback Strategy section has concrete scoped content — not just "TBD" — even if the full runbook is an Open Issue (VP-94).
- [ ] A per-endpoint API contract matrix covers every endpoint named anywhere in the SDS, including deprecated or cronjob-only ones (VP-68).
- [ ] Every HIGH-risk item names both a manual and an automated verification approach.
- [ ] Entry and Exit Criteria are both objectively checkable — no subjective language, and build/version confirmation is explicit, not assumed from "latest."
- [ ] Coverage Analysis never marks a dimension "covered" when its trigger/mapping is undefined — it's marked blocked instead.
- [ ] Every out-of-scope exclusion carries its own one-line justification, not a shared blanket sentence (VP-12).
- [ ] Test Artifacts names a specific, durable evidence-store location distinct from defect-tracker linkage — or, if genuinely unknown, tracks its absence as its own Open Issue rather than leaving it vague (VP-54).
- [ ] Every cited, heavily-used reference either states its own version/revision identifier or is explicitly flagged as lacking one (G-05).
- [ ] Roles & Responsibilities use only names actually present in the source documents; anything unnamed is an Open Issue.
- [ ] Open Issues lists every real ambiguity found — or explicitly says "None identified" — with stable IDs carried forward, never renumbered to look tidier than the source, and each tagged Maker-fixable or Externally-blocked.
- [ ] Any risk-accepted behaviour named in the SDS appears as a confirmation test, not a suppressed defect.
- [ ] Status is Draft (about to move to In Review) — never self-marked Approved.
- [ ] Nothing in the plan is invented — every scenario traces to something actually in the PRD/Impact Doc/SDS/design; anything not traceable is an Open Issue, not a guess.
- [ ] Checked for a comparable/parallel VP and cross-checked consistency on any shared issue before raising the PR.

---

## 4. What "good" looks like for InstaECG/CardioNet specifically

- A **regulatory-gated toggle** (e.g. "Make Results Available") is rated HIGH risk with the specific compliance statement it enforces quoted or cited, not treated as an ordinary config flag, and named with both a manual and an automated verification approach.
- A **Release Type & Coverage Floor** declaration derives its mandatory scenarios mechanically (e.g. "mixed release → reverse App↔Backend and mixed-version device coexistence are both mandatory") rather than asserting coverage without deriving it from the release type.
- A **hazard-register caveat** is stated plainly in the Risk Profile itself (e.g. "this qualitative table does not substitute for ISO 14971 hazard-ID-level traceability; that remains OI-24") rather than left for a reader to infer.
- A **report-delivery coordination** requirement (waits, caps, subsequent dispatch, stuck-record recovery) gets explicit boundary-value test rows at each numeric threshold — not just a happy-path row.
- A **multi-state label/outcome** requirement (e.g. 5+ result states across case list, detail, and report) gets every state enumerated in its own matrix row, including states confirmed only by design or SDS config and absent from the PRD's own definitions section.
- A **data migration** (e.g. old→new config value mapping) with no stated mapping is marked as a blocked test-design row, not silently assumed or skipped from the combination matrix.
- A **risk-accepted edge case** stated explicitly in the SDS (e.g. an accepted duplicate-alarm scenario) is written as a confirmation scenario, phrased so a tester doesn't mistake the accepted behaviour for a bug to file.

---

## 5. Escalation

If a PRD/Impact Doc/SDS is genuinely silent on something needed to draft a section (not just under-detailed, but missing entirely — e.g. a trigger condition, a migration mapping, a target version number), that's an Open Issue: flag it in the plan and raise it to Haripriya rather than inventing a plausible answer to keep drafting moving.

---

*Companion to `verification-plan-guardrails-hp.md`. Sibling to `test-execution-skillset-hp.md` for the next stage. When an NR review finds a drafting gap this skillset didn't anticipate, add it here so the next draft starts from a stronger position.*
