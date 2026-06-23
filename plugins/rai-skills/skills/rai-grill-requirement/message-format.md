# Question format

How to lay out each grilling turn so the question can't get buried, the options are pickable, the reasoning reads easily, and the scrollback becomes a clean record of decisions. Read the `<how-it-feels>` rules in `SKILL.md`; come here for the shape.

A live turn leads with the **previous answer as a Q/A card**, a divider, then the **new open question** — proportional markdown, no code block, no box. Offer the question's options as **selectable choices the user can click** where your platform supports it (see the Claude Code note below); the written layout here is the fallback.

## A turn

> **Q:** When should the reminder fire?
> **A:** 30 days before renewal ✅

---

> **QUESTION**
> Who should get the renewal reminder?

**Options**
- **(a) The owning account manager** — the one who can act on it ⭐
- **(b) The whole account team**
- **(c) The customer directly**

**Why (a):** the owner is the only one positioned to act before the renewal lapses; cc'ing the whole team turns a clear responsibility into a diffuse one.

## The parts

- **Q/A card** — the question just answered, paired with what was chosen, in words (drop the option letter — it's meaningless once the list is gone). Record what was *actually* chosen, even when it overrides your ⭐, so the trail is honest. The first question of an area has no card above it.
- **`---` divider** — the settled → active boundary: above is done, below is now. The only hard line in a turn.
- **QUESTION** — the anchor; one question per turn.
- **Options** — offer them as **selectable choices the user can pick directly**: recommended choice first, each choice's gloss attached, and an always-available "something else" for a free-text or "park it" answer. As written markdown (the fallback) it's a bullet list — bold each short label, **⭐** at the end of the recommended one. Cap at **4**.
- **Why (x):** — one or two sentences on why the ⭐ option beats the alternatives. An explanation, not an essay. (As a selectable choice, it rides in the recommended option's description.)
- **Open questions stay text.** Offer choices only when the question has genuine discrete options. A genuinely open question ("walk me through the last time…") asked as buttons just leads the user — keep it free text.

## Selectable choices — Claude Code

On Claude Code, "selectable choices" is the **ask-question tool** (`AskUserQuestion`). Present the question and its options *through* the tool — don't also duplicate them as a markdown list:

- **One question per call** — keep one-at-a-time (the tool allows four; use one).
- **Recommended choice first**, with "(Recommended)" in its label — that's the ⭐; put the **Why** in its description.
- **Each option's description** carries its gloss, so a click is informed.
- **"Other"** is the built-in escape — park it / not sure / a custom answer.
- **multiSelect** for "select all that apply"; a short **header** (≤12 chars) names the area.
- After the answer, emit the **resolved Q/A card** in markdown as usual.

Other agents ignore this section and render the written list above.

## Keeping it lean

- **No discrete choices?** Drop the options; give the pick in one line — **My take:** <answer> — then the **Why**. The card still pairs Q and A.
- Keep the **Why** short; the question and options carry the turn.

## End of an area — the ledger

Stack the area's cards under a heading — `## ✅ Settled — <area>` — with even spacing and no dividers (a uniform set). This *is* the reflect-back: a Q→A record, not a prose paragraph.

> **Q:** When should the reminder fire?
> **A:** 30 days before renewal ✅

> **Q:** Who should get the renewal reminder?
> **A:** The owning account manager ✅

## The escape hatch

"park it / move on / not sure yet" is stated once at the session open, then resurfaced only on a genuinely hard question — not every turn. With selectable choices it's always available via "Other".
