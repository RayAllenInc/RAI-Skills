---
name: rai-setup-skills
description: Onboard a teammate to the rai-skills toolkit — verify prerequisites (git, GitHub CLI, Node) and install the critical RAI and community skills every RAI machine should have. Use when someone is setting up rai-skills for the first time, onboarding a new teammate, or mentions rai setup or onboarding.
disable-model-invocation: true
---

# Onboard a teammate to rai-skills

Get a teammate's machine ready to use the `rai-*` skills and the wider RAI skill set. Walk the steps **in order**, fix each before moving on, and finish with a short ✅ / ❌ summary so it's obvious what (if anything) still needs doing.

Most checks run non-interactively. The two interactive ones (`gh auth login`, the in-app marketplace install) are handed to the teammate — ask them to type the `! <command>` in the prompt and follow it through.

## Step 1 — Prerequisites

Check each; install only what's missing, then re-check before continuing.

- **Git + identity** — `git --version` succeeds, and `git config --global user.name` and `git config --global user.email` both return a value. If an identity is unset, set it. Install Git: Windows `winget install --id Git.Git`; macOS `brew install git`; Linux `sudo apt install git`.
- **GitHub CLI** — `gh --version`. Install: Windows `winget install --id GitHub.cli` (or `scoop install gh` / `choco install gh`); macOS `brew install gh`; Linux see https://github.com/cli/cli#installation.
- **GitHub CLI auth** — `gh auth status` shows an authenticated github.com account. If not, `gh auth login` is interactive: ask the teammate to type **`! gh auth login`** and choose **GitHub.com → HTTPS → Login with a web browser**, then re-run the check.
- **Node.js + npx** — `node --version` and `npx --version` both succeed (the skill installer runs through `npx`). Install: Windows `winget install --id OpenJS.NodeJS.LTS`; macOS `brew install node`; Linux see https://nodejs.org.

## Step 2 — RAI marketplace + plugin

The `rai-*` skills ship through the `rai` Claude Code marketplace. If you're running this skill, it's already installed — confirm and move on. To set up a fresh Claude Code, hand these to the teammate:

```
/plugin marketplace add RayAllenInc/RAI-Skills
/plugin install rai-skills@rai
```

- **Whole team at once:** commit the auto-provision snippet from `README.md` into a shared repo's `.claude/settings.json` — teammates are prompted to install when they trust the folder.
- **Other agents (Codex, etc.):** `npx --yes skills@latest add RayAllenInc/RAI-Skills --full-depth -g -y -a codex` — name the agents the teammate actually uses, space-separated (e.g. `-a codex cursor`); avoid `-a '*'`, which targets every agent skills.sh knows.

## Step 3 — Install the critical skill set

Install every skill listed in [critical-skills.md](./critical-skills.md), using the RAI standard flags documented there (global, Claude Code, non-interactive). Run each command; if one fails, record it and continue with the rest.

Pass on the heads-up in that file: installed skills run with full agent permissions, and `skills.sh` shows a security rating per skill at install time.

Then prune stale skills: run `npx --yes skills@latest update -g -y` and, if it warns that any skills were *deleted upstream*, remove the named ones with `npx --yes skills@latest remove <names> -g -y -a claude-code` (see "Pruning stale skills" in [critical-skills.md](./critical-skills.md)). This clears leftovers a major upstream release renamed or dropped — e.g. a `write-a-skill` or `diagnose` copy from before mattpocock/skills v1.0.0 — that would otherwise sit alongside their replacements.

## Finish — report the result

Print a short ✅ / ❌ summary, for example:

- Git + identity: ✅ `git 2.54`, user configured
- GitHub CLI: ✅ `gh 2.x`, authenticated as `<user>`
- Node + npx: ✅ `v20.x`
- RAI marketplace: ✅ `rai-skills@rai`
- Critical skills: ✅ 6/6 roster commands succeeded (or ❌ name the ones that failed)
- Stale skills pruned: ✅ removed N deleted-upstream skills (or "none to prune")

If everything is ✅, tell them they're set up. Point to the per-project next step: run `setup-matt-pocock-skills` once inside a repo so the engineering skills (`to-prd`, `triage`, …) learn where that repo tracks issues and which triage labels to use. If anything is ❌, say exactly what's left to do.
