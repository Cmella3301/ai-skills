---
name: github-workflow
description: Set up Git and GitHub CLI on Windows, create repos, push projects, and manage day-to-day commits. Use this skill when a user wants to push a project to GitHub, make commits, or needs help with Git on Windows for the first time.
---

# Skill: GitHub Workflow (Windows)

Use this skill for:
- First-time Git + GitHub setup on Windows
- Pushing an existing local project to a new GitHub repo
- Day-to-day commit and push workflow
- Managing multiple repos

---

## Step 1 — Install Git & GitHub CLI

If `git` or `gh` are not recognized, install both via winget:

```powershell
winget install Git.Git
winget install GitHub.cli
```

> ⚠️ Both will prompt for UAC (admin) approval — click Yes.

### Fix PATH in the current session (avoids reopening terminal):
```powershell
$env:PATH = "C:\Program Files\Git\cmd;C:\Program Files\GitHub CLI;" + $env:PATH
```

Verify both are working:
```powershell
git --version
gh --version
```

---

## Step 2 — Authenticate with GitHub

```powershell
$env:PATH = "C:\Program Files\Git\cmd;C:\Program Files\GitHub CLI;" + $env:PATH
gh auth login --web -h github.com
```

This will:
1. Display a **one-time code** (e.g. `9276-0E11`) — copy it
2. Open **https://github.com/login/device** in the browser
3. Sign in to GitHub, paste the code, and click **Authorize GitHub CLI**

Check auth status:
```powershell
gh auth status
```

---

## Step 3 — Configure Git Identity

Only needed once per machine:
```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## Step 4 — Create `.gitignore`

Always create a `.gitignore` before the first commit. Common entries:

```gitignore
# Node.js
node_modules/

# Database / local data
data/
*.db
*.db-journal

# Logs
*.log
output*.txt

# Environment secrets
.env
.env.local

# OS files
.DS_Store
Thumbs.db
desktop.ini
```

---

## Step 5 — Initialize Repo & First Commit

```powershell
cd "C:\path\to\your\project"
git init
git add .
git commit -m "🚀 Initial commit"
```

---

## Step 6 — Create GitHub Repo & Push

Use GitHub CLI to create the repo and push in one command:

```powershell
$env:PATH = "C:\Program Files\Git\cmd;C:\Program Files\GitHub CLI;" + $env:PATH

# Public repo
gh repo create <repo-name> --public --description "Your description" --source=. --remote=origin --push

# Private repo
gh repo create <repo-name> --private --description "Your description" --source=. --remote=origin --push
```

The URL will be printed on success:
```
https://github.com/YourUsername/repo-name
```

---

## Day-to-Day: Save Changes to GitHub

Every time you make changes you want to keep:

```powershell
git add .
git commit -m "describe what changed"
git push
```

### Good commit message examples:
```
✨ Add water tracking feature
🐛 Fix habit streak not resetting
🎨 Update nav bar styling
📝 Update README
🔒 Fix auth issue
```

---

## Useful Git Commands

```powershell
# Check what's changed
git status

# See commit history
git log --oneline -10

# Undo last commit (keeps changes)
git reset --soft HEAD~1

# Discard all local changes (careful!)
git checkout -- .

# Pull latest from GitHub
git pull

# See all branches
git branch -a

# Create a new branch
git checkout -b feature/my-new-feature

# Switch branch
git checkout main
```

---

## Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| `git` not recognized | `$env:PATH = "C:\Program Files\Git\cmd;" + $env:PATH` |
| `gh` not recognized | `$env:PATH = "C:\Program Files\GitHub CLI;" + $env:PATH` |
| `gh auth login` code expired | Re-run `gh auth login --web` for a new code |
| Push rejected (non-fast-forward) | Run `git pull --rebase` then `git push` |
| Accidentally committed `node_modules` | Add to `.gitignore`, run `git rm -r --cached node_modules/`, then commit |
| Wrong email in commits | `git config --global user.email "correct@email.com"` |

---

## Repo Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| App/project | `kebab-case` | `locke-in-app` |
| Skills/docs collection | descriptive noun | `ai-skills` |
| Personal configs | `dotfiles` | `dotfiles` |

---

## Notes (Cmella3301's Setup)

- GitHub username: **Cmella3301**
- Git installed at: `C:\Program Files\Git\cmd`
- GitHub CLI installed at: `C:\Program Files\GitHub CLI`
- Active repos:
  - 🔒 [locke-in-app](https://github.com/Cmella3301/locke-in-app) — self-hosted PWA habit tracker
  - 🧠 [ai-skills](https://github.com/Cmella3301/ai-skills) — skills collection
