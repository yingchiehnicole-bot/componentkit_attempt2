---
description: Before starting new work: first-time setup if needed, sync main, create branch, start Storybook
---

When a designer is about to start a new feature or component, follow these steps.

## Step 0: First-Time Setup (checked every run, skipped when already done)

The very first `/start` on a new computer or a freshly unzipped project sets everything up. Every later run just re-checks each item below — each check takes seconds, and anything that's already in place is skipped silently. Only speak up about the items that actually need work.

While doing any of this:

- I am a designer. Tell me in plain language what you're about to do and why, one step at a time — never paste raw commands or error output at me without explaining them.
- Some steps need me to type a password, press Enter, or click something. Your own shell can't take that kind of input, so for those steps open a **separate terminal window** for me (see 0c), tell me exactly what I'll see and what to do, and wait for me to say I'm done before checking again.
- If an install is refused because I don't have admin rights on this computer, or the network blocks a download (proxy / certificate errors), stop and tell me in plain language to ask my IT team, and what to ask them for.
- If something was half-finished last time (e.g. I closed a window midway), just pick up from the first item that isn't done yet.

### 0a. Check the project folder

- If the folder path contains `.zip` or a temporary folder (e.g. `\AppData\Local\Temp\`), I opened the project from inside the zip without extracting it. Stop and tell me: right-click the zip → **Extract All**, then open the extracted folder in VS Code.
- If the path is inside iCloud Drive (`Library/Mobile Documents`) or OneDrive (path contains `OneDrive`), stop and tell me cloud syncing will break the project. Ask me to move the folder to a non-synced place (e.g. a `Projects` folder in my home folder) and reopen it from there.

### 0b. Check the tools

Figure out whether this is a Mac or Windows. On Windows your shell may be PowerShell — use PowerShell equivalents for everything in this file.

Check for each of these:

| Tool | Check | Needed for |
|------|-------|------------|
| Node.js `^20.19.0 \|\| >=22.12.0` | `node -v` | Storybook, building |
| Git | `git --version` | saving work |
| GitHub CLI | `gh --version` | only if needed — see 0d and 0f; don't install it up front |

**Mac**

1. If `xcode-select -p` fails, run `xcode-select --install`. A system window pops up — tell me to click **Install** and that it takes about 5–15 minutes. Re-check `xcode-select -p` every minute or so until it succeeds. (This also provides Git.)
2. Homebrew is only needed to install Node or GitHub CLI. If one of them is needed and Homebrew is missing (`/opt/homebrew/bin/brew` and `/usr/local/bin/brew` both absent), install it in a separate Terminal window (0c) with the official command:
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
   Warn me first: it will ask for my **Mac login password**, nothing appears on screen while I type it (that's normal), then press Enter. It may also ask me to press Enter to continue.
3. Your shell won't see Homebrew right after installing it. Prefix every command that needs it with:
   ```bash
   eval "$(/opt/homebrew/bin/brew shellenv 2>/dev/null || /usr/local/bin/brew shellenv)"
   ```
   And make it permanent for future sessions — append that same line to `~/.zprofile` if it isn't already there.
4. `brew install node` if Node is missing or older than required; `brew install gh` only when 0d or 0f calls for it.

**Windows**

1. Install whatever is missing with winget (GitHub CLI only when 0d or 0f calls for it):
   ```powershell
   winget install --id Git.Git -e --source winget --accept-package-agreements --accept-source-agreements
   winget install --id OpenJS.NodeJS.LTS -e --source winget --accept-package-agreements --accept-source-agreements
   ```
   GitHub CLI, when needed: `winget install --id GitHub.cli -e --source winget --accept-package-agreements --accept-source-agreements`

   Warn me first: Windows may ask "Do you want to allow this app to make changes to your device?" — click **Yes**.
2. If `winget` itself is missing, ask me to install **App Installer** from the Microsoft Store, then continue.
3. Your shell won't see newly installed tools until the path is reloaded. Reload it in the same session instead of asking me to restart VS Code:
   ```powershell
   $env:Path = [Environment]::GetEnvironmentVariable('Path','Machine') + ';' + [Environment]::GetEnvironmentVariable('Path','User')
   ```
4. If `npm` fails with "running scripts is disabled on this system", run:
   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
   ```

### 0c. How to open a separate terminal window for me

- **Mac:** write the commands into a temporary file ending in `.command` (e.g. in `$TMPDIR`), `chmod +x` it, then `open` it — Terminal launches and runs it.
- **Windows:** write the commands into a temporary `.ps1` file, then:
  ```powershell
  Start-Process powershell -ArgumentList '-NoExit','-ExecutionPolicy','Bypass','-File','<path to the .ps1>'
  ```

### 0d. Log in to GitHub

I may already be logged in to GitHub on this computer in some way. Use whatever already works — don't make me log in again. Check, in this order, and stop at the first one that works:

1. **GitHub CLI**: `gh auth status` says I'm logged in. Also run `gh auth setup-git` (harmless if already done), so Git uses this login too.
2. **SSH key**: `ssh -T -o BatchMode=yes -o StrictHostKeyChecking=accept-new git@github.com` replies `Hi <username>!` (it exits with code 1 even on success — go by the text).
3. **Saved HTTPS login** (Keychain / Windows Credential Manager) — print the username only, never the password:
   ```bash
   printf 'protocol=https\nhost=github.com\n\n' | GIT_TERMINAL_PROMPT=0 GCM_INTERACTIVE=never git credential fill 2>/dev/null | grep '^username='
   ```

Remember which method worked and my GitHub username — 0e and 0f use them.

If one of them works, tell me in one short sentence that this computer is already logged in to GitHub, so I can skip the login steps (including the README section "Logging in to GitHub, step by step") — nothing for me to do here.

If none of these work, tell me this computer isn't logged in to GitHub yet, so I need to do the one-time login now. Then log me in with GitHub CLI: install it (0b) if missing, then run this in a separate terminal window (0c):

```bash
gh auth login --hostname github.com --git-protocol https --web
```

Tell me beforehand what I'll see, and point me to the README section **"Logging in to GitHub, step by step"**, which has screenshots: the window asks "Authenticate Git with your GitHub credentials?" — I type `y` and press Enter; it shows a one-time code (already copied) and asks me to press Enter; my browser opens GitHub; I paste the code, click **Continue**, then **Authorize github**; the window says "Logged in as …" and I come back here.

After I confirm, re-check `gh auth status`, then run `gh auth setup-git` so Git uses this login and never asks me for a username or password.

All of this is a one-time thing: the login is saved on the computer, so later runs pass the first check that worked and move on.

### 0e. Set my name for saved work

If `git config --global user.name` or `git config --global user.email` is empty, fill them in from my public GitHub profile, using the username from 0d — don't ask me:

```bash
curl -s https://api.github.com/users/<username>
```

- `user.name` → the profile's `name`, or the username if `name` is empty.
- `user.email` → `<id>+<username>@users.noreply.github.com`, using the profile's `id`. This "noreply" email keeps my real email private and is never rejected by GitHub's email-privacy setting.

### 0f. Connect this project to my GitHub

Skip this if the project already has a `.git` folder and `git remote get-url origin` works.

If `origin` points at `VyntaHQ/component-kit-builder`, the zip I received was packed incorrectly — stop and tell me to contact the team that sent it.

Otherwise:

1. **Ask what to call the library.** Offer the current folder name as the default, converted to lowercase-with-hyphens (e.g. `My Design System` → `my-design-system`) — package names can only use lowercase letters, numbers and hyphens, so convert whatever I answer and confirm the result with me. If the folder still has the original name (contains `component-kit-builder`), remind me that I can rename the folder to match once this is done: **File → Close Folder** in VS Code, rename it in Finder / File Explorer, then reopen it.

2. **Ask where to put it**, in one question: "Which GitHub account or organization should this live under? Paste the URL of its page — for example `https://github.com/your-account-name`." Parse the owner out of the URL. If I paste a repo URL instead (`https://github.com/<owner>/<repo>`), use both the owner and that repo name. Confirm it exists with `curl -s https://api.github.com/users/<owner>` — its `type` is `User` (personal account) or `Organization`.

3. **Pick the repo address** in the same style as the login that worked in 0d: `git@github.com:<owner>/<name>.git` for SSH, `https://github.com/<owner>/<name>.git` otherwise. Then **check the name is free** with `GIT_TERMINAL_PROMPT=0 git ls-remote <address>`:
   - Fails with "repository not found" (the message may be in another language) → doesn't exist yet, it'll be created in step 6.
   - Succeeds with no output → it exists and is empty; use it as-is in step 6.
   - Succeeds and lists anything → it already has content. Stop and ask me for a different name. Don't touch the existing repo.

4. **Rename the project:** set `name` in `package.json` to the library name, and the two top-level `name` fields at the start of `package-lock.json` to match. Don't add a `@scope/` prefix — `/publish` takes care of that.

5. **Save a first version:** confirm there's no `.npmrc` file, then:
   ```bash
   git init -b main
   git add .
   git commit -m "chore: initial commit"
   ```

6. **Create the repo on GitHub and upload.** It must always be **private** — the kit is licensed to me for my own use, so its files can't be made public. Tell me that in one short sentence.
   - **New repo:** creating one needs GitHub CLI logged in. If it isn't (0d found an SSH key or a saved login instead), ask me which I prefer:
     - **Log in once more through the browser so Claude creates it** (recommended) — install GitHub CLI if missing and log in as in 0d, but skip `gh auth setup-git` since my existing login already works for saving.
     - **Create it myself on GitHub** — tell me: go to [github.com/new](https://github.com/new), choose the owner, type the name, select **Private**, leave every "Add …" option unticked, click **Create repository**, and tell you when done. Then continue as an existing empty repo.

     Once GitHub CLI is logged in:
     ```bash
     gh repo create <owner>/<name> --private
     ```
   - **Then, for a new or existing empty repo:**
     ```bash
     git remote add origin <address from step 3>
     git push -u origin main
     ```
   If creating it under an organization fails with a permission error, tell me plainly that the organization doesn't allow me to create repos, and offer two choices: put it under my personal account instead, or ask an organization admin to create an empty repo with this name and then paste me its URL.

7. Tell me the repo is ready and give me its link: `https://github.com/<owner>/<name>`.

## Step 1: Sync Main Branch

If `git status --porcelain` isn't empty, I have unsaved changes from before. Stop and tell me, in plain language, to run `/done` first to save them (or tell you to discard them).

Then look for earlier work that never made it into `main` — e.g. a branch where I stopped before running `/done`:

```bash
git fetch origin --prune
git for-each-ref --format='%(refname:short)' refs/heads refs/remotes/origin
```

For each branch other than `main` / `origin/main` / `origin/HEAD`, count the saved changes that aren't in `main` with `git rev-list --count origin/main..<branch>`. Treat a local branch and its `origin/` copy as one. If any have a count above 0, tell me in plain language — e.g. "You have unfinished work on **product card** (3 saved changes) that isn't in the main version yet, so it won't show up in today's work or the next release." — and ask what to do with each:

- **Add it to the main version now** → do what `/done` does from Step 4 (merge into main) for that branch, then carry on.
- **Keep working on it today** → switch to that branch (`git checkout <branch>`), skip the sync below and Step 2, and continue from Step 3.
- **Leave it for now** → carry on without it.

Then sync:

```bash
git checkout main
git pull origin main
```

## Step 2: Ask for Branch Name

Ask me which component I'm working on today (e.g. "product card", "nav bar"), then create a branch based on my answer. Format: `feature/<description>`

Examples: `feature/product-card`, `feature/nav-bar`

```bash
git checkout -b feature/<description>
```

## Step 3: Install Dependencies (if needed)

Check if `node_modules` exists. If it doesn't, run:

```bash
npm install
```

Skip this step if `node_modules` already exists. The first install can take a few minutes — tell me so I don't think it's stuck.

## Step 4: Start Storybook

Start it **in the background**, so it keeps running without blocking this conversation:

```bash
npm run storybook
```

If port 6006 is already in use, Storybook is probably already running from earlier — just point me to it.

When done, tell me: the branch name, that Storybook is available at http://localhost:6006, and that they can use `/generate-component` to create a new component.
