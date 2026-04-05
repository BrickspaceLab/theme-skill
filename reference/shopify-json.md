# Shopify Online Store 2.0 — JSON templates reference

This file supplements [SKILL.md](../SKILL.md) with terminology and links. For block-type patterns when the theme uses `blocks/`, follow **Discover allowed block types** and **Analyze block and section schemas** in SKILL.md. Optional theme-specific recipes live under [examples/](../examples/). **Always** confirm `type` values and settings against the **target theme**’s `sections/`, `blocks/`, and `{% schema %}`.

## Official documentation

<!-- source: https://shopify.dev/docs/storefronts/themes/architecture -->
- [Theme architecture](https://shopify.dev/docs/storefronts/themes/architecture) — how templates, sections, blocks, and assets relate
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates -->
- [JSON templates](https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates) — structure, `sections`, `order`, limitations
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/templates -->
- [Templates](https://shopify.dev/docs/storefronts/themes/architecture/templates) — alternate templates, customer templates, layout key
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/sections -->
- [Sections](https://shopify.dev/docs/storefronts/themes/architecture/sections) — Liquid sections and schema overview
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema -->
- [Section schema](https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema) — `{% schema %}` tag, settings, blocks, presets, limit
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/section-groups -->
- [Section groups](https://shopify.dev/docs/storefronts/themes/architecture/section-groups) — optional grouping in JSON / layouts where applicable
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/blocks -->
- [Blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks) — block files, `@theme`, `@app` references, nesting rules
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks -->
- [Theme blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks) — standalone block files under `blocks/`, how they differ from section-defined blocks

<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates -->
## Standard JSON template filenames

These names are conventional for Shopify themes (exact set depends on the theme):

| File | Typical use |
|------|----------------|
| `templates/index.json` | Homepage |
| `templates/product.json` | Product pages |
| `templates/collection.json` | Collection pages |
| `templates/page.json` | Online Store pages |
| `templates/blog.json` | Blog index |
| `templates/article.json` | Blog articles |
| `templates/cart.json` | Cart |
| `templates/search.json` | Search results |
| `templates/list-collections.json` | Collection list (if used) |
| `templates/404.json` | Not found (if JSON) |
| `templates/password.json` | Password page (if JSON) |

**Alternate templates:** `\<resource\>.\<suffix\>.json` — e.g. `product.preorder.json`, `page.contact.json`.

**Customer account JSON** (when used): under `templates/customers/` per current Shopify theme conventions.

<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates -->
## Limits (platform)

Per Shopify’s JSON templates documentation:

- Up to **25 sections** per template; up to **50 blocks** per section
- Up to **1,000** JSON templates per theme
- Root may include optional **`layout`** (layout filename or `false`) and **`wrapper`** (HTML wrapper for all sections); see the docs for syntax

<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates -->
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema -->
<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks -->
## Concepts

- **Section `type`:** String matching a section the theme provides (usually the basename of `sections/\<type\>.liquid`).
- **Section instance id:** Key in the template’s `sections` object; can differ from `type` (e.g. two `"type": "image-banner"` sections with ids `image_banner_top` and `image_banner_bottom`).
- **Block `type`:** Defined in the section schema’s `blocks` array; each block entry may allow nested `blocks` and `block_order`.
- **`order`:** Top-level array listing section instance keys in render order.
- **`block_order`:** Per-level array listing block instance keys under that parent.

<!-- source: https://shopify.dev/docs/storefronts/themes/architecture/templates -->
## Relationship to Liquid templates

Themes may still use `.liquid` templates for some routes. JSON templates reference sections only; the Liquid in each section still controls markup. Do not confuse **template JSON** with **section Liquid** — both are needed, but this skill focuses on **JSON template files**.
