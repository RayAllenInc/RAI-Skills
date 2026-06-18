# rai-build-story Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `rai-build-story` skill — Phase 4 of the AI-native pipeline — that turns one repo-pure, `ready-for-agent` story into a reviewed, test-backed pull request.

**Architecture:** A model-invoked Agent Skill: a thin `SKILL.md` carrying the `red → green → review → PR` build loop (each step gated by a checkable completion criterion) plus one disclosed sibling `build-checklist.md` for the mechanics. The skill restates its build loop inline so it runs standalone, and only *leans on* the community `tdd`/`review` skills when present.

**Tech Stack:** Markdown on the open [Agent Skills standard](https://agentskills.io) (`SKILL.md` + sibling); the Claude Code plugin marketplace (`rai`); PowerShell + git for verification. **No runtime code** — so per-task verification is structural (frontmatter parse, section/step coverage, tool-neutrality, line count) plus a tabletop dry-run eval for behavior, not unit tests.

**Spec:** `docs/superpowers/specs/2026-06-17-rai-build-story-design.md` (this repo, committed on the branch).

## Global Constraints

Every task implicitly includes these (copied verbatim from the spec and repo conventions):

- **Tool-neutral `SKILL.md`** — describe *what* to do, never a Claude-only tool or command. Portable to Codex/Gemini/Cursor.
- **Tight `SKILL.md`** (≈ ≤100 lines, hard ceiling 120); push detail one level deep into the sibling.
- **Every skill is prefixed `rai-`**; one skill per folder: `plugins/rai-skills/skills/<name>/SKILL.md` (+ siblings beside it).
- **Model-invoked** — keep the description; do **not** set `disable-model-invocation`.
- **Soft dependency** — model on `implement`, lean on `tdd`/`review` when present, but restate the loop inline; never hard-require them (`implement`/`tdd` are not installed today).
- **Leading words, used verbatim (no synonyms):** `branch-write, prod-blind` · `repo-pure` · `red → green` · `the fix belongs in the context` · `build against the contract` · `the seam`.
- **Behavioral envelope baked into the text:** branch-write/prod-blind; repo-pure (one side per run); stops at "PR opened."
- **Version bump** — adding the skill bumps `plugins/rai-skills/.claude-plugin/plugin.json` or installs won't pick it up.

All work continues on the branch that holds the spec. **Renamed in Task 1, Step 1** from `feature/rai-build-story-spec` → `feature/rai-build-story`. All `git`/verify commands assume the repo root `C:\Development\rai-skills`; run them from there (or with `-C`).

---

### Task 1: Author `SKILL.md`

**Files:**
- Create: `plugins/rai-skills/skills/rai-build-story/SKILL.md`

**Interfaces:**
- Consumes: nothing (first task).
- Produces: the skill folder + `SKILL.md` that Task 2 adds a sibling beside and Task 4 lists as built. The sibling link it emits is exactly `./build-checklist.md` (Task 2 must create that filename).

- [ ] **Step 1: Rename the working branch**

```powershell
git -C "C:\Development\rai-skills" branch -m feature/rai-build-story-spec feature/rai-build-story
git -C "C:\Development\rai-skills" branch --show-current
```
Expected: prints `feature/rai-build-story`.

- [ ] **Step 2: Create the skill folder + `SKILL.md` with the full content below**

File: `plugins/rai-skills/skills/rai-build-story/SKILL.md`

````markdown
---
name: rai-build-story
description: Build one ready-for-agent story into a reviewed, test-backed pull request — test-first, on a branch that never touches main. Phase 4 of the AI-native pipeline. Use when an agent or developer is ready to implement a single repo-pure story (it links a PRD and names the contract it targets) and open its PR, or when an AFK runtime picks up a ready-for-agent story to build. Follows to-issues; precedes human review.
---

<built-on>

This skill restates the test-first build loop inline — **red → green → review → PR** — so it runs standalone. It *leans on* the community **tdd** (the red→green loop) and **review** (the standards/spec pass) skills when they are installed, and is modeled on **implement**, but it does **not** require them: `implement` and `tdd` are not guaranteed present. This is a deliberate departure from the grill skills, which hard-depend on their engines.

Write every instruction **tool-neutrally** — describe *what* to do, never a tool or command specific to one agent — so the skill is portable to any skills-compatible agent.

</built-on>

<preconditions>

Confirm the **environment contract** before building. If one is missing, stop and ask for it rather than improvising:

1. A **writable checkout of the target code repo**, on a clean integration branch (backend `develop`, frontend `dev`), with push rights to **branches only**.
2. A **readable Context repo** (NexaContext): `CONTEXT.md`, `docs/adr/`, `features/<slug>/PRD.md`, `features/<slug>/contract/`.
3. A **test runner and a run command** for the repo. (Standing one up where there is none is a one-time per-repo decision, not this skill's job — see build-checklist.md.)
4. **One repo-pure story** — single side, single repo — that links its PRD and names the **contract version** and shared shape it targets.

How those got there — a bootstrapped workspace, an AFK sandbox, a laptop — is not this skill's concern.

</preconditions>

<read-first>

Before writing anything, read:

- the **story** (the unit of work — one side, one repo);
- its **PRD** (`features/<slug>/PRD.md`) — the acceptance criteria are your definition of "done";
- the **contract** (`features/<slug>/contract/`) and the shared shape the story references — you **build against the contract**;
- the **glossary** (`CONTEXT.md`) — use its exact vocabulary;
- the **ADRs** touching this area — honor them; if your build would contradict one, **surface it**, never silently override;
- the **code in place** at the seams the story names.

</read-first>

<the-build-loop>

Run these in order. Each step is done only when its check holds.

1. **Branch.** Create `feature/<slug>` off the integration branch; you drive git, not the product person.
   *Done when:* the branch exists off the integration branch, you are on it, and `main` is untouched.

2. **Red.** Write the acceptance test from the PRD's acceptance criteria. If the module has no test setup, scaffold the test file and its local wiring (see build-checklist.md); the runner itself is assumed present.
   *Done when:* the test exists and **runs red for the right reason** — it fails because the behavior is absent, not because the harness is broken.

3. **Green.** Write the minimal code to pass — **build against the contract** (or its mock), at the named seams. Run typecheck and the test command after each meaningful change.
   *Done when:* the acceptance test and the full test command run **green**, and typecheck passes.

4. **Skeptical review.** Review the change with a red-team posture: does it meet every acceptance criterion? does it follow the repo's standards and the ADRs? where are the holes? *(This review is **the seam** where the v2 second-model challenge will slot in.)*
   *Done when:* every acceptance criterion is accounted for, every touched ADR is honored or its contradiction surfaced, and each finding is addressed or deferred with a stated reason.

5. **The fix belongs in the context.** Capture anything the build surfaced that belongs in the durable context — an ambiguous acceptance criterion, a missing ADR, glossary drift — as a note for the reviewer to fold back into NexaContext. Note it; do **not** edit the Context repo mid-build.
   *Done when:* every decision or gap the build surfaced is captured as a context-feedback note, or there were none.

6. **PR from template.** Commit, push the branch, and open a pull request using the repo's template (fields in build-checklist.md). Then stop.
   *Done when:* a PR is open from the branch, linking the story and PRD, naming the contract version built against, and listing the tests added and the context-feedback notes.

</the-build-loop>

<guardrails>

- **branch-write, prod-blind** — write branches never `main`; deploy preview/QA never prod; use synthetic/fixture data never customer data.
- **repo-pure** — build exactly one side. A feature that spans backend and frontend is two separate runs; reference the shared shape, don't re-describe it.
- **Stop at the PR.** Human review and merge (Phase 5) and regression (Phase 6) are downstream, not yours.

</guardrails>

<when-done>

Hand off the open PR to human review (Phase 5). Release notes are Phase 7 (`rai-release-notes`). The second-model challenge — a second model attacking the build before the PR — is the planned upgrade at the review seam (step 4); it is not part of v1.

</when-done>

See **[build-checklist.md](./build-checklist.md)** for harness scaffolding, contract/mock mechanics, the prod-blind checklist, and the PR-template fields.
````

- [ ] **Step 3: Verify frontmatter + all six build steps are present**

```powershell
$p = "C:\Development\rai-skills\plugins\rai-skills\skills\rai-build-story\SKILL.md"
Select-String -Path $p -Pattern '^name: rai-build-story$','^description: Build one' | ForEach-Object { $_.Line }
(Select-String -Path $p -Pattern '^\d\. \*\*(Branch|Red|Green|Skeptical review|The fix belongs in the context|PR from template)\.\*\*').Count
(Select-String -Path $p -Pattern '\*Done when:\*').Count
```
Expected: the two frontmatter lines print; first count is `6` (all six steps); second count is `6` (every step has a completion criterion).

- [ ] **Step 4: Verify tool-neutrality, leading words, and line count**

```powershell
$p = "C:\Development\rai-skills\plugins\rai-skills\skills\rai-build-story\SKILL.md"
Select-String -Path $p -Pattern 'Edit tool','Bash tool','Skill tool','Write tool','claude code','/rai-'   # expect: no output
$lw = 'branch-write, prod-blind','repo-pure','red → green|red→green','the fix belongs in the context','build against the contract','the seam'
$lw | ForEach-Object { "{0}: {1}" -f $_, ([bool](Select-String -Path $p -Pattern $_)) }
(Get-Content $p | Measure-Object -Line).Lines
```
Expected: the first `Select-String` prints nothing (tool-neutral); every leading word reports `True`; line count is `≤ 120`.

- [ ] **Step 5: Commit**

```powershell
git -C "C:\Development\rai-skills" add "plugins/rai-skills/skills/rai-build-story/SKILL.md"
git -C "C:\Development\rai-skills" commit -m "feat(rai-build-story): add SKILL.md (Phase 4 build loop)"
```

---

### Task 2: Author the `build-checklist.md` sibling

**Files:**
- Create: `plugins/rai-skills/skills/rai-build-story/build-checklist.md`

**Interfaces:**
- Consumes: the `./build-checklist.md` link emitted by `SKILL.md` (Task 1) — this file must sit beside `SKILL.md` under that exact name.
- Produces: the five mechanics sections the SKILL.md steps point at (greenfield harness, contract mechanics, prod-blind checklist, PR template fields, repo-pure reminder).

- [ ] **Step 1: Create the sibling with the full content below**

File: `plugins/rai-skills/skills/rai-build-story/build-checklist.md`

````markdown
# Build checklist & mechanics

The detail behind `rai-build-story`'s steps. Read the step in `SKILL.md`; come here for *how*.

## Greenfield harness (step 2)

The **runner** is a precondition — the repo must already have a test framework installed and a command that runs the suite. Standing one up where there is none is a one-time decision (which framework, how it wires into CI) that belongs in an ADR, not in a story build. If there is no runner at all, **stop and ask** for one.

The **first test in a previously-untested module** is yours, though. When the runner exists but this module has never been tested:

- create the test file beside the code it covers, following the repo's existing test-file naming;
- add only the local wiring that one test needs (imports, a fixture, a minimal setup helper) — not a new framework;
- write fixtures as **synthetic data**, never copied from prod or a customer.

Keep the scaffolding minimal: enough to make this story's acceptance test run red, no speculative harness for tests you have not written.

## Contract mechanics (steps 2–3)

You **build against the contract** — the shared shape the two sides exchange, which for NexaCore is a **versioned OpenAPI spec** in `features/<slug>/contract/`. ADR-0002 in NexaContext makes the contract the authority; do not drift back to a looser "shared shape" reading.

- **Backend story:** implement the endpoint so it satisfies the contract, and **provider-test** against it — prove the response matches the shape the contract promises.
- **Frontend story:** build against the **mock generated from the contract**, so the frontend is buildable and testable before the backend merges.

Each story names the **contract version** it targets; record that version in the PR (step 6) so "what we built against" survives the integration branches moving underneath you.

## Prod-blind checklist (guardrails)

Before you push, confirm:

- [ ] branched off the integration branch, not `main`;
- [ ] no commit lands on `main`; no force-push;
- [ ] no deploy beyond preview/QA;
- [ ] every fixture is synthetic — no prod or customer data anywhere in the diff.

## PR template fields (step 6)

If the repo ships a PR template, fill it; otherwise include at least:

- **Story** — link to the issue this builds;
- **PRD** — link to `features/<slug>/PRD.md`;
- **Contract version** — the version built against;
- **Tests** — what was added and what it proves;
- **Context feedback** — the notes from step 5 (decisions, ambiguous ACs, missing ADRs, glossary drift) for the reviewer to fold back into NexaContext — or "none".

## Repo-pure reminder (guardrails)

One run builds one side. If the story seems to need both sides at once, it was sliced wrong upstream — flag it rather than building both. The other side is its own story, built against the same shared shape.
````

- [ ] **Step 2: Verify all five sections + the synthetic-data and contract rules are present**

```powershell
$p = "C:\Development\rai-skills\plugins\rai-skills\skills\rai-build-story\build-checklist.md"
(Select-String -Path $p -Pattern '^## (Greenfield harness|Contract mechanics|Prod-blind checklist|PR template fields|Repo-pure reminder)').Count
[bool](Select-String -Path $p -Pattern 'synthetic data')
[bool](Select-String -Path $p -Pattern 'ADR-0002')
```
Expected: count is `5`; both `[bool]` checks print `True`.

- [ ] **Step 3: Verify the SKILL.md → sibling link resolves**

```powershell
$dir = "C:\Development\rai-skills\plugins\rai-skills\skills\rai-build-story"
[bool](Select-String -Path "$dir\SKILL.md" -Pattern '\(\./build-checklist\.md\)')
Test-Path "$dir\build-checklist.md"
```
Expected: both print `True` (the pointer in SKILL.md matches a file that exists beside it).

- [ ] **Step 4: Commit**

```powershell
git -C "C:\Development\rai-skills" add "plugins/rai-skills/skills/rai-build-story/build-checklist.md"
git -C "C:\Development\rai-skills" commit -m "feat(rai-build-story): add build-checklist.md sibling"
```

---

### Task 3: Tabletop dry-run eval

Validate **behavior**, not just structure: have a fresh agent follow the skill on a synthetic repo-pure story and narrate each step + its completion check, then score it against a rubric. This is the v1 acceptance test; a full live build against NexaContext (the real acceptance test) is the follow-up noted at the end of this plan.

**Files:**
- Create: `docs/superpowers/evals/2026-06-17-rai-build-story-tabletop.md` (the synthetic story + the eval result)

**Interfaces:**
- Consumes: the finished `SKILL.md` + `build-checklist.md` from Tasks 1–2.
- Produces: a recorded eval verdict; if it fails, fixes loop back into Tasks 1–2 before Task 4.

- [ ] **Step 1: Write the synthetic story fixture**

Create `docs/superpowers/evals/2026-06-17-rai-build-story-tabletop.md` with this fixture at the top:

```markdown
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
<!-- filled in Step 3 -->
```

- [ ] **Step 2: Dispatch the eval (or run inline)**

Give a fresh agent ONLY the two skill files and the synthetic story, with this instruction:

> Follow the `rai-build-story` skill on Story FE-1. Do **not** write real code — instead, narrate what you would do at each step of the build loop and state how you would know its completion criterion is met. Then answer: did you hit all six steps in order? where would the first test go and would it run red for the right reason? what would you build against? where do you stop?

- [ ] **Step 3: Score against the rubric and record the verdict**

Append the verdict to the eval file. PASS requires **all** of:

```
[ ] Confirms the four preconditions before building (or asks for a missing one)
[ ] Walks all six steps in order: Branch → Red → Green → Skeptical review → Context feedback → PR
[ ] Step 2 (Red): scaffolds the FIRST test for the untested module but does NOT stand up a new runner
[ ] Step 3 (Green): builds against the contract v1 MOCK (frontend), not a guessed shape
[ ] Honors repo-pure: builds only the frontend side, never reaches for the backend
[ ] Honors branch-write/prod-blind: branch off integration, no main, synthetic data
[ ] Stops at "PR opened" — does not review/merge or write release notes
[ ] PR names the contract version (v1) and lists context-feedback notes
```
If any box fails, fix the skill (Task 1/2), re-run Step 2, and re-score. Do not proceed to Task 4 on a fail.

- [ ] **Step 4: Commit**

```powershell
git -C "C:\Development\rai-skills" add "docs/superpowers/evals/2026-06-17-rai-build-story-tabletop.md"
git -C "C:\Development\rai-skills" commit -m "test(rai-build-story): tabletop dry-run eval (PASS)"
```

---

### Task 4: Ship it — version bump + roadmap/README integration

Declare the skill built: bump the plugin version so installs pick it up, and move `rai-build-story` from "roadmap, not yet built" to built in the repo's docs.

**Files:**
- Modify: `plugins/rai-skills/.claude-plugin/plugin.json` (version)
- Modify: `README.md` (the "What's inside" table)
- Modify: `CLAUDE.md` (the "Skill suite & roadmap" section)

**Interfaces:**
- Consumes: a PASS verdict from Task 3.
- Produces: the released state — v0.5.0, docs listing the skill as built.

- [ ] **Step 1: Bump the plugin version `0.4.0` → `0.5.0`**

In `plugins/rai-skills/.claude-plugin/plugin.json`, change:
```json
  "version": "0.4.0",
```
to:
```json
  "version": "0.5.0",
```

- [ ] **Step 2: Verify the JSON is valid and the version took**

```powershell
$v = (Get-Content "C:\Development\rai-skills\plugins\rai-skills\.claude-plugin\plugin.json" -Raw | ConvertFrom-Json).version
$v
Get-Content "C:\Development\rai-skills\.claude-plugin\marketplace.json" -Raw | ConvertFrom-Json | Out-Null; "marketplace OK"
```
Expected: prints `0.5.0` then `marketplace OK` (both files parse; marketplace untouched).

- [ ] **Step 3: Add `rai-build-story` to the README "What's inside" table**

In `README.md`, add this row to the table (after the `rai-grill-architect` row):
```markdown
| `rai-build-story` | Builds one repo-pure, `ready-for-agent` story into a reviewed, test-backed pull request — test-first (`red → green`), branch-write/prod-blind, stopping at PR. Phase 4 of the pipeline; restates its loop inline and leans on community `tdd`/`review` when present. |
```

- [ ] **Step 4: Update the `CLAUDE.md` roadmap — move it from "Next" to built**

In `CLAUDE.md`, in the "Skill suite & roadmap" section:

1. Add a built-skill bullet after the `rai-grill-architect` bullet:
```markdown
- **`rai-build-story`** (built) — Phase 4 AFK build. A thin orchestrator that restates the test-first loop inline (`red → green → review → PR`) so it runs standalone, leaning on community `tdd`/`review` when present. Takes one repo-pure `ready-for-agent` story, builds against the contract/mock, stays branch-write/prod-blind, and stops at an opened PR. Detail in the skill's `build-checklist.md`. Spec: `docs/superpowers/specs/2026-06-17-rai-build-story-design.md`.
```
2. In the "**Next (roadmap, not yet built):**" paragraph, **remove** the `rai-build-story` clause (the parenthetical describing it) so only `rai-release-notes` and the demoted docs/infra items remain.

- [ ] **Step 5: Verify the docs now list it as built and drop it from "Next"**

```powershell
[bool](Select-String -Path "C:\Development\rai-skills\README.md" -Pattern '\| `rai-build-story` \|')
[bool](Select-String -Path "C:\Development\rai-skills\CLAUDE.md" -Pattern '\*\*`rai-build-story`\*\* \(built\)')
(Select-String -Path "C:\Development\rai-skills\CLAUDE.md" -Pattern 'Next \(roadmap, not yet built\).*rai-build-story').Count
```
Expected: first two print `True`; the third count is `0` (no longer advertised as un-built).

- [ ] **Step 6: Commit**

```powershell
git -C "C:\Development\rai-skills" add "plugins/rai-skills/.claude-plugin/plugin.json" "README.md" "CLAUDE.md"
git -C "C:\Development\rai-skills" commit -m "feat(rai-build-story): ship v0.5.0 — list as built in README + roadmap"
```

---

## Follow-ups (out of this plan)

- **Live dry-run** of `rai-build-story` against a real NexaContext workspace + a thin NexaCore story (the true acceptance test) — run when standing up Story #1.
- **Push + PR** the `feature/rai-build-story` branch to `origin` and open a PR against `main` (left local until you ask).
- **v2 seam:** the second-model Codex challenge at the review step (step 4).
- **Reconcile** the `.scratch/<feature>/` vs `features/<slug>/` parking-lot path mismatch across the pipeline (noted in the spec's open items).

## Self-Review

**1. Spec coverage** — every spec section maps to a task: §3 shape/description → Task 1 frontmatter; §4 leading words → Task 1 Step 4 check; §5 preconditions/inputs/loop/boundaries/deps/out-of-scope → Task 1 body; §6 sibling → Task 2; §7 skeleton → Tasks 1–2 content; the three decisions (D1 agnostic preconditions, D2 defer second model, D3 hybrid harness) appear in the SKILL.md text and are exercised by the Task 3 rubric; §8 open items → Follow-ups; the version bump (§8) → Task 4. No gaps.

**2. Placeholder scan** — no `TBD`/`TODO`/"handle edge cases"; both skill files are given in full; every verify step has a real command + expected output. The one `<!-- filled in Step 3 -->` is an intended fixture slot, filled within the same task.

**3. Type/name consistency** — `./build-checklist.md` is the link in Task 1 and the filename created in Task 2. Folder `plugins/rai-skills/skills/rai-build-story/` is identical across tasks. Version `0.4.0 → 0.5.0` matches the current `plugin.json`. The six step names in the SKILL.md content match the six the Task 1 Step 3 regex and the Task 3 rubric check for. Leading-word strings match between the SKILL.md text and the Task 1 Step 4 grep.
