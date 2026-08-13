# Build Notes & Technical Refactoring Report

**Project:** Purelane Shopify OS 2.0 Theme Refactor  
**Baseline Theme:** Shopify Dawn  
**Source Prototype:** `purelane-homepage.html`

---

### 1. Flaws Spotted in the Prototype

* **Massive Inlined Base64 Assets:** The original prototype embedded huge base64 SVG data URIs directly into `:root` CSS variables (`--p-combo2`, `--p-dish`, `--p-kitchen`, `--p-laundry`, etc.). This caused significant render-blocking bloat, inflated stylesheet size, and completely prevented browser image preloading or native lazy loading.
* **Accessibility Violations:** The interactive rotator under "Why it works" used non-semantic `<i>` tags (`<i class="on"></i>`) for slide navigation dots rather than accessible `<button>` elements, breaking keyboard focus and screen reader navigation.
* **Non-Functional Form Controls:** The email newsletter signup form was hardcoded with `onsubmit="return false"`, rendering it useless for customer capture.
* **Global Scope Contamination:** Interactive scripts relied on hardcoded global IDs (`document.getElementById('hstage')`, `document.getElementById('rot')`), which causes script failures if a merchant duplicates a section in Shopify's Theme Customizer.
* **Lack of Layout Resiliency:** Product cards lacked multiline clamping on `<h4>` titles and fixed aspect ratios on image containers, causing vertical misalignment across grid columns when titles spanned multiple lines.

---

### 2. What I Changed & Why

* **Performance & Data:** Stripped out all base64 CSS backgrounds. Replaced them with native HTML `<img>` elements using Shopify's `image_url` filter (`srcset`, `sizes`, `loading: 'lazy'`). For the above-the-fold Hero stage, set `loading: 'eager'` and `fetchpriority: 'high'` to minimize LCP (Largest Contentful Paint).
* **Semantics & Accessibility (a11y):** Refactored interactive dots into semantic `<button>` elements with dynamic `aria-label`, `aria-selected`, and `role="tab"` attributes. Preserved `:focus-visible` outlines (`2px solid var(--accent)`) and added `@media (prefers-reduced-motion: reduce)` rules to pause animations for motion-sensitive users.
* **Theme Editor Modularity & Scoping:** Wrapped sections in custom web components (`<shop-grid-section>`, `<hero-section>`) and scoped JS using `this.container.querySelector()`. Scoped all CSS under `#shopify-section-{{ section.id }}` to guarantee zero style bleeding into Dawn's global theme styles.
* **Dynamic Integration & Schema:** Replaced hardcoded static articles with dynamic Liquid `{% for %}` loops connected to Shopify `collection.products` and custom `{% schema %}` blocks, allowing merchants to customize all content directly via the Theme Customizer.
* **Resilient Fallback Handling:** Implemented SVG `placeholder_svg_tag` fallback for missing product images and applied CSS 2-line title clamping (`-webkit-line-clamp: 2`, `min-height: 2.4em`) with `margin-top: auto` pricing alignment.

---

### 3. What I Would Have Done Differently With More Time

* **Extract Botanical Vector Line Art:** Extract the heavy inline SVGs used for the "Ingredients" botanical line art into reusable Liquid snippets (`snippets/icon-coconut.liquid`, `snippets/icon-neem.liquid`) to keep section files pristine.
* **Integrate Predictive Search & Cart Drawer APIs:** Replace the static search button (`<button class="ico hide-s" aria-label="Search">`) with Shopify's Predictive Search drawer API and connect the cart counter bubble (`<span class="dot">0</span>`) directly to Dawn's Section Rendering API and Ajax cart drawer.
* **Metaobject Product Bundle Builder App:** Build a custom Shopify App Proxy / Metaobject endpoint allowing customers to dynamically build custom 3-pack or 5-pack bundle boxes directly on the client side with instant price updates.
