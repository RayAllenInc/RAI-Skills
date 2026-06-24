# Build checklist & mechanics

The detail behind `rai-implement-story`'s steps. Read the step in `SKILL.md`; come here for *how*.

## Greenfield harness (step 2)

The **runner** is a precondition — the repo must already have a test framework installed and a command that runs the suite. Standing one up where there is none is a one-time decision (which framework, how it wires into CI) that belongs in an ADR, not in a story build. If there is no runner at all, **stop and ask** for one.

The **first test in a previously-untested module** is yours, though. When the runner exists but this module has never been tested:

- create the test file beside the code it covers, following the repo's existing test-file naming;
- add only the local wiring that one test needs (imports, a fixture, a minimal setup helper) — not a new framework;
- write fixtures as **synthetic data**, never copied from prod or a customer.

Keep the scaffolding minimal: enough to make this story's acceptance test run red, no speculative harness for tests you have not written.

## Seam mechanics (steps 2–3)

You **build against the shaped seam** the story names — its concrete interface, the decision already made upstream so you don't invent it. How heavy the seam is depends on the archetype:

- **Single-repo tool/library:** the seam is an **internal interface** (e.g. a Core↔adapter boundary). Build to the signatures the story shaped, and test through that seam with a fake / in-memory implementation. There is no separate contract artifact.
- **Cross-repo product (NexaCore):** the seam is a **versioned cross-repo contract** — a **versioned OpenAPI spec** in `features/<slug>/contract/`; ADR-0002 in NexaContext makes it the authority. The **backend** implements the endpoint to satisfy it and **provider-tests** against it; the **frontend** builds against the **mock generated from it**, so it's buildable before the backend merges.

When the seam is a versioned contract, name the **version** the story targets and record it in the PR (step 6) so "what we built against" survives the integration branches moving underneath you.

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
- **Context feedback** — the notes from step 5 (decisions, ambiguous ACs, missing ADRs, glossary drift) for the reviewer to fold back into the durable context (the Context repo, or this same repo in a single-repo tool) — or "none".

## Repo-pure reminder (guardrails)

One run builds one side. If the story seems to need both sides at once, it was sliced wrong upstream — flag it rather than building both. The other side is its own story, built against the same shared shape.
