# Critical skills roster

The skills every RAI machine should have. They're installed with the cross-agent
[skills.sh](https://skills.sh) installer (`npx skills add`), which places each skill in the
agent's skills directory.

**To change what `rai-setup-skills` provisions, edit this file** — the procedure in `SKILL.md`
doesn't change.

## RAI standard install flags

| Setting | Flag |
| --- | --- |
| Global (user-level) scope | `-g` |
| Target agent — Claude Code (RAI primary) | `-a claude-code` |
| Non-interactive (skip every prompt) | `-y` |

- **Agent target.** RAI standardises on **Claude Code**, so installs are scoped with `-a claude-code`. Do **not** use `-a '*'` — that installs into *all 71* agents skills.sh knows about (most not on the machine) and errors on some. To also cover an agent a teammate actually uses, extend the list space-separated, e.g. `-a claude-code codex cursor`.
- **Method.** Symlink is the default; pass `--copy` only to force copies. A single-agent global install lands as a copy under `~/.claude/skills/` — that's expected and fine.

## The roster

Run each command. If one fails, note it in the summary and keep going.

| Skill(s) | Upstream | Command |
| --- | --- | --- |
| **All Matt Pocock skills** — `grilling`, `domain-modeling`, `codebase-design`, `grill-with-docs`, `grill-me`, `to-prd`, `to-issues`, `triage`, `tdd`, `implement`, `prototype`, `resolving-merge-conflicts`, `writing-great-skills`, `diagnosing-bugs`, `handoff`, `humanizer`, … (closes the grill → PRD → issues → triage pipeline) | `mattpocock/skills` (MIT) | `npx --yes skills@latest add mattpocock/skills -s '*' -g -y -a claude-code` |
| `find-skills` | `vercel-labs/skills` | `npx --yes skills@latest add https://github.com/vercel-labs/skills --skill find-skills -g -y -a claude-code` |
| `frontend-design` | `anthropics/skills` | `npx --yes skills@latest add https://github.com/anthropics/skills --skill frontend-design -g -y -a claude-code` |
| `agent-browser` | `vercel-labs/agent-browser` | `npx --yes skills@latest add https://github.com/vercel-labs/agent-browser --skill agent-browser -g -y -a claude-code` |
| `ui-ux-pro-max` | `nextlevelbuilder/ui-ux-pro-max-skill` | `npx --yes skills@latest add https://github.com/nextlevelbuilder/ui-ux-pro-max-skill --skill ui-ux-pro-max -g -y -a claude-code` |
| `vercel-react-best-practices` | `vercel-labs/agent-skills` | `npx --yes skills@latest add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices -g -y -a claude-code` |

## Notes

- **Quote the asterisk** in `-s '*'` (Matt Pocock's "all skills") so the shell doesn't expand it — works in PowerShell and bash alike.
- **Narrow Matt Pocock's set** if you don't want the whole set: swap `-s '*'` for the names you want, e.g. `-s grilling domain-modeling grill-with-docs to-prd to-issues triage`.
- **New in mattpocock/skills v1.0.0, worth knowing** (all install with `-s '*'`): `grilling` + `domain-modeling` are the engines the `rai-grill-*` skills build on (the `grill-with-docs` wrapper just composes them); `codebase-design` owns the deep-module vocabulary (module · interface · depth · seam · adapter) the architect grill leans on; `implement` (a new build skill) and `resolving-merge-conflicts` (an integration-phase loop) are both candidates for the roadmapped `rai-build-story`; `decision-mapping` (still `in-progress` upstream) drives multi-session planning.
- **`agent-browser` also ships a CLI tool**; its own `SKILL.md` documents any extra setup (e.g. a browser binary) on first use.
- **Skills run with full agent permissions.** `skills.sh` shows a per-skill security rating (Gen / Socket / Snyk) at install time — `agent-browser` and `find-skills`, for example, currently rate *medium risk*. Review before rolling out fleet-wide.
- **Updating:** `npx --yes skills@latest update -g -y` refreshes installed skills to the latest upstream.
- **Pruning stale skills.** `update` *detects* skills deleted upstream but **won't remove them under `-y`** — non-interactive mode only prints a "deleted upstream" warning and skips the deletion. After an upstream release that drops or renames skills, read that warning and remove the named ones explicitly: `npx --yes skills@latest remove <names> -g -y -a claude-code`. (mattpocock/skills v1.0.0 deleted `caveman`, `diagnose`, `write-a-skill`, `zoom-out` — `diagnose` became `diagnosing-bugs` and `write-a-skill` became `writing-great-skills`, so the old copies are stale dead weight once the new ones land.)
