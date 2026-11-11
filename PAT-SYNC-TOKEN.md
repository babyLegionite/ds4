# SYNC_TOKEN — what this PAT is for (and how to finish the setup)

**Status: PENDING — need a classic PAT to fully automate upstream-sync PR creation.**

## Why we need it

`.github/workflows/upstream-sync.yml` (in `babyLegionite/ds4`) runs on cron
(Wed + Sun 04:30 UTC) and keeps the fork current with `antirez/ds4`. It:
1. fetches upstream/main
2. merges the fork's changes (workflow file) into `sync/upstream`
3. force-pushes the branch
4. creates/updates the sync PR

Steps 1–3 work with the `GITHUB_TOKEN`. Step 4 (creating/updating the PR) does
**not**, because:
- This repo has **"Allow GitHub Actions to create and approve pull requests"**
  disabled (a UI-only setting — cannot be changed via the REST API). That blocks
  the bot token from ALL PR mutations (`createPullRequest`, `addComment`), on
  both schedule and dispatch.
- Schedule-triggered tokens are additionally read-only for PRs regardless of
  `permissions:`.

A **personal access token (PAT)** bypasses both limits: it's a real user token,
so the repo setting and the schedule read-only rule don't apply.

## What to do (when you get the PAT)

1. Create a **classic PAT** with the `repo` scope:
   github.com → Settings → Developer settings → Personal access tokens → **Tokens (classic)** → Generate new token → select `repo` (and `workflow` if you want it to also manage Actions) → copy the `ghp_...` value.
2. Add it as the `SYNC_TOKEN` secret in the repo:
   `gh secret set SYNC_TOKEN --repo babyLegionite/ds4 --body "ghp_..."`  (or Settings → Secrets and variables → Actions → New repository secret).
3. Trigger a manual dispatch to confirm:
   `gh workflow run upstream-sync.yml --repo babyLegionite/ds4`
   The run should now reach `gh pr create` / `gh pr comment` and post the PR body.

## Until the PAT is added (current behavior — already fixed)

The cron still runs cleanly every Wed + Sun:
- it always merges + pushes the `sync/upstream` branch,
- the open sync PR **auto-tracks the branch head**, so it stays current,
- it only skips posting the PR body comment (emits a `::warning::` telling you
  to add the SYNC_TOKEN secret).

So the fork stays up to date even without the PAT; the PAT just completes the
PR-body automation.

## How to check the workflow is healthy

- `gh run list --repo babyLegionite/ds4 --workflow upstream-sync.yml`
- The latest runs should show `success` (green).
- `gh pr list --repo babyLegionite/ds4` should show the open `sync/upstream` PR.
