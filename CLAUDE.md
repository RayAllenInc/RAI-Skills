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
- **`rai-grill-architect`** (PLANNED) — stage 2. Consumes `architect-questions.md` (PM outcomes become the architect's constraints) and flips the persona to the architect (the "how": build mechanics, non-functionals). Mirror the structure and rule-discipline of `rai-grill-requirement`.

When building `rai-grill-architect` or any new skill, read `plugins/rai-skills/skills/rai-grill-requirement/SKILL.md` first — it's the reference for house style and the rule-discipline to copy.

## Distribution

- **Claude Code (primary, org-wide):** the marketplace + a committed `.claude/settings.json` (`extraKnownMarketplaces` + `enabledPlugins`) auto-provisions the team — see `README.md`.
- **Other agents:** point their skills directory at `plugins/rai-skills/skills/` (Codex: `~/.agents/skills/` or `<repo>/.agents/skills/`). No edits needed — same `SKILL.md`.
- **Team-plan caveat:** the Claude Team/Enterprise "Org settings → Skills" console deploys to Claude.ai apps (web/desktop/Cowork), **not** Claude Code. Central deployment for Claude Code = this marketplace, not that console.
