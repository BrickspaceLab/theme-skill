---
name: theme-skill  
description: Edits Shopify JSON template files using your theme's actual block and section types. Just describe what you want or drop in a screenshot.
---

## When to use

Use when the task involves Shopify **JSON templates** under `templates/` (e.g. `product.json`, `index.json`, alternates like `page.contact.json`), **section JSON** that references **theme blocks** from `blocks/` in block-based themes, or theme design settings that can be safely expressed through `config/settings_data.json`, section settings, or block settings.

**This skill alone does not fully cover:**

- `config/settings_schema.json` schema development
- Checkout branding or Checkout UI extensions
- Theme app extension code under `extensions/`
- Replacing **Liquid** `.liquid` templates where the theme has not adopted JSON for that route

If the theme uses a **build step** that generates `sections/` or `blocks/`, read that theme’s README and follow its source-of-truth paths before editing compiled output.

## Quick start

1. **Understand requirements** — Parse prompts or images for layout, columns/rows, content types, and styling (see **Design images and mockups** when the input is visual).
2. **Discover allowed block types** — List `blocks/` and read parent `{% schema %}` so every `type` you use exists in this theme (required; see below).
3. **(Optional) Check bundled examples** — If `examples/` is present, follow **Choosing the best example to reference** in [examples/README.md](examples/README.md): compare the workspace theme to each indexed example and open at most one best-matched file. If nothing fits, copy patterns from existing `templates/*.json` in the workspace theme instead. Never emit `type` strings from a bundled example until they appear in the target theme’s allowed set.
4. **Map to real blocks** — Choose types only from that allowed set; resolve section `type` from `sections/*.liquid` for template-level JSON.
5. **Analyze schemas** — Read each block or section `{% schema %}` for allowed children, setting IDs, types, defaults, and presets.
6. **Apply theme design settings** — Map colors, typography, spacing, borders, and behavior to real global/section/block settings declared by the target theme.
7. **Compile JSON** — Build valid `sections`, nested `blocks`, and `order` / `block_order` arrays per Shopify’s template structure.
8. **Validate** — Run the checklist and JSON-focused Theme Check before finishing.

## Workflow

### Step 1 — Understand user requirements

**From text:**

- Layout: grid, flex, slider, inline, etc.
- Column/row counts
- Content: images, buttons, text, product cards, etc.
- Styling: spacing, colors, responsive behavior

**From images:**

- Visual layout, columns and rows, elements and relationships, spacing and alignment
- Visual hierarchy (headline vs body vs CTAs), approximate alignment (left/center/right), and content categories (hero, cards, logos, footer, etc.)
- **What is ambiguous** — Exact breakpoints, font sizes, and pixel spacing are not reliably readable from a static image; call that out instead of guessing.

**Design images and mockups**

When the user provides a screenshot, Figma export, or design mockup:

- **Extract:** structure (sections, columns, stacking order), content types, and qualitative spacing rhythm (tight vs airy), not exact pixel values unless provided separately.
- **Ignore images in source files:** do not copy, crop, recreate, download, duplicate, generate, or invent image assets from screenshots, Figma exports, mockups, or other source files. Use those images only to infer where image/media blocks belong.
- **Use empty image blocks or sections:** when the design calls for hero, product, lifestyle, logo, or decorative imagery, add the valid image/media block or section with its image setting omitted or blank. Assume the user or merchant will provide final images in the theme editor.
- **Only wire explicit assets:** set image/media values only when the user provides existing theme asset filenames, Shopify media IDs/references, already-selected section settings, or files they explicitly ask to use as storefront assets.
- **Do not infer** concrete Shopify `type` strings, setting keys, or enum values from the image alone. **Always** reconcile with **Discover allowed block types** on the target theme.
- **Limits:** JSON templates do not express every visual detail. Arbitrary typography, one-off CSS, or global palette changes may require **theme settings**, **section settings**, or **custom CSS** outside the core JSON-template workflow—say so when the design cannot be matched with schema-defined settings only.
- If the design **cannot** be built with the theme’s available blocks and settings, **state the gap** and offer the closest achievable structure.

### Step 2 — Discover allowed block types

Do this **before** naming or emitting any `type` in JSON. This is the operational way to satisfy the validation rule that every `type` exists in the theme.

1. **List** files in `blocks/` — For themes using theme blocks, the basename of `blocks/<name>.liquid` is typically a valid block `type` (confirm in schema).
2. **Read** `{% schema %}` on each **section** you add or edit in the template, and on each **block** file you nest. Collect every block `type` the parent allows (specific types, `@theme`, `@app`, etc., per Shopify rules for that parent).
   - Validate **each immediate parent/child edge**, not just whether a block exists somewhere in the theme. If a layout block only allows specific item-wrapper blocks, content blocks cannot be direct children even when they are valid theme blocks elsewhere. They must be nested under an allowed child if that child permits them.
   - Treat explicit `blocks` arrays as allow-lists. Only `@theme` permits general theme blocks; a specific list permits only those listed types plus any listed `@app`.
   > **`_`-prefix caveat:** Blocks whose filename starts with `_` (e.g. `_stat-bar.liquid`) may **not** be matched by `@theme` in a parent schema, even if they have `presets`. Shopify theme check treats `_`-prefixed blocks as private/static and may require them to be **explicitly listed** in the parent’s `blocks` array. Before nesting a `_`-prefixed block inside a parent that only declares `@theme`, verify by checking existing templates or presets for a precedent. If none exists, the parent schema needs a new `{ "type": "_block-name" }` entry—which is a Liquid edit, not a JSON-only change.
3. **Build the allowed set** — Union of: types from `blocks/` that the parent schema permits, types declared in the parent’s `blocks` array, and section `type` values from `sections/<type>.liquid` for template-level `sections`.
4. **Only use types in that set.** If the user asks for a layout that no block provides, propose the **closest real types**, copy a working template in the same theme, or note that **adding a new block** is theme development work outside JSON-only edits.

### Step 3 — Check bundled examples (optional)

If `examples/` is present in this skill package, follow **Choosing the best example to reference** in [examples/README.md](examples/README.md):

- Compare the workspace theme to each indexed example (theme name, README, overlap between `blocks/` basenames and the example’s described types).
- Open **at most one** best-matched file for patterns and sample JSON.
- If nothing fits, copy patterns from existing `templates/*.json` in the workspace theme instead.
- **Never** emit `type` strings from a bundled example until they appear in the target theme’s allowed set.

### Step 4 — Map requirements to real blocks

Within the **allowed type set** from Step 2:

- Search the theme’s `blocks/` directory and section schemas for blocks that match the layout and content needs.
- For **template-level** sections, list `sections/` and map JSON `type` to section files (usually basename of `sections/<type>.liquid`).

### Step 5 — Analyze block and section schemas

For each block or section, read `{% schema %}`:

1. **Nested blocks** — `"blocks": [{ "type": "@theme" }]` vs specific types vs `"blocks": []`. Build a small parent-to-child map for every nested level you emit, and check each proposed child against its immediate parent's schema before writing JSON.
2. **Settings** — Required vs optional, defaults, types. Pay attention to:
   - **`range`** — Values must land exactly on `min + (N * step)`, where `N` is a whole number, even when the setting is hidden by `visible_if`. A range with `min: 10, max: 48, step: 2` rejects `13` (use `14`). A range with `min: 100, step: 5` rejects `158` (use `160` or `155`). When copying or inventing settings, calculate the nearest valid step before writing JSON.
   - **`select`** — Values must exactly match one of the `options[].value` strings. Do not invent values even if they look like valid CSS—only the listed options pass validation.
   - **`visible_if`** — Settings gated by `visible_if` may still be validated even when hidden. Prefer omitting them entirely rather than setting values that won't take effect (e.g. don't set `font_size` when `type_preset != 'custom'`).
3. **Presets** — Recommended configurations.

### Step 6 — Apply theme design controls

Prefer schema-defined design settings over arbitrary `custom_css`. Read the relevant schema first and only emit setting keys and values that exist in the target theme.

**Global design settings (`config/settings_data.json`):**

- Use global settings for broad storefront-wide changes, such as site-wide colors, typography systems, spacing defaults, radii, and border defaults.
- Before editing global values, read `config/settings_schema.json` and `config/settings_data.json`. Preserve the existing `current` object shape and only update keys that exist in the schema.
- Do not create arbitrary color scheme IDs, utility class values, or token names. Use existing settings data and schema-listed options.
- Use global changes only when the requested design should affect the whole storefront. For one page, one template, or one section, prefer section/block settings.

**Section settings (`sections/*.liquid`):**

- Use section settings for broad bands of layout and styling: spacing, width, margins, colors, borders, alignment, visibility, behavior, and background media when the section schema supports them.
- Select values must be exact schema options. Do not invent pixel strings, CSS utility strings, animation names, or class names.
- If a section exposes custom CSS, entries must be complete CSS rules with selectors, e.g. `.rte { text-transform: uppercase; }`. Do not use standalone declarations or selectorless rules.

**Block settings (`blocks/*.liquid`):**

- Use block settings for local visual adjustments inside a section: spacing, colors, borders, shape, typography, layout, visibility, inheritance, and animation when the block schema supports them.
- Respect inheritance settings when a block should use parent section/container styles. Only override local colors, typography, or borders when the destination block schema declares those settings and the design requires it.
- Typography, layout, and animation settings vary by block. Validate every setting ID, range step, select option, and enum-like string against the destination block schema.
- Layout container blocks often have restricted child rules. Follow each parent block's schema for allowed children, and copy nesting patterns only from the same target theme or a verified matching bundled example.
- Do not copy a setting from one block type into another unless that destination block schema declares the same setting ID and accepts the same value.

### Step 7 — Compile JSON structure

Follow Shopify’s JSON template shape. If the theme ships **Cursor rules** (e.g. `.cursor/rules/templates.mdc`, `blocks.mdc`, `schemas.mdc`), follow those for the project you are editing.

**Template structure:**

```json
{
  "sections": {
    "<sectionId>": {
      "type": "<sectionType>",
      "settings": {},
      "blocks": {
        "<blockId>": {
          "type": "<blockType>",
          "name": "t:blocks.block_name",
          "settings": {},
          "blocks": {},
          "block_order": []
        }
      },
      "block_order": ["<blockId>"]
    }
  },
  "order": ["<sectionId>"]
}
```

**Block instance IDs:**

- Use descriptive prefixes that reflect role (e.g. layout, container, item, image, text)—match conventions you see in the theme’s existing JSON.
- Do **not** start block instance IDs with `_`. Private block **types** may start with `_` (for example, `"type": "_grid-products"`), but the JSON object key for that block must be a valid instance ID such as `"grid_products_best"`.
- Append a short unique suffix if needed.
- Keep IDs unique within the template (and within each nested `blocks` scope per schema rules).

**Static blocks and `block_order`:**

- Static blocks (`"static": true`) are declared in the parent `blocks` object but must **not** be listed in that parent's `block_order`.
- `block_order` is only for dynamic/reorderable sibling blocks. If a parent contains only static child blocks, use `"block_order": []` or omit it if the existing template pattern allows omission.
- Before finishing, compare every sibling `block_order` against sibling `blocks`: each listed ID must exist and must not point to a block object with `"static": true`.

**Settings (block-based themes):**

- Layout blocks often expose settings for column counts, rows, gaps, padding, and responsive behavior. Use only the setting IDs and values declared by the target schema.
- Prefer translation keys for block `name` when the theme does: e.g. `"t:blocks.image"`.
- Align color and spacing settings with the theme’s design system.

**Richtext content rules:**

Shopify sanitizes HTML in `richtext` and `text` (rich) settings. Only a limited set of tags and attributes are allowed:

- **Allowed tags:** `p`, `h1`–`h6`, `ul`, `ol`, `li`, `a`, `br`, `strong`, `em`, `span`
- **No `style` attributes** — `<span style="color: red">` will be stripped. Use `<em>` or `<strong>` tags instead, then add CSS rules in the section/block stylesheet targeting those tags (e.g. `.my-section em { color: var(--accent); font-style: normal; }`).
- **No `class` attributes** on inline elements in richtext.
- If the design requires colored text within a single text block, this is a **CSS customization gap**—note it and add a stylesheet rule rather than using inline styles.

### Step 8 — Validate

Run the **Validation checklist** below before finishing.

For JSON template edits, also run JSON-focused Theme Check from the **theme root**:

```sh
shopify theme check --config .json-check.yml --fail-level error --output json --no-color
```

Inspect the JSON output for offenses whose `path` is the changed template (for example, `templates/index.json`) and fix those before finishing. Do not use `--path templates/index.json` to check a single file: Shopify CLI treats `--path` as the theme root and will look for sibling theme directories like `locales/` under that path. If unrelated files report warnings or errors, do not change them unless they block the requested template upload.

Some JSON-template validations are enforced by Shopify only during upload (for example, invalid custom CSS values, range-step violations, invalid block IDs, and static blocks listed in `block_order`). When a store connection is available, run an upload-scoped validation after local checks:

```sh
shopify theme push --strict --only templates/index.json
```

This is not a dry run; it uploads the specified file if validation passes. Use it only when uploading that template is acceptable. If it fails, copy the exact console error into the validation checklist and add a local guard for that class of error before retrying.

When `npm run dev` or `shopify theme dev` is already running, use its upload log as the practical validation loop for JSON template edits:

1. Save or touch the edited template so the dev server attempts to sync it.
2. Inspect the dev terminal for `Failed to upload file "templates/<name>.json" to remote theme`.
3. Treat the message below that line as authoritative Shopify validation, even if Theme Check passed.
4. Fix the specific class of error and add it to this skill/checklist if it is not already covered.
5. Repeat until the dev terminal reports a successful sync or no new upload error for the template.

Do not start a duplicate dev server if one is already running. Prefer the existing terminal output. Known upload-only JSON template errors to guard locally include:

- `Invalid CSS value`: custom CSS array entries must be complete CSS rules with selectors, not standalone declarations.
- `Setting '<id>' must be a step in the range`: range settings must use exact `min + (N * step)` values, including settings hidden by `visible_if`.
- `'<id>' is not a valid block id`: block instance IDs must not use invalid characters or leading `_`.
- `static block with id '<id>' must not be present in 'block_order'`: remove static block IDs from sibling `block_order`.

- JSON is valid
- Section and block instance IDs are unique in scope
- Block instance IDs do not start with `_`; only block `type` strings may use private `_` prefixes
- Every `block_order` matches the keys in its sibling `blocks` object
- No block with `"static": true` appears in its sibling `block_order`
- Top-level `order` lists every section key to render
- Each `type` exists in `sections/` or `blocks/` and is allowed by the parent schema
- Nested block types are valid for the parent
- Setting keys and value types match `{% schema %}`
- Range values land on valid steps (`min + N*step`) for every range setting present in JSON, including values hidden by `visible_if`
- Select values match an `options[].value` exactly
- Images from source files are ignored; use empty image/media blocks or sections unless explicit storefront asset values were provided
- No `style` or `class` attributes in richtext content strings
- `_`-prefixed blocks are explicitly listed (not just `@theme`) in every parent they nest inside

## Error handling


| Problem                 | What to do                                                       |
| ----------------------- | ---------------------------------------------------------------- |
| Block/section not found | Search `blocks/` and `sections/`; suggest close matches          |
| Invalid nesting         | Re-read parent `{% schema %}` `blocks` array                     |
| `_` block not allowed   | Block has `_` prefix; add it explicitly to parent schema `blocks` array—`@theme` won't match it |
| `style` attr stripped   | Shopify sanitizes richtext; use `<em>`/`<strong>` + CSS instead of inline styles |
| Range step violation    | Read the schema `step` value; round your value to the nearest valid step       |
| Invalid select value    | Only use values from the schema `options` array; don't invent CSS expressions  |
| Mockup image copied     | Remove screenshot-derived image values; preserve the layout with valid empty image/media blocks |
| Schema too large        | Copy patterns from a working template in the same theme          |
| Ambiguous request       | Ask which template file and which section/block instances change |
| Generated theme output  | Edit source per theme docs, then build                           |


## Further reading


| File                                                   | Purpose                                             |
| ------------------------------------------------------ | --------------------------------------------------- |
| [reference/shopify.md](reference/shopify.md) | Official docs links, filenames, platform limits     |
| [examples/README.md](examples/README.md)               | How to pick a bundled theme example; index of files |
| [docs/adding-theme-support.md](docs/adding-theme-support.md) | Theme partners: add bundled block standards under `examples/` |


In a **theme repository**, also use project rules such as `templates.mdc`, `blocks.mdc`, and `schemas.mdc` when present under `.agent/rules/`.

## Sources

These are the Shopify documentation pages this skill's content is derived from. When Shopify updates these pages, review the corresponding sections above (indicated by `<!-- source: ... -->` comments).


| Shopify doc                                                                                         | Sections in this file                                  |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| [Theme architecture](https://shopify.dev/docs/storefronts/themes/architecture)                      | When to use this skill                                 |
| [JSON templates](https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates) | Step 4 — Compile JSON structure, Validation checklist  |
| [Sections](https://shopify.dev/docs/storefronts/themes/architecture/sections)                       | Discover allowed block types, Step 3 — Analyze schemas |
| [Section schema](https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema)  | Discover allowed block types, Step 3 — Analyze schemas |
| [Blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks)                           | Discover allowed block types                           |
| [Theme blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks)        | Discover allowed block types, Step 3 — Analyze schemas |
