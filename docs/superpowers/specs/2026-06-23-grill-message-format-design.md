# Grill message format — design spec

**Date:** 2026-06-23 · **Owner:** Avinash Gadiraju · **Status:** Draft for review

A formatting upgrade for the **questions the grill skills ask during a querying session** — the one-question-at-a-time interview run by `rai-grill-requirement` and `rai-grill-architect`. Goal: **decrease the reader's cognitive load** so each turn is easy to read and easy to act on. This is a presentation change to existing skills, not a new skill and not a change to *what* gets asked.

The chosen format is an **ASCII boxed-question layout** rendered in a code block: the question framed in a Unicode box (the anchor), with the recommendation and options as labelled lines around it.

Written to the `writing-great-skills` principles: the format lives as a thin `<how-it-feels>` pointer plus **one disclosed sibling** (`message-format.md`) per skill, the layout is a **checkable shape** the skill produces every turn, and it stays **tool-neutral** (plain monospace any agent emits — no Claude-only tool named).

---

## 1. The problem

A grilling turn is really several parts — an optional reflect-back, an optional "why this matters", **the question**, the recommended answer, sometimes an option set, and the escape hatch. Today the skills give *no* guidance on laying those parts out (`grilling` only says "one question at a time"), so the agent free-styles each turn into a prose blob.

The user named three sources of load (a fourth, "no sense of progress," was explicitly **not** a problem — so progress breadcrumbs are out of scope):

| Pain | What it means |
|---|---|
| **Question gets buried** | reflect-back, "why", and the ask blur into one paragraph — hard to spot what's being decided |
| **Hard to pick an answer** | the recommendation and options aren't scannable — you re-read to find them |
| **Too much text per turn** | each message is simply longer than it needs to be |

The tension: "buried" pulls toward *more* structure; "too much text" pulls toward *less*. The resolution is **structure via a fixed visual frame, governed for brevity** — box the one thing that must not get buried (the question), and keep everything else lean.

---

## 2. Scope & decisions

**Scope: presentation only ("V1").** This pass changes *how the message looks*, not *what is asked or in what order*. The framework research surfaced higher-leverage **intent-elicitation** changes (ask-open-before-recommending, laddering, incident-grounding, complex reflections) — these are **captured in Appendix A for a later pass and deliberately not adopted here**, to keep this change tight and low-risk.

| # | Decision | Rationale |
|---|---|---|
| **D1 — Boxed question** | Frame **only the question** in a Unicode box (`╭─ QUESTION ─╮ … ╰─╯`); render `RECAP` above it and `MY TAKE` / `OPTIONS` below it as labelled lines. | The user's chosen look. The box is the unmissable anchor, which directly kills pain 1 ("question gets buried"). Boxing *only* the question (not the whole message) limits the alignment-sensitive region to one short span. |
| **D2 — Code-block emission** | The whole turn is emitted inside a code block so monospace alignment and the box characters survive the markdown renderer. This trades markdown **bold**/*italic* for the box + label caps as the emphasis. | Box-drawing and column alignment only render reliably in monospace; outside a code block the renderer mangles `│`/`─`. Accepted cost: no inline bold/links inside the turn. |
| **D3 — Fixed width + graceful fallback** | Use a fixed inner width so the right border always lines up. If clean right-edge padding isn't achievable, **drop the right border** (open box) rather than ship a jagged one. | A closed Unicode box requires right-edge padding on every line — the one place generated text misaligns. A fixed width makes padding mechanical; the open-box fallback guarantees it never *looks* broken. |
| **D4 — Conditional labels** | Render only the labels that carry real content this turn. Floor = the boxed `QUESTION` + `MY TAKE`. | Stops the layout from being verbose on simple turns (pain 3). Most turns show the box + a take, not every label. |
| **D5 — One line per label** | `RECAP`, `MY TAKE` and each option are one line (wrapped with a hanging indent if long). The only list is `OPTIONS`. | The brevity governor. Labels organize; they don't license rambling. Preserves the spirit of the old "never a wall of sub-bullets" rule while allowing the options menu. |
| **D6 — Escape hatch once; reflect-back stays prose** | "park it / move on / not sure yet" is stated **once** at session open, resurfaced only on a hard question. The **end-of-area** reflect-back is a short **prose** summary under a light marker — *not* the boxed layout (which is for asking, not summarizing). | Per-turn escape lines are furniture that fight brevity. The boxed layout is the *question* unit; an area summary is a different message type. |

**Out of scope (deliberately):** progress breadcrumbs (user did not want them); the intent-elicitation behavior in Appendix A; `rai-setup-skills` ✅/❌ summaries and `rai-build-story` output (not querying-session messages); the community `grilling` / `domain-modeling` engines (RAI vendors its own skills and orchestrates third-party ones — the format guidance lives in our wrappers, not in skills we don't own).

---

## 3. The format

Only the **question** is boxed; `RECAP` (above) and `MY TAKE` / `OPTIONS` (below) are unboxed labelled lines. Labels sit left-justified in an 8-column field with content in a hanging indent. The whole turn is one code block.

**Full turn (weighty, multi-option):**

```
  RECAP     account manager owns the renewal, not the customer
╭─ QUESTION ───────────────────────────────────╮
│  Who should get the renewal reminder?         │
╰───────────────────────────────────────────────╯
  MY TAKE   owning account manager — the one who
            can act on it
  OPTIONS   (a) owning AM
            (b) whole account team
            (c) the customer
```

**Collapsed (simple question — `RECAP`/`OPTIONS` just disappear, D4):**

```
╭─ QUESTION ───────────────────────────────────╮
│  Should the reminder repeat if it's ignored?  │
╰───────────────────────────────────────────────╯
  MY TAKE   yes — once after 3 days, then stop
```

### The box recipe (D3)

A fixed inner width keeps the right border aligned:

1. Pick a fixed inner text width **W** (default **46**).
2. Wrap the question to ≤ W characters per line.
3. Each content line is `│ ` + text + right-pad-with-spaces-to-W + ` │`.
4. Top border: `╭─ QUESTION ` + `─`-fill + `╮`; bottom: `╰` + `─`-fill + `╯`; both total **W + 4** wide.
5. `RECAP` / `MY TAKE` / `OPTIONS` are unboxed and indented 2 spaces (the box sits flush at the left margin so it stands out); label left-justified in an 8-column field, content in a hanging indent; wrap long values to the indent.
6. **Fallback:** if clean padding isn't achievable, drop the trailing ` │` (open box) — an aligned open box beats a broken closed one.

### Architect register

Identical layout, build-language content:

```
  RECAP     reads go through the 10-min cache; writes are sync
╭─ QUESTION ───────────────────────────────────╮
│  Sync or async for the recompute — and where  │
│  does it queue?                               │
╰───────────────────────────────────────────────╯
  MY TAKE   async via the existing job runner
  OPTIONS   (a) sync inline
            (b) async job
            (c) hybrid: sync write, async fan-out
```

---

## 4. Where it lives (information hierarchy)

Mirrors the `rai-build-story` + `build-checklist.md` shape: a thin pointer in `SKILL.md`, detail one level deep in a sibling.

- **New sibling `message-format.md` in each skill dir** — `rai-grill-requirement/` and `rai-grill-architect/`. Each holds the layout + the box recipe + the brevity rules + register-appropriate examples + the escape-hatch / reflect-back notes. Duplicated across the two dirs **on purpose**: each skill stays self-contained and portable (Codex copies skill dirs standalone), and the worked examples differ by register anyway.
- **`<how-it-feels>` in each `SKILL.md`** keeps its principles but (a) points one level deep to `message-format.md`, (b) rewords the "never a wall of sub-bullets" bullet so it no longer reads as forbidding the options menu, and (c) folds in the state-once escape hatch and prose reflect-back (D6).
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
- Render each question in the boxed layout — the question framed, my take and options below — so it can't get buried and the options are pickable. See [message-format.md](./message-format.md). The options list is the menu for that one question, not a "wall of sub-bullets."
- A light "why this matters" when it isn't obvious — fold it into the recommendation in one line, not every time.
- After each area, reflect back what you captured (a short prose summary, not the boxed layout) so I see the thinking take shape, then move on.
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
- Render each question in the boxed layout — the question framed, my take and options below — so it can't get buried and the options are pickable. See [message-format.md](./message-format.md). The options list is the menu for that one question, not a "wall of sub-bullets."
- A light "why this matters" when it isn't obvious — fold it into the recommendation in one line.
- After each area, reflect back the decision you captured (a short prose summary, not the boxed layout) so I see it take shape, then move on.
- Allow "park it / move on / not sure yet" — say so once up front, then resurface only on a hard question. Note the gap rather than force a weak answer.
```

### 5c. New file `rai-grill-requirement/message-format.md`

````
# Question format

How to lay out each grilling turn so the question can't get buried, the options are
pickable, and the text stays lean. Read the `<how-it-feels>` rules in `SKILL.md`; come
here for the shape.

## The layout

Box **only the question** — it's the one thing to answer, so it gets the strongest frame.
`RECAP` (above) and `MY TAKE` / `OPTIONS` (below) are unboxed labelled lines. Emit the
whole turn inside a code block, so the monospace alignment and box characters survive;
that trades markdown **bold** for the box + label caps as the emphasis.

```
  RECAP     account manager owns the renewal, not the customer
╭─ QUESTION ───────────────────────────────────╮
│  Who should get the renewal reminder?         │
╰───────────────────────────────────────────────╯
  MY TAKE   owning account manager — the one who
            can act on it
  OPTIONS   (a) owning AM
            (b) whole account team
            (c) the customer
```

## The box recipe

A fixed inner width keeps the right border aligned:

1. Pick a fixed inner text width W (default 46).
2. Wrap the question to ≤ W characters per line.
3. Each content line is "│ " + text + right-pad-to-W + " │".
4. Top border "╭─ QUESTION " + ─-fill + "╮"; bottom "╰" + ─-fill + "╯"; both W+4 wide.
5. RECAP / MY TAKE / OPTIONS are unboxed and indented 2 spaces (the box sits flush left
   so it stands out); label left-justified in an 8-column field, content hanging-indented.
6. Fallback: if clean padding isn't achievable, drop the trailing " │" (open box). An
   aligned open box beats a broken closed one.

## Two rules that keep it lean

The boxed layout is the *most* verbose if every label fires every turn. Two rules stop that:

1. **Conditional labels — render only the labels that carry content this turn.** The floor
   is the boxed `QUESTION` + `MY TAKE`. Drop `RECAP` on the first question of an area
   (nothing captured yet). Fold "why it matters" into `MY TAKE` in one clause; give it its
   own line only when the stakes aren't obvious. Drop `OPTIONS` when there are no discrete
   choices.
2. **One line per label.** `RECAP`, `MY TAKE`, and each option are one line (wrap with a
   hanging indent if long). The only list is `OPTIONS`, and it's the menu for one question —
   never a dump of sub-points; cap it at ~4–5 choices.

## Collapsed example (simple question)

Empty labels just disappear:

```
╭─ QUESTION ───────────────────────────────────╮
│  Should the reminder repeat if it's ignored?  │
╰───────────────────────────────────────────────╯
  MY TAKE   yes — once after 3 days, then stop
```

## The two other message types

- **The escape hatch** ("park it / move on / not sure yet") is stated **once** at the
  session open, then resurfaced only on a genuinely hard question — not every turn.
- **The end-of-area reflect-back** is a *different* message from a question turn: a short
  prose summary of what the area settled, under a light marker — not the boxed layout.
````

### 5d. New file `rai-grill-architect/message-format.md`

Identical to 5c except the framing voice and the worked examples use the **build register**:

```
  RECAP     reads go through the 10-min cache; writes are sync
╭─ QUESTION ───────────────────────────────────╮
│  Sync or async for the recompute — and where  │
│  does it queue?                               │
╰───────────────────────────────────────────────╯
  MY TAKE   async via the existing job runner
  OPTIONS   (a) sync inline
            (b) async job
            (c) hybrid: sync write, async fan-out
```

Collapsed:

```
╭─ QUESTION ───────────────────────────────────╮
│  Does the recompute need to be idempotent?    │
╰───────────────────────────────────────────────╯
  MY TAKE   yes — key it on the renewal id so retries are safe
```

### 5e. Version bump

Bump `version` in `plugins/rai-skills/.claude-plugin/plugin.json` — per the project rule, installs won't pick up the new siblings + edits otherwise.

---

## 6. Verification

The change is to skill *prose*, so verification is by inspection plus a render check:

- [ ] Both `<how-it-feels>` sections point to `message-format.md` and no longer contain the bare "never a wall of sub-bullets" phrasing that conflicts with the options menu.
- [ ] Both `message-format.md` files exist, carry the layout + box recipe + the two brevity rules + the escape-hatch / reflect-back notes, and differ only in register/examples.
- [ ] Render check: paste the full and collapsed examples into a markdown terminal renderer; the box borders align (or degrade to a clean open box), and the layout reads clearly.
- [ ] Each `SKILL.md` stays within the ≤~100-line budget.
- [ ] `plugin.json` version is bumped.
- [ ] Dry-run read-through: an agent following the new `<how-it-feels>` + sibling produces the boxed layout and collapses empty labels.

---

## 7. Framework grounding (the "good message" basis)

The formatting decisions are grounded in established low-cognitive-load principles, not just taste:

- **Information Mapping** (Robert Horn) — decompose into blocks, label each by type, same labels every time. Backs the fixed `RECAP`/`QUESTION`/`MY TAKE`/`OPTIONS` landmarks. <https://www.tcbok.org/development/information-mapping/>
- **F-pattern / scanning** (Nielsen Norman Group) — left-aligned text is scanned, not read; front-load the line. Backs boxing the question as the anchor and starting each line with its load-bearing word. <https://www.nngroup.com/articles/f-shaped-pattern-reading-web-content/>
- **Hick's Law** + **Miller/Cowan (~4 chunks)** — cap options (~4–5) and keep the whole turn to a few chunks. <https://lawsofux.com/hicks-law/>, <https://lawsofux.com/millers-law/>
- **Choice architecture / defaults** (Thaler & Sunstein) — a recommended answer collapses an N-way choice to accept/override. Backs `MY TAKE` with the reason folded in. <https://www.behavioraleconomics.com/resources/mini-encyclopedia-of-be/nudge/>
- **Grice's maxim of Quantity** + **Krug, "Don't Make Me Think"** — say no more than needed; omit needless words. Backs the conditional-label and one-line-per-label governors. <https://en.wikipedia.org/wiki/Cooperative_principle>
- **Plain Language** — active voice, second person, no jargon, bullets for option sets. <https://www.plainlanguage.gov/>

---

## Appendix A — Intent-elicitation findings (captured for a later pass; NOT adopted here)

The framework research turned up changes that improve *how well the interview surfaces true intent*. They alter questioning **behavior** (not just format), so they are recorded here for a future stage rather than built now.

**The anchoring tension + fix.** Recommending an answer to *every* question *before* the user has spoken is textbook **anchoring** / **leading bias** (The Mom Test; JTBD both warn against it). Proven resolution, which keeps the recommendation:
1. **Funnel ordering** (NN/g) — ask the open question first, capture the user's unprompted answer, *then* recommend. <https://www.nngroup.com/articles/the-funnel-technique-in-qualitative-user-research/>
2. **Recommendation as a correctable apprentice's guess** — "my best guess is X — where's that wrong?" (Contextual Inquiry's master–apprentice stance + an MI complex reflection), sometimes with a plausible alternative so "just agree" isn't the easy path.
3. **Withhold the recommendation when ungrounded** — instead ask for a concrete past instance.

**Elicitation techniques (a future "technique menu").**
- **Laddering / means-end** (Reynolds & Gutman) — when the user names a feature, ask "what does that get you?" 1–3 times to reach the value beneath it (the real intent). <https://www.uxmatters.com/mt/archives/2009/07/laddering-a-research-interview-technique-for-uncovering-core-values.php>
- **The Mom Test** (Rob Fitzpatrick) + **Critical Incident Technique** (Flanagan, via NN/g) — ground in real past incidents; treat generic/future/hypothetical answers as "fluff flags." <https://www.momtestbook.com/>, <https://www.nngroup.com/articles/critical-incident-technique/>
- **Motivational Interviewing — OARS** (Miller & Rollnick) — the per-area reflect-back is best modeled as a **Summary** + a **complex reflection** (a guess at underlying intent, not a paraphrase). <https://motivationalinterviewing.org/understanding-motivational-interviewing>
- **JTBD four forces** (Moesta) — push/pull/anxiety/habit, for change-shaped features. **Socratic questioning** + **5 Whys** — assumption/root-cause probes inside the completeness sweep, used sparingly.
- **Cross-cutting caveat:** laddering, 5 Whys, and Socratic probing all read as interrogation if overused and *raise* load — guard with expectation-setting, varied phrasing, concrete examples, and affirmations.

---

## 8. References

- `rai-grill-requirement/SKILL.md`, `rai-grill-architect/SKILL.md` — the `<how-it-feels>` sections this edits; house style.
- `rai-build-story/SKILL.md` + `build-checklist.md` — the thin-SKILL + disclosed-sibling pattern this mirrors.
- `grilling` (community) — the engine both skills run; "one question at a time" is the only formatting rule it carries, which this fills in. Not edited.
- `CLAUDE.md` — authoring rules (tool-neutral, ≤100-line SKILL.md, detail one level deep, bump `plugin.json` on release).
