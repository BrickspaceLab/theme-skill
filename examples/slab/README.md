# Slab

This file holds bundled knowledge for the Slab Shopify theme. 



## Block reference


| Role              | Typical block types                                                                                                                                  |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Layout**        | `layout__grid` (children: `_g__grid-item`), `layout__flex` (`_g__flex-item`), `layout__slider` (`_g__slider-item`), `layout__inline`, `g__container` |
| **Content**       | `image`, `g__button`, `richtext`, `g__product-card`, `g__article-card`, `g__collection-card`                                                         |
| **Item wrappers** | `_g__grid-item`, `_g__flex-item`, `_g__slider-item`                                                                                                  |




## Examples

### 3-column grid with image and button per column

Use this when you want equal-width columns driven by `layout__grid` with `row_desktop: 3`, repeating the same block pattern in each `_g__grid-item`.

**Structure:**

```
layout__grid (row_desktop: 3)
  └── _g__grid-item (×3)
      └── image
      └── g__button
```

**Illustrative nested block JSON** — [three-column-grid-image-button.json](three-column-grid-image-button.json) (verify types/settings in the target theme).



### Flex row with multiple items

Use this for row-based layouts where each column’s width is controlled on `_g__flex-item` (for example fractional or custom widths) instead of an implicit grid.

**Structure:**

```
layout__flex (direction: flex-row)
  └── _g__flex-item (×3)
      └── image
      └── richtext
```

**Illustrative nested block JSON** — [flex-row-multiple-items.json](flex-row-multiple-items.json) (verify types/settings in the target theme).



### Nested containers

Use this when a single grid or flex cell should wrap a shared inner frame (`g__container`) so spacing and grouping apply to several blocks together.

**Structure:**

```
_g__grid-item
  └── g__container (optional)
      └── image
      └── richtext
      └── g__button
```

**Illustrative nested block JSON** — [nested-containers.json](nested-containers.json) (verify types/settings in the target theme).



### 3-column flex, nested image grid, footer row

Two stacked flex layouts under one `section`—a main row with custom column widths (navigation column, bio column, wide column with stacked richtext and a nested two-cell image grid) and a footer row with a top border and split left/right richtext.

**Structure:**

```
section (type: section)
  ├── layout__flex (layout_flex_main_3col)
  │   ├── _g__flex-item (flex_item_nav_left)
  │   │   └── richtext
  │   ├── _g__flex-item (flex_item_bio_center)
  │   │   └── richtext
  │   └── _g__flex-item (flex_item_right_content)
  │       ├── richtext
  │       └── layout__grid (layout_grid_images)
  │           ├── _g__grid-item (grid_item_img_left) → image
  │           └── _g__grid-item (grid_item_img_right) → image
  └── layout__flex (layout_flex_footer_row)
      ├── _g__flex-item (flex_item_footer_left) → richtext
      └── _g__flex-item (flex_item_footer_right) → richtext
```

**Full section JSON** — [three-column-flex-nested-image-grid-footer-row.json](three-column-flex-nested-image-grid-footer-row.json) (verify types/settings in the target theme)

