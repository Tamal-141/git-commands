# Playwright Learning — Challenges & Solutions

## Challenge 1: Two Playwright Config Files Conflicting
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

## Challenge 2: ENV Variable Not Working on Windows
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

## Challenge 3: networkidle Timing Out
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

## Challenge 4: domcontentloaded Not Working for React Re-renders
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

## Challenge 5: target="_blank" Links Failing URL Assertions
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

## Challenge 6: Test Timeout Too Short for Multiple Navigations
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

## Challenge 7: DOM Staleness After Filter Click
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
