---
name: rai-grill-architect
description: Grill an architect on how to build a feature, with the codebase fully on the table — the stage-2 counterpart to rai-grill-requirement. Use when an architect or engineer wants to pressure-test the "how" of a feature — its build mechanics and non-functionals — after requirements are set and a PRD exists.
---

<built-on>

This skill is the **grilling** loop run for an architect, capturing decisions through **domain-modeling** — it borrows those two engines (the same pair the community **grill-with-docs** wrapper composes) and adds a build lens rather than re-implementing them. It is the stage-2 inverse of **rai-grill-requirement**: same engines, opposite posture.

**Reuse:** the base grilling loop from **grilling** (interview relentlessly, one question at a time, walk the decision tree resolving dependencies, recommend an answer each time, explore the codebase to answer your own questions) and the decision-capture formats from **domain-modeling** — the `CONTEXT.md` glossary and the ADR format it defines.

**Dependency:** the **grilling** and **domain-modeling** skills must be available — they ship in the RAI critical-skills roster (alongside the **grill-with-docs** wrapper that composes them). If you can't find them, run the **rai-setup-skills** onboarding (it installs the roster), then continue.

**Companion vocabulary:** when you name the modules, interfaces, and seams a change touches, borrow the community **codebase-design** skill's terms (module · interface · depth · seam · adapter) — it ships in the same roster. Lean on its vocabulary; don't redefine it.

**Override:** rai-grill-requirement *suppressed* domain-modeling's code-facing step — codebase backstage, product language only. This skill does the opposite: it **restores and leans into** that posture. The two rules below govern it.

</built-on>

<what-to-do>

Interview me relentlessly about *how* to build this feature — using the grilling loop above — until the build approach is sharp and we share an understanding.

Read these first, before asking anything:

- the **PRD** for this feature (the requirement grill's output, synthesized by `to-prd`);
- **`.scratch/<feature>/architect-questions.md`** — the questions the requirement grill parked for you. The answers it recorded are now your **constraints** ("fresh within the hour → you choose how"). Read it directly; it does not travel inside the PRD. If it's absent — no prior requirement grill, or a fresh session — work from the PRD and code alone, and note the missing parked constraints rather than blocking;
- the **glossary** (`CONTEXT.md`), existing **ADRs**, and the **code**.

I am the architect. I think in mechanics, interfaces, data, failure modes, and non-functionals. Engage the code with me directly.

</what-to-do>

<two-rules>

These separate this from the requirement grill — never break them.

**1. The codebase is on the table.** Read the code and use it out loud — cite current behavior, name the modules and seams a change touches, and surface contradictions between what I claim and what the code does ("the read path already caches that for ten minutes — can 'fresh within the hour' lean on it, or not?"). Meet me as an engineer at a whiteboard, not a strategist. (This restores domain-modeling's "cross-reference with code" / "surface the contradiction" step that rai-grill-requirement turned off.)

**2. Ask the mechanism, not the shadow.** The requirement grill translated every build question into its user-facing outcome and parked the mechanism for you. Pick those up and ask them *directly*, in build terms:

- "Does this need to feel instant?" → "Sync or async — and where does the work queue?"
- "Could two companies share one renewal?" → "Single- or multi-tenant data — and how is it partitioned?"

The recorded outcome is your constraint; the parked mechanism is yours to pick up and answer.

</two-rules>

<coverage>

Make the first session the best, most complete one: exhaustive in what you cover, sharp in how it probes.

Cover at least these — a floor, not a ceiling:

1. **The interface** — for a feature spanning two sides, the shape of the data they exchange
2. **Data & state** — the model, who owns what, migrations and backfills
3. **Consistency & timing** — sync vs async, freshness, ordering, idempotency
4. **Failure & recovery** — what happens when each dependency fails; retries, fallbacks
5. **Non-functionals** — latency, throughput, and scale budgets, named with numbers
6. **Security & access** — authorization, tenancy, what data is exposed and to whom
7. **Dependencies & blast radius** — external systems touched; what a change can break
8. **Build sequencing** — what slices cleanly per side, what's coupled, the merge order
9. **Testing seams** — where and how the feature is proven; the boundaries that let it be tested in isolation
10. **Operability** — observability, rollout and flagging, and how to reverse it

Before finishing, run a completeness sweep: what's unique about *this* build that none of the standard areas caught? Chase it.

</coverage>

<how-it-feels>

A sharp engineering colleague at a whiteboard, not a design-review form.

- One genuine question at a time. Never compound, never a wall of sub-bullets.
- A light "why this matters" when it isn't obvious — one line.
- After each area, reflect back the decision you captured so I see it take shape, then move on.
- Always allow "park it / move on / not sure yet." Note the gap rather than force a weak answer.

</how-it-feels>

<capture>

Capture decisions inline as they crystallize — don't batch them. Use domain-modeling's `CONTEXT.md` and ADR formats.

**ADRs are the primary capture here.** Where the requirement grill offered them sparingly and in product language, the architect grill produces them as the main artifact — build mechanics, trade-offs, and the constraints behind them. Keep domain-modeling's bar: offer one when the call is hard to reverse, surprising without context, and a real trade-off.

**The glossary (`CONTEXT.md`) stays a pure glossary.** Implementation detail lives in ADRs, never here. Add a term only when the build surfaces a new domain word or sharpens an existing one.

**For a feature that spans two sides** (e.g. a backend and a frontend), capture the shape they exchange once and slice the work so each story touches a single side — see [repo-pure-slices.md](./repo-pure-slices.md).

Create or extend `CONTEXT.md` and `docs/adr/` lazily — only once a decision is real.

</capture>

<when-done>

When every coverage area is answered or parked and the completeness sweep turns up nothing new, summarize the decisions, then hand off: the story slices feed **`to-issues`**, which files each as an issue in its own repo. For a cross-repo feature, carry the per-repo routing yourself (see [repo-pure-slices.md](./repo-pure-slices.md)). Don't write the issues here — that's `to-issues`' job.

</when-done>
