# Self-Assessment Report: IA#3 — Three Answers to One Brief

- **Student ID**: 24120184
- **Repository**: https://github.com/LqHung06/24120184-IA3-three-answers
- **Branch**: `main` (Public repository)
- **Course**: CSC13008 — Web Application Development
- **Session**: Session 5 — "Code That Waits"
- **Claimed Score**: **100 / 100**

---

## 1. Executive Summary & Grading Breakdown

| Item | Criterion | Max Points | Claimed Points | Evidence / Git Commit |
|---|---|---|---|---|
| **1** | **Predictions before Running** | 10 | 10 | `VERDICTS.md` (Commit `ea1f514`): Detailed predictions for all 3 versions written strictly from reading code prior to executing any script. |
| **2** | **Independent Assistant Review** | 10 | 10 | `ASSISTANT-REVIEW.md` (Commit `3afee41`): Raw, unedited review in Vietnamese from Gemini 3.8 Flash High, with line citations and initial verdicts. |
| **3** | **AI Interaction Log (`AI-LOG.md`)** | 10 | 10 | `AI-LOG.md` (Commits `2cc0f80`, `79b72d7`, `fec14aa`): 3 structured entries (Review, Fix A, Fix B) with exact prompts, diffs, table re-run confirmation, and rejected items. |
| **4** | **Empirical Test Results & Evidence** | 20 | 20 | `VERDICTS.md` (Commit `9fae4c2`) & `evidence/`: 30 verified test rows across all 5 test conditions (`normal`, `slow`, `error`, `hang`, `empty`), console checks, `@ts-check`, parallel checks, with screenshots (`a-row2.png`, `b-row6.png`, `b-row8.png`). |
| **5** | **Minimal, Correct Fixes (A & B)** | 20 | 20 | `public/versions/a.js` (Commit `332af4a`): Awaited parallel stock requests with `Promise.all` + `reduce`. `public/versions/b.js` (Commit `c7c1f61`): Fixed 5s timeout with `AbortSignal.timeout(5000)`. Version C kept untouched. All 3 versions 10/10 PASS (Commit `5263dbc`). |
| **6** | **CSR vs SSR Comparative Analysis** | 15 | 15 | `CSR-SSR.md` (Commit `56cfcad`): 5 observation questions answered in English with exact request counts (13–14 vs 2), DOM analysis, XSS escaping comparison (`escapeHtml` vs `textContent`), and single-sentence architectural conclusion. |
| **7** | **Verification Tables** | 10 | 10 | `ASSISTANT-REVIEW.md`: Table 1 (Assistant Claims vs Actual Runs) evaluating all claims, and Table 2 (Failing Rows Missed) identifying Row 8 (`// @ts-check`). |
| **8** | **Git Discipline & Repository Hygiene** | 5 | 5 | Clean linear commit history matching the step-by-step workflow, atomic commits with informative messages, no scope creep (`git diff 0de5088 HEAD --stat`), and no leftover `// @ts-check`. |
| **TOTAL** | | **100** | **100** | |

---

## 2. Detailed Evidence & Criteria Justification

### Criterion 1: Predictions Before Running (10/10)
- **Commit**: `ea1f514` ("predictions before running")
- **Details**: Written strictly before starting `server.mjs`. Predicted that Version A would fail Row 2 due to `forEach(async ...)` not awaiting; predicted Version B would fail Row 6 (`hang` mode) and Row 8 (`// @ts-check`) due to `{ timeout: 5000 }` not being a standard Web API option; predicted Version C would pass all rows (`keep`). Predictions completely matched subsequent empirical runs.

### Criterion 2: Independent Assistant Review (10/10)
- **Commit**: `3afee41` ("assistant review, raw answer")
- **Details**: Preserved verbatim in Vietnamese from an independent Gemini session without any post-editing or smoothing. Successfully diagnosed the core issues in Version A and Version B.

### Criterion 3: AI Interaction Log (10/10)
- **Commits**:
  - `2cc0f80`: `ai-log: assistant review` (Entry 1)
  - `79b72d7`: `ai-log: fix A` (Entry 2)
  - `fec14aa`: `ai-log: fix B` (Entry 3)
- **Details**: Every entry follows the required schema (`Tool & Model`, `Date`, `Request`, `Result`, `Accepted`, `Changed / Rejected`). Includes the exact one-sentence directive from `VERDICTS.md`, confirms that re-running the entire test table produced 0 regressions, and documents concrete rejected items (such as rejecting unneeded code refactoring and correcting corrupted markdown formatting).

### Criterion 4: Empirical Test Results Before Fixes (20/20)
- **Commit**: `9fae4c2` ("results before fixes")
- **Details**: Documented all 27 rubric rows plus 3 parallel verification checks. Recorded exact server log response times (e.g., `3003 ms` under `slow`, `30011 ms` / `30016 ms` under `hang`).
- **Screenshots**:
  - `evidence/a-row2.png`: Shows `#stock-value` as `0 đ` under `normal` mode.
  - `evidence/b-row6.png`: Shows `/api/products` hanging past 30 seconds before receiving HTTP 504.
  - `evidence/b-row8.png`: Shows red squiggly error in VS Code under `{ timeout: 5000 }`.

### Criterion 5: Minimal & Correct Fixes for A and B (20/20)
- **Commit `332af4a` ("fix A: wait for all stock requests before writing the total")**:
  - Touched only lines 51–53 in `public/versions/a.js`.
  - Replaced un-awaited `forEach` with `await Promise.all(products.map(...))` and `products.reduce(...)`.
  - Maintained parallel stock fetching (~190 ms start timestamp, no waterfall staircase). Row 2 now displays `12.360.000 đ`.
- **Commit `c7c1f61` ("fix B: real 5 s timeout with AbortSignal")**:
  - Touched strictly 1 line (`public/versions/b.js | 2 +-`): replaced `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`.
  - Fixes Row 6 (aborts at exactly 5.00 s in `hang` mode) and Row 8 (0 red squiggly underlines under `// @ts-check`).
- **Version C**: Left 100% untouched (`keep`).
- **Results after fixes**: All 3 versions verified 10/10 PASS in `VERDICTS.md` (Commits `f397868`, `5263dbc`).

### Criterion 6: CSR vs SSR Comparative Analysis (15/15)
- **Commit**: `56cfcad` ("CSR vs SSR answers")
- **Details**: `CSR-SSR.md` provides in-depth answers to all 5 observation questions from `README.md`. Includes code citations (`escapeHtml` in `server.mjs`), network metrics (13–14 requests for CSR vs 2 for SSR), JavaScript degradation behavior, and a single-sentence architectural rule of thumb.

### Criterion 7: Verification Tables in Assistant Review (10/10)
- **File**: `ASSISTANT-REVIEW.md`
- **Details**:
  - Table 1 cross-examines all claims made by the assistant against empirical test runs, distinguishing True claims from Inexact line citations and edge cases.
  - Table 2 explicitly identifies that the assistant missed Row 8 (`// @ts-check`) for Version B, citing `evidence/b-row8.png`.

### Criterion 8: Git Hygiene & Packaging (5/5)
- **Details**: Clean git log, atomic commits for every milestone, no merge commits, working tree clean. No temporary files or leftover `// @ts-check` comments in production version scripts. Deliverable packaged into `<StudentID>_100.zip` containing the full `.git/` history.

---

## 3. Conclusion

All requirements and constraints of the assignment brief, `README.md`, and guidance document have been rigorously satisfied. The final submission achieves a verified score of **100 / 100**.
