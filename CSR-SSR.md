# CSR vs SSR: Comparing the Same Catalogue Built in Two Places

- **Student ID**: 24120184
- **Repository**: https://github.com/LqHung06/24120184-IA3-three-answers
- **Course**: CSC13008 — Web Application Development
- **Assignment**: IA#3 — Three answers to one brief

---

## 1. View Page Source (`Ctrl+U`)

- **Client-Side Rendered (`/?v=c` / `index.html`)**:
  - The product catalogue list in the raw HTML source is completely empty: `<ul class="catalogue" id="catalogue"></ul>`.
  - The stock value output displays only the initial placeholder character: `<output id="stock-value">—</output>`.
  - Product names and stock values do not exist anywhere in the static HTML payload; they are only represented as an uninstantiated client-side `<template id="card-template">` structure and are fetched asynchronously via client-side JavaScript (`/api/products` and `/api/stock/:id`) after script execution begins.
  - Confirmed in the source view (`evidence/csr-source.png`): the delivered HTML contains an empty `<ul class="catalogue" id="catalogue"></ul>` and `<output id="stock-value">—</output>`, with no product data.
- **Server-Side Rendered (`/ssr`)**:
  - Product names are directly embedded inside the HTML payload within pre-rendered `<li>` elements (e.g., `<h2 class="name">T-shirt</h2>`, `<h2 class="name">Hoodie</h2>`, etc.).
  - The calculated stock total is already rendered into the markup delivered from the server: `<output id="stock-value">12.360.000 đ</output>`.
  - No client-side JavaScript execution or secondary network requests are required to view the content.
  - Confirmed in the source view (`evidence/ssr-view-source.png`): all 8 `<li class="card">` elements and the stock total are present in the HTML the server sent, line 16 `<output id="stock-value">12.360.000 đ</output>`.

---

## 2. The Scarf and Security: `&lt;i&gt;` vs `textContent`

- **Why it reads `&lt;i&gt;` in `/ssr` source**:
  - In `server.mjs`, `renderSsrPage()` sanitizes dynamic product names through the `escapeHtml(s)` helper function (lines 67–71):
    ```javascript
    function escapeHtml(s) {
      return String(s)
        .replaceAll('&', '&amp;').replaceAll('<', '&lt;').replaceAll('>', '&gt;')
        .replaceAll('"', '&quot;').replaceAll("'", '&#39;');
    }
    ```
    This transforms literal `<` to `&lt;` and `>` to `&gt;`, emitting `<h2 class="name">Scarf &lt;i&gt;limited&lt;/i&gt;</h2>`.
  - Confirmed in the source view (`evidence/ssr-view-source.png`, line 48): `<h2 class="name">Scarf &lt;i&gt;limited&lt;/i&gt;</h2>`.
- **What it has in common with `textContent`**:
  - Both `escapeHtml()` on the server and `element.textContent = product.name` in client-side JavaScript treat external data strictly as raw character sequences rather than executable DOM markup.
  - Both mechanisms prevent Cross-Site Scripting (XSS) vulnerabilities and guarantee that angle brackets are rendered literally on screen as `Scarf <i>limited</i>` rather than being interpreted as HTML italic formatting tags.

---

## 3. Disabling JavaScript

- **Client-Side Rendered (`/?v=c`)**:
  - When JavaScript is disabled, the page remains completely blank aside from the static header toolbar and test condition container. The catalogue list remains empty (`<ul></ul>`), the status line is blank, and `#stock-value` remains `—`. The page is completely non-functional without the client runtime engine.
  - Evidence: `evidence/csr-js-off.png` — no cards, `#stock-value` is `—`, and the Network tab shows only 3 requests (document, `style.css`, `favicon.ico`); no script or API request runs.
- **Server-Side Rendered (`/ssr`)**:
  - When JavaScript is disabled, the page renders 100% identically to its normal state. All 8 product cards, their formatted prices, and the grand stock total (`12.360.000 đ`) remain fully visible and usable because the server produced complete semantic HTML, and `/ssr` does not load or depend on any JavaScript files.
  - Evidence: `evidence/ssr-js-off.png` — all 8 cards and the total visible with JavaScript off.

---

## 4. Network Tab Requests and Performance

Counted in the Network tab on 31 October 2026 (Disable cache on, filter All, counted from the status bar).

- **Client-Side Rendered (`/?v=c`)**:
  - Generates **15 HTTP requests** before the final catalogue and stock value are visible (`evidence/csr-requests.png`):
    1. `GET /?v=c` (HTML skeleton)
    2. `GET /style.css` (Stylesheet)
    3. `GET /panel.js` (Panel script)
    4. `GET /__mode` (Server mode API probe)
    5. `GET /versions/c.js` (Client catalogue loader)
    6. `GET /api/products` (Product array JSON)
    7–14. Eight concurrent `GET /api/stock/:id` requests (individual product stock queries).
    15. `GET /favicon.ico` (404 — requested automatically by the browser).
  - Excluding the automatic `favicon.ico` request, the count is **14**.
- **Server-Side Rendered (`/ssr`)**:
  - Generates **3 HTTP requests** (`evidence/ssr-requests.png`):
    1. `GET /ssr` (Fully rendered HTML document)
    2. `GET /style.css` (Stylesheet)
    3. `GET /favicon.ico` (404 — requested automatically by the browser).
  - Excluding the automatic `favicon.ico` request, the count is **2**.
- **Performance Comparison**:
  - A search engine crawler or a user on a low-end mobile device on a slow network will see the **SSR page first**. SSR delivers the content ready-to-paint in the initial response body (instant First Contentful Paint / Largest Contentful Paint), whereas CSR suffers from a multi-stage request waterfall and requires JavaScript parsing and execution before any meaningful content can be displayed.
  - In the counted runs, CSR finished in **711 ms** versus SSR in **63 ms** (Network status bar, `evidence/csr-requests.png` and `evidence/ssr-requests.png`).

---

## 5. Architectural Trade-off (One Sentence)

Choose Server-Side Rendering when first-load speed, search engine indexability (SEO), resilience against slow networks, and accessibility without client runtime dependencies are paramount (such as public e-commerce catalogues, blogs, or landing pages); choose Client-Side Rendering when building complex, stateful, app-like interfaces requiring instant client-side transitions and rich dynamic interactions without full-page reloads after the initial bundle has loaded (such as dashboards, admin portals, or interactive software tools).
