# rai-skills

Ray Allen Inc's shared **Agent Skills** for AI-native development — one neutral source of truth that works in Claude Code today and any skills-compatible agent (Codex, Gemini CLI, Cursor, …) tomorrow.

Skills follow the open [Agent Skills standard](https://agentskills.io) (`SKILL.md` folders), so they're portable by construction. This repo adds a thin Claude Code **plugin marketplace** wrapper for one-command install — other tools read the same `SKILL.md` folders directly.

## What's inside

| Skill | What it does |
| --- | --- |
| `rai-grill-requirement` | Grills a product manager on a feature idea in plain, outcome-focused language, turning it into sharp requirements plus a shared glossary and decisions, then hands off to `to-prd`. Builds on the community `grilling` + `domain-modeling` skills (the pair the `grill-with-docs` wrapper composes; installed by `rai-setup-skills`). |
| `rai-grill-architect` | The stage-2 inverse: grills an architect on *how* to build the feature with the codebase fully on the table, turning a PRD and the parked build questions into ADRs, **shaped seams** (internal interfaces, plus a tiered cross-repo contract when two sides merge separately), and repo-pure story slices, then hands off to `to-issues`. Also builds on `grilling` + `domain-modeling`. |
| `rai-ready-to-code` | The **ReadyToCode gate** (Phase 3): a mandatory, independent, adversarial "no loose ends" review that gates a designed story to `ready-for-agent` before any build — review lenses + a red-team refute against a checklist (incl. "seams shaped, not just named"), recording the review and the architect's approval. |
| `rai-implement-story` | Implements one `ready-for-agent` story (single-repo tool *or* cross-repo product) into a reviewed, test-backed pull request — test-first (`red → green`), built against the shaped seam, branch-write/prod-blind, stopping at PR. Phase 4; restates its loop inline and leans on community `tdd`/`review` when present. |
| `rai-setup-skills` | Onboarding check for a new teammate (today: GitHub CLI installed + authenticated). |

The canonical skill folders live under **`plugins/rai-skills/skills/`**.

## Install for Claude Code (primary)

```
/plugin marketplace add RayAllenInc/RAI-Skills
/plugin install rai-skills@rai
```

Skills are then available in every project, namespaced — e.g. `/rai-skills:rai-grill-requirement`.

### One step for the whole team

Commit this to a repo your team already opens, in `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "rai": { "source": { "source": "github", "repo": "RayAllenInc/RAI-Skills" } }
  },
  "enabledPlugins": { "rai-skills@rai": true }
}
```

Teammates get prompted to install automatically when they trust the project folder.

## Use with Codex or other agents (optional)

The folders under `plugins/rai-skills/skills/` are standard Agent Skills — no edits required. To use them in OpenAI Codex, point its skills directory at them by cloning this repo and symlinking (or copying) the skill folders into:

- `~/.agents/skills/` (personal), or
- `<your-repo>/.agents/skills/` (per project)

Codex reads the same `SKILL.md`. The same idea works for any other tool on the [Agent Skills](https://agentskills.io) list — drop the folders into that tool's skills directory.

## Authoring rule (keep skills portable)

Write every `SKILL.md` **tool-neutrally**: describe *what* to do ("explore the codebase", "capture the decision"), never a Claude-only tool or command. That single habit is what keeps these skills agnostic across agents.

## Conventions

- Every skill is prefixed `rai-`.
- One skill per folder: `plugins/rai-skills/skills/<rai-skill-name>/SKILL.md`, plus any supporting files alongside it.
- Bump `version` in `plugins/rai-skills/.claude-plugin/plugin.json` on each release so installs pick up changes.
