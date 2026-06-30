---
name: rai-implement-story
description: Implement one ready-for-agent story into a reviewed, test-backed pull request — Phase 4 of the AI-native pipeline. Use when an agent or developer is ready to implement a single repo-pure story (it links a PRD and shapes the seam it builds against — an internal interface or a cross-repo contract) and open its PR, or when an AFK runtime picks up a ready-for-agent story to build. Follows to-issues; precedes human review.
---

<built-on>

This skill restates the test-first build loop inline — **red → green → review → PR** — so it runs standalone. It leans on the community **tdd** and **review** skills when they are installed, and is modeled on **implement**, but does not require them — `implement` and `tdd` are not guaranteed present.

</built-on>

<preconditions>

Confirm the **environment contract** before building. If one is missing, stop and ask for it rather than improvising. Its shape depends on the **archetype** — a two-repo product (e.g. NexaCore: a code repo + a separate Context repo) or a single-repo tool/library (code and context in **one** repo, where "the contract" is the story's shaped internal seam):

1. A **writable checkout of the target code repo**, on a clean integration branch (NexaCore: backend `develop`, frontend `dev`; single-repo: that repo's integration branch), with push rights to **branches only**. *(Exception — the **bootstrap story** of a net-new product: there is no repo yet; standing it up is what this story does. See build-checklist.md.)*
2. **Readable context** — `CONTEXT.md`, `docs/adr/`, the feature's `PRD.md`, and the seam the story builds against. In a two-repo product this is the Context repo (NexaContext: `features/<slug>/PRD.md`, `features/<slug>/contract/`); in a single-repo tool it's the same repo.
3. A **test runner that can run *this story's* test in the loop** — locally, without infra the checkout lacks. A runner that only works against a live app, a real database, or a network the build can't reach (e.g. an E2E-only frontend with no mock or component harness) does **not** satisfy this: if that's all there is, **stop and flag it** rather than writing a test you can never run red→green. (Standing up a runner where there is none is normally a one-time per-repo decision, not this skill's job — **except on the bootstrap story of a net-new product, where creating the repo, runner, and CI per the founding ADRs _is_ the story** — see build-checklist.md.)
4. **One story scoped to one side, one repo** that links its PRD and names *and points at* the **shaped seam** it targets — a versioned cross-repo contract, or an internal interface — and that earned its **`ready-for-agent`** label at the **ReadyToCode gate** (the `rai-ready-to-code` skill: an independent, adversarial review, architect-approved).

How those got there — a bootstrapped workspace, an AFK sandbox, a laptop — is not this skill's concern.

</preconditions>

<read-first>

Before writing anything, read:

- the **story** (the unit of work — one side, one repo);
- its **PRD** — the feature's `PRD.md` at the layout the repo's `docs/agents/issue-tracker.md` specifies (`features/<slug>/PRD.md` in NexaContext; `.scratch/<feature>/PRD.md` when no tracker is configured yet, e.g. a net-new repo) — the acceptance criteria are your definition of "done";
- the **shaped seam** the story builds against — a cross-repo contract (`features/<slug>/contract/`) or an internal interface — you **build against the seam**;
- the **glossary** (`CONTEXT.md`) — use its exact vocabulary;
- the **ADRs** touching this area — honor them; if your build would contradict one, **surface it**, never silently override;
- the **code in place** at the seams the story names.

**The PRD's "done" is rulebook-defined — verify it, don't assume it.** Preview the PRD (and any artifact this story produces, e.g. a `contract/`) with `rai-adlc check <feature>`: a `done` means the bar the rulebook sets is met; an `incomplete` with named missing pieces is a loose end the ReadyToCode gate should have caught — surface it rather than building on a half-spec. The agent self-checks here so a human never has to type "validate".

</read-first>

<the-build-loop>

Run these in order. Each step is done only when its check holds.

1. **Branch.** Start from a clean tree, then **fetch `origin` and branch `feature/<slug>` off the latest `origin/<integration>`** (NexaCore: backend `develop`, frontend `dev`; single-repo tool: that repo's integration branch, e.g. `main`) — never a stale local ref; if the local integration branch is behind, fast-forward it first. Branching off a stale integration branch silently builds against outdated code. You drive git, not the product person. **Bootstrap story (net-new repo):** there is no `origin` to fetch — `git init`, commit the scaffold, and branch off the local integration branch; push when a remote exists. This no-origin path fires *only* when there is genuinely no remote, never when a fetch merely failed (build-checklist.md).
   *Done when:* the branch exists off the **freshly-fetched** integration tip (or, for a bootstrap story, the freshly-initialized repo), you are on it, and `main` is untouched.

2. **Red.** Write the acceptance test from the PRD's acceptance criteria. If the module has no test setup, scaffold the test file and its local wiring — an untested module is not a missing runner (see build-checklist.md); the runner is assumed present **except on the bootstrap story, where standing it up per the founding ADRs is part of this story's green step**.
   *Done when:* the test exists and **runs red for the right reason** — it fails because the behavior is absent, not because the harness is broken.

3. **Green.** Write the minimal code to pass — **build against the shaped seam** (its signatures, or a contract/mock for cross-repo work), at the seams the story names. Run typecheck and the test command after each meaningful change.
   *Done when:* the acceptance test and the full test command run **green**, and typecheck passes.

4. **Independent skeptical review.** Attack the build with a red-team posture — ideally a fresh reviewer that did **not** write the code (a second agent/model, or at minimum a clean pass), because the author's own tests encode the author's blind spot. Does it meet every acceptance criterion? does it follow the repo's standards and the ADRs? where are the holes? Treat a green suite as suspect, not proof: **green tests ≠ acceptance criteria met** when the test setup quietly dodges the hard case. See build-checklist.md for what this pass catches.
   *Done when:* every acceptance criterion is accounted for, every touched ADR is honored or its contradiction surfaced, and each finding is addressed or deferred with a stated reason.

5. **The fix belongs in the context.** Capture anything the build surfaced that belongs in the durable context — an ambiguous acceptance criterion, a missing ADR, glossary drift — as a note for the reviewer to fold back into the durable context (the Context repo in a two-repo product, or this same repo's `CONTEXT.md`/`docs/adr/` in a single-repo tool). Capture it as a note; don't scope-creep the build by editing the context mid-build.
   *Done when:* every decision or gap the build surfaced is captured as a context-feedback note, or there were none.

6. **PR from template.** Commit, push the branch, and open a pull request using the repo's template (fields in build-checklist.md). Then stop.
   *Done when:* a PR is open from the branch, linking the story and PRD, naming the seam (and, for a cross-repo contract, its version) built against, and listing the tests added and the context-feedback notes.

</the-build-loop>

<guardrails>

- **branch-write, prod-blind** — write branches never `main`; deploy preview/QA never prod; use synthetic/fixture data never customer data.
- **repo-pure** — build exactly one side. A feature that spans backend and frontend is two separate runs; reference the shared shape, don't re-describe it.
- **Stop at the PR.** Human review and merge (Phase 5) and regression (Phase 6) are downstream, not yours.
- **Dry run when asked, or when you can't publish.** For a dry run — or in a throwaway/sandbox checkout with no push rights — do steps 1–5, then **emit the PR body as text and stop**: no commit-to-push, no PR. Honoring "branch-write" never means inventing a push you can't make.

</guardrails>

<when-done>

Hand off the open PR to human review (Phase 5). Release notes are Phase 7 (`rai-release-notes`). The step-4 review runs **before** the PR — it gates human review, it doesn't replace it; a solo runtime with no second agent still runs it against its own work and never skips.

</when-done>

See **[build-checklist.md](./build-checklist.md)** for harness scaffolding, seam & contract/mock mechanics, the prod-blind checklist, and the PR-template fields.
