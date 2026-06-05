---
name: rai-setup-skills
description: Onboard a teammate to the rai-skills toolkit by checking prerequisites are in place. Today it verifies the GitHub CLI (gh) is installed and authenticated; more setup steps will be added over time. Use when someone is setting up rai-skills for the first time, onboarding a new teammate, or mentions rai setup or onboarding.
disable-model-invocation: true
---

# Onboard a teammate to rai-skills

Get a teammate's machine ready to use the `rai-*` skills. Walk the steps **in order**, fix each before moving on, and finish with a short ✅ / ❌ summary so it's obvious what (if anything) still needs doing.

This is deliberately small right now — it confirms the **GitHub CLI** is installed and logged in. More steps (installing the skills marketplace, project config, and so on) will be added to this same flow later, so keep the structure extensible.

## Step 1 — Is the GitHub CLI installed?

The `rai-*` skills lean on GitHub (issues, PRs, and the skills repo itself), so `gh` needs to be on the PATH.

Run:

```
gh --version
```

- **Prints a version** → ✅ continue to Step 2.
- **Command not found** → install it for this machine's OS, then re-run the check:
  - **Windows:** `winget install --id GitHub.cli` (or `scoop install gh` / `choco install gh`)
  - **macOS:** `brew install gh`
  - **Linux:** see https://github.com/cli/cli#installation (for example `sudo apt install gh`)

Don't move on until `gh --version` succeeds.

## Step 2 — Is the GitHub CLI logged in?

Run:

```
gh auth status
```

- **Shows an authenticated github.com account** → ✅ done.
- **Not logged in** → `gh auth login` is interactive, so hand it to the teammate: ask them to type **`! gh auth login`** in the prompt and follow it through (choose **GitHub.com → HTTPS → Login with a web browser**). When they're back, re-run `gh auth status` to confirm.

## Finish — report the result

Print a short summary, for example:

- GitHub CLI installed: ✅ `gh version 2.x`
- GitHub CLI authenticated: ✅ signed in as `<user>`

If both are ✅, tell them they're set up and point to the next step (installing the rai-skills marketplace — added to this flow soon). If anything is ❌, say exactly what's left to do.
