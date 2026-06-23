# Question format

How to lay out each grilling turn so the question can't get buried, the options are pickable, and the reasoning stays easy to read. Read the `<how-it-feels>` rules in `SKILL.md`; come here for the shape.

## The turn

Three parts, in normal proportional markdown — no code block, no monospace. The only frame is a quote bar on the question; the reasoning stays as prose so it reads easily.

> **QUESTION**
> Sync or async for the renewal recompute — and where does it queue?

**Options**
- **(a) Sync inline**
- **(b) Async via the existing job runner** ⭐
- **(c) Hybrid — sync write, async fan-out**

**Why (b):** the recompute is too slow to block the write path, and the job runner already gives us retries and backpressure; the hybrid buys a freshness gain nobody asked for at the cost of two code paths to maintain.

## The parts

- **QUESTION** — in a quote bar with a bold label, so it's the anchor and can't get buried. One question per turn.
- **Options** — a bullet list; bold the short label of each. Mark the recommendation with a **⭐ at the end of its line** — the star *is* the recommendation. Cap at ~4–5.
- **Why (x):** — one or two sentences on why the ⭐ option beats the alternatives. An explanation, not an essay.

## Keeping it lean

- **Recap** only when something was just captured: a short italic lead-in above the question — *Recap: reads already go through the 10-min cache.* Skip it on the first question of an area.
- **No discrete choices?** Drop the Options list and give the pick in one line — **My take:** <answer> — then the **Why**.
- The question and options carry the turn; keep the **Why** short so it supports rather than buries them.

## Collapsed example (open question, no options)

> **QUESTION**
> Does the recompute need to be idempotent?

**My take:** yes — key it on the renewal id so retries are safe.

**Why:** the async runner retries on failure, and without an idempotency key a double-run double-counts; keying on the renewal id makes a retry a harmless no-op.

## The other two messages

- **The escape hatch** ("park it / move on / not sure yet") is stated once at the session open, then resurfaced only on a genuinely hard question — not every turn.
- **The end-of-area reflect-back** is a short prose summary of what the area settled, under a light marker — distinct from a question turn.
