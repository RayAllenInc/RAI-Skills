# Question format

How to lay out each grilling turn so the question can't get buried, the options are pickable, and the text stays lean. Read the `<how-it-feels>` rules in `SKILL.md`; come here for the shape.

## The box

Frame **only the question** — it's the one thing to answer, so it gets the strongest frame. `RECAP` (above) and `MY TAKE` / `OPTIONS` (below) sit unboxed around it. Emit the whole turn in a code block so the monospace alignment and the box characters survive; that trades markdown bold for the box and label caps as the emphasis.

```
  RECAP     reads go through the 10-min cache; writes are sync
╭─ QUESTION ───────────────────────────────────╮
│  Sync or async for the recompute — and where │
│  does it queue?                              │
╰──────────────────────────────────────────────╯
  MY TAKE   async via the existing job runner
  OPTIONS   (a) sync inline
            (b) async job
            (c) hybrid: sync write, async fan-out
```

## Building the box

A fixed box width keeps the right border aligned:

1. Pick a fixed box width and keep every line that wide.
2. Wrap the question to fit inside the borders.
3. Content lines: `│  ` + text + spaces, so the closing `│` lands at the fixed width.
4. Top is `╭─ QUESTION ` + `─`-fill + `╮`; bottom is `╰` + `─`-fill + `╯`.
5. `RECAP` / `MY TAKE` / `OPTIONS` are unboxed and indented 2 spaces (the box sits flush left so it stands out); label in an 8-column field, content hanging-indented.
6. If clean padding isn't achievable, drop the right border — an aligned open box beats a broken closed one.

## Keeping it lean

The box is the *most* verbose format if every label fires every turn. Two rules stop that:

1. **Only the labels that carry content this turn.** The floor is the boxed `QUESTION` + `MY TAKE`. Drop `RECAP` on the first question of an area. Fold "why it matters" into `MY TAKE`; give it its own line only when the stakes aren't obvious. Drop `OPTIONS` when there are no discrete choices.
2. **One line per label.** Each label's value and each option is one line. `OPTIONS` is the only list — the menu for one question, capped at ~4–5, never a dump of sub-points.

Empty labels just disappear:

```
╭─ QUESTION ───────────────────────────────────╮
│  Does the recompute need to be idempotent?   │
╰──────────────────────────────────────────────╯
  MY TAKE   yes — key it on the renewal id so retries are safe
```

## The other two messages

- **The escape hatch** ("park it / move on / not sure yet") is stated once at the session open, then resurfaced only on a genuinely hard question — not every turn.
- **The end-of-area reflect-back** is prose, not a box: a short summary of what the area settled, under a light marker.
