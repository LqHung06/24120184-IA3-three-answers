# Self-Assessment Report: IA#3 — Three Answers to One Brief

- **Student ID**: 24120184
- **Repository**: https://github.com/LqHung06/24120184-IA3-three-answers
- **Branch**: `main` (Public repository)
- **Course**: CSC13008 — Web Application Development
- **Session**: Session 5 — "Code That Waits"
- **Claimed Total Score**: **98 / 100**

---

## 1. Rubric Score Table

*Aligned directly with the 6 criteria defined in the official assignment rubric (`IA3-rubric-three-answers.pdf`).*

| # | Criterion | Max Points | Claimed Points | Primary Evidence / Commits |
|---|---|---|---|---|
| **1** | **Verdicts and evidence — `VERDICTS.md`, `evidence/`** | 30 | 30 | `VERDICTS.md` (Commits `ea1f514`, `9fae4c2`): All 27 rubric rows + 3 parallel rows present with what was seen, responsible code lines, verdicts, and one-sentence prompts. Screenshots in `evidence/`: `a-row2.png`, `b-row6.png`, `b-row8.png` (retaken 31/10 with hover tooltip and Problems panel), `a-stockfail.png`, `b-stockfail.png`. |
| **2** | **After your fixes, every version passes every row** | 25 | 25 | `public/versions/a.js` (Commit `332af4a`): Fixed Row 2 via `Promise.all` parallel fetching (`12.360.000 đ`). `public/versions/b.js` (Commit `c7c1f61`): Fixed Rows 6 & 8 with `AbortSignal.timeout(5000)`. `c.js` untouched (`keep`). All 3 versions 10/10 PASS (Commit `5263dbc`). |
| **3** | **The assistant's review, checked — `ASSISTANT-REVIEW.md`** | 15 | 14 | `ASSISTANT-REVIEW.md` (Commits `3afee41`, `629697d`, plus the 31/10 addendum): Unedited raw Vietnamese review from Gemini 3.8 Flash High; Table 1 checking every claim against actual row runs; Table 2 identifying that the assistant missed Row 8 (`// @ts-check`); the addendum corrects one wrong Table 1 verdict (B catch claim, False → True, caught by `b-stockfail.png`) and checks one claim Table 1 missed entirely (Retry never appears under hang — Inexact, caught by `b-row6.png`). One point deducted because the original Table 1 contained that wrong verdict. |
| **4** | **CSR vs SSR — `CSR-SSR.md`** | 10 | 10 | `CSR-SSR.md` (Commit `56cfcad`, plus the 31/10 revision): All 5 questions answered from direct observation: DOM comparison, `escapeHtml` vs `textContent` entity escaping, disabling JS, request counts counted in the Network tab (CSR **15** vs SSR **3**, or 14 vs 2 excluding the automatic `favicon.ico` request), and a single-sentence architectural rule. Evidence: `csr-requests.png`, `ssr-requests.png`, `csr-js-off.png`, `ssr-js-off.png`, `ssr-view-source.png`. |
| **5** | **`AI-LOG.md`** | 10 | 9 | `AI-LOG.md` (Commits `2cc0f80`, `79b72d7`, `fec14aa`, plus the 31/10 addendum): 3 entries plus an addendum naming tool/model, exact request, code result, and specific rejected items, including the corrupted `Results after fixes` draft and the corrected Table 1 verdicts. One point deducted because Entries 2–3 and the addendum were written after the work they describe, not while it happened. |
| **6** | **Scope and commit history** | 10 | 10 | Clean linear git history (20 commits): starter commit first (`0de5088`), predictions before results, one commit per fix, only `a.js` and `b.js` modified among starter files (`git diff 0de5088 HEAD --stat`), 0 leftover `// @ts-check`. |
| **TOTAL** | | **100** | **98** | **File name: `24120184_98.zip`** |

---

## 2. Detailed Justification by Criterion

### Criterion 1: Verdicts and evidence (30 / 30 points)
- **Predictions**: Committed in `ea1f514` before any server execution. Accurately predicted Row 2 failure in A (`forEach` not awaiting), Row 6 & 8 failures in B (`{ timeout: 5000 }` not a valid Web API option), and full pass for C.
- **Empirical Results**: Committed in `9fae4c2`. All 27 rubric rows (9 rows × 3 versions) plus 3 parallel waterfall checks are documented with exact observations and server log timings (`3003 ms` under `slow`, `30011 ms` / `30016 ms` under `hang`).
- **Evidence Screenshots**:
  - `evidence/a-row2.png`: Shows `#stock-value` displaying `0 đ` instead of `12.360.000 đ`.
  - `evidence/b-row6.png`: Shows request hanging past 30 seconds before 504 gateway timeout, then `Could not load products.` and the Retry button.
  - `evidence/b-row8.png` (retaken 31/10): Shows the hover tooltip `Object literal may only specify known properties, and 'timeout' does not exist in type 'RequestInit'. ts(2353)` under `{ timeout: 5000 }`, with the Problems panel listing 1 problem at `[Ln 17, Col 5]`.
  - `evidence/a-stockfail.png`: With stock requests blocked, 8 × `Uncaught (in promise) TypeError: Failed to fetch` from `a.js:20` / `a.js:53`, stock stays `0 đ`.
  - `evidence/b-stockfail.png`: With stock requests blocked, all 8 cards stay visible with `Could not load products.` and the Retry button.
- **Actionable Sentences**: Each failing version has a specific one-sentence directive citing the row, the evidence, and the constraints to keep.

### Criterion 2: After your fixes, every version passes every row (25 / 25 points)
- **Version A**: Fixed in commit `332af4a`. Replaced lines 51–53 with `await Promise.all(products.map(p => getStock(p.id)))` and `products.reduce(...)`. Total is `12.360.000 đ`. All 8 stock requests start concurrently at ~190 ms (verified via Network waterfall, no staircase).
- **Version B**: Fixed in commit `c7c1f61`. Replaced line 16 `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`. Aborts after exactly 5.00 s in `hang` mode (Row 6 PASS); 0 red squiggly underlines under `// @ts-check` (Row 8 PASS).
- **Version C**: Left untouched baseline (`keep`).
- **Post-Fix Verification**: All 3 versions verified 10/10 PASS across all 5 test conditions, documented cleanly in `VERDICTS.md` (Commit `5263dbc`).

### Criterion 3: The assistant's review, checked (14 / 15 points)
- **Raw Answer**: Verbatim Vietnamese response from Gemini 3.8 Flash High committed in `3afee41` without edits or summaries.
- **Table 1 (Claims vs Runs)**: Checks all assistant claims against empirical test rows. Confirms the two main flaws in A and B; points out inexact line numbers cited by the assistant (lines 48–52 vs 50–56 in A, line 13 vs 16 in B); and disproves the assistant's claim regarding DOM cleanup in B under standard `error` mode.
- **Table 2 (Failing Rows Missed)**: Identifies that the assistant's reading missed Row 8 (`// @ts-check`), which was caught by running VS Code with `jsconfig.json` (evidence: `evidence/b-row8.png`).
- **Addendum (31/10)**: Extra stock-failure runs corrected Table 1 in two places — the B catch claim changes from False to True (`b-stockfail.png`), the A unhandled-rejection claim changes from theoretical to confirmed by run (`a-stockfail.png`) — and checked one claim Table 1 had missed entirely: the review's "Retry never appears" under hang is Inexact, because Retry appears after 30 seconds when the server returns 504 (`b-row6.png`). The addendum also notes the review's verdicts were two-headed (`reject (or fix)`, `fix (or reject)`), so they never commit to a single verdict.
- **Why 14, not 15**: The original Table 1 contained one wrong verdict (B catch, marked False) that was only caught and corrected by the supplementary runs on 31/10. The correction is documented honestly, but the original error is on record.

### Criterion 4: CSR vs SSR (10 / 10 points)
- **Observations**: `CSR-SSR.md` (Commit `56cfcad`, revised 31/10) thoroughly answers all 5 questions from `README.md`.
- **Security & Escaping**: Compares server-side `escapeHtml()` in `server.mjs` (lines 67–71) with browser `textContent`, explaining how both treat data as plain text to prevent XSS and prevent `<i>limited</i>` from rendering as italics. Confirmed in `evidence/ssr-view-source.png`, line 48: `<h2 class="name">Scarf &lt;i&gt;limited&lt;/i&gt;</h2>`.
- **Metrics**: Contrasts **15** requests for CSR against **3** requests for SSR, counted in the Network tab (14 vs 2 excluding the automatic `favicon.ico` 404), with finish times of 711 ms vs 63 ms in the counted runs. Explaining First Contentful Paint advantages for slow devices.
- **Rule of Thumb**: Concludes with strictly one sentence defining when to use SSR vs CSR.

### Criterion 5: AI-LOG.md (9 / 10 points)
- **Log Structure**: Commits `2cc0f80`, `79b72d7`, and `fec14aa`, plus the 31/10 addendum. Contains 3 entries corresponding to each assistant engagement (Review, Fix A, Fix B), plus a supplementary entry for the extra runs and corrections.
- **Honest Records**: Each entry details the prompt, the code proposed, verification that re-running the full table broke no other rows, and specific items in `Changed / Rejected` (including rejecting scope creep outside lines 51–53 in A, rejecting the corrupted markdown draft of the after-fix results, and correcting the two wrong Table 1 verdicts).
- **Why 9, not 10**: Entries 2–3 and the addendum were written after the work they describe rather than while it happened. The entries are accurate about what was done and when, but they were not contemporaneous logs.

### Criterion 6: Scope and commit history (10 / 10 points)
- **History**: Linear 20-commit history starting with untouched starter (`0de5088`).
- **Scope**: `git diff 0de5088 HEAD --stat` confirms only `public/versions/a.js` (7 lines) and `public/versions/b.js` (2 lines) were touched among starter code. All other starter files (`server.mjs`, `panel.js`, `index.html`, `style.css`, `c.js`, `jsconfig.json`) remain identical to starter.
- **Code Cleanliness**: 0 occurrences of `// @ts-check` left in `public/versions/*.js`.

---

## 3. What I Did Not Manage

1. **AI-LOG entries written after the work, not during it**:
   The AI-LOG entries for the fixes (Entries 2–3) and the 31/10 addendum were written after the work they describe rather than while it happened. The entries are accurate about what was done and when, but they were not contemporaneous logs.
2. **Three review claims checked late, not in the original verification**:
   The claims about unhandled rejections in A, the catch not clearing cards in B, and "Retry never appears" under hang were only checked by running extra stock-failure tests on 31/10, not during the original Step 8 verification. The original Table 1 verdict on the B catch claim (False) was wrong and is corrected in the addendum, not in the original table.
3. **Earlier self-assessment mistakes, now corrected**:
   The previous self-assessment (commit `f19aa54`) claimed a "14-commit history" when the history already had 15 commits, and claimed every review claim had been checked against a row/run when the B catch verdict had only been checked by reading code. Both are corrected in this report.
4. **After-fix rerun numbers are generic**:
   The "Results after fixes" section in `VERDICTS.md` gives generic timings ("3 s", "5.00 s") without server-log milliseconds for the after-fix runs, and no separate after-fix screenshots were taken. The after-fix passes were verified by re-running the full table, but the written evidence for those runs is thinner than for the before-fix runs.
5. **Assistant line discrepancies preserved verbatim**:
   In `ASSISTANT-REVIEW.md`, the assistant's initial review cited slightly shifted line numbers (citing lines 48–52 instead of lines 50–56 in `a.js`, and line 13 instead of line 16 in `b.js`). Rather than altering or smoothing the assistant's raw review text, I preserved the assistant's raw text verbatim as required by the rubric and documented the exact line discrepancies in Table 1 of the verification section.
