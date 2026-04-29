# Git Learning — Challenges & Solutions

## Challenge 1: Unrelated Histories
**What happened:**
Tried to push from `C:\TTB` (personal repo - Tamal-141) to daksha repo.
Git rejected with: `fatal: refusing to merge unrelated histories`

**Why:**
Two completely different repos have completely different git histories.
Git cannot merge them — like trying to merge two separate books with different page numbers.

**Fix:**
Always work FROM the correct repo. We cloned daksha repo into `C:\DakshaRepo`
and pushed from there — same repo, same history, no conflict.

**Lesson:**
Before pushing, always confirm which repo you cloned from:
```
git remote -v
```

---

## Challenge 2: Wrong PAT Scope
**What happened:**
First push to daksha failed even with correct credentials.
Error: `refusing to allow a Personal Access Token to create or update workflow`

**Why:**
The Personal Access Token (PAT) was created without the `workflow` scope.
Without it, GitHub blocks pushes that touch certain files.

**Fix:**
Go to GitHub → Settings → Developer Settings → Personal Access Tokens
→ Edit token → check `repo` AND `workflow` scopes.

**Lesson:**
When creating a PAT always select both `repo` and `workflow` scopes.

---

## Challenge 3: git credential-manager erase Got Stuck
**What happened:**
Tried `git credential-manager erase` to clear saved credentials — terminal froze waiting for input.

**Fix:**
Press Ctrl+C to cancel. Then use this instead (4 lines, paste all at once):
```
git credential reject
protocol=https
host=github.com

```

**Lesson:**
`git credential reject` is safer and more reliable on Windows than credential-manager erase.

---

## Challenge 4: Non-Fast-Forward Rejection
**What happened:**
Pushed from TTB to daksha earlier (wrong repo). That created a `tamal/test-suite-fixes` branch
on remote with wrong history. When we tried pushing the correct branch from DakshaRepo:
Error: `Updates were rejected because the tip of your current branch is behind`

**Why:**
Remote branch already existed with different commits. Git cannot reconcile them.

**Fix:**
Delete the old wrong remote branch, then push fresh:
```
git push origin --delete tamal/test-suite-fixes
git push origin tamal/test-suite-fixes
```

**Lesson:**
If push is rejected with non-fast-forward, check if an old wrong branch exists on remote first.

---

## Key Git Commands

```bash
git remote -v                          # check which repo you are connected to
git branch                             # list all branches (* = current branch)
git checkout -b branchname             # create new branch and switch to it
git checkout branchname                # switch to existing branch
git add .                              # stage all changes
git commit -m "message"               # commit with message
git push origin branchname             # push YOUR branch (never push to main)
git push origin --delete branchname    # delete a remote branch
git pull origin branchname             # pull remote branch changes
git status                             # see what files are changed/untracked
```

## Branch Naming Convention
```
yourname/description-of-work
example: tamal/test-suite-fixes
```
Never use random names like `superman` — your team needs to know whose branch it is.

## Golden Rules
1. Never push directly to `main`
2. Always create a branch first
3. Always confirm which repo you are in: `git remote -v`
4. Never clone one repo inside another repo
5. After pushing your branch → create a PR → let senior review → they merge
