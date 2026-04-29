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

## Challenge 5: Two Playwright Config Files Conflicting
**What happened:**
All tests failed with `Cannot navigate to invalid URL` after switching to relative paths.

**Why:**
Two config files existed: `playwright.config.ts` AND `playwright.config.js`
The `.ts` file takes priority over `.js` — it had `baseURL` commented out,
so Playwright had no base URL and relative paths like `/fpc` became invalid.

**Fix:**
Deleted `playwright.config.ts` — only keep one config file.

**Lesson:**
Never have both `.ts` and `.js` versions of the same config file. Always check:
```
ls playwright.config.*
```

---

## Challenge 6: ENV Variable Not Working on Windows
**What happened:**
`ENV=qa playwright test` worked on Mac/Linux but not on Windows.

**Why:**
Windows does not support inline environment variable syntax (`KEY=value command`).

**Fix:**
Install `cross-env` package and use npm scripts:
```
npm install cross-env
```
```json
"scripts": {
  "test:qa":  "cross-env ENV=qa playwright test",
  "test:uat": "cross-env ENV=uat playwright test"
}
```
Then run: `npm run test:qa`

**Lesson:**
Always use `cross-env` when setting env variables in npm scripts for Windows compatibility.

---

## Challenge 7: networkidle Timing Out
**What happened:**
Added `waitForLoadState('networkidle')` after filter button click — tests started timing out.

**Why:**
`networkidle` waits for 500ms of zero network activity.
The FPC page has continuous background requests (analytics, trackers) that never stop.
So networkidle NEVER fires → timeout.

**Fix:**
Removed networkidle. Used `waitForTimeout(1000)` instead for React re-renders.

**Lesson:**
`networkidle` only works on simple pages with no analytics/trackers.
For React re-renders use `waitForTimeout()` or `expect(locator).toBeVisible()`.

---

## Challenge 8: domcontentloaded Not Working for React Re-renders
**What happened:**
Used `waitForLoadState('domcontentloaded')` after filter button click — DOM staleness still happened.

**Why:**
`domcontentloaded` fires when the HTML page first loads — not when React updates the DOM.
Filter buttons trigger client-side React re-renders, not a new page load.

**Fix:**
Use `waitForTimeout(1000)` to let React finish re-rendering.

**Lesson:**
| Event | When it fires |
|-------|-------------|
| `domcontentloaded` | Full page HTML loaded |
| `networkidle` | No network requests for 500ms |
| `waitForTimeout(ms)` | Just waits — good for React re-renders |

---

## Challenge 9: target="_blank" Links Failing URL Assertions
**What happened:**
Clicking Loan Planning and Will & Estate Planning links → `page.url()` returned wrong URL.
Expected `/loan-planning` but received the FPC page URL (or empty string).

**Why:**
These links have `target="_blank"` — they open in a NEW TAB.
The original `page` object stays on the FPC page, URL never changes.

**Fix:**
Check the `href` attribute directly instead of clicking and asserting URL:
```js
const href = await link.getAttribute('href');
expect(href).toMatch(/loan-planning/);
```

**Lesson:**
If `toHaveURL` receives the same URL you started on, the link probably opens in a new tab.
For new-tab links, check `href` attribute instead of navigating.

---

## Challenge 10: Test Timeout Too Short for Multiple Navigations
**What happened:**
TC6 visits 3 different pages on QA server (slow). Hit 30s timeout before finishing.

**Fix:**
Use `test.setTimeout()` INSIDE the test body — not in the test signature:
```js
test('TC6: Navigation links', async ({ page }) => {
    test.setTimeout(90000); // ← inside the test
    // ...
});
```

**Lesson:**
`test('name', { timeout: 90000 }, async...)` syntax does NOT always work.
`test.setTimeout(90000)` inside the test body is guaranteed to work.

---

## Challenge 11: DOM Staleness After Filter Click
**What happened:**
After clicking a location filter button, `scrollIntoViewIfNeeded()` threw:
`Element is not attached to the DOM`

**Why:**
Filter button triggers React re-render. Between `count()` and `scrollIntoViewIfNeeded()`,
React swaps out the old DOM elements with new ones. Old reference becomes invalid.

**Fix:**
Wait after filter click to let React finish:
```js
await fpc.filterByLocation('Delhi');
await page.waitForTimeout(1000);
const eventCount = await fpc.workshopCards.count();
```

**Lesson:**
React re-renders happen asynchronously. Always add a wait after actions that trigger re-renders
before trying to interact with the updated elements.

---

## Key Git Commands Learned

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
