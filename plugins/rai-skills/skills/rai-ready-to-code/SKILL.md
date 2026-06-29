---
name: rai-ready-to-code
description: Run the mandatory ReadyToCode review that gates a designed story to `ready-for-agent` — an independent, adversarial "no loose ends" pass before any build. Use at the architect→development handoff, before an AFK/agent build, or whenever asked to gate, sign off, or check whether a story is ready to code.
---

<built-on>

This is the **gate** between *designed* and *built* — Phase 3 of the RAI pipeline, the handoff from `rai-grill-architect` (and `to-issues`) into `rai-implement-story`. It restates its review loop inline so it runs standalone, and leans on the community **review** skill when that is installed.

</built-on>

<what-this-is>

A story earns the **`ready-for-agent`** label only when this review finds **no surviving loose end** and the architect approves. The bar is one question: *could an agent build this cold, with nobody in the room?* The review is how you answer it — not a glance, a **red-team** pass. Never offer a "the agent will figure it out" or "start building" shortcut that skips this gate.

</what-this-is>

<the-review>

Run the gate as an **independent, adversarial** review — independent of whoever wrote the story. Thoroughness is the point, so don't do one friendly read-through; run it as a panel:

1. **Review by lens.** Cover the story from several angles at once, one pass per lens, each against the checklist below:
   - *spec* — does it match its PRD slice, no more, no less?
   - *architecture* — does it honor the ADRs and **shape its seams**?
   - *buildability* — could a builder finish in one context window with nothing left to invent?
   - *gate integrity* — links, dependencies, the seam it targets (its contract, for a cross-repo seam).
2. **Red-team it.** Take each finding and try to refute it; then hunt for what the lenses missed. A finding is a real blocker only if it **survives** a skeptic trying to wave it away — this kills plausible-but-wrong nitpicks and surfaces the loose ends a friendly read skips.
3. **Verdict.** PASS only if no blocker survives. Otherwise **NOT-READY**: list each loose end with its one-line fix and send the story back to be written down properly.

Because the review — not the reviewer's identity — is what stays independent, a story's own author may run it; the architect gives the final approval on a PASS.

</the-review>

<the-checklist>

Every line must be **yes**:

- [ ] **Dependencies satisfied** — everything the story needs already exists or is a named earlier story; nothing it claims to "reuse" is actually unbuilt (open the code and check). *(Greenfield: nothing to open yet — check against the founding ADRs instead.)*
- [ ] **Seams shaped, not just named** — every interface the story builds against is written down concretely (signatures, the data shape, the error/edge cases), not referenced by name only. A *named* seam is a decision punted to the builder; a *shaped* seam is the decision made. For a cross-repo seam the shaped form is the contract — see the architect grill's `seams-and-contracts.md`.
- [ ] **Acceptance test is concrete and runnable** — a builder could turn it red *today*, with the runner and data the repo actually has. No "test against the live app." *(Bootstrap-story exception: on a net-new repo the runner may not exist yet — instead require the story to stand it up per the founding ADRs, with a test concrete enough to run the moment it does.)*
- [ ] **Every real decision is an ADR** — the story rests on recorded decisions, not on what was said in the room; a cold reader sees the *why*.
- [ ] **Repo-pure** — one side, one repo (always trivially true for a single-repo tool).
- [ ] **One-window** — a single agent can hold the whole story, its PRD slice, and the seams it touches at once.
- [ ] **Glossary-clean** — every domain word it uses is defined in `CONTEXT.md`, no overloaded term left ambiguous.
- [ ] **Links its PRD and the ADRs it rests on** — the artifact a builder reads carries them, not just the conversation.

</the-checklist>

<sign-off>

On a PASS, record the gate on the story so it's auditable — the review that cleared it and the architect who approved it:

```
Status: ready-for-agent
ReadyToCode: <the independent review — e.g. "adversarial AI review, 2026-06-24">
Approved-by: <architect>
```

A story marked `ready-for-agent` with no recorded `ReadyToCode:` line **has not passed the gate** — treat it as not-ready and send it back. On a clean PASS, hand the story to `rai-implement-story`.

**Setup (once per repo):** the `ready-for-agent` label must exist in the repo's tracker; create it if missing. *(A net-new product's **bootstrap story** has no tracker yet — gate it in `.scratch/<feature>/`, record the PASS inline in the story file, and defer the label until the bootstrap story creates the tracker.)*

</sign-off>
