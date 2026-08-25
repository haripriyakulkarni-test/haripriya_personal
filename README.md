# HP — Maker (QA Engineer) Artifacts

This repo holds HP's Maker-side artifacts across the agentic maker-checker QA SDLC: guardrails, skillsets, and drafted work products for each stage, independent of any single product.

Each stage folder below is self-contained:

- **Guardrails/** — the non-negotiable checks a work product must pass before it's sent for Checker review.
- **Skillset/** — the drafting workflow and self-review checklist for producing that stage's work product.
- **Drafts/** — the actual work products drafted at this stage (e.g. a specific product's Verification Plan, Test Case set, or Execution record).

## Stages

1. **[01_Verification_Plan](01_Verification_Plan/)** — drafting the Verification Plan before it's ready for Checker review.
2. **[02_Test_Case_Writing](02_Test_Case_Writing/)** — authoring test cases from an approved Verification Plan.
3. **[03_Test_Execution](03_Test_Execution/)** — executing test cases and recording evidence.

A Checker (e.g. NR) independently reviews each stage's output against their own counterpart guardrails/skillset before it moves forward — the maker-checker gate only works if that review is genuinely separate from this Maker's own self-review.
