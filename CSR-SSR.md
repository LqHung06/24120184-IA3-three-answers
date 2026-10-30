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
- **Server-Side Rendered (`/ssr`)**:
  - Product names are directly embedded inside the HTML payload within pre-rendered `<li>` elements (e.g., `<h2 class="name">T-shirt</h2>`, `<h2 class="name">Hoodie</h2>`, etc.).
  - The calculated stock total is already rendered into the markup delivered from the server: `<output id="stock-value">12.360.000 đ</output>`.
  - No client-side JavaScript execution or secondary network requests are required to view the content.

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
- **What it has in common with `textContent`**:
  - Both `escapeHtml()` on the server and `element.textContent = product.name` in client-side JavaScript treat external data strictly as raw character sequences rather than executable DOM markup.
  - Both mechanisms prevent Cross-Site Scripting (XSS) vulnerabilities and guarantee that angle brackets are rendered literally on screen as `Scarf <i>limited</i>` rather than being interpreted as HTML italic formatting tags.

---

## 3. Disabling JavaScript

- **Client-Side Rendered (`/?v=c`)**:
  - When JavaScript is disabled, the page remains completely blank aside from the static header toolbar and test condition container. The catalogue list remains empty (`<ul></ul>`), the status line is blank, and `#stock-value` remains `—`. The page is completely non-functional without the client runtime engine.
- **Server-Side Rendered (`/ssr`)**:
  - When JavaScript is disabled, the page renders 100% identically to its normal state. All 8 product cards, their formatted prices, and the grand stock total (`12.360.000 đ`) remain fully visible and usable because the server produced complete semantic HTML, and `/ssr` does not load or depend on any JavaScript files.

---

## 4. Network Tab Requests and Performance

- **Client-Side Rendered (`/?v=c`)**:
  - Generates approximately **13 to 14 HTTP requests** before the final catalogue and stock value are visible:
    1. `GET /?v=c` (HTML skeleton)
    2. `GET /style.css` (Stylesheet)
    3. `GET /panel.js` (Panel script)
    4. `GET /__mode` (Server mode API probe)
    5. `GET /versions/c.js` (Client catalogue loader)
    6. `GET /api/products` (Product array JSON)
    7–14. Eight concurrent `GET /api/stock/:id` requests (individual product stock queries).
- **Server-Side Rendered (`/ssr`)**:
  - Generates exactly **2 HTTP requests**:
    1. `GET /ssr` (Fully rendered HTML document)
    2. `GET /style.css` (Stylesheet)
- **Performance Comparison**:
  - A search engine crawler or a user on a low-end mobile device on a slow network will see the **SSR page first**. SSR delivers the content ready-to-paint in the initial response body (instant First Contentful Paint / Largest Contentful Paint), whereas CSR suffers from a multi-stage request waterfall and requires JavaScript parsing and execution before any meaningful content can be displayed.

---

## 5. Architectural Trade-off (One Sentence)

Choose Server-Side Rendering when first-load speed, search engine indexability (SEO), resilience against slow networks, and accessibility without client runtime dependencies are paramount (such as public e-commerce catalogues, blogs, or landing pages); choose Client-Side Rendering when building complex, stateful, app-like interfaces requiring instant client-side transitions and rich dynamic interactions without full-page reloads after the initial bundle has loaded (such as dashboards, admin portals, or interactive software tools).
