# rai-skills — Project Context

## What this is

Ray Allen Inc's shared **Agent Skills** toolkit (part of the Pillar 3 AI-native dev initiative). Reusable, version-controlled skills authored on the open [Agent Skills standard](https://agentskills.io) (`SKILL.md`), shipped to Claude Code via a plugin marketplace and usable by any skills-compatible agent (Codex, Gemini CLI, Cursor, …). For install and usage, see `README.md`.

## Layout & conventions

- Skills live at `plugins/rai-skills/skills/<name>/SKILL.md` (+ any supporting files beside them).
- **Every skill is prefixed `rai-`** (e.g. `rai-grill-requirement`).
- Claude Code wrapper: `.claude-plugin/marketplace.json` (marketplace `rai`) + `plugins/rai-skills/.claude-plugin/plugin.json` (plugin `rai-skills`). **Bump `version` in `plugin.json` on each release** or installs won't pick up changes.
- Installed commands are namespaced: `/rai-skills:rai-<name>`.

## Authoring rules (non-negotiable)

- **Tool-neutral `SKILL.md`.** Describe *what* to do ("explore the codebase", "capture the decision"), never a Claude-only tool or command. This is what keeps skills portable — Codex reads `.agents/skills/`, Claude Code reads `.claude/skills/`, same files.
- Keep `SKILL.md` tight (≈ ≤100 lines); push detail into sibling files referenced one level deep.
- Use the **`write-a-skill`** skill when adding or editing a skill.

## Skill suite & roadmap

A two-stage, persona-split grilling pipeline that feeds the existing PRD workflow (`to-prd` → `to-issues` → triage):

`PM idea → rai-grill-requirement → CONTEXT.md glossary + ADRs (+ parked architect questions) → to-prd → PRD → rai-grill-architect → architectural review`

- **`rai-grill-requirement`** (built) — grills a *product manager* in plain, outcome-only language. Two hard rules baked into its SKILL.md: **(1) the codebase is backstage** — read it to know what to ask, never cite it in a question; **(2) ask the outcome shadow** — translate every consideration, even a technical one, into the plain user-facing question its answer hinges on, and park the rest. Coverage = 10 product dimensions as a *floor, not a ceiling* + a completeness sweep. Captures `CONTEXT.md` + ADRs, parks deferred build questions at `.scratch/<feature>/architect-questions.md`, hands to `to-prd`.
- **`rai-setup-skills`** (built) — onboarding command (`disable-model-invocation: true`; sequential steps; ✅/❌ summary). Today it verifies `gh` is installed + authenticated. Built to grow (self-install the marketplace, project config).
- **`rai-grill-architect`** (built) — stage 2, the inverse of the requirement grill on the same `grill-with-docs` engine. Two rules, **inverted** from stage 1: **(1) the codebase is on the table** — cite current behavior, name the seams a change touches, surface contradictions aloud (restores what stage 1 suppressed); **(2) ask the mechanism, not the shadow** — pick up the parked questions from `.scratch/<feature>/architect-questions.md` (the PM's recorded outcomes are now constraints) and ask them directly in build terms. Coverage = 10 build / non-functional dimensions as a *floor* + a completeness sweep. ADRs are the primary capture; `CONTEXT.md` stays a pure glossary. For cross-repo features it captures **the shared shape** — the data two sides exchange, stated once with both sides' stories referencing it, deliberately *not* a formal "contract" (no versioning/packaging/contract-test ceremony) — and slices the work **repo-pure** (one side per story), then hands the slices to `to-issues` with per-repo routing. Detail in the skill's `repo-pure-slices.md`.

**Next (roadmap, not yet built):** `rai-build-story` (Phase 4 AFK build — a *thin* skill built on `tdd` + a build-checklist sibling: read context in place, build against the shared shape/mock, branch-write/prod-blind, PR from template) and `rai-release-notes` (Phase 7 — net-new; aggregate both repos' merged PRs, anchor to the PRD, emit a Support handoff). Demoted to docs/infra, *not* skills: ReadyToCode gate (community `triage` + a checklist doc), cross-repo PR review (community `review` + a rubric doc; a thin override only if cross-repo review is needed), the contract *format* (a one-time ADR), regression/CI, and Jira intake. This is the consolidated ~4-skill cut (down from an initial 12).

When building any new skill, read `plugins/rai-skills/skills/rai-grill-requirement/SKILL.md` and `plugins/rai-skills/skills/rai-grill-architect/SKILL.md` first — they are the reference for house style and the rule-discipline to copy (a tight tool-neutral `SKILL.md` that builds on an engine skill, with detail pushed one level deep into sibling files).

## Distribution

- **Claude Code (primary, org-wide):** the marketplace + a committed `.claude/settings.json` (`extraKnownMarketplaces` + `enabledPlugins`) auto-provisions the team — see `README.md`.
- **Other agents:** point their skills directory at `plugins/rai-skills/skills/` (Codex: `~/.agents/skills/` or `<repo>/.agents/skills/`). No edits needed — same `SKILL.md`.
- **Team-plan caveat:** the Claude Team/Enterprise "Org settings → Skills" console deploys to Claude.ai apps (web/desktop/Cowork), **not** Claude Code. Central deployment for Claude Code = this marketplace, not that console.
