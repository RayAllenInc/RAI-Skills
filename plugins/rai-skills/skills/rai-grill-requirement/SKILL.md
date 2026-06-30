---
name: rai-grill-requirement
description: Grill a product manager about a feature idea in plain, outcome-focused language — the stage-1 counterpart to rai-grill-architect. Use when a PM, or anyone steering product direction, wants to pressure-test a feature idea or firm up requirements before it becomes a PRD.
---

<built-on>

This skill is the **grilling** loop run for a product manager, capturing decisions through **domain-modeling** — it borrows those two engines (the same pair the community **grill-with-docs** wrapper composes) and adds a product lens rather than re-implementing them.

**Reuse:** the base grilling loop from **grilling** (interview relentlessly, one question at a time, walk the decision tree resolving dependencies, recommend an answer each time, explore the codebase to answer your own questions) and the decision-capture formats from **domain-modeling** — the `CONTEXT.md` glossary and the ADR format it defines.

**Dependency:** the **grilling** and **domain-modeling** skills must be available — they ship in the RAI critical-skills roster (alongside the **grill-with-docs** wrapper that composes them). If you can't find them, run the **rai-setup-skills** onboarding (it installs the roster), then continue.

**Override:** **domain-modeling** carries a code-facing step — when the user states how something works, cross-reference the code and surface contradictions aloud. The two rules below **replace** that posture for this product grill. Anything code-facing stays backstage; persona, framing, and what you may say are governed here, not there.

</built-on>

<what-to-do>

Interview me relentlessly about this feature idea — using the grilling loop above — until the requirements are sharp and we share an understanding.

Start from the idea I bring — if I haven't stated it, ask me to in a sentence.

I am a product manager. I think in users, outcomes, and value, not code. Keep every question answerable from that seat.

</what-to-do>

<frame-first>

Two quick framings **before** the relentless interview — they decide what you're even grilling. Do these first, in plain language, then start the loop.

**1. Size it: story, feature, or epic?** If the idea spans several independent user-facing capabilities (e.g. a whole *reporting module*, not one report), it's an **epic** — too big for a single grill. Stop and help me name the **features** inside it, then grill **one feature at a time** (one `features/<slug>/` each). One grill = one feature = a handful of stories. If the idea is a whole **product** (several epics), sketch the **epic map** first and record it beside the feature folder, pick the first epic, then grill one feature at a time — a single grill that swallows an epic (let alone a product) yields a shapeless PRD and a marathon session.

**2. Greenfield or brownfield?** Ask up front whether this is net-new or an **expansion of something that already exists** — usually the latter in our shop. Record **whichever it is** as a binding constraint so it lands in the PRD and the architect inherits it: brownfield → *"expansion of `<X>`: reuse what's there, don't rebuild it"* (don't make me keep repeating "we already have RBAC, don't reinvent it"); greenfield → *"net-new: no existing system to reuse — design from scratch"* (so the architect designs the to-be instead of hunting for code to cite). (You still keep the codebase backstage per rule 1 — on brownfield, read the code to *know* what already exists; just don't cite it at me.)

</frame-first>

<two-rules>

These are what separate this from the base grill. Never break them.

**1. The codebase is backstage.** Read the code to learn what to ask and how sharply — but no question ever references it. No file names, no "the system currently does X," no internals. I don't know the underlying code. Meet me as a strategist who understands the product, never as a code reviewer. (This replaces domain-modeling's "cross-reference with code" / "surface the contradiction" step — keep that for your own thinking only.)

**2. Ask the outcome shadow.** Every consideration — even a deeply technical one — has a user-facing question its answer hinges on. Ask *that*, in plain language:

- "Sync or async?" → "Does this need to feel instant, or is a short wait fine?"
- "Single- or multi-tenant data?" → "Could two companies ever share one renewal?"

If a consideration has no outcome shadow I could weigh in on, don't ask it — park it for the architect.

</two-rules>

<coverage>

Make the first session the best, most complete one: exhaustive in what you cover, humane in how it feels.

Cover at least these — a floor, not a ceiling:

1. **The user** — who exactly; primary vs secondary
2. **The problem** — the pain, how they cope today, why now
3. **Success** — what "this worked" looks like; how we'd know
4. **The words** — what key terms mean; disambiguate overloaded ones (→ glossary)
5. **Smallest valuable slice** — the first version worth shipping
6. **Out of scope** — what we're deliberately not doing
7. **The experience** — what the user sees and does; the happy path
8. **The edges** — user-visible "what happens when…" cases
9. **Trade-offs** — when two good things collide, which wins and why (→ ADR)
10. **Risks & bets** — what we're assuming; what would sink it
11. **Which one exactly** — when the outcome rests on "the X" but several similar things could be that X (which date, which status, which user), make me name the exact one. Overloaded words hide the real decision.
12. **Already half-solved?** — does the product already do part of this somewhere else? If so, reconcile with it so there's one source of truth — don't quietly ship a second version that can disagree with the first.

Before finishing, run a completeness sweep: what's unique about *this* idea that none of the standard areas caught? Chase it — the best questions live there.

</coverage>

<how-it-feels>

A sharp colleague over coffee, not an intake form.

- One genuine question at a time — never two at once.
- Lay every turn out the same way — open with the previous answer as a Q/A card, a divider, then the new question in a quote bar with its options (⭐ on your pick) and a short **Why**. See [message-format.md](./message-format.md).
- Keep the **Why** to a sentence or two — why the pick beats the alternatives, including what's at stake when that isn't obvious.
- After each area, reflect back the settled decisions as the Q/A ledger so I see what's locked, then move on.
- Allow "park it / move on / not sure yet" — say so once up front, then resurface only on a hard question. Note the gap (or send it to the architect pile) rather than force a weak answer.

</how-it-feels>

<capture>

Capture decisions inline as they crystallize — don't batch them. Use domain-modeling's `CONTEXT.md` and ADR formats, with one override: **keep every entry in product language** — outcomes and trade-offs from the user's and business's point of view, never build mechanics.

**Glossary (`CONTEXT.md`).** When a term resolves, or I use a word that conflicts with an existing one or is vague ("account" — the company paying us, or the person logging in?), pin it down and record it. Keep it a glossary — no implementation details.

**Decisions (ADRs).** Offer one only when the call is hard to reverse, surprising without context, and a real trade-off — framed in pure product language.

**Architect parking lot.** Deferred build questions go inline into `architect-questions.md` in this feature's folder. **Resolve that folder from the repo — don't assume it:** if `docs/agents/issue-tracker.md` exists, follow the layout it specifies (e.g. NexaContext uses `features/<slug>/`); only if there is no such config, fall back to `.scratch/<feature>/`. Keep it beside where the PRD will live. Many derive from my own answers — record the answer as the constraint: "Data must be fresh within the hour → architect: choose a refresh approach that meets that." This file feeds the later architect grill.

Read any existing `CONTEXT.md` and `docs/adr/` before exploring; create them lazily, only once there's something to write.

</capture>

<when-done>

When coverage is solid and the sweep is clean, summarize the decisions in the conversation, then hand off: tell me to run `to-prd`, which synthesizes the PRD for the architect to review from this conversation and the glossary and ADRs we captured. (`to-prd` doesn't read the parked-questions file — that's for the later architect grill — so anything the PRD needs must surface in the conversation.) Don't write the PRD here — that's `to-prd`'s job.

**Then commit the PRD on the feature branch** where `docs/agents/issue-tracker.md` specifies (e.g. `features/<slug>/PRD.md`; `.scratch/<feature>/PRD.md` when none is configured), not only as a tracker issue. The branch is the source of truth: a committed `PRD.md` is what lights up the *Requirements* stage for status tooling and what `rai-implement-story` reads as its definition of done. (`to-prd` is a community skill, so this on-branch commit is the rai convention layered on top of it — do it after `to-prd` runs.) For a **net-new product with no repo yet**, the PRD and ADRs live in `.scratch/<feature>/` and this on-branch commit is **deferred to the bootstrap story** — the first build story, which creates the repo — restoring the chain once the repo exists.

**What "a complete PRD" means is not described here — it lives once in the central rulebook** (the versioned, per-artifact definition of "done", so the bar can't drift across skills). Preview the PRD against it with `rai-adlc check <feature>`: it reports **done / incomplete / absent** and names the required sections still missing, so the PRD is finished before handoff instead of a human catching the gap stages later. Cite the rulebook and run the check — don't restate the required-section list here.

</when-done>
