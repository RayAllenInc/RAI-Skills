# Build checklist & mechanics

The detail behind `rai-implement-story`'s steps. Read the step in `SKILL.md`; come here for *how*.

## The runner: established repo vs. bootstrap story (steps 1–2)

**Established repo (the default).** The **runner** is a precondition — the repo already has a test framework installed and a command that runs the suite. Standing one up where there is none is *not* this skill's job here: it is a one-time decision (which framework, how it wires into CI) that belongs in an ADR. If a runner is expected but missing, **stop and ask** for one rather than improvising a parallel framework. The **first test in a previously-untested module** is still yours, though — when the runner exists but this module has never been tested:

- create the test file beside the code it covers, following the repo's existing test-file naming;
- add only the local wiring that one test needs (imports, a fixture, a minimal setup helper) — not a new framework;
- write fixtures as **synthetic data**, never copied from prod or a customer.

Keep the scaffolding minimal: enough to make this story's acceptance test run red, no speculative harness for tests you have not written.

**Bootstrap story of a net-new product (the one exception).** When the architect grill marked story #1 `Bootstrap: yes` (a net-new repo with no runner yet), standing up the project *is* the story — executed against the **founding ADRs** the architect recorded (language/runtime, framework, persistence, repo layout, test strategy, runner, CI — see the architect grill's `foundational-architecture.md`). Its green step:

- `git init` (no `origin` yet — see "Branching a net-new repo" below) and scaffold the project skeleton per the founding ADRs;
- install and wire the **test runner** + **CI**;
- **commit the `.scratch/<feature>/` PRD and ADRs into the new repo** at the configured layout, restoring "the branch is the source of truth";
- write the repo's **`docs/agents/issue-tracker.md`** and create the **`ready-for-agent`** label, so the *remaining* stories can be filed by `to-issues` and gated normally;
- prove the runner with a **real** red→green: bundle the first thin vertical feature (one acceptance test) into this story so something meaningful turns red. A pure-scaffold story whose only test is a trivial smoke test is weak test-first evidence — prefer a real first slice, and say so on the story if there genuinely isn't one.

## Branching a net-new repo (step 1)

The bootstrap story has no `origin` to fetch. `git init`, commit the scaffold, and branch `feature/<slug>` off the local integration branch; **push once a remote exists**. This no-origin path fires **only** when there is genuinely no remote — never as a fallback when a `fetch` merely failed (that still stops, to protect the freshness guarantee on an established repo).

## Seam mechanics (steps 2–3)

You **build against the shaped seam** the story names — its concrete interface, the decision already made upstream so you don't invent it. How heavy the seam is depends on the archetype:

- **Single-repo tool/library:** the seam is an **internal interface** (e.g. a Core↔adapter boundary). Build to the signatures the story shaped, and test through that seam with a fake / in-memory implementation. There is no separate contract artifact.
- **Cross-repo product (NexaCore):** the seam is a **versioned cross-repo contract** — a **versioned OpenAPI spec** in `features/<slug>/contract/`; ADR-0002 in NexaContext makes it the authority. The **backend** implements the endpoint to satisfy it and **provider-tests** against it; the **frontend** builds against the **mock generated from it**, so it's buildable before the backend merges.

When the seam is a versioned contract, name the **version** the story targets and record it in the PR (step 6) so "what we built against" survives the integration branches moving underneath you.

## Skeptical review (step 4)

The highest-value pass in the loop, and the one a green test bar most tempts you to skip. Run it **independent** of whoever wrote the code — ideally a fresh agent/model — because the author's tests encode the author's blind spot: the case the code gets wrong is often the exact case the author's test quietly avoids (a "safe" default that sidesteps the failure path, an input that never exercises the guard). So attack the *guarantee the story exists to make*, not the tests that are already green:

- **Re-derive each acceptance criterion from scratch** and confirm the build meets it — don't re-read the tests and nod along.
- **For every guarantee** (never-hangs, never-loses-data, byte-for-byte parity), find the input that would break it and confirm a test actually drives that input — a green suite that never feeds the dangerous case proves nothing about it.
- **If the same agent wrote and reviewed in one pass, say so in the PR** — a self-review is weaker evidence, and a second pass before merge is cheap insurance.

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
- **Seam** — the interface built against (and its version, for a cross-repo contract);
- **Tests** — what was added and what it proves;
- **Self-review** — whether the step-4 review was independent (a different agent/model) or the author reviewing their own build; flag the latter;
- **Context feedback** — the notes from step 5 (decisions, ambiguous ACs, missing ADRs, glossary drift) for the reviewer to fold back into the durable context (the Context repo, or this same repo in a single-repo tool) — or "none".

## Repo-pure reminder (guardrails)

One run builds one side. If the story seems to need both sides at once, it was sliced wrong upstream — flag it rather than building both. The other side is its own story, built against the same shared shape.
