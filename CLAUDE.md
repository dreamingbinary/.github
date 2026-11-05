# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

An org-level `.github` repository: default community-health files plus automation that pushes shared config into *other* repos in the org. It is a copy of `googleworkspace/.github`, now hosted at `dreamingbinary/.github`. Most files still hardcode the `googleworkspace` org, its bot account and its teams. Assume any org-specific value is stale until it has been changed.

There is no application code, build or test suite. The only code is `.github/workflows/workflows.sh` (bash) and YAML.

## Architecture

- **`sync-files/defaults/`**: templates pushed to target repos. `.github/sync-files-all.yml` maps each source to a destination. `replace: false` writes the file only once (the target repo can then edit it). `replace: true` overwrites it on every sync (automation.yml, sync-repo-settings.yaml, linters, SECURITY.md).
- **`sync-files.yml`** (weekly cron): runs `gh repo list <org>` to append every public, non-archived, non-fork repo to the `repos:` block of `sync-files-all.yml`. It then runs `BetaHuhn/repo-file-sync-action`, which opens PRs in each target repo. That `repos:` list is generated at runtime; do not edit it by hand.
- **`workflows.yml` + `workflows.sh`** (weekly cron): rebuilds the badge table in README.md and opens a PR. The script deletes everything from the `<!-- WORKFLOWS_INSERT_AFTER -->` marker onward and rewrites it, so never put hand-written content below that marker.
- **`automation.yml`**: auto-approves and merges Dependabot PRs, and mirrors the `main`/`master` branches. It is also synced to every target repo.
- Bot configs (blunderbuss, auto-label, stale, do-not-merge, snippet-bot) are read by GitHub Apps; no workflow uses them.
- Files in `.github/` and in `sync-files/defaults/.github/` that share a path (lint.yml, test.yml, linters/, CODEOWNERS, sync-repo-settings.yaml) are separate copies. Edit both when a change should apply here and in the target repos.

## Known breakage (as of this fork)

- Every write-path workflow uses `secrets.GOOGLEWORKSPACE_BOT_TOKEN`, which almost certainly does not exist in `dreamingbinary`.
- `gh repo list googleworkspace` (sync-files.yml, workflows.sh) and `ORG=` in workflows.sh still target the upstream org. The README badge table therefore lists googleworkspace repos.
- The `lint.yml` and `test.yml` placeholders run `exit 1`, so they always fail. The templates in sync-files carry the same placeholder.
- CODEOWNERS, `sync-repo-settings.yaml`, and the bot git identity (`googleworkspace-bot@google.com`) refer to upstream teams and users.
- Actions are outdated (`actions/checkout@v2`, and `@master` in sync-files.yml).

## Local checks

```bash
shellcheck .github/workflows/workflows.sh
yamllint -c .github/linters/.yaml-lint.yml .github/
```
