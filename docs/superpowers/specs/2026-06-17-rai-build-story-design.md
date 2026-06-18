# rai-build-story — design spec

**Date:** 2026-06-17 · **Owner:** Avinash Gadiraju · **Status:** Draft for review (build gated on this)

The Phase 4 skill of the AI-native pipeline — the **AFK build**. It is the biggest remaining gap in the `rai-skills` back half (the front half — `rai-grill-requirement`, `rai-grill-architect` — is built; `to-prd`/`to-issues` are reused community deps).

This spec is written to the `writing-great-skills` principles: it names the **leading words** the skill leans on, fixes the **information hierarchy** (a thin `SKILL.md` + one disclosed sibling), and gives every build step a **checkable completion criterion** so the skill runs the same *process* every time.

---

## 1. What it does

Takes **one** repo-pure, `ready-for-agent` story and produces a reviewed, test-backed pull request on a branch in that story's code repo — built against the durable context, **branch-write / prod-blind** throughout.

Position in the pipeline (`NexaContext/CLAUDE.md`):

```
… → rai-grill-architect → to-issues → per-repo story (ready-for-agent) → [rai-build-story] → review/merge (P5) → regression (P6) → rai-release-notes (P7)
```

It **stops at "PR opened."** Human review + merge (P5) and regression CI (P6) are deliberately downstream and out of scope.

---

## 2. Decisions locked (brainstorming, 2026-06-17)

| # | Decision | Rationale |
|---|---|---|
| **D1 — Agnostic preconditions** | The skill declares *what must be present* (a writable code-repo checkout + a readable Context repo + a test runner), never *how* it got there. | Portable and tool-neutral. The bootstrap workspace satisfies it today, an AFK sandbox later, a laptop in assisted mode in between — the skill doesn't change. The sandbox itself is a not-yet-decided Phase-4 bake-off, so the skill must not bind to it. |
| **D2 — Defer the second model to v2** | v1 = test-first build + a skeptical standards/spec `review` + PR. The spine's "Codex tries to break it" beat becomes a v2 add at a named **seam**. | Keeps v1 a thin orchestrator. Cross-model wiring depends on the unbuilt AFK sandbox anyway. The `review` pass already provides a self-critique; the seam keeps the upgrade clean. |
| **D3 — Hybrid test harness** | Precondition: the repo has a test **runner + run command** (a one-time per-repo ADR). The skill then handles *first test in a previously-untested module* — it scaffolds the test file + local wiring and proceeds. | NexaCore is near-zero-test (decision G), so Story #1 lands in an untested module immediately. Assuming a full harness would wall the first story; bootstrapping the whole framework would scatter framework choices. The hybrid lets the first story run while keeping framework choice an explicit decision. |

**Baked-in defaults (carried from the design, not vetoed):**

- **One story per run.** The skill takes a *specific* story; it does **not** self-select from the `ready-for-agent` queue. Picking the next story is the AFK trigger's job, not the skill's.
- **Single side per run.** A backend+frontend feature is two `rai-build-story` runs, never one. This is the **repo-pure** rule the architect grill already enforces upstream.
- **Soft dependency, not hard.** Unlike the grill skills (which *require* `grilling`/`domain-modeling`), this skill **restates its loop inline** and only *leans on* `tdd`/`review`/`implement` when present — because `implement`/`tdd` are confirmed not installed today.

---

## 3. Skill shape (information hierarchy)

**Invocation: model-invoked.** It needs autonomous reach — an AFK runtime (or an orchestrating agent) fires it on a `ready-for-agent` story; a developer can also type it. So it keeps a description and pays the context load.

**Two files, detail one level deep** (mirrors `rai-grill-architect` + `repo-pure-slices.md`):

- **`SKILL.md`** — the ordered build loop (steps) + a short reference block (preconditions, output/boundaries, dependencies, out-of-scope). Tight, ≤~100 lines.
- **`build-checklist.md`** (sibling, disclosed) — the mechanics each step needs, pulled out so the top stays legible: greenfield-harness handling, BE-provider vs FE-mock specifics, the branch-write/prod-blind enforcement checklist, the PR-template fields + context-feedback note format, the repo-pure reminder. Reached by a context pointer from the relevant steps.

### Draft description (model-invoked)

> Build one `ready-for-agent` story into a reviewed, test-backed pull request — test-first, on a branch that never touches `main`. Phase 4 of the AI-native pipeline. Use when an agent or developer is ready to implement a single **repo-pure** story (it links a PRD and names the contract it targets) and open its PR, or when an AFK runtime picks up a `ready-for-agent` story to build. Follows `to-issues`; precedes human review.

One trigger per branch (implement-a-story / AFK-picks-up-a-story), a position clause for reach, leading word **Build** front-loaded.

---

## 4. Leading words

The predictability anchors. Most already live in the project's shared language — which is exactly why the skill should reuse them (shared language → reliable invocation *and* execution). The implementer should use these words verbatim, not synonyms.

- **branch-write, prod-blind** — the whole access envelope in one phrase: write branches never `main`, deploy preview/QA never prod, synthetic/fixture data never customer.
- **repo-pure** — one side, one repo per story.
- **red → green** — the TDD loop's state. The acceptance test goes **red** first (for the right reason), then the build turns it **green**. Binary, observable.
- **the fix belongs in the context** — the locked sub-thesis driving the context-feedback step: a decision or gap the build surfaces is noted *back to the context*, not buried in the diff.
- **build against the contract** — BE is provider-tested against the OpenAPI contract; FE builds against the **mock** generated from it, so each side is buildable without the other present.
- **the seam** — `codebase-design`'s word. The build happens at the seams the story names; the v2 second-model challenge slots into **the review seam**.
- **the environment contract** — the agnostic preconditions (D1).

---

## 5. SKILL.md content

### Preconditions — the environment contract *(reference)*

At start, these must hold (the skill checks, and stops with a clear ask if one is missing):

1. A **writable checkout of the target code repo**, on a clean integration branch (BE `develop` / FE `dev`), with push rights to **branches only**.
2. A **readable Context repo** (NexaContext): `CONTEXT.md`, `docs/adr/`, `features/<slug>/PRD.md`, `features/<slug>/contract/`.
3. A **test runner + run command** for the repo (the one-time per-repo ADR from D3).
4. **One repo-pure story** that links its PRD and names the **contract version** + the shared shape it targets.

### Inputs to read *(first legwork, before writing anything)*

The **story** (one side, one repo) · its **PRD** (the acceptance criteria = "done") · the **OpenAPI contract** + shared shape it references · the **glossary** (use its exact vocabulary) · the **ADRs** touching this area (honor them; if the build would contradict one, **surface it** — never silently override) · the **code in place** at the named seams.

### The build loop *(steps — each ends on a checkable completion criterion)*

| # | Step | Completion criterion |
|---|---|---|
| 1 | **Branch.** Create `feature/<slug>` off the integration branch; the agent drives git. | A `feature/<slug>` branch exists off the integration branch, you are on it, and `main` is untouched. |
| 2 | **Red.** Write the acceptance test from the PRD's ACs. If the module has no test setup, scaffold the test file + local wiring (→ `build-checklist.md`); the runner itself is assumed present. | The acceptance test exists and **runs red for the right reason** — it fails because the behavior is absent, not because the harness is broken. |
| 3 | **Green.** Write the minimal code, **against the contract** / mock, at the named seams. Run typecheck + the test command after each meaningful change. | The acceptance test and the full test command run **green**, and typecheck passes. |
| 4 | **Skeptical review.** Run a standards-and-spec `review` pass with an explicit red-team posture — meets the ACs? follows repo standards + the ADRs? where are the holes? *(This is the review **seam** the v2 second model slots into.)* | **Every** PRD acceptance criterion is accounted for, **every** touched ADR honored or its contradiction surfaced, and each review finding is addressed or explicitly deferred with a reason. |
| 5 | **The fix belongs in the context.** Capture any decision, ambiguous AC, missing ADR, or glossary drift the build surfaced — as a note for the P5 review to fold back into NexaContext. The skill **notes** it; it does not edit the Context repo mid-build. | Every decision/gap the build surfaced is captured as a context-feedback note (or there were none). |
| 6 | **PR from template.** Commit, push the branch, open a PR using the repo's template (→ `build-checklist.md` for fields). Stop. | A PR is open from the branch using the template, linking the story + PRD, naming the contract version built against, listing the tests added and the context-feedback notes. |

### Output + boundaries *(reference)*

A branch + an open PR; tests green; review addressed; context-feedback noted. **branch-write / prod-blind** throughout. Lower boundary: **PR opened** — P5 review/merge and P6 regression are downstream.

### Dependencies + graceful degradation *(reference)*

Models on `implement`; leans on `tdd` (the red→green loop) and `review` (step 4) **when present**. Because `implement`/`tdd` are not installed today, the loop is **restated inline** and runs standalone. Tool-neutral throughout, per the repo's authoring rule — no Claude-only tools or commands named.

### Out of scope (v1) *(reference)*

Second-model Codex challenge (v2, at the review seam) · self-selecting the next story (the AFK trigger's job) · P5 review/merge · P6 regression CI · P7 release notes (`rai-release-notes`) · cross-repo orchestration (each side is its own story/run).

---

## 6. The sibling — `build-checklist.md`

What it holds (the mechanics each step points at):

- **Greenfield-harness handling** — how to scaffold the first test in a previously-untested module (test file + local wiring), and when to stop and ask for a runner instead (D3's boundary: the framework is a precondition, the first test is not).
- **Contract mechanics** — BE provider-tested against the OpenAPI contract; FE built against the generated **mock**. (For NexaCore the shared shape *is* a versioned OpenAPI contract per ADR-0002, which consciously overrides the architect skill's lighter "shared shape" — state this so the build doesn't drift back to the lighter reading.)
- **branch-write / prod-blind enforcement** — the concrete checklist: branch off integration not `main`; never push `main`; no prod deploy; fixtures/synthetic data only.
- **PR template fields** — story link, PRD link, contract version, tests added, context-feedback notes — and the note format step 5 produces.
- **Repo-pure reminder** — build exactly one side; the other side is its own story; reference the shared shape, don't re-describe it.

---

## 7. Draft SKILL.md skeleton (for the implementer)

```
---
name: rai-build-story
description: <the model-invoked description from §3>
---

<built-on>
Restates the test-first build loop inline (red → green → review → PR). Leans on the
community tdd and review skills when present, but does not require them — implement/tdd
are not guaranteed installed. Tool-neutral throughout. Contrast the grill skills, which
hard-depend on their engines; this one degrades gracefully.
</built-on>

<preconditions>   … the environment contract (§5) …            </preconditions>
<read-first>      … inputs (§5) …                               </read-first>
<the-build-loop>  … steps 1–6, each with its completion criterion … </the-build-loop>
<guardrails>      branch-write / prod-blind; repo-pure; stop at PR </guardrails>
<when-done>       PR open → hands to P5 human review …          </when-done>

See build-checklist.md for harness scaffolding, contract/mock mechanics, the
prod-blind checklist, and the PR-template fields.
```

---

## 8. Open items (resolve at build time, do not block the spec)

- **Story-input convention.** How a run is pointed at its story — issue URL / number, or a `.scratch/<feature>/` path. Note the known `.scratch/` vs `features/<slug>/` parking-lot mismatch from the dry-run: PRD + contract live in NexaContext `features/<slug>/`; the *story* lives in the code repo's tracker. The skill reads context from the former and is handed the latter.
- **PR template source.** Whether each code repo already ships a PR template the skill fills, or the skill carries a default.
- **Assisted vs AFK posture.** v1 runs to PR and stops; "assisted/supervised" is the human choosing to watch and interrupt, not skill branching. Confirm no built-in approval checkpoint is wanted before code is written (the staged-autonomy decision lives in the operating model, not the skill).
- **Version bump.** Adding the skill bumps `plugins/rai-skills/.claude-plugin/plugin.json` (silent-drift: installs won't pick it up otherwise).

---

## 9. References

- `rai-skills/CLAUDE.md` — "Skill suite & roadmap" (the `rai-build-story` sketch this sharpens).
- `NexaContext/CLAUDE.md` — the pipeline, the Workspace, agent-drives-git, the contract.
- `rai-grill-architect/SKILL.md` + `repo-pure-slices.md` — house style and the repo-pure/shared-shape rules this consumes.
- ADR-0002 (NexaContext) — OpenAPI contract-first; the concrete "build against the contract."
- AI-NativeDev decisions G (TDD-from-day-one), J (branch-write/prod-blind), L (staged autonomy).
