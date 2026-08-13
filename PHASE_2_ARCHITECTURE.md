# Phase 2 Architecture Document: Shopify OS 2.0 Theme Mapping

**Brand Target:** Purelane (Plant-based Homecare)  
**Theme Baseline:** Shopify Dawn / OS 2.0 Architecture  
**Source Prototype:** `purelane-homepage.html`

---

## 1. Reusable Snippets Identification

Scanning the prototype reveals 4 core repeating UI components across the grid, combo, tier, and review sections. These will be implemented as modular, isolated Liquid snippets under `snippets/`:

| Snippet Name | Source Section HTML Selector | Visual / Functional Purpose |
| :--- | :--- | :--- |
| `snippet-product-card.liquid` | `#shop .shelf > article.card` | Individual product card with image/SVG fallback, dynamic pricing, badges, line-clamped title, and single-click cart integration. |
| `snippet-combo-card.liquid` | `#combos .comborail > article.combo` | Pre-built bundle card featuring visual product item stacks, total savings callouts, items list, and bundle quick-picker drawer triggers. |
| `snippet-bundle-tier-card.liquid` | `#bundles .tiers > article.tier` | Mix-and-match tier box card (2, 3, or 5 products) displaying flat rate per product, tiered discounts, feature bullet lists, and build trigger. |
| `snippet-review-card.liquid` | `#reviews .revtrack > article.rcard` | Social proof customer review card with star ratings, testimonial text, verified buyer indicator, and product reference line. |

---

## 2. Shopify Native Data Mapping

Below is the mapping of static HTML elements to native Shopify Liquid attributes and variables:

### A. Product Card (`#shop`) -> `product` Object

| HTML Prototype Element | Static Example in Prototype | Native Shopify Liquid Object Mapping |
| :--- | :--- | :--- |
| Container | `<article class="glass card rv">` | `<article class="glass card rv" data-product-id="{{ product.id }}">` |
| Image | `<span class="pimg p-tap">` | `{{ product.featured_media \| image_url: width: 400 \| image_tag: class: 'pimg pimg-img', loading: 'lazy' }}` |
| Title | `<h4>Tap cleaner & limescale remover</h4>` | `<h4><a href="{{ product.url }}">{{ product.title }}</a></h4>` |
| Selling Price | `<strong>₹200</strong>` | `<strong>{{ product.price \| money }}</strong>` |
| Compare Price | `<s>₹299</s>` | `{% if product.compare_at_price > product.price %}<s>{{ product.compare_at_price \| money }}</s>{% endif %}` |
| Discount % | `<em>33% off</em>` | `<em>{{ product.compare_at_price \| minus: product.price \| times: 100.0 \| divided_by: product.compare_at_price \| round }}% off</em>` |
| Availability / CTA | `<button class="btn btn-ghost btn-sm">` | `{% if product.available %}{% form 'product', product %}...<button type="submit">Add to cart</button>{% endform %}{% else %}<button disabled>Sold Out</button>{% endif %}` |

### B. Combo Card (`#combos`) -> Bundle `product` Object

| HTML Prototype Element | Static Example in Prototype | Native Shopify Liquid Object Mapping |
| :--- | :--- | :--- |
| Combo Title | `<h3>Kitchen essentials</h3>` | `<h3>{{ product.title }}</h3>` |
| Selling Price | `<strong>₹499</strong>` | `<strong>{{ product.price \| money }}</strong>` |
| Compare Price | `<s>₹897</s>` | `<s>{{ product.compare_at_price \| money }}</s>` |
| Product Count | `<div class="cnt">3 products</div>` | `<div class="cnt">{{ product.variants.first.options.size \| default: product.metafields.custom.combo_items.value.count }} products</div>` |

---

## 3. Custom Data (Metafields & Metaobjects) Schema

Standard Shopify Liquid objects lack native support for custom badges, product stack features, tier perk lists, and review ratings without third-party apps or hardcoding.

The JSON schema below defines the required **Shopify Metafields** and **Metaobjects** to make the entire UI dynamic via Shopify Admin.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Purelane_Shopify_Custom_Data_Schema",
  "description": "Metafield & Metaobject definitions for Purelane OS 2.0 Theme",
  "metafields": {
    "product_metafields": [
      {
        "namespace": "custom",
        "key": "badge_text",
        "name": "Custom Product Badge",
        "type": "single_line_text_field",
        "description": "Overrides default badge text (e.g., 'Best seller', 'Top rated', 'New', 'Most popular')."
      },
      {
        "namespace": "custom",
        "key": "rating_summary",
        "name": "Rating Summary Override",
        "type": "json",
        "description": "Fallback rating metadata when external review app is absent.",
        "example": {
          "rating": 4.8,
          "review_count": 237
        }
      },
      {
        "namespace": "custom",
        "key": "combo_tagline",
        "name": "Combo Savings Tagline",
        "type": "single_line_text_field",
        "description": "Header savings banner for combo cards (e.g., 'You save ₹398')."
      },
      {
        "namespace": "custom",
        "key": "combo_items_included",
        "name": "Combo Included Products Breakdown",
        "type": "list.single_line_text_field",
        "description": "Itemized list of products included in the combo box for card description text."
      },
      {
        "namespace": "custom",
        "key": "combo_item_highlights",
        "name": "Combo Visual Stack Highlights",
        "type": "json",
        "description": "Array of included product references and highlight subtext for the visual stack tray.",
        "example": [
          { "product_handle": "kitchen-cleaner", "subtext": "Cuts grease instantly" },
          { "product_handle": "dishwash-gel", "subtext": "Squeaky clean dishes" },
          { "product_handle": "tap-cleaner", "subtext": "Melts hard water stains" }
        ]
      }
    ]
  },
  "metaobjects": {
    "bundle_tier": {
      "name": "Bundle Tier Definition",
      "type": "bundle_tier",
      "fields": [
        {
          "key": "tier_name",
          "name": "Tier Tag",
          "type": "single_line_text_field",
          "example": "Starter / Most popular / Whole home"
        },
        {
          "key": "product_quantity",
          "name": "Product Count",
          "type": "number_integer",
          "example": 3
        },
        {
          "key": "tier_price",
          "name": "Flat Bundle Price",
          "type": "number_decimal",
          "example": 499.00
        },
        {
          "key": "compare_at_price",
          "name": "Original Combined Price",
          "type": "number_decimal",
          "example": 897.00
        },
        {
          "key": "unit_rate_text",
          "name": "Per Product Price Note",
          "type": "single_line_text_field",
          "example": "Flat ₹166 per product"
        },
        {
          "key": "perks_list",
          "name": "Feature Bullet Points",
          "type": "list.single_line_text_field",
          "example": [
            "Pick any three products",
            "Covers kitchen and laundry",
            "Free shipping across India"
          ]
        }
      ]
    },
    "customer_review": {
      "name": "Customer Review",
      "type": "customer_review",
      "fields": [
        {
          "key": "rating_stars",
          "name": "Star Rating (1-5)",
          "type": "number_integer",
          "example": 5
        },
        {
          "key": "headline",
          "name": "Review Title",
          "type": "single_line_text_field",
          "example": "Works like a charm"
        },
        {
          "key": "body_text",
          "name": "Review Body",
          "type": "multi_line_text_field",
          "example": "Finally an eco option that cleans as well as chemical detergent..."
        },
        {
          "key": "author_name",
          "name": "Customer Name",
          "type": "single_line_text_field",
          "example": "Anita"
        },
        {
          "key": "product_tag",
          "name": "Purchased Product Name",
          "type": "single_line_text_field",
          "example": "Laundry detergent"
        },
        {
          "key": "is_verified",
          "name": "Verified Buyer Flag",
          "type": "boolean",
          "example": true
        }
      ]
    }
  }
}
```

---

### Architectural Readiness Summary
1. **Zero Hardcoding Strategy:** All non-standard visual text (savings ribbons, custom pill badges, stack bullets, tier rules) is routed through structured metafields and metaobjects.
2. **Standard Theme Compatibility:** Fits seamlessly into Dawn OS 2.0 dynamic sections and blocks (`templates/index.json`).
3. **Execution Plan:** Ready for Phase 3 section building (`sections/shop-grid.liquid`, `sections/combos-carousel.liquid`, `sections/bundle-picker.liquid`, `sections/reviews-marquee.liquid`).
