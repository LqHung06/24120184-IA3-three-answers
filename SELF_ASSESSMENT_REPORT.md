# Self-Assessment Report: IA#3 — Three Answers to One Brief

- **Student ID**: 24120184
- **Repository**: https://github.com/LqHung06/24120184-IA3-three-answers
- **Branch**: `main` (Public repository)
- **Course**: CSC13008 — Web Application Development
- **Session**: Session 5 — "Code That Waits"
- **Claimed Total Score**: **100 / 100**

---

## 1. Rubric Score Table

*Aligned directly with the 6 criteria defined in the official assignment rubric (`IA3-rubric-three-answers.pdf`).*

| # | Criterion | Max Points | Claimed Points | Primary Evidence / Commits |
|---|---|---|---|---|
| **1** | **Verdicts and evidence — `VERDICTS.md`, `evidence/`** | 30 | 30 | `VERDICTS.md` (Commits `ea1f514`, `9fae4c2`): All 27 rubric rows + 3 parallel rows present with what was seen, responsible code lines, verdicts, and one-sentence prompts. Screenshots in `evidence/`: `a-row2.png`, `b-row6.png`, `b-row8.png`. |
| **2** | **After your fixes, every version passes every row** | 25 | 25 | `public/versions/a.js` (Commit `332af4a`): Fixed Row 2 via `Promise.all` parallel fetching (`12.360.000 đ`). `public/versions/b.js` (Commit `c7c1f61`): Fixed Rows 6 & 8 with `AbortSignal.timeout(5000)`. `c.js` untouched (`keep`). All 3 versions 10/10 PASS (Commit `5263dbc`). |
| **3** | **The assistant's review, checked — `ASSISTANT-REVIEW.md`** | 15 | 15 | `ASSISTANT-REVIEW.md` (Commits `3afee41`, `629697d`): Unedited raw Vietnamese review from Gemini 3.8 Flash High; Table 1 checking every claim against actual row runs; Table 2 identifying that the assistant missed Row 8 (`// @ts-check`). |
| **4** | **CSR vs SSR — `CSR-SSR.md`** | 10 | 10 | `CSR-SSR.md` (Commit `56cfcad`): All 5 questions answered from direct observation: DOM comparison, `escapeHtml` vs `textContent` entity escaping, disabling JS, request counts (13–14 vs 2), and single-sentence architectural rule. |
| **5** | **`AI-LOG.md`** | 10 | 10 | `AI-LOG.md` (Commits `2cc0f80`, `79b72d7`, `fec14aa`): 3 discrete entries (Step 3 Review, Fix A, Fix B) naming tool/model, exact one-sentence request, code result, confirmation of 0 regressions, and specific rejected items. |
| **6** | **Scope and commit history** | 10 | 10 | Clean linear git history (14 commits): starter commit first (`0de5088`), predictions before results, one commit per fix, only `a.js` and `b.js` modified among starter files (`git diff 0de5088 HEAD --stat`), 0 leftover `// @ts-check`. |
| **TOTAL** | | **100** | **100** | **File name: `24120184_100.zip`** |

---

## 2. Detailed Justification by Criterion

### Criterion 1: Verdicts and evidence (30 / 30 points)
- **Predictions**: Committed in `ea1f514` before any server execution. Accurately predicted Row 2 failure in A (`forEach` not awaiting), Row 6 & 8 failures in B (`{ timeout: 5000 }` not a valid Web API option), and full pass for C.
- **Empirical Results**: Committed in `9fae4c2`. All 27 rubric rows (9 rows × 3 versions) plus 3 parallel waterfall checks are documented with exact observations and server log timings (`3003 ms` under `slow`, `30011 ms` / `30016 ms` under `hang`).
- **Evidence Screenshots**:
  - `evidence/a-row2.png`: Shows `#stock-value` displaying `0 đ` instead of `12.360.000 đ`.
  - `evidence/b-row6.png`: Shows request hanging past 30 seconds before 504 gateway timeout.
  - `evidence/b-row8.png`: Shows red squiggly underline in VS Code under `{ timeout: 5000 }`.
- **Actionable Sentences**: Each failing version has a specific one-sentence directive citing the row, the evidence, and the constraints to keep.

### Criterion 2: After your fixes, every version passes every row (25 / 25 points)
- **Version A**: Fixed in commit `332af4a`. Replaced lines 51–53 with `await Promise.all(products.map(p => getStock(p.id)))` and `products.reduce(...)`. Total is `12.360.000 đ`. All 8 stock requests start concurrently at ~190 ms (verified via Network waterfall, no staircase).
- **Version B**: Fixed in commit `c7c1f61`. Replaced line 16 `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`. Aborts after exactly 5.00 s in `hang` mode (Row 6 PASS); 0 red squiggly underlines under `// @ts-check` (Row 8 PASS).
- **Version C**: Left untouched baseline (`keep`).
- **Post-Fix Verification**: All 3 versions verified 10/10 PASS across all 5 test conditions, documented cleanly in `VERDICTS.md` (Commit `5263dbc`).

### Criterion 3: The assistant's review, checked (15 / 15 points)
- **Raw Answer**: Verbatim Vietnamese response from Gemini 3.8 Flash High committed in `3afee41` without edits or summaries.
- **Table 1 (Claims vs Runs)**: Checks all assistant claims against empirical test rows. Confirms the two main flaws in A and B; points out inexact line numbers cited by the assistant (lines 48–52 vs 50–56 in A, line 13 vs 16 in B); and disproves the assistant's claim regarding DOM cleanup in B under standard `error` mode.
- **Table 2 (Failing Rows Missed)**: Identifies that the assistant's reading missed Row 8 (`// @ts-check`), which was caught by running VS Code with `jsconfig.json` (evidence: `evidence/b-row8.png`).

### Criterion 4: CSR vs SSR (10 / 10 points)
- **Observations**: `CSR-SSR.md` (Commit `56cfcad`) thoroughly answers all 5 questions from `README.md`.
- **Security & Escaping**: Compares server-side `escapeHtml()` in `server.mjs` (lines 67–71) with browser `textContent`, explaining how both treat data as plain text to prevent XSS and prevent `<i>limited</i>` from rendering as italics.
- **Metrics**: Contrasts ~13–14 requests for CSR against 2 requests for SSR, explaining First Contentful Paint advantages for slow devices.
- **Rule of Thumb**: Concludes with strictly one sentence defining when to use SSR vs CSR.

### Criterion 5: AI-LOG.md (10 / 10 points)
- **Log Structure**: Commits `2cc0f80`, `79b72d7`, and `fec14aa`. Contains 3 entries corresponding to each assistant engagement (Review, Fix A, Fix B).
- **Honest Records**: Each entry details the prompt, the code proposed, verification that re-running the full table broke no other rows, and specific items in `Changed / Rejected` (including rejecting scope creep outside lines 51–53 in A and rejecting corrupted markdown drafts).

### Criterion 6: Scope and commit history (10 / 10 points)
- **History**: Linear 14-commit history starting with untouched starter (`0de5088`).
- **Scope**: `git diff 0de5088 HEAD --stat` confirms only `public/versions/a.js` (7 lines) and `public/versions/b.js` (2 lines) were touched among starter code. All other starter files (`server.mjs`, `panel.js`, `index.html`, `style.css`, `c.js`, `jsconfig.json`) remain identical to starter.
- **Code Cleanliness**: 0 occurrences of `// @ts-check` left in `public/versions/*.js`.

---

## 3. What I Did Not Manage

1. **Hover Tooltip in `evidence/b-row8.png`**:
   The screenshot in `evidence/b-row8.png` captures the red squiggly underline beneath `{ timeout: 5000 }` in VS Code at line 17, but the screenshot was captured without the hovering mouse tooltip popup. The exact TypeScript compiler error string (`Object literal may only specify known properties, and 'timeout' does not exist in type 'RequestInit'`) was verified in VS Code and transcribed into `VERDICTS.md` and `ASSISTANT-REVIEW.md`, but is not visually displayed in a popup box within the PNG file itself.
2. **Assistant Line Discrepancies**:
   In `ASSISTANT-REVIEW.md`, the assistant's initial review cited slightly shifted line numbers (citing lines 48–52 instead of lines 50–56 in `a.js`, and line 13 instead of line 16 in `b.js`). Rather than altering or smoothing the assistant's raw review text, I preserved the assistant's raw text verbatim as required by the rubric and documented the exact line discrepancies in Table 1 of the verification section.
