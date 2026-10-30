# VERDICTS

## Predictions (written from reading only, before running anything)

### Version A
- Prediction: In a.js (lines 52–56), the stock total is built with
  `products.forEach(async (p) => { ... })`, but `stockValue.textContent`
  is set to `formatVnd(total)` right after it. Since `forEach` does not
  wait for the async callbacks to finish, `total` is likely to be written
  before any of the stock requests come back, so the page would probably
  show `0 đ` instead of `12.360.000 đ`. I predict Version A will fail
  Row 2.
- Expected to fail: Row 2 — total written at line 56 before the stock
  requests in lines 52–55 have finished.
- Expected to pass: Row 1 — cards are rendered in API order with
  formatted prices (lines 26–35, 46–48); Row 3 — names use `textContent`
  (line 30), so `Scarf <i>limited</i>` should show as text; Row 4 —
  `Loading products…` is shown while waiting (line 44) and cleared
  after the cards appear (line 48); Row 5 — the catch block (lines
  57–60) clears the list, shows `Could not load products.` and shows
  Retry, and Retry calls `load()` again; Row 6 — `AbortSignal.timeout(5000)`
  is used (line 14), so the request should give up after 5 seconds;
  Row 7 — I saw no `console.error` and no obvious uncaught error in
  rows 1–6; Row 8 — I saw no invalid fetch option or obvious type
  error, so `// @ts-check` should show no red underline.
- Row empty: I expect a pass — with `[]`, the status should read
  `No products yet.` (line 48) and the total should be `0 đ`.
- Parallel: I expect a pass — the stock requests are started inside
  `forEach` (lines 52–55), so they should start together, even though
  the total itself is written too early.
- Tentative verdict: fix, decided by Row 2. To be confirmed by running.

### Version B
- Prediction: In b.js (line 16), `loadProducts()` calls
  `fetch('/api/products', { timeout: 5000 })`. Under the browser's
  standard Web API, `fetch` does not accept a `timeout` option; the
  timeout has to be given as `signal: AbortSignal.timeout(5000)`. So in
  hang mode, where the server does not answer for 30 seconds, the page
  probably will not give up after 5 seconds as required, but will keep
  waiting much longer. I also expect `// @ts-check` to flag a type
  error on the `timeout` property. I predict Version B will fail
  Row 6 and Row 8.
- Expected to fail: Row 6 — `{ timeout: 5000 }` (line 16) is silently
  ignored, so the page should still be loading after 5 seconds in hang
  mode; Row 8 — `// @ts-check` should underline `timeout`, because it
  is not a property of `RequestInit`.
- Expected to pass: Row 1 — cards are rendered in API order with
  formatted prices (lines 28–34, 46–48); Row 2 — stock is loaded with
  `Promise.all` (line 51) and the total is calculated after it, so it
  should be `12.360.000 đ`; Row 3 — names use `textContent` (line 31);
  Row 4 — `Loading products…` is shown while waiting (line 44) and
  cleared after the cards appear (line 48); Row 5 — the catch block
  (lines 54–56) shows `Could not load products.` and Retry, and the
  list is cleared at the start of `showCatalogue()` (line 42); Row 7 —
  I saw no `console.error` and no obvious uncaught error in rows 1–6.
- Row empty: I expect a pass — with `[]`, the status should read
  `No products yet.` (line 48) and `Promise.all([])` should give a
  total of `0 đ`.
- Parallel: I expect a pass — all stock requests are started together
  with `Promise.all` (line 51).
- Tentative verdict: fix, decided by Row 6. To be confirmed by running.

### Version C
- Prediction: In c.js, line 16 already uses the correct
  `signal: AbortSignal.timeout(5000)` for the 5-second limit, the code
  checks `res.ok`, and the catch block shows the error and the Retry
  button. The stock calculation in lines 54–57 uses `Promise.all` to
  run the stock requests in parallel and waits for all of them before
  writing the total, and product names are set with `textContent`.
  I predict Version C meets the brief well and is likely to pass
  the test rows.
- Expected to fail: None that I spotted on reading.
- Expected to pass: Row 1 — cards are rendered in API order with
  formatted prices (lines 28–38, 49–51); Row 2 — stock is loaded with
  `Promise.all` (line 54) and the total is written only after it
  (line 57), so it should be `12.360.000 đ`; Row 3 — names use
  `textContent` (line 33); Row 4 — `Loading products…` is shown while
  waiting (line 47) and cleared after the cards appear (line 51);
  Row 5 — the catch block (lines 58–61) clears the list, shows
  `Could not load products.` and shows Retry, and Retry calls
  `start()` again; Row 6 — `AbortSignal.timeout(5000)` is used
  (line 16); Row 7 — I saw no `console.error` and no obvious uncaught
  error in rows 1–6; Row 8 — I saw no invalid fetch option or obvious
  type error, so `// @ts-check` should show no red underline.
- Row empty: I expect a pass — with `[]`, the status should read
  `No products yet.` (line 51) and the total should be `0 đ`.
- Parallel: I expect a pass — all stock requests are started together
  with `Promise.all` (line 54).
- Tentative verdict: keep, subject to the runs confirming it.
  To be confirmed by running.

---

## Results before fixes (run against server.mjs under all 5 test conditions)

### Version A
- Prediction check: My prediction was confirmed. Version A failed Row 2 because `forEach` did not wait for stock requests before writing the stock value.
- Row 1: PASS. Under `normal` mode, 8 cards rendered in API order, prices formatted like `180.000 đ`, status line cleared.
- Row 2: FAIL. Under `normal` mode, `Stock value:` displays `0 đ` instead of expected `12.360.000 đ` (evidence: `evidence/a-row2.png`).
- Row 3: PASS. Scarf name reads `Scarf <i>limited</i>` with literal angle brackets visible and text not in italics.
- Row 4: PASS. Under `slow` mode, `Loading products…` shown for about 3.00 s (server log: `3003 ms`), then 8 cards appeared and status cleared.
- Row 5: PASS. Under `error` mode, shows `Could not load products.` and Retry button is visible with list cleared; switching to `normal` and clicking Retry reloads 8 cards and clears error message and button.
- Row 6: PASS. Under `hang` mode, browser aborted request after 5.00 s (server log: `the browser gave up after 5006 ms`), status shows `Could not load products.` and Retry is shown.
- Row 7: PASS. Console remained completely free of `Uncaught …` errors throughout rows 1–6 (only browser's own red `GET … 500` under error mode).
- Row 8: PASS. Opened folder in VS Code, added `// @ts-check` at line 1 of `a.js`: 0 red squiggly lines (0 problems). Line removed immediately afterwards.
- Row empty: PASS. Under `empty` mode, status displays `No products yet.`, `Stock value:` displays `0 đ`, catalogue is empty.
- Row Parallel: PASS. Under `normal` mode, all 8 `/api/stock/…` requests started at the exact same timestamp (~60 ms on Network waterfall, no staircase).
- Code responsible: `public/versions/a.js`, lines 52–56 (`products.forEach(async (p) => { ... })` does not await async callbacks before executing `stockValue.textContent = formatVnd(total)` at line 56).
- Verdict: **fix** (decided by Row 2).
- One sentence to assistant: *"Row 2 (normal): stock value shows `0 đ` instead of `12.360.000 đ` because `products.forEach(async ...)` does not await before writing `#stock-value`; await all stock requests using `Promise.all` keeping them in parallel (do not use `await` inside a `for` loop), then write the formatted total; preserve `AbortSignal.timeout(5000)`, `textContent`, and touch no other file."*

### Version B
- Prediction check: My prediction was confirmed. Version B failed Row 6 because `{ timeout: 5000 }` was silently ignored, and failed Row 8 because TypeScript flagged `timeout` as an invalid option.
- Row 1: PASS. Under `normal` mode, 8 cards rendered in API order, prices formatted like `180.000 đ`, status line cleared.
- Row 2: PASS. Under `normal` mode, `Stock value:` correctly displays `12.360.000 đ` after stock requests complete via `Promise.all`.
- Row 3: PASS. Scarf name reads `Scarf <i>limited</i>` with literal angle brackets visible and text not in italics.
- Row 4: PASS. Under `slow` mode, `Loading products…` shown for 3.00 s (server log: `3000 ms`), then 8 cards appeared and status cleared.
- Row 5: PASS. Under `error` mode, shows `Could not load products.` and Retry button is visible; switching to `normal` and clicking Retry reloads 8 cards.
- Row 6: FAIL. Under `hang` mode, `/api/products` hung for 30.02 s (server log: `30011 ms` and `30016 ms` before 504), failing to give up after 5 seconds because `{ timeout: 5000 }` is not a valid fetch option in browser Web API (evidence: `evidence/b-row6.png`).
- Row 7: PASS. Console remained free of `Uncaught …` errors throughout rows 1–6 (only browser's own red `GET … 504`/`500` network status).
- Row 8: FAIL. Opened folder in VS Code, added `// @ts-check` at line 1 of `b.js`: red squiggly line under `{ timeout: 5000 }` at line 17 with error `Object literal may only specify known properties, and 'timeout' does not exist in type 'RequestInit'` (evidence: `evidence/b-row8.png`). Line removed immediately afterwards.
- Row empty: PASS. Under `empty` mode, status displays `No products yet.`, `Stock value:` displays `0 đ`.
- Row Parallel: PASS. Under `normal` mode, all 8 `/api/stock/…` requests started simultaneously via `Promise.all`.
- Code responsible: `public/versions/b.js`, line 16 (or line 17 in VS Code: `fetch('/api/products', { timeout: 5000 })` using non-existent `timeout` property instead of `signal: AbortSignal.timeout(5000)`).
- Verdict: **fix** (decided by Row 6 and Row 8).
- One sentence to assistant: *"Rows 6 and 8 fail: under `hang` mode `/api/products` hangs for 30 seconds instead of aborting after 5 seconds, and `// @ts-check` reports that `timeout` does not exist in `RequestInit` at line 16; replace `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`; preserve `Promise.all` parallel requests for stock, `res.ok` check, `textContent`, and touch no other file."*

### Version C
- Prediction check: My prediction was confirmed. Version C passed all rows under all conditions.
- Row 1: PASS. Under `normal` mode, 8 cards rendered in API order, prices formatted like `180.000 đ`, status line cleared.
- Row 2: PASS. Under `normal` mode, `Stock value:` correctly displays `12.360.000 đ` after stock requests complete via `Promise.all`.
- Row 3: PASS. Scarf name reads `Scarf <i>limited</i>` with literal angle brackets visible and text not in italics.
- Row 4: PASS. Under `slow` mode, `Loading products…` shown for about 3.00 s, then 8 cards appeared and status cleared.
- Row 5: PASS. Under `error` mode, shows `Could not load products.` + Retry; Retry under `normal` reloads 8 cards.
- Row 6: PASS. Under `hang` mode, the page gave up right at 5.00 s (server log: `the browser gave up after 5015 ms` and `4999 ms`), status shows `Could not load products.` and Retry button appears.
- Row 7: PASS. Console clean, no `Uncaught …` errors throughout rows 1–6.
- Row 8: PASS. Added `// @ts-check` at line 1 of `c.js`: 0 red underlines (0 problems). Line removed immediately afterwards.
- Row empty: PASS. Under `empty` mode, status displays `No products yet.`, `Stock value:` displays `0 đ`.
- Row Parallel: PASS. All 8 `/api/stock/…` requests start simultaneously via `Promise.all`.
- Code responsible: None (no failing rows).
- Verdict: **keep** (passes rows 1–8, empty, and parallel check).
- One sentence to assistant: *"No fixes required: passes rows 1–8, empty, and parallel check. Keep untouched."*


## Results after fixes (run against server.mjs under all 5 test conditions)

### Version A (fixed in commit 332af4a)
- Fix applied: In `public/versions/a.js`, replaced `products.forEach(async ...)` at lines 51–53 with `const stocks = await Promise.all(products.map((p) => getStock(p.id)));` and calculated `total` using `products.reduce(...)` before writing to `stockValue.textContent`.
- Row 1: PASS. Under `normal` mode, 8 cards rendered in API order with formatted prices (e.g. `180.000 đ`), status line cleared.
- Row 2: PASS. Under `normal` mode, `Stock value:` now correctly displays `12.360.000 đ` instead of `0 đ` after all stock requests complete.
- Row 3: PASS. Scarf name displays literal `Scarf <i>limited</i>` with angle brackets visible and not in italics (via `textContent`).
- Row 4: PASS. Under `slow` mode, `Loading products…` shows for 3 seconds while waiting, then 8 cards appear and status line is cleared.
- Row 5: PASS. Under `error` mode, shows `Could not load products.` with catalogue cleared and Retry button shown; clicking Retry under `normal` reloads 8 cards.
- Row 6: PASS. Under `hang` mode, browser aborts after 5.00 s, status shows `Could not load products.` and Retry button appears.
- Row 7: PASS. Console clean, 0 uncaught errors throughout all conditions.
- Row 8: PASS. Opened in VS Code with `// @ts-check` at line 1: 0 red squiggly underlines (0 errors).
- Row empty: PASS. Under `empty` mode, status displays `No products yet.`, `Stock value:` displays `0 đ`, catalogue is empty.
- Row Parallel: PASS. Under `normal` mode, all 8 `/api/stock/…` requests started simultaneously at the same timestamp on Network waterfall (~190 ms, no staircase).
- Final status: **10/10 PASS**.

### Version B (fixed in commit c7c1f61)
- Fix applied: In `public/versions/b.js` line 16, replaced `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`.
- Row 1: PASS. Under `normal` mode, 8 cards rendered in API order with formatted prices, status line cleared.
- Row 2: PASS. Under `normal` mode, `Stock value:` correctly displays `12.360.000 đ` via `Promise.all`.
- Row 3: PASS. Scarf name displays literal `Scarf <i>limited</i>` with angle brackets visible and not in italics (via `textContent`).
- Row 4: PASS. Under `slow` mode, `Loading products…` shows for 3 seconds while waiting, then 8 cards appear and status line is cleared.
- Row 5: PASS. Under `error` mode, shows `Could not load products.` with catalogue cleared and Retry button shown; clicking Retry under `normal` reloads 8 cards.
- Row 6: PASS. Under `hang` mode, `/api/products` aborted after exactly 5.00 s (canceled in Network panel, time: `5.00 s`), status shows `Could not load products.` and Retry button appears.
- Row 7: PASS. Console clean, 0 uncaught errors throughout all conditions.
- Row 8: PASS. Opened in VS Code with `// @ts-check`: red squiggly underline under `fetch` option is completely gone (0 errors).
- Row empty: PASS. Under `empty` mode, status displays `No products yet.`, `Stock value:` displays `0 đ`, catalogue is empty.
- Row Parallel: PASS. Under `normal` mode, all 8 `/api/stock/…` requests start simultaneously via `Promise.all` (starting together at timeline mark, no staircase).
- Final status: **10/10 PASS**.

### Version C (untouched baseline — verdict: keep)
- Row 1: PASS. Under `normal` mode, 8 cards rendered in API order with formatted prices, status line cleared.
- Row 2: PASS. Under `normal` mode, `Stock value:` correctly displays `12.360.000 đ` via `Promise.all`.
- Row 3: PASS. Scarf name displays literal `Scarf <i>limited</i>` with angle brackets visible and not in italics (via `textContent`).
- Row 4: PASS. Under `slow` mode, `Loading products…` shows for 3 seconds while waiting, then 8 cards appear and status line is cleared.
- Row 5: PASS. Under `error` mode, shows `Could not load products.` with catalogue cleared and Retry button shown; clicking Retry under `normal` reloads 8 cards.
- Row 6: PASS. Under `hang` mode, browser aborts after 5.00 s, status shows `Could not load products.` and Retry button appears.
- Row 7: PASS. Console clean, 0 uncaught errors throughout all conditions.
- Row 8: PASS. Opened in VS Code with `// @ts-check`: 0 red squiggly underlines (0 errors).
- Row empty: PASS. Under `empty` mode, status displays `No products yet.`, `Stock value:` displays `0 đ`, catalogue is empty.
- Row Parallel: PASS. Under `normal` mode, all 8 `/api/stock/…` requests start simultaneously via `Promise.all`.
- Final status: **10/10 PASS**.
