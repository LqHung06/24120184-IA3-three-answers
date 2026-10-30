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
