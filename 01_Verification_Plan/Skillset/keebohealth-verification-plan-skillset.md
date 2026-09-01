# KeeboHealth — Verification Plan Drafting Skillset (Maker / QA Engineer)

> **Product:** KeeboHealth (Tricog RPM / chronic care platform — patient mobile app + doctor web portal)
> **Role:** HP (QA Engineer / Maker) — drafts and hardens a Verification Plan before raising a PR for Manager (Checker) to review.
> **Regulatory frame:** ISO 13485 · IEC 62304 · ISO 14971 · IEC 62366
> **Companion:** `keebohealth-verification-plan-guardrails.md`
> **Derived from:** the org-wide `Guardrails_QC/VerificationPlan_Guardrails` non-negotiables, turned into a drafting skill for KeeboHealth specifically — not copied wording.

---

## 1. Role & objective

**Role:** QA Engineer acting as the drafting agent for a KeeboHealth Verification Plan — pulling the release's PRD/SDS/Impact Doc, the approved baseline template, and the guardrails, and producing a plan that's genuinely ready for independent review, not a first pass that offloads the real thinking onto the Checker.

**Objective:** by the time the PR opens, the plan should already satisfy every rule in the guardrails file — structure complete, every requirement traced, every method concrete, every gap named instead of hidden. The Checker's job is to independently verify that, not to do the first read.

**Why draft quality matters here specifically:** KeeboHealth is a chronic-care platform — a vitals requirement that's under-verified isn't just a missed test case, it's a path for a real deterioration signal (a dropped SpO₂ reading, a threshold that never fires) to go unnoticed. A weak first draft costs real review cycles on a product where those cycles matter.

---

## 2. Drafting workflow

1. **Pull inputs** — the release's PRD, SDS, Impact Doc, the current baseline VP template, and the current guardrails file. Never draft from memory of a past release's PRD.
2. **Work the Impact Doc first** — before writing requirements, work out the actual blast radius: which of patient app / doctor web portal / backend / database this release touches, since that becomes the Module Overview.
3. **Draft section by section, in template order** — don't skip to Functional Requirements before Scope and References are solid; downstream sections depend on scope being right.
4. **Name every open question as an Open Gap** the moment it's found — don't resolve it silently and move on, and don't let it pile up to fix "later."
5. **Self-review against every guardrail rule** (§1–5 of the guardrails file) before raising the PR — treat this as if Manager were already reading it, because effectively they are.
6. **Raise the PR only once the self-review pass is clean** — a plan raised with known gaps still open should have those gaps flagged in the PR description, not discovered by the Checker for the first time.

---

## 3. Self-review checklist (map to guardrails)

Before opening the PR, confirm each of these against the actual draft — not from memory of having "covered it":

- [ ] All 14 required sections present, in template order, none renamed or merged.
- [ ] Objective ties directly to what the PRD actually changes this release.
- [ ] Every Scope item points at a specific PRD section/requirement ID; every exclusion has a reason.
- [ ] Every reference (PRD, SDS, Impact Doc, Figma, tracking ID) is a live link, not a description.
- [ ] Open Gaps lists every real ambiguity found — or explicitly says "None identified."
- [ ] Test Strategy is written for *this* release's actual scope, not reused boilerplate.
- [ ] Module Overview covers every touched module (patient app / doctor web / backend / DB) with requirement counts and platforms.
- [ ] Scenario Combinations are genuine multi-condition cases, not restated single-condition scenarios.
- [ ] Every Functional Requirement has a tracking ID, requirement text, platform(s), and scenarios.
- [ ] Every NFR category is populated or marked "Not Applicable + reason" — Security is never skipped silently for anything touching patient input or vitals data.
- [ ] Entry and Exit Criteria are both objectively checkable — no subjective language.
- [ ] Every verification method names a concrete technique — no "will be tested."
- [ ] Any patient-data/safety/clinical-decision requirement is flagged and cross-referenced in Risks & Assumptions.
- [ ] Status is Draft (about to move to In Review) — never self-marked Approved.
- [ ] Nothing in the plan is invented — every scenario traces to something actually in the PRD/SDS/Impact Doc/Figma; anything not traceable is an Open Gap, not a guess.

---

## 4. What "good" looks like for KeeboHealth specifically

- A **Blood Sugar / new-vitals** requirement includes the reading-type selector (Fasting/Random/Post-meal) as its own scenario, not folded silently into a generic "enter value" case.
- A change touching both the patient app and backend gets an explicit note on **version skew during rollout** (older app build against updated backend) somewhere in Scope or Risks — even if the full compatibility matrix belongs to a later, more detailed pass.
- Anything that depends on the **Smart Alerts engine's readiness** is called out as an assumption or an Open Gap, not treated as guaranteed to exist.
- A **mobile-specific** requirement (offline entry, background sync) names the platform(s) it applies to rather than assuming "mobile" means one thing.

---

## 5. Escalation

If a PRD/SDS/Impact Doc is genuinely silent on something needed to draft a section (not just under-detailed, but missing entirely), that's an Open Gap — flag it in the plan and raise it to Haripriya rather than inventing a plausible answer to keep drafting moving.

---

*Companion to `keebohealth-verification-plan-guardrails.md`. When a Checker review finds a drafting gap this skillset didn't anticipate, add it here so the next draft starts from a stronger position.*
