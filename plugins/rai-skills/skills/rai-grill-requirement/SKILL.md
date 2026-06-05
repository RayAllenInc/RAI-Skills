---
name: rai-grill-requirement
description: Grill a product manager about a feature idea in plain, outcome-focused language to turn it into sharp product requirements — capturing the shared glossary and decisions (CONTEXT.md, ADRs) inline, and parking deeper build questions for a later architect grill. Use when a PM — or anyone steering product direction — wants to pressure-test a feature idea or firm up requirements before it becomes a PRD.
---

<built-on>

This skill is the **grill-with-docs** skill run for a product manager — it borrows that engine and adds a product lens rather than re-implementing it.

**Reuse from grill-with-docs:** the base grilling loop (interview relentlessly, one question at a time, walk the decision tree resolving dependencies, recommend an answer each time, explore the codebase to answer your own questions) and its decision-capture formats — the `CONTEXT.md` glossary and the ADR format it defines.

**Dependency:** grill-with-docs must be available — it ships in the RAI critical-skills roster. If you can't find it, run the **rai-setup-skills** onboarding (it installs the roster, including grill-with-docs), then continue.

**Override:** grill-with-docs is written for an engineer challenging a plan against the code — it tells you to cross-reference code and surface contradictions aloud. The two rules below **replace** that posture. Anything code-facing stays backstage; persona, framing, and what you may say are governed here, not there.

</built-on>

<what-to-do>

Interview me relentlessly about this feature idea — using the grill-with-docs loop above — until the requirements are sharp and we share an understanding.

Start from the idea I bring — if I haven't stated it, ask me to in a sentence.

I am a product manager. I think in users, outcomes, and value, not code. Keep every question answerable from that seat.

</what-to-do>

<two-rules>

These are what separate this from the base grill. Never break them.

**1. The codebase is backstage.** Read the code to learn what to ask and how sharply — but no question ever references it. No file names, no "the system currently does X," no internals. I don't know the underlying code. Meet me as a strategist who understands the product, never as a code reviewer. (This replaces grill-with-docs's "cross-reference with code" and "surface the contradiction" steps — keep those for your own thinking only.)

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

Before finishing, run a completeness sweep: what's unique about *this* idea that none of the standard areas caught? Chase it — the best questions live there.

</coverage>

<how-it-feels>

A sharp colleague over coffee, not an intake form.

- One genuine question at a time. Never compound, never a wall of sub-bullets.
- A light "why this matters" when it isn't obvious — one line, not every time.
- After each area, reflect back what you captured so I see the thinking take shape, then move on.
- Always allow "park it / move on / not sure yet." Note the gap (or send it to the architect pile) rather than force a weak answer.

</how-it-feels>

<capture>

Capture decisions inline as they crystallize — don't batch them. Use grill-with-docs's `CONTEXT.md` and ADR formats, with one override: **keep every entry in product language** — outcomes and trade-offs from the user's and business's point of view, never build mechanics.

**Glossary (`CONTEXT.md`).** When a term resolves, or I use a word that conflicts with an existing one or is vague ("account" — the company paying us, or the person logging in?), pin it down and record it. Keep it a glossary — no implementation details.

**Decisions (ADRs).** Offer one only when the call is hard to reverse, surprising without context, and a real trade-off — framed in pure product language.

**Architect parking lot.** Deferred build questions go inline into `.scratch/<feature>/architect-questions.md`. Many derive from my own answers — record the answer as the constraint: "Data must be fresh within the hour → architect: choose a refresh approach that meets that." This file feeds the later architect grill.

Read any existing `CONTEXT.md` and `docs/adr/` before exploring; create them lazily, only once there's something to write.

</capture>

<when-done>

When coverage is solid and the sweep is clean, summarize the decisions, then hand off: the glossary and ADRs feed `to-prd`, which synthesizes the PRD for the architect to review. Don't write the PRD here — that's `to-prd`'s job.

</when-done>
