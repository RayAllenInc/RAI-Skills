# Question format

How to lay out each grilling turn so the question can't get buried, the options are pickable, the reasoning reads easily, and the scrollback becomes a clean record of decisions. Read the `<how-it-feels>` rules in `SKILL.md`; come here for the shape.

A live turn leads with the **previous answer as a Q/A card**, a divider, then the **new open question** — proportional markdown, no code block, no box.

## A turn

> **Q:** Sync or async for the recompute?
> **A:** Async via the existing job runner ✅

---

> **QUESTION**
> Does the recompute need to be idempotent — and how do we key it?

**Options**
- **(a) Idempotent, keyed on the renewal id** — a retry is a harmless no-op ⭐
- **(b) Non-idempotent, rely on at-most-once delivery**

**Why (a):** the async runner retries on failure; without a key a double-run double-counts, and at-most-once is hard to guarantee.

## The parts

- **Q/A card** — the question just answered, paired with what was chosen, in words (drop the option letter — it's meaningless once the list is gone). Record what was *actually* chosen, even when it overrides your ⭐, so the trail is honest. The first question of an area has no card above it.
- **`---` divider** — the settled → active boundary: above is done, below is now. The only hard line in a turn.
- **QUESTION** — in a quote bar with a bold label, the anchor. One question per turn.
- **Options** — a bullet list; bold each short label; a **⭐ at the end** of the recommended line (the star *is* the recommendation). Cap at ~4–5.
- **Why (x):** — one or two sentences on why the ⭐ option beats the alternatives. An explanation, not an essay.

## Keeping it lean

- **No discrete choices?** Drop the Options list; give the pick in one line — **My take:** <answer> — then the **Why**. The card still pairs Q and A.
- Keep the **Why** short; the question and options carry the turn.
- Group tight, separate clean: the card is one unit, the options are one unit, the divider is the only rule.

## End of an area — the ledger

Stack the area's cards under a heading — `## ✅ Settled — <area>` — with even spacing and no dividers (a uniform set). This *is* the reflect-back: a Q→A record, not a prose paragraph.

> **Q:** Sync or async for the recompute?
> **A:** Async via the existing job runner ✅

> **Q:** Does the recompute need to be idempotent?
> **A:** Yes — keyed on the renewal id ✅

## The escape hatch

"park it / move on / not sure yet" is stated once at the session open, then resurfaced only on a genuinely hard question — not every turn.
