# Seams & contracts

How the architect grill turns build decisions into stories a builder can implement without guessing — by **shaping the seams** each story builds against, and, when a seam crosses two separately-merged repos, choosing how heavily to formalize it as a **contract**.

## Shape the seam, don't just name it

A **seam** is any boundary a story builds against — an internal module interface (an `ISourceGateway` between a Core and its adapters), or the data shape two sides exchange. Naming the seam ("it talks to an `ISourceGateway`") is a decision *deferred to the builder*; **shaping** it — the method signatures, the data shape, the error/edge cases — is the decision *made*. Two builders handed a named-but-unshaped seam invent two incompatible versions, and the mismatch only surfaces later.

So for every story, write its seams down concretely. This is what the ReadyToCode gate's "seams shaped, not just named" line checks. Most seams are internal and live in the story (or in an ADR when there's a real choice to justify). Two kinds need more than a one-liner: the **internal** seam — the common case, and the one most often left named-but-unshaped — and the **cross-repo** one.

## Shaping an internal seam (the common case)

For an internal interface, *shaped* means a builder could implement it with nothing left to decide:

- **the operations** — each one's name, inputs, and outputs (not "it talks to a board gateway" but the exact methods and their types);
- **the data shape** each returns, and the **error/edge cases** — empty, absent, malformed, timed-out;
- **which side is pure vs. which does the I/O** — the decision stories drop most often. If one side is declared **pure / I/O-free** (typically the Core), no operation on it may read or write a file, hit the network, spawn a process, or mutate external state — that work lives in an **adapter** on the other side, injected so the pure side is tested through a fake. An acceptance criterion that asks the pure side to *do* I/O (a `reads` / `removes` / `fetches` verb on a pure method) is the seam shaped wrong — catch it here, not at the gate.

**Worked example — a launcher's board read.** *"Show each repo's branch rows on the status board."*

- **Named (not enough):** "the Core reads branch rows via a source gateway."
- **Shaped:** `GetBranchRows(repo)` returns `BranchRow { name, ahead, behind, lastCommit }`; a missing repo yields an empty list, an unreachable one surfaces a `ReadFailed` the caller renders — not an exception that hangs the board. The Core composes the rows **purely**; the git/process reads live in an adapter behind the gateway, injected as a fake in tests, with the never-hang watchdog a parameter of the seam rather than buried in the I/O.

## The cross-repo contract — a tiered decision

A seam that crosses **two repos that merge separately** is special: the two sides go live at different times, so if each guesses the shape, the guesses drift (`{ value, source }` on one side, `{ val, sourceSystem }` on the other) and collide at integration. That drift is the *only* problem a "contract" solves — so reach for one **only when the sides merge independently**, and formalize it only as heavily as that independence demands.

Ask first: **do these two sides have to merge independently?** Then pick the lightest tier that holds:

| The two sides… | Do this | Artifact |
| --- | --- | --- |
| can merge **together** (one story/PR owns both, or a monorepo) | a **vertical slice** through both sides; test the real integration | none — drift is impossible |
| merge separately, **same team, fast cadence** | state the shape **once** in one side's story; the other references it | a snippet / interface, not a spec |
| merge separately, **different teams or cadences** (one side ships before the other exists) | **contract-as-authority**: a versioned spec both sides build against | a versioned OpenAPI/schema + a provider test + a generated mock |

Don't default to the bottom row. Versioning, generated mocks, and dedicated contract tests are weight you add when the independence is real — this is **NexaCore's** case (backend `develop` and frontend `dev` merge separately), so its cross-repo seam is a versioned contract, the authority `rai-implement-story` builds against. For everything lighter, a stated shape both sides reference is enough.

When you do write a cross-repo contract, land it in the repo that **owns** the shape (typically the producing/backend side's `docs/adr/`) and have the consuming side's story reference it across the boundary. State it once; both sides point to it; never re-describe it per side.

**Relation to `domain-modeling`'s multi-context model.** `domain-modeling` handles multiple bounded contexts *inside one repo* via a root `CONTEXT-MAP.md`. That map can't span repos — two separately-merged repos each keep their own root `CONTEXT.md` + `docs/adr/`. So treat a cross-repo contract as the inter-repo analog: the shared shape is the cross-boundary reference standing in for a shared map, owned by the repo that owns the shape.

## Repo-pure slices (cross-repo features only)

When a feature spans two repos, decompose it so each slice:

- touches a **single side / repo**;
- is written **against the shaped seam**, so it's buildable and verifiable without the other side present (build the frontend against the contract while the backend is still in flight); and
- **links back to the PRD.**

A slice so tightly coupled that splitting it is artificial can stay whole — note *why*, so the exception is deliberate rather than a default.

### Reconciling with `to-issues`' "vertical slice" rule

`to-issues` normally insists each slice cut through **all** layers (schema → API → UI) and be demoable on its own, treating a single-layer slice as the horizontal slice to avoid. A repo-pure slice looks horizontal by that definition, so be explicit: **for a cross-repo feature, redefine the slice axis** — each slice runs vertical through *its own* repo's layers (schema → endpoint → tests on the backend; component → state → tests on the frontend), single-side across repos. The shaped seam is what keeps a single-side slice independently verifiable, honouring `to-issues`' "verifiable on its own" intent without the other repo present. Single-repo work keeps `to-issues`' normal all-layers slices untouched.

## Routing to `to-issues`

`to-issues` owns turning slices into issues, ordering them, and filing them — let it. But it files into **one tracker: the one configured for the repo it's run in** (it reads that repo's `docs/agents/issue-tracker.md`). It has no multi-repo mode, so carry cross-repo routing yourself:

- **Separate trackers (the usual case):** run `to-issues` **once from inside each repo**, handing it that repo's slices.
- **One shared tracker:** encode the target repo in each slice's title/body (e.g. a `[backend]` / `[frontend]` prefix).
- Capture the **cross-repo merge order** (which side must be live first) as a decision — `to-issues` only orders issues *within* a tracker.

## Worked example — NEXA-412

*"Show which source system each field on a customer's golden record came from."*

- **Shaped seam (a versioned contract, in the backend repo):** each field on the golden record is returned as `{ value, source }` — the sides merge separately, so it earns the heavy tier.
- **Backend slice:** the golden-record response returns `source` for each field, matching the contract. Verified by exercising the API.
- **Frontend slice:** the customer screen renders an origin badge from `source`, built against the contract's mock while the backend slice is still in flight.
- Both slices link the PRD and reference the one contract. Run `to-issues` from each repo; recorded merge order is backend first, then frontend.
