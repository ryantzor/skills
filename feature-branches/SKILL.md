---
name: feature-branches
description: >
  REQUIRED when creating feature branches, working on features in parallel with
  multiple agents, creating pull requests, or syncing branches with main.
  Use before branching off or coordinating work across agents.
  Triggers: creating a branch, multi-agent coordination, PR creation, branch sync,
  rebasing, DB migration conflicts.
---

# Feature Branch Workflow

Parallel feature development across multiple agents.

## Branch Naming

```
feature/<short-description-with-dashes>
```

Examples: `feature/add-dark-mode`, `feature/customer-email-notification`

## Trunk-Based Development Rules

1. Always branch from `main`
2. Keep branches short-lived — 1-2 days, max 5
3. Rebase frequently — sync with main at least daily
4. PR back to main when complete
5. Delete branch after merge

## Starting a New Feature

```bash
git checkout main && git pull origin main
git checkout -b feature/<short-description>
```

## Keeping In Sync

### Preferred: Rebase (local / unpushed branches)

```bash
git fetch origin main && git rebase origin/main
```

Conflicts: resolve files, `git add <files>`, `git rebase --continue`

### Alternative: Merge (pushed / shared branches)

```bash
git fetch origin main && git merge origin/main
```

## Multi-Agent Coordination

- One branch per agent per feature
- Document the branch name in the task prompt
- Communicate if work touches shared code paths (DB queries, lib modules, auth)
- When one agent merges first, others rebase

### DB Schema Changes

Only one agent should add migration files to avoid conflicts.

- Create numbered migrations in `supabase/migrations/` using the next available number
- **If agents create migrations in parallel**: first to merge keeps their number; others rebase and renumber to the next available number
- Run `npm run db:pull` to capture schema changes as a migration

## PR Workflow

### Creating a PR

```bash
git push -u origin feature/my-feature
gh pr create --title "feature: <short description>" --fill
```

### Before Merging Checklist

- [ ] `npm run check` passes (TypeScript)
- [ ] `npm test` passes (Playwright E2E)
- [ ] Branch rebased on latest `main`
- [ ] Migration files (if any) numbered sequentially
- [ ] Branch pushed to remote

### Merging

```bash
gh pr merge --squash --delete-branch
```

Or via GitHub UI: **Squash and merge**.

### Local Cleanup

After the PR is merged and remote branch deleted, prompt the user before cleaning up the local branch:

> Always prompt — never delete a local branch without a confirm dialog.

Ask the user (use the `question` tool):

> "Branch 'feature/xxx' has been merged. Delete local branch?"
> Options: "Yes, clean up", "Keep local branch"

If confirmed, run:

```bash
git checkout main && git pull origin main && git branch -d feature/xxx
```

## Commit Messages

Free-form on feature branches — they get squashed into one message on merge.

## When NOT to Use

- Single-agent, single-task changes (commit directly or use one branch)
- Trivial fixes that don't need isolation
