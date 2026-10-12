---
description: Finish current work: commit, merge into main, push to remote
---

The current changes are complete. Please submit them.

## Step 0: Make Sure Setup Is Done

If there's no `.git` folder or `git remote get-url origin` fails, this project hasn't been connected to GitHub yet. Stop and tell me, in plain language, to run `/start` first — it sets everything up the first time.

## Step 1: Check Current Status

```bash
git status
git diff --stat
```

## Step 2: Generate Commit Message

Based on the git diff, write a properly formatted commit message:

- Added a component → `feat: add <ComponentName> component`
- Style changes → `style: adjust <ComponentName> styles`
- Bug fix → `fix: fix <ComponentName> <description>`
- Docs / config → `docs: <description>` or `chore: <description>`

Do not ask me — infer the message from the changes.

## Step 3: Save

If there are changes, commit them (skip this if there's nothing to commit — the branch may still have earlier saved work to merge):

```bash
git add .
git commit -m "<commit message>"
```

## Step 4: Merge Into Main

Finished work goes straight into `main` — there's no review step — so the next `/start` always begins from everything done so far, and `/publish` releases all of it.

If we're already on `main`, just run `git push origin main` and skip to Step 5.

Otherwise, first bring in anything new from `main` while still on this branch, so any conflict gets sorted out here rather than on `main`:

```bash
BRANCH=$(git branch --show-current)
git fetch origin
git merge origin/main
```

If this reports a conflict, explain it to me in plain language (which component or file, what each side changed), suggest how to combine them, and resolve it once I agree. Then run `npm run typecheck` to make sure nothing broke.

Then move `main` forward to include this work and upload it:

```bash
git checkout main
git pull origin main
git merge --ff-only "$BRANCH"
git push origin main
git branch -d "$BRANCH"
git push origin --delete "$BRANCH" 2>/dev/null || true
```

## Step 5: Report

Tell me, in plain language, that my work is saved and included in the main version of the library. Then tell me:
- To keep working (on this or anything else), run `/start` — it creates a fresh working branch.
- When I'm ready to release it to the frontend team, run `/publish`.
