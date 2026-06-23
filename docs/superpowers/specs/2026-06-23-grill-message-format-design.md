# Grill message format — design spec

**Date:** 2026-06-23 (rev. 2026-06-23) · **Owner:** Avinash Gadiraju · **Status:** Draft for review

A formatting upgrade for the **questions the grill skills ask during a querying session** — the one-question-at-a-time interview run by `rai-grill-requirement` and `rai-grill-architect`. Goal: **decrease the reader's cognitive load** so each turn is easy to read and easy to act on. Presentation only — not a new skill, and not a change to *what* gets asked.

The format is **proportional markdown**: the question in a quote bar (the anchor), a bulleted **Options** list with a **⭐ on the recommended option**, then a short **Why** explaining the pick.

> **Revision note (boxed → proportional).** This began as a monospace **boxed-question** layout — a Unicode box in a code block. Seen rendered live, the closed box came out **ragged** (the live agent couldn't reliably pad every line to a fixed column) and the **monospace prose** in `MY TAKE` was tiring to read. We pivoted to proportional markdown: nothing to misalign, and the reasoning reads in a normal font. The boxed design is preserved in git history (commits `7ed75ba`, `74392ca`).

Written to the `writing-great-skills` principles: a thin `<how-it-feels>` pointer plus one disclosed sibling (`message-format.md`) per skill; the layout is a checkable shape; tool-neutral (plain markdown any agent emits).

---

## 1. The problem

A grilling turn is several parts — an optional reflect-back, the question, a recommended answer, sometimes an option set, and the escape hatch. The base `grilling` engine only says "one question at a time," so the agent free-styles each turn into a prose blob. Three sources of load (a fourth, "no sense of progress," was explicitly **not** a problem, so progress breadcrumbs are out of scope):

| Pain | What it means |
|---|---|
| **Question gets buried** | reflect-back, "why", and the ask blur into one paragraph |
| **Hard to pick an answer** | the recommendation and options aren't scannable |
| **Too much text per turn** | each message is longer than it needs to be |

The first attempt (a monospace box) fixed "buried" but reintroduced "too much text" — fixed-width prose reads slower, and the box rendered ragged. Proportional markdown resolves all three: the quote bar anchors the question, the ⭐ makes the pick scannable, and normal-font prose is the most readable per word.

---

## 2. Decisions

**Scope: presentation only.** The higher-leverage **intent-elicitation** findings (ask-open-before-recommending, laddering, Mom Test, OARS) stay parked in **Appendix A** for a later pass — not built here.

| # | Decision | Rationale |
|---|---|---|
| **D1 — Proportional markdown, no monospace** | The turn is normal markdown; the only frame is a quote bar on the question. No code block, no box. | Live render proved the monospace box ragged + prose hard to read. Proportional has nothing to misalign and reads fastest per word (Plain Language, F-pattern). |
| **D2 — Question in a quote bar** | `> **QUESTION**` then the question on the next line. | The quote bar + bold label make the question the unmissable anchor — kills pain 1 — without a fragile box. |
| **D3 — Options list, ⭐ marks the pick** | Bulleted `Options`; bold each short label; a **⭐ at the end** of the recommended line. No separate `MY TAKE` when options exist — the star *is* the recommendation. | One scannable list; the eye lands on the ⭐ (pain 2). A recommended default collapses the choice (Hick's Law / defaults). |
| **D4 — Why after the options** | `**Why (x):**` — one or two sentences on why the ⭐ option beats the alternatives. | The user asked for an explicit reason. Placing it *after* the options keeps the question + choices on top, the rationale in support. |
| **D5 — Lean by default** | `Recap` only when something was just captured (a short italic lead-in). No discrete choices → drop `Options`, give a one-line `My take:` + `Why`. Keep `Why` to a sentence or two. | The brevity governor (Grice-Quantity, Krug). The 6-line `MY TAKE` seen live is the failure this prevents. |
| **D6 — Escape hatch once; reflect-back is prose** | "park it / move on / not sure yet" stated once at session open. The end-of-area reflect-back is a short prose summary, distinct from a question turn. | Per-turn escape lines are furniture; the area summary is a different message type. |

**Out of scope:** progress breadcrumbs; the Appendix A intent work; `rai-setup-skills` / `rai-build-story` messages; the community `grilling` / `domain-modeling` engines (RAI owns the wrappers, not those).

---

## 3. The format

```
> **QUESTION**
> Who should get the renewal reminder?

**Options**
- **(a) The owning account manager** — the one who can act on it ⭐
- **(b) The whole account team**
- **(c) The customer directly**

**Why (a):** the owner is the only one positioned to act before the renewal
lapses; cc'ing the whole team turns a clear responsibility into a diffuse one,
and the customer can't act on our behalf.
```

Rendered, that is a quote-barred question, a scannable starred list, and a short reason. **Collapsed** (open question, no discrete options): drop `Options`, give a one-line `My take:` then `Why`. The architect grill uses the identical shape in build register. Full worked examples live in each skill's `message-format.md`.

---

## 4. Where it lives (information hierarchy)

Mirrors the `rai-build-story` + `build-checklist.md` shape: a thin pointer in `SKILL.md`, detail one level deep in a sibling.

- **`message-format.md` in each skill dir** — the layout, the parts, the lean rules, and register-appropriate examples. Duplicated across the two dirs on purpose (each skill stays self-contained and portable; examples differ by register).
- **`<how-it-feels>` in each `SKILL.md`** points one level deep to the sibling and states the shape in one bullet.
- **No edits** to community `grilling` / `domain-modeling`.

---

## 5. Exact edits

### `<how-it-feels>` (both skills) — the changed bullets

**After (requirement; architect mirrors it in whiteboard voice):**

```
- One genuine question at a time — never two at once.
- Lay every turn out the same way — the question in a quote bar, the options with a ⭐ on your pick, then a short **Why** — so it stays scannable and easy to read. See [message-format.md](./message-format.md).
- Keep the **Why** to a sentence or two — why the pick beats the alternatives, including what's at stake when that isn't obvious.
- After each area, reflect back what you captured (a short prose summary) so I see the thinking take shape, then move on.
- Allow "park it / move on / not sure yet" — say so once up front, then resurface only on a hard question. Note the gap … rather than force a weak answer.
```

### Siblings + version

- `rai-grill-requirement/message-format.md` and `rai-grill-architect/message-format.md` hold the full layout + examples (the canonical source — the spec does not duplicate them).
- `plugin.json` version bumped (`0.9.0 → 0.10.0`) so installs pick up the change.

---

## 6. Verification

By inspection plus a render check:

- [ ] Both `<how-it-feels>` point to `message-format.md` and describe the quote-bar / starred-options / Why shape.
- [ ] Both `message-format.md` use proportional markdown — no code block, no box — with `⭐` on exactly the recommended option and a one-or-two-sentence `Why`.
- [ ] Render check: the question reads as a quote bar, options as a scannable list, the ⭐ is obvious, and the `Why` is short.
- [ ] Each `SKILL.md` stays within the ≤~100-line budget.
- [ ] `plugin.json` version bumped.

---

## 7. Framework grounding (the "good message" basis)

- **F-pattern / scanning** (Nielsen Norman Group) — front-load the line; the question and the bold option labels lead. <https://www.nngroup.com/articles/f-shaped-pattern-reading-web-content/>
- **Hick's Law** + **defaults** (Thaler & Sunstein) — a marked recommendation (the ⭐) collapses an N-way choice to accept/override. <https://lawsofux.com/hicks-law/>, <https://www.behavioraleconomics.com/resources/mini-encyclopedia-of-be/nudge/>
- **Miller / Cowan (~4 chunks)** — cap options at ~4–5. <https://lawsofux.com/millers-law/>
- **Grice's maxim of Quantity** + **Krug, "Don't Make Me Think"** — say no more than needed; the lean rules and the short `Why`. <https://en.wikipedia.org/wiki/Cooperative_principle>
- **Plain Language** — active voice, second person, real bullets, normal prose; the reason this format reads easily where the monospace box did not. <https://www.plainlanguage.gov/>
- **Information Mapping** (Robert Horn) — consistent labels (`QUESTION` / `Options` / `Why`) as learned landmarks; the principle survived the box's removal. <https://www.tcbok.org/development/information-mapping/>

---

## Appendix A — Intent-elicitation findings (captured for a later pass; NOT adopted here)

Changes that improve *how well the interview surfaces true intent*. They alter questioning **behavior** (not just format), so they are recorded for a future stage.

**The anchoring tension + fix.** Recommending an answer to *every* question *before* the user has spoken is textbook **anchoring** / **leading bias** (The Mom Test; JTBD warn against it). Proven resolution, which keeps the recommendation:
1. **Funnel ordering** (NN/g) — ask the open question first, capture the unprompted answer, *then* recommend. <https://www.nngroup.com/articles/the-funnel-technique-in-qualitative-user-research/>
2. **Recommendation as a correctable apprentice's guess** — "my best guess is X — where's that wrong?" (Contextual Inquiry + an MI complex reflection), sometimes with an alternative.
3. **Withhold the recommendation when ungrounded** — instead ask for a concrete past instance.

**Elicitation techniques (a future "technique menu").**
- **Laddering / means-end** (Reynolds & Gutman) — when the user names a feature, ask "what does that get you?" 1–3 times to reach the value beneath it. <https://www.uxmatters.com/mt/archives/2009/07/laddering-a-research-interview-technique-for-uncovering-core-values.php>
- **The Mom Test** (Fitzpatrick) + **Critical Incident Technique** (Flanagan, via NN/g) — ground in real past incidents; treat generic/future/hypothetical answers as "fluff flags." <https://www.momtestbook.com/>, <https://www.nngroup.com/articles/critical-incident-technique/>
- **Motivational Interviewing — OARS** (Miller & Rollnick) — model the reflect-back as a **Summary** + a **complex reflection** (a guess at underlying intent). <https://motivationalinterviewing.org/understanding-motivational-interviewing>
- **JTBD four forces** (Moesta); **Socratic** + **5 Whys** for assumption/root-cause probes in the completeness sweep — sparingly.
- **Cross-cutting caveat:** laddering, 5 Whys, and Socratic probing read as interrogation if overused — guard with expectation-setting, varied phrasing, concrete examples, and affirmations.

---

## 8. References

- `rai-grill-requirement/SKILL.md`, `rai-grill-architect/SKILL.md` — the `<how-it-feels>` sections this edits; house style.
- `rai-build-story/SKILL.md` + `build-checklist.md` — the thin-SKILL + disclosed-sibling pattern this mirrors.
- `grilling` (community) — the engine both skills run; "one question at a time" is the only formatting rule it carries. Not edited.
- `CLAUDE.md` — authoring rules (tool-neutral, ≤100-line SKILL.md, detail one level deep, bump `plugin.json` on release).
