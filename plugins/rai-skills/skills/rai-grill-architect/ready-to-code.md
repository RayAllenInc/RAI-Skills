# ReadyToCode gate (Phase 3)

The human hinge between "designed" and "built." A story earns the **`ready-for-agent`** label only when a gatekeeper can answer **yes** to every line below. This is pure judgment — *"could an agent build this cold, with nobody in the room?"* Yes → AFK build. No → send it back to be written down properly. You earn AFK by context quality, not by trusting the agent more.

## The checklist

- [ ] **Dependencies satisfied** — everything this story needs already exists, or is a named earlier story. Nothing it claims to "reuse" is actually unbuilt (open the code and check).
- [ ] **Acceptance test is concrete and runnable** — a builder could turn it into a red test *today*, with the runner and data the repo actually has. No "test against the live app" hand-waves.
- [ ] **Every real decision is an ADR** — the story rests on recorded decisions, not on what was said in the room. A cold reader sees the *why*.
- [ ] **Repo-pure** — one side, one repo. The contract version it targets is named.
- [ ] **Thin enough for one context window** — a single agent can hold the whole story, its PRD slice, and the seams it touches at once.
- [ ] **Glossary-clean** — every domain word it uses is defined in `CONTEXT.md`, with no overloaded term left ambiguous.

## Who confirms — and why it can't be the author

The gatekeeper is **someone other than the story's author** — a rotating co-architect. A gate the author applies to their own work isn't a gate. Record the confirmer on the story so it's auditable:

```
Status: ready-for-agent
Gated-by: <name>      # MUST differ from the author
```

A story marked `ready-for-agent` with no distinct `Gated-by:` line **has not passed the gate** — treat it as not-ready and send it back.

## PR-review rubric (Phase 5 — the other side of the gate)

When the agent's PR comes back, the reviewer checks:

- [ ] Meets **every** acceptance criterion in the PRD — not most.
- [ ] Honors the ADRs it touches; any contradiction is surfaced, not silently overridden.
- [ ] The test added actually runs, and fails without the change.
- [ ] Repo-pure and within the named contract version; no scope creep into the other side.
- [ ] Context feedback the build surfaced (ambiguous criteria, a missing ADR, glossary drift) is folded back into the Context repo — **the fix belongs in the context, not just the code.**
