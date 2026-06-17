# Repo-pure slices & the shared shape

How the architect grill turns build decisions into work that two separately-merged repos can build independently, then hands them to `to-issues` to file.

**This applies only when a feature spans two repos.** A single-repo feature skips everything here — slice it and hand it to `to-issues` normally; the shared shape and repo-pure rules apply only when the work crosses two sides.

## The problem this solves

Many features need both a **backend** change and a **frontend** change. The two repos merge through separate gates, at different times — so one side is always live before the other, and whoever builds one side often does it before the other side exists. If each side guesses the shape of the data they exchange, the guesses drift (`{ value, source }` on one side, `{ val, sourceSystem }` on the other) and only collide at integration. Writing the shape down once removes the guess.

## The shared shape

When a feature spans two sides, record the shape of the data they exchange **once**:

- as an **ADR** when there's a real choice to justify ("each field carries its origin as `{ value, source }`, not parallel arrays, because…"), or
- stated plainly **in one side's story** when it's obvious.

Then make the *other* side's story **reference that same statement**. Stated once, both sides point to it; never re-described on each side. Describe it in plain terms or a small snippet — no particular format is required, and there is no separate "contract" artifact to stand up and maintain.

This usually **sharpens** what the PRD already sketched (the shared shape maps onto an "API contracts" item under `to-prd`'s Implementation Decisions) rather than inventing it from scratch. Land the ADR in the repo that **owns** the shape — typically the producing (backend) side's `docs/adr/` — and have the consuming side's story reference it across the repo boundary.

That single discipline — one statement, both sides reference it — is the whole point. Versioning, generated mocks, and dedicated contract tests are optional weight you can add later or never; don't bake them in here.

**Relation to `domain-modeling`'s multi-context model.** `domain-modeling` already handles multiple bounded contexts *inside one repo* via a root `CONTEXT-MAP.md` that points to each context's own `CONTEXT.md` + `docs/adr/`. That map can't span repos, though — two separately-merged repos each keep their own root `CONTEXT.md` and `docs/adr/`. So treat a cross-repo feature as the inter-repo analog: the **shared shape is the cross-boundary reference** that stands in for a shared map, and the ADR lives with the repo that owns the shape (above).

## Repo-pure slices

Decompose the work so each slice:

- touches a **single side / repo** (backend-only or frontend-only);
- is written **against the shared shape**, so it's buildable and verifiable without the other side present (the frontend can be built against the shape while the backend is still in flight); and
- **links back to the PRD.**

A slice so tightly coupled that splitting it is artificial can stay whole — note *why* it resists splitting, so the exception is deliberate rather than a default.

### Reconciling with `to-issues`' "vertical slice" rule

`to-issues` normally insists each slice cut through **all** layers end-to-end (schema → API → UI) and be demoable on its own, and treats a single-layer slice as the horizontal slice to avoid. A repo-pure slice looks horizontal by that definition, so be explicit: **for a cross-repo feature, redefine the slice axis.** A slice that spans every layer is impossible inside one repo, so each slice runs vertical through *its own* repo's layers (schema → endpoint → tests on the backend; component → state → tests on the frontend), single-side across repos. The **shared shape is what keeps a single-side slice independently verifiable** — you can prove the backend returns the shape, and build the frontend against the shape — which honours `to-issues`' "verifiable on its own" intent without the other repo present. This override applies to cross-repo features only; single-repo work keeps `to-issues`' normal all-layers slices untouched. (It is the same kind of deliberate override the skill's two rules make over domain-modeling's code-facing step.)

## Routing to `to-issues`

`to-issues` already owns turning slices into issues, ordering them, and filing them — let it; don't re-implement that. But it files into **one tracker: the one configured for the repo it's run in** (it reads that repo's `docs/agents/issue-tracker.md`, written per-repo during setup). It has no multi-repo mode and no arbitrary-tagging step. So carry the cross-repo routing yourself:

- **Separate trackers (the usual case):** run `to-issues` **once from inside each repo**, handing it that repo's slices — each run files into that repo's own tracker.
- **One shared tracker:** encode the target repo in each slice's **title/body** (e.g. a `[backend]` / `[frontend]` prefix); `to-issues` won't invent a repo label on its own.
- Capture the **cross-repo merge order** (which side must be live first) as a decision — `to-issues` only orders issues *within* a tracker. If the routing convention isn't written down yet, record it as an ADR so the next feature inherits it.

## Worked example — NEXA-412

*"Show which source system each field on a customer's golden record came from."*

- **Shared shape (ADR, in the backend repo):** each field on the golden record is returned as `{ value, source }`.
- **Backend slice:** the golden-record response returns `source` for each field, matching the shared shape. Verified by exercising the API.
- **Frontend slice:** the customer screen renders a small origin badge from `source`, built against the shared shape while the backend slice is still in flight.
- Both slices link the PRD and reference the one shape statement. Run `to-issues` from the backend repo to file the backend slice and from the frontend repo to file the frontend slice; the recorded merge order is backend first, then frontend.
