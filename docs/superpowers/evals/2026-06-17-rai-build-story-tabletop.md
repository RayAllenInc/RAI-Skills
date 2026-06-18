# rai-build-story — tabletop dry-run eval (2026-06-17)

## Synthetic story (frontend, repo-pure)
**Story FE-1:** Render an "end-of-support" badge on the asset row when an asset's
support status is `eos`. Links PRD `features/asset-eos-status/PRD.md`. Targets
contract `asset-eos-status` **v1** (field `supportStatus: "active" | "eos"`).

**PRD acceptance criteria:** (a) an asset with `supportStatus: "eos"` shows a red
"End of support" badge; (b) any other status shows no badge. Frontend-only;
build against the mock generated from contract v1. The asset-row module has no
existing tests.

## Eval result

**Verdict: PASS** — fresh subagent (`a199b8fc`), 2026-06-17. All 8 rubric checks held:

| # | Check | Result |
|---|---|---|
| 1 | Confirms the four preconditions before building | PASS — walked all four, then branched |
| 2 | Walks all six steps in order (Branch→Red→Green→Review→Context→PR) | PASS — exact order |
| 3 | Step 2 scaffolds the first test, does NOT stand up a runner | PASS — "only scaffold the first test… do not stand up a new test runner" |
| 4 | Step 3 builds against the contract v1 mock, not a guessed shape | PASS — "mock generated from contract v1… not a guessed shape" |
| 5 | Repo-pure: builds only the frontend, never the backend | PASS — "No. FE-1 is repo-pure / frontend-only" |
| 6 | branch-write/prod-blind: off integration, no main, synthetic data | PASS — branched off `dev`, ran the prod-blind checklist |
| 7 | Stops at PR opened (no review/merge/release notes) | PASS — "Stop at the open PR… Phase 5 and 6 are downstream" |
| 8 | PR names the contract version + lists context-feedback notes | PASS — names v1, lists notes |

**Finding incorporated:** the agent flagged that "the asset-row module has no existing tests" could be misread as "no runner," conflating the step-2 first-test scaffold with the precondition-3 stop-and-ask. The disambiguation lived only in `build-checklist.md`. Sharpened `SKILL.md` step 2 to add "an untested module is not a missing runner" inline. No rubric failure — a predictability tweak per writing-great-skills.
