# Git Commands - Beginner Guide

---

## The Golden Rule
> Always create a new branch before making changes. Never work directly on `main`.

---

## Daily Workflow (Do this every time)

```
1. git pull                        → get latest changes from GitHub
2. git checkout -b yourname/task   → create new branch
3. make your code changes
4. git add .                       → stage all changes
5. git commit -m "what you did"    → save with a message
6. git push origin yourname/task   → send to GitHub
7. go to GitHub → create Pull Request → ask senior to review
```

---

## Starting Fresh (First time on a project)

```bash
# Clone the team repo to your computer (do this only once)
git clone https://github.com/daksha-1finance/Playwright-1Finance-Website.git

# Go inside the folder
cd Playwright-1Finance-Website
```

---

## Branches

```bash
# See which branch you are on
git branch

# Create a new branch and switch to it
git checkout -b tamal/feature-name

# Switch to an existing branch
git checkout main

# See all branches (including remote/GitHub branches)
git branch -a

# Delete a branch (after it is merged)
git branch -d tamal/feature-name
```

---

## Saving Changes

```bash
# See what files you changed
git status

# Stage all changed files
git add .

# Stage a specific file only
git add tests/calculators/myTest.spec.js

# Save staged files with a message
git commit -m "fixed FAQ locator in ITR page"

# See your recent commits
git log --oneline
```

---

## Syncing with GitHub

```bash
# Download latest changes from GitHub (always do this before starting work)
git pull

# Send your branch to GitHub
git push origin tamal/your-branch-name

# First time pushing a new branch
git push -u origin tamal/your-branch-name
```

---

## Checking Things

```bash
# See what branch you are on and what files changed
git status

# See the actual changes inside files
git diff

# See recent commits in one line each
git log --oneline

# See all remotes (GitHub repo addresses)
git remote -v
```

---

## Remotes (GitHub repo addresses)

```bash
# See which repos are connected
git remote -v

# Add a new remote (team repo)
git remote add daksha https://github.com/daksha-1finance/Playwright-1Finance-Website.git

# Remove a remote
git remote remove daksha
```

---

## Mistakes and Fixes

```bash
# Undo staged files (before commit)
git restore --staged .

# Undo changes in a file (before commit) - WARNING: changes are lost
git restore filename.js

# Change your last commit message (before pushing only)
git commit --amend -m "corrected message"
```

---

## Common Words Explained

| Word | Meaning |
|------|---------|
| `origin` | Your GitHub repo address (default name) |
| `main` | The main/production branch - don't touch directly |
| `branch` | Your own copy of the code to work on safely |
| `commit` | A saved snapshot of your changes |
| `push` | Send your commits to GitHub |
| `pull` | Download latest commits from GitHub |
| `clone` | Copy a GitHub repo to your computer |
| `PR` | Pull Request - asking senior to review and merge your branch |
| `merge` | Combining your branch into main |
| `remote` | A GitHub repo address saved in Git |

---

## The Mistake We Made Today (Learn from it)

**Wrong way:**
- Working in personal repo → trying to push to team repo → no shared history → PR fails

**Correct way:**
- Clone team repo → create branch → make changes → push → create PR

```bash
# Correct way - always start from the team repo
git clone https://github.com/daksha-1finance/Playwright-1Finance-Website.git
cd Playwright-1Finance-Website
git checkout -b tamal/your-task
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Get latest code | `git pull` |
| Create branch | `git checkout -b name` |
| See changes | `git status` |
| Stage all | `git add .` |
| Save changes | `git commit -m "message"` |
| Send to GitHub | `git push origin branch-name` |
| See branches | `git branch` |
| Switch branch | `git checkout branch-name` |
| See remotes | `git remote -v` |
