# AI Workflow & Tooling Notes

**Project:** Purelane Shopify OS 2.0 Theme Refactor  
**Focus:** Reflection on AI Delegation, Failure Points, and Workflow Systematization  

---

### What I Delegated

I primarily used the AI to generate boilerplate Liquid `{% schema %}` JSON for section settings and block structures. Parsing the static HTML to figure out which text blocks, images, and links needed to be merchant-editable is tedious, so I fed the HTML to the AI to rapidly map out schema arrays for complex sections like the Best-Selling Combos and the Reviews marquee rail. I also delegated the quick conversion of static HTML product cards into native Shopify Liquid objects (`{{ product.title }}`, `{{ product.price | money }}`).

---

### Where the AI Failed / Hallucinated

The AI struggled heavily with the prototype's unconventional CSS architecture. Because the original file used CSS background images driven by custom aspect-ratio utility classes (e.g., `.p-combo2 { background-image: var(--p-combo2); aspect-ratio: 1.1247 }`), the AI initially tried to inject Shopify Liquid tags directly into CSS `<style>` blocks. I had to manually intervene, rip out the CSS background logic, and rewrite it using standard HTML `<img>` elements powered by Shopify's `image_url` filter to ensure proper CDN media optimization and responsive `srcset` generation. The AI also occasionally hallucinated legacy Online Store 1.0 Liquid syntax instead of modern Dawn theme architecture.

---

### Systematizing for the Future

If I had to build 20 more sections like this, I would establish a strict, pre-defined AI Rulebook. I would create a system prompt that explicitly bans the use of CSS background images for product media, enforces semantic HTML `<button>` elements over `<i>` tags, and supplies a standardized JSON schema template. I would delegate only data mapping and schema generation to the AI, while keeping CSS refactoring and JavaScript web component scoping strictly under manual control to ensure full compatibility with the Shopify Theme Editor without breaking.
