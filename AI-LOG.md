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
