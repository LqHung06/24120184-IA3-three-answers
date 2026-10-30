# AI Interaction Log (AI-LOG.md)

- **Student ID**: 24120184
- **Repository**: https://github.com/LqHung06/24120184-IA3-three-answers
- **Course**: CSC13008 — Web Application Development
- **Assignment**: IA#3 — Three answers to one brief

---

## Entry 1 — Independent Review of the 3 Original Versions

- **Tool & Model**: Gemini 3.8 Flash (High) (new independent session)
- **Date**: 2026-10-31
- **Request**:
  Asked the assistant to independently review the three original files (`a.js`, `b.js`, `c.js`) against `brief.md` by reading code only (no running). Requested listing all violations of requirements or constraints with code line numbers, stating which test condition (`normal`, `slow`, `error`, `hang`, `empty`) or check exposes each flaw, and giving a verdict of `keep`, `fix`, or `reject` for each file.
- **Result**:
  The assistant provided a detailed review for all three files:
  - Version A: Verdict `reject` (or `fix`); identified that `products.forEach(async ...)` does not await the async callbacks, causing `#stock-value` to be written synchronously as `0 đ` under `normal`/`slow` conditions, and warned of unhandled promise rejections if stock requests fail.
  - Version B: Verdict `fix` (or `reject`); identified that `{ timeout: 5000 }` is not a standard Web API fetch option, causing fetch to hang indefinitely under `hang` mode; flagged that the catch block does not clear `#catalogue` on stock failure; and noted that clearing DOM at the start causes unnecessary flashing.
  - Version C: Verdict `keep`; confirmed compliance with all 6 brief requirements and all constraints.
- **Accepted**:
  - Recorded the raw, unedited assistant response verbatim into `ASSISTANT-REVIEW.md`.
  - The two primary technical findings (`forEach(async ...)` in A and `{ timeout: 5000 }` in B) directly align with the core suspect areas to be verified against the *Done means* table.
- **Changed / Rejected**:
  - Held off on accepting or rejecting secondary claims (such as the DOM cleanup claim on B, or slightly mismatched line number citations) until empirical verification across all 9 rows in Step 4.
  - Will document a rigorous cross-examination of each claim and any missed rows (such as Row 8 `// @ts-check`) in the verification table in Step 8.

---

## Entry 2 — Prompting Assistant to Fix Version A (Row 2)

- **Tool & Model**: Antigravity (Gemini)
- **Date**: 2026-10-31
- **Request**:
  Provided `brief.md`, `public/versions/a.js`, and the exact one-sentence directive from `VERDICTS.md`:
  > *"Row 2 (normal): stock value shows `0 đ` instead of `12.360.000 đ` because `products.forEach(async ...)` does not await before writing `#stock-value`; await all stock requests using `Promise.all` keeping them in parallel (do not use `await` inside a `for` loop), then write the formatted total; preserve `AbortSignal.timeout(5000)`, `textContent`, and touch no other file."*
  Enforced hard rules: only modify `public/versions/a.js`, change only lines 52–56, and do not touch any other files or restructure existing code.
- **Result**:
  The assistant proposed replacing lines 51–53 in `public/versions/a.js` with:
  ```javascript
  const stocks = await Promise.all(products.map((p) => getStock(p.id)));
  const total = products.reduce((sum, p, i) => sum + p.price * stocks[i], 0);
  stockValue.textContent = formatVnd(total);
  ```
- **Accepted**:
  - Applied the minimal diff to lines 51–53 in `public/versions/a.js`.
  - Verified scope with `git diff --stat`: strictly 1 file changed (2 insertions, 5 deletions).
  - Reran all test conditions on `?v=a`:
    - Row 2 (`normal`): stock value correctly updated to `12.360.000 đ`.
    - Row Parallel: all 8 `/api/stock/...` requests started concurrently at ~190 ms on Network waterfall.
    - Checked Rows 1, 3, 4, 5, 6, 7, 8, empty: confirmed no regression occurred; no other rows were broken.
  - Committed as `332af4a` ("fix A: wait for all stock requests before writing the total").
- **Changed / Rejected**:
  - Rejected any modification outside lines 51–53; rejected changes to error handling, template rendering, or variable naming.
  - Rejected the assistant's initial generated draft of the `## Results after fixes` markdown section due to corrupted text formatting, truncated sentences, and missing row prefixes; subsequently rewrote the section cleanly in commit `5263dbc`.

---

## Entry 3 — Prompting Assistant to Fix Version B (Rows 6 and 8)

- **Tool & Model**: Antigravity (Gemini)
- **Date**: 2026-10-31
- **Request**:
  Provided `brief.md`, `public/versions/b.js`, and the exact one-sentence directive from `VERDICTS.md`:
  > *"Rows 6 and 8 fail: under `hang` mode `/api/products` hangs for 30 seconds instead of aborting after 5 seconds, and `// @ts-check` reports that `timeout` does not exist in `RequestInit` at line 16; replace `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`; preserve `Promise.all` parallel requests for stock, `res.ok` check, `textContent`, and touch no other file."*
  Enforced hard rules: only modify line 16 of `public/versions/b.js`, preserve `Promise.all` parallel fetching, and touch no other file.
- **Result**:
  The assistant proposed replacing line 16 in `public/versions/b.js`:
  ```javascript
  const response = await fetch('/api/products', { signal: AbortSignal.timeout(5000) });
  ```
  replacing `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`.
- **Accepted**:
  - Applied the exact 1-line diff to line 16 of `public/versions/b.js`.
  - Verified scope with `git diff --stat`: strictly 1 file changed (1 insertion, 1 deletion).
  - Reran all test conditions on `?v=b`:
    - Row 6 (`hang`): `/api/products` aborted after exactly 5.00 s (`(canceled)` in Network tab), displaying `Could not load products.` and showing the Retry button.
    - Row 8 (`@ts-check`): TypeScript error disappeared; 0 red squiggly underlines.
    - Row 2 (`normal`) & Parallel: stock value remained `12.360.000 đ`, all 8 stock requests ran concurrently.
    - Checked Rows 1, 3, 4, 5, 7, empty: confirmed no regression occurred; no other rows were broken.
  - Committed as `c7c1f61` ("fix B: real 5 s timeout with AbortSignal").
- **Changed / Rejected**:
  - Rejected any changes to `public/versions/c.js`, helper functions, or DOM event listeners in `b.js`.
  - Rejected the commit message placeholder string previously used in documentation headers, enforcing the exact short SHA `c7c1f61` in the final verdict records.
