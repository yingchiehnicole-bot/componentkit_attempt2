---
description: Publish component library: build, bump version, publish to GitHub Packages for the frontend team to install
---

Bundle and publish the component library to **GitHub Packages** for the frontend team to install.

Releases always happen on `main`, which holds all finished work (`/done` merges each batch into it).

## Step 0: Make Sure Everything Is Saved

1. If there's no `.git` folder or `git remote get-url origin` fails, this project hasn't been connected to GitHub yet. Stop and tell me, in plain language, to run `/start` first — it sets everything up the first time.
2. If `git status --porcelain` isn't empty, I have unsaved changes. Stop and tell me to run `/done` first.
3. If I'm on a branch other than `main` and it has work that isn't in `main` yet (`git rev-list --count main..HEAD` is above 0), stop and tell me to run `/done` first — it saves that work into the main version so it's included in the release.

Then switch to `main` and get the latest:

```bash
git checkout main
git pull origin main
```

## Step 1: One-Time Setup (only if not done yet)

Check whether this project is already configured:

- `.npmrc` exists and its token line no longer says `YOUR_TOKEN_HERE`
- `package.json`'s `name` is scoped (`@something/...`) **and** the scope matches the repo owner from `git remote get-url origin`, lowercased — a mismatch makes publishing fail
- `package.json` has a `publishConfig` and a `repository` field, and `repository.url` points at this repo

If **all** of these are in place, skip this whole step and go straight to Step 2.

We're on `main` now, where a safety check blocks direct file edits. So make the changes in (c) and (d) with shell commands (e.g. `cp`, `npm pkg set`), not by editing the files.

Otherwise, walk me through only the parts that are missing, one part at a time — don't dump all of it on me at once, and never ask me to paste a secret token into this chat. For example, if only `repository` is missing, just do (a) to read the remote, then (d) and (e) — don't ask me for a new token.

**a. Determine the scope**

Run:
```bash
git remote get-url origin
```
Parse the GitHub org or username out of the URL (works for both `https://github.com/<org>/<repo>.git` and `git@github.com:<org>/<repo>.git`). Lowercase it and tell me directly: "Your scope is `@<org-lowercase>`." Mention briefly that npm scopes must be all lowercase, so this is what we use even if the GitHub org/username has capital letters.

**b. Generate a publish token**

Tell me to:
1. Go to [github.com/settings/tokens/new](https://github.com/settings/tokens/new)
2. Select **Classic token**
3. **Only check `write:packages`** — leave every other box (`repo`, `workflow`, `admin:org`, etc.) unchecked. That single scope is all this needs.
4. Set **Expiration** to a long period (e.g. 1 year) — when the token expires, publishing stops working until I make a new one
5. Generate and copy the token
6. **Organization repos only:** if the token list shows a **Configure SSO** button next to the new token, click it and **Authorize** the organization — otherwise the organization rejects the token

**c. Fill in `.npmrc`**

If `.npmrc` doesn't exist yet, copy `.npmrc.example` to `.npmrc`. (If it exists but only the token is missing or expired, leave the rest alone and just ask me to paste the new token.) Then fill in the scope line using the value from (a) — keep only the line matching the account type that applies (org vs personal), remove or comment out the other.

Leave the `YOUR_TOKEN_HERE` line as a placeholder — do **not** ask me to paste the token here in chat, it's a secret credential. Instead tell me clearly, in plain language: "Open `.npmrc` in the file explorer on the left side of VS Code — it's in the project root, next to `package.json`. It won't show up in git since it's gitignored. Paste your token in place of `YOUR_TOKEN_HERE` on the last line, then save the file." Wait for me to confirm before moving on.

**d. Update `package.json`**

Update the `name` field to `@<scope>/<existing-library-name>` (keep the current library name, just prefix it with the scope from (a)), and add the fields below — `npm pkg set name=... publishConfig.registry=... repository.type=git repository.url=...` does it in one command:
```json
"publishConfig": {
  "registry": "https://npm.pkg.github.com"
},
"repository": {
  "type": "git",
  "url": "https://github.com/<owner>/<repo>.git"
}
```

Fill `repository.url` from the remote URL read in (a), always in the `https://github.com/<owner>/<repo>.git` form (convert `git@github.com:<owner>/<repo>.git` if needed). This links the package to the repo on GitHub, so anyone who can see the repo can also see and install the package — without it, the package stays visible only to its owner.

**e. Commit the config change**

Run `git status --porcelain` and confirm `.npmrc` does **not** appear (if it does, stop and flag it — don't continue). Only `package.json` should show as changed. Then commit automatically — don't ask whether to, just tell me you're doing it:
```bash
git add package.json
git commit -m "chore: configure package for GitHub Packages publishing"
```
Tell me this step always auto-commits, and that it gets uploaded together with the release at the end.

Once setup is confirmed complete, continue to Step 2.

## Step 2: Build

Build right before releasing, on the up-to-date `main`, so the package contains exactly what's in `main` now:

```bash
npm run typecheck && npm run build
```

Confirm the build succeeds with no errors before continuing. Do not proceed if either command fails.

## Step 3: Bump Version

Ask me which version to bump:
- **patch** (1.0.0 → 1.0.1): small bug fixes, style tweaks
- **minor** (1.0.0 → 1.1.0): new components, new features
- **major** (1.0.0 → 2.0.0): breaking changes, major rework

After I answer, change only the version number — no commit or tag yet, those come in Step 5 once the release is confirmed:
```bash
npm version <patch|minor|major> --no-git-tag-version
```

## Step 4: Publish to GitHub Packages

This is the step that actually makes the library installable by the frontend team.

```bash
npm publish
```

**If it fails**, first undo the version change so the next attempt doesn't skip a number:
```bash
git checkout -- package.json package-lock.json
```
Then explain to me in plain language what went wrong and what to do, based on the error:

| Error mentions | Meaning | What to do |
|---|---|---|
| `401` / unauthenticated | The token is wrong or has expired | Make a new token (Step 1 b) and paste it into `.npmrc` (Step 1 c), then run `/publish` again |
| `403` / permission | The token is missing `write:packages`, or the scope doesn't match the repo owner. For an organization, also: the token isn't authorized for the organization's single sign-on, the organization blocks classic tokens, or it doesn't let members publish packages | Check the token's scope and the name in `package.json`. For an organization: click **Configure SSO → Authorize** next to the token at [github.com/settings/tokens](https://github.com/settings/tokens); if that's not it, ask an organization admin to allow classic tokens and package publishing for members |
| `404` | The name's scope or `publishConfig` doesn't match this repo | Re-check Step 1 (a) and (d) |
| `409` / "cannot publish over" | This version number was already released | Run `/publish` again and pick the next version |

In every failure case, **stop — don't run Step 5**.

**If it succeeds**, verify it, reading the real name and version from `package.json`:

1. The new version is really on GitHub Packages:
   ```bash
   npm view <package-name>@<new-version> version
   ```
   It should print the new version. GitHub can take a few seconds to show a new version, so if it doesn't, wait about 10 seconds and try again, up to 3 times. If it still prints nothing or errors, undo the version change as above, tell me in plain language that the release didn't go through, and **stop — don't run Step 5**. (A version tag on GitHub should always mean the package is really installable.)

2. The package is linked to this repo — without the link, only I can see it, even after I add collaborators. Read the token out of `.npmrc` into a shell variable (never print it), and ask GitHub which repo the package belongs to:
   ```bash
   TOKEN=$(sed -n 's#^//npm.pkg.github.com/:_authToken=##p' .npmrc)
   curl -s -H "Authorization: Bearer $TOKEN" https://api.github.com/<users|orgs>/<owner>/packages/npm/<library-name>
   ```
   Use `users` for a personal account, `orgs` for an organization, and `<library-name>` without the `@scope/`. Read `repository.full_name` from the response (parse it with `node -e`). If it isn't `<owner>/<repo>`, the release still worked — carry on — but remember to tell me in Step 6 how to link it.

## Step 5: Save the Version and Push

Only now record the release in git and upload it:
```bash
git add package.json package-lock.json
git commit -m "chore: release v<new-version>"
git tag v<new-version>
git push origin main v<new-version>
```

## Step 6: Report

Tell me the new version number and how the frontend team should update, reading the real package name from `package.json`:

```bash
npm install <package-name>@<new-version>
```

Then tell me, in plain language, who can see the package (a package linked to its repo is visible to the same people as the repo):
- If the repo belongs to a **personal account**: only people added to this repo can see and install it. Tell me to add each frontend teammate under the repo's **Settings → Collaborators**.
- If the repo belongs to an **organization**: anyone in the organization with access to this repo can see and install it.

Give me the package page link so I can see the release for myself: `https://github.com/<owner>/<repo>/pkgs/npm/<library-name>`. Tell me that page should show the new version marked **Latest**, and point me to the README section **"Check that it was published"**, which has screenshots of what to look for. Tell me teammates find it on the repo's main page, under **Packages** on the right side — it won't appear on their own GitHub profile. To install it, each of them needs their own token with `read:packages` (README section "For Frontend Engineers").

If the check in Step 4 found the package isn't linked to this repo (e.g. versions were published before the `repository` field existed), tell me to link it once: open my GitHub profile → **Packages** tab → this package → **Connect Repository** (or **Package settings**) → choose this repo. Until then, only I can see it, even if I've added collaborators.
