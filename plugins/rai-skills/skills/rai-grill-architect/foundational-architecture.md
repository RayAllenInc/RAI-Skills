# Foundational architecture & the bootstrap story

How the architect grill handles a **net-new product** — one with no repo, no stack, and no test runner yet. This fires **only on the first feature of a net-new product** (the requirement grill recorded a *greenfield* constraint). Brownfield, and every later feature of a product already underway, skip everything here: the stack is already chosen, so coverage area 11 is a no-op and there is no bootstrap story.

## The founding ADRs (coverage area 11)

Before any feature seam can be shaped, a net-new product has to make the decisions a brownfield feature inherits for free. Capture each as an ADR — these are the **founding ADRs**, and they are the input the bootstrap story builds against:

- **Language / runtime** — and the version floor.
- **Framework(s)** — web/app framework, and the libraries that come with a real commitment (an ORM, a router).
- **Persistence** — the datastore and the access pattern (migrations, schema ownership).
- **Repo layout** — single repo vs. the two-repo product archetype; package/module boundaries; where `CONTEXT.md`, `docs/adr/`, and `features/<slug>/` live.
- **Test strategy** — the test levels (unit / integration / e2e), and which level each story's acceptance test targets.
- **The test runner** — the concrete framework and the command that runs the suite. This is the decision `rai-implement-story` otherwise *requires to already exist*; on a net-new product it is decided here and stood up by the bootstrap story.
- **CI** — where the suite runs on push, and the green bar a PR must clear.

Keep domain-modeling's bar: an ADR per decision that is hard to reverse and a real trade-off. A throwaway prototype can collapse several into one "stack" ADR; a product meant to live should not.

## The bootstrap story

These founding decisions need an **execution** owner, and no design skill builds. So the first story of a net-new product is a **bootstrap story**: its job is to bring the repo into existence per the founding ADRs. Mark it on the slice explicitly so the gate and the build can read it:

```
Bootstrap: yes
Founding-ADRs: <links to the language/runtime, framework, persistence, repo-layout, test-strategy, runner, CI ADRs>
```

`rai-implement-story` reads that marker and treats standing up the skeleton, runner, and CI as the story's green step (see its `build-checklist.md`), rather than stopping for a missing runner.

## Why it routes around `to-issues` (the ordering)

`to-issues` files into a repo's tracker, and the ReadyToCode gate's `ready-for-agent` label has to live in that tracker — but on a net-new product **the repo and tracker do not exist yet; the bootstrap story is what creates them.** That is a chicken-and-egg: the steps that need the repo run before the step that makes it.

So break the cycle by running the bootstrap story **out of band**:

1. Write the bootstrap story to `.scratch/<feature>/` (not through `to-issues`).
2. The gate reviews it there and records its PASS **inline in the story file** — the `ready-for-agent` label is deferred (no tracker yet).
3. `rai-implement-story` builds it: it `git init`s the repo, scaffolds, stands up the runner + CI, commits the `.scratch/<feature>/` PRD and ADRs into the repo (restoring "the branch is the source of truth"), and creates the tracker config (`docs/agents/issue-tracker.md`) plus the `ready-for-agent` label.
4. **Only now** does the standard flow resume: `to-issues` files every *remaining* story into the now-real tracker, and the gate uses the real label.

The mental model: **the bootstrap story brings the repo into existence; every story after it is the ordinary pipeline.**

## One honest strain — the bootstrap story's red test

Pure scaffolding has nothing meaningful to turn red — only a trivial smoke test — which thins the test-first discipline `rai-implement-story` rests on. Prefer to **bundle the first thin vertical feature** (one real acceptance test) into the bootstrap story, so the runner is proven by a genuine red→green rather than a smoke test. If the product truly has no shippable first slice yet, accept the smoke test, but say so on the story — it is weaker evidence that the harness works.
