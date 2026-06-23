# Grill message format — design spec

**Date:** 2026-06-23 · **Owner:** Avinash Gadiraju · **Status:** Draft for review

A formatting upgrade for the **questions the grill skills ask during a querying session** — the one-question-at-a-time interview run by `rai-grill-requirement` and `rai-grill-architect`. Goal: **decrease the reader's cognitive load** so each turn is easy to read and easy to act on. This is a presentation change to existing skills, not a new skill and not a change to *what* gets asked.

Written to the `writing-great-skills` principles: the format lives as a thin `<how-it-feels>` pointer plus **one disclosed sibling** (`message-format.md`) per skill, the skeleton is a **checkable shape** the skill produces every turn, and it stays **tool-neutral** (plain markdown any agent emits — no Claude-only tool named).

---

## 1. The problem

A grilling turn is really several parts — an optional reflect-back, an optional "why this matters", **the question**, the recommended answer, sometimes an option set, and the escape hatch. Today the skills give *no* guidance on laying those parts out (`grilling` only says "one question at a time"), so the agent free-styles each turn into a prose blob.

The user named three sources of load (a fourth, "no sense of progress," was explicitly **not** a problem — so progress breadcrumbs are out of scope):

| Pain | What it means |
|---|---|
| **Question gets buried** | reflect-back, "why", and the ask blur into one paragraph — hard to spot what's being decided |
| **Hard to pick an answer** | the recommendation and options aren't scannable — you re-read to find them |
| **Too much text per turn** | each message is simply longer than it needs to be |

The tension: "buried" pulls toward *more* structure; "too much text" pulls toward *less*. The resolution is **structure via fixed labels, governed for brevity** — not structure via more prose.

---

## 2. Decision

| # | Decision | Rationale |
|---|---|---|
| **D1 — Labeled sections** | Lay each question out as labeled sections with **bold labels as fixed landmarks**, in a fixed order: `Recap` · `Question` · `Why it matters` · `My take` · `Options`. | Chosen by the user over a lean-typography and a one-liner option. Fixed positions become muscle memory: the eye jumps to the label it wants. A bold `Question` line **can't get buried** (pain 1); a marked `Options` line is **pickable** (pain 2). |
| **D2 — Conditional labels** | Render only the labels that carry real content this turn. Floor = `Question` + `My take`. | This is what stops labeled sections from being the *most* verbose format (pain 3). Most turns show two or three labels, not five. |
| **D3 — One line per label** | A label's value never spills to a second line or sub-bullets. The only list permitted is `Options`. | The brevity governor. Labels *organize*; they don't license rambling. Preserves the spirit of the old "never a wall of sub-bullets" rule while allowing the options menu. |
| **D4 — Escape hatch stated once** | "park it / move on / not sure yet" is stated **once** at session open, then resurfaced only on a genuinely hard question — not a trailing line every turn. | A per-turn escape line is pure furniture that fights brevity. Confirmed default. |
| **D5 — Reflect-back stays prose** | The end-of-area reflect-back is a *different* message from a question turn: a short prose summary under a light marker, **not** the labeled skeleton. | The skeleton is for asking; the area summary is for showing the thinking settle. Confirmed default. |

**Out of scope (deliberately):** progress breadcrumbs (user did not want them); `rai-setup-skills` ✅/❌ summaries and `rai-build-story` output (not querying-session messages); the community `grilling` / `domain-modeling` engines (RAI vendors its own skills and orchestrates third-party ones — the format guidance lives in our wrappers, not in skills we don't own).

---

## 3. The skeleton

Bold labels as landmarks, fixed order:

> **Recap** — &lt;one line: what just got captured&gt;
> **Question** — &lt;the single question&gt;
> **Why it matters** — &lt;one line&gt;
> **My take** — &lt;the recommended answer, naming the option&gt;
> **Options** — (a) … · (b) … · (c) …

**Conditional rendering (D2).** Floor is `Question` + `My take`. Drop `Recap` on the first question of an area (nothing captured yet). Drop `Why it matters` when the stakes are obvious. Drop `Options` when there are no discrete choices.

**One line each (D3).** No label's value runs to a second line or grows sub-bullets. The single exception is `Options`: when the choices are too long for one `·`-separated line, they become a tight bulleted list — one per line, the recommended one **bold**. That list is the *menu for one question*, not the "wall of sub-bullets" the one-question rule forbids.

### Worked examples — `rai-grill-requirement` (product register)

Weighty, multi-option:

> **Recap** — account manager owns the renewal, not the customer.
> **Question** — who should get the renewal reminder?
> **Why it matters** — decides who can actually act on it.
> **My take** — the owning account manager *(a)*.
> **Options** — (a) owning AM · (b) whole team · (c) customer

Simple — empty labels just disappear:

> **Question** — should the reminder repeat if it's ignored?
> **My take** — yes, once after 3 days, then stop.

### Worked examples — `rai-grill-architect` (build register)

Same skeleton, build language:

> **Recap** — reads go through the 10-min cache; writes are synchronous.
> **Question** — sync or async for the renewal recompute, and where does it queue?
> **Why it matters** — sets the freshness ceiling and the failure surface.
> **My take** — async via the existing job runner *(b)*.
> **Options** — (a) sync inline · (b) async job · (c) hybrid: sync write, async fan-out

> **Question** — does the recompute need to be idempotent?
> **My take** — yes; key it on the renewal id so retries are safe.

---

## 4. Where it lives (information hierarchy)

Mirrors the `rai-build-story` + `build-checklist.md` shape: a thin pointer in `SKILL.md`, detail one level deep in a sibling.

- **New sibling `message-format.md` in each skill dir** — `rai-grill-requirement/` and `rai-grill-architect/`. Each holds the skeleton + the two brevity rules + register-appropriate examples + the escape-hatch / reflect-back notes. Duplicated across the two dirs **on purpose**: each skill stays self-contained and portable (Codex copies skill dirs standalone), and the worked examples differ by register anyway.
- **`<how-it-feels>` in each `SKILL.md`** keeps its principles but (a) points one level deep to `message-format.md`, (b) rewords the "never a wall of sub-bullets" bullet so it no longer reads as forbidding the options menu, and (c) folds in the state-once escape hatch (D4) and prose reflect-back (D5). Net change stays within the ≤~100-line budget.
- **No edits** to community `grilling` / `domain-modeling`.

---

## 5. Exact edits

### 5a. `rai-grill-requirement/SKILL.md` — replace the `<how-it-feels>` body

**Before:**

```
A sharp colleague over coffee, not an intake form.

- One genuine question at a time. Never compound, never a wall of sub-bullets.
- A light "why this matters" when it isn't obvious — one line, not every time.
- After each area, reflect back what you captured so I see the thinking take shape, then move on.
- Always allow "park it / move on / not sure yet." Note the gap (or send it to the architect pile) rather than force a weak answer.
```

**After:**

```
A sharp colleague over coffee, not an intake form.

- One genuine question at a time — never two at once.
- Lay each question out as labeled sections (`Question` · `My take` · …) so it can't get buried and the options are pickable — see [message-format.md](./message-format.md). The options list is the menu for that one question, not a "wall of sub-bullets."
- A light "why this matters" when it isn't obvious — one line, not every time.
- After each area, reflect back what you captured (a short prose summary, not the question skeleton) so I see the thinking take shape, then move on.
- Allow "park it / move on / not sure yet" — say so once up front, then resurface only on a hard question. Note the gap (or send it to the architect pile) rather than force a weak answer.
```

### 5b. `rai-grill-architect/SKILL.md` — replace the `<how-it-feels>` body

**Before:**

```
A sharp engineering colleague at a whiteboard, not a design-review form.

- One genuine question at a time. Never compound, never a wall of sub-bullets.
- A light "why this matters" when it isn't obvious — one line.
- After each area, reflect back the decision you captured so I see it take shape, then move on.
- Always allow "park it / move on / not sure yet." Note the gap rather than force a weak answer.
```

**After:**

```
A sharp engineering colleague at a whiteboard, not a design-review form.

- One genuine question at a time — never two at once.
- Lay each question out as labeled sections (`Question` · `My take` · …) so it can't get buried and the options are pickable — see [message-format.md](./message-format.md). The options list is the menu for that one question, not a "wall of sub-bullets."
- A light "why this matters" when it isn't obvious — one line.
- After each area, reflect back the decision you captured (a short prose summary, not the question skeleton) so I see it take shape, then move on.
- Allow "park it / move on / not sure yet" — say so once up front, then resurface only on a hard question. Note the gap rather than force a weak answer.
```

### 5c. New file `rai-grill-requirement/message-format.md`

```
# Question format

How to lay out each grilling turn so the question can't get buried, the options are
pickable, and the text stays lean. Read the `<how-it-feels>` rules in `SKILL.md`; come
here for the shape.

## The skeleton

Labeled sections, bold labels as fixed landmarks, in this order:

> **Recap** — <one line: what just got captured>
> **Question** — <the single question>
> **Why it matters** — <one line>
> **My take** — <the recommended answer, naming the option>
> **Options** — (a) … · (b) … · (c) …

Same order every turn, so the labels become landmarks the eye jumps to.

## Two rules that keep it lean

Labeled sections are the *most* verbose format if every label fires every turn. Two
rules stop that:

1. **Conditional labels — render only the labels that carry content this turn.** The
   floor is just `Question` + `My take`. Drop `Recap` on the first question of an area
   (nothing captured yet). Drop `Why it matters` when the stakes are obvious. Drop
   `Options` when there are no discrete choices.
2. **One line per label.** A label's value never spills to a second line or sub-bullets.
   The only list allowed is `Options`, and it's the menu for one question — never a dump
   of sub-points. If the options are too long for one `·`-separated line, make them a
   tight bulleted list (one per line, the recommended one **bold**); that is still the
   menu for a single ask, not the "wall" the one-question rule forbids.

## Worked examples

Weighty, multi-option:

> **Recap** — account manager owns the renewal, not the customer.
> **Question** — who should get the renewal reminder?
> **Why it matters** — decides who can actually act on it.
> **My take** — the owning account manager *(a)*.
> **Options** — (a) owning AM · (b) whole team · (c) customer

Simple — empty labels just disappear:

> **Question** — should the reminder repeat if it's ignored?
> **My take** — yes, once after 3 days, then stop.

## The two other message types

- **The escape hatch** ("park it / move on / not sure yet") is stated **once** at the
  session open, then resurfaced only on a genuinely hard question — not a trailing line
  every turn.
- **The end-of-area reflect-back** is a *different* message from a question turn: a short
  prose summary of what the area settled, under a light marker — not the labeled skeleton.
```

### 5d. New file `rai-grill-architect/message-format.md`

Identical to 5c except the framing and the worked examples use the **build register**:

```
## Worked examples

Weighty, multi-option:

> **Recap** — reads go through the 10-min cache; writes are synchronous.
> **Question** — sync or async for the renewal recompute, and where does it queue?
> **Why it matters** — sets the freshness ceiling and the failure surface.
> **My take** — async via the existing job runner *(b)*.
> **Options** — (a) sync inline · (b) async job · (c) hybrid: sync write, async fan-out

Simple — empty labels just disappear:

> **Question** — does the recompute need to be idempotent?
> **My take** — yes; key it on the renewal id so retries are safe.
```

### 5e. Version bump

Bump `version` in `plugins/rai-skills/.claude-plugin/plugin.json` — per the project rule, installs won't pick up the new siblings + edits otherwise.

---

## 6. Verification

The change is to skill *prose*, so verification is by inspection, not a test run:

- [ ] Both `<how-it-feels>` sections point to `message-format.md` and no longer contain the bare "never a wall of sub-bullets" phrasing that conflicts with the options menu.
- [ ] Both `message-format.md` files exist, carry the skeleton + the two brevity rules + the escape-hatch / reflect-back notes, and differ only in register/examples.
- [ ] Each `SKILL.md` stays within the ≤~100-line budget.
- [ ] `plugin.json` version is bumped.
- [ ] Dry-run read-through: an agent following the new `<how-it-feels>` + sibling produces the skeleton in §3 and collapses empty labels.

---

## 7. References

- `rai-grill-requirement/SKILL.md`, `rai-grill-architect/SKILL.md` — the `<how-it-feels>` sections this edits; house style.
- `rai-build-story/SKILL.md` + `build-checklist.md` — the thin-SKILL + disclosed-sibling pattern this mirrors.
- `grilling` (community) — the engine both skills run; "one question at a time" is the only formatting rule it carries, which this fills in. Not edited.
- `CLAUDE.md` — authoring rules (tool-neutral, ≤100-line SKILL.md, detail one level deep, bump `plugin.json` on release).
