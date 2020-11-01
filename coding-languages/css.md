---
title: CSS - The Cascade, the Box Model, and Modern Layout
description: CSS separates how a page looks from what it contains. How the browser turns CSS into pixels, the selector types, the cascade and specificity rules that resolve conflicts, the box model, and the modern layout systems - Flexbox for one dimension, Grid for two - plus preprocessors, BEM, and native custom properties.
date: 2020-11-01
draft: false
slug: /coding-languages/css
tags:
  - Programming Languages
  - CSS
  - Web
---

If [HTML](/citadel/coding-languages/html) is the structure of a page, **CSS (Cascading Style Sheets)** is its appearance: colours, fonts, spacing, layout, animation. The core idea is **separation** — content and structure in HTML, presentation in CSS — which keeps content readable if styles fail to load, lets many pages share one stylesheet, and makes responsive layouts possible. CSS is more than a list of colours, though; it has its own rules for which style wins when several apply, and its own layout systems. This post covers those.

## How CSS becomes pixels

Visiting a page, the browser runs a **rendering pipeline**:

1. **Parse HTML** into the **DOM tree** — one node per element.
2. **Fetch and parse CSS** into the **CSSOM** — a tree of the styles that apply to elements. CSS files are static assets, usually served from a [CDN](/citadel/interview/cdn).
3. **Combine** DOM and CSSOM into the **render tree** — only the nodes that will actually display (`display: none` is excluded), each with its computed styles.
4. **Layout (reflow)** — compute the exact position and size of every visible element.
5. **Paint** — fill in pixels: colours, backgrounds, borders, text.
6. **Composite** (when needed) — for layered content (`transform`, `opacity`, animations), paint onto separate layers and let the GPU combine them.

See [frontend performance](/citadel/interview/fe-performance) for why layout and paint cost matters.

## Selectors, properties, values

A rule is a selector plus a declaration block:

```css
/* selector { property: value; } */
p.important-text {
  color: navy;
  font-weight: bold;
}
```

**Selector types:**

- **Type / element** — `p` (all paragraphs).
- **Class** — `.my-button` (elements with `class="my-button"`).
- **ID** — `#main-header` (the one element with that id).
- **Attribute** — `input[type="text"]`.
- **Pseudo-classes** — state or position: `:hover`, `:focus`, `:nth-child(n)`, `:first-child`.
- **Pseudo-elements** — parts of an element: `::before`, `::after`, `::first-letter`.
- **Combinators** — relationships: descendant (`div p`), child (`ul > li`), adjacent sibling (`h1 + p`), general sibling (`h1 ~ p`).

A **declaration** is a **property** (`color`, `margin`, `border`) and a **value** (`red`, `16px`, `10px auto`, `solid 1px black`).

## The cascade, specificity, inheritance

The "cascading" part decides which rule wins when several target the same element.

- **Cascade** — resolves conflicts using, in order:
  1. **Origin and importance** — browser default, user stylesheet, author stylesheet; `!important` outranks normal rules. Author `!important` usually wins.
  2. **Specificity** — among rules of the same origin, the more specific selector wins.
  3. **Source order** — same origin and specificity: the one written later wins.
- **Specificity** — a score per selector: inline styles highest, then IDs, then classes / attributes / pseudo-classes, then types / pseudo-elements. The universal selector `*` and combinators add nothing.
- **Inheritance** — some properties pass from parent to child by default (`font-family`, `color`, `font-size`, `line-height`); box properties (`border`, `padding`, `margin`, `width`) do not.

## The box model

Every element is a rectangular box with four parts, from the inside out:

- **Content** — text and images, sized by `width` and `height`.
- **Padding** — transparent space between content and border.
- **Border** — a line around the padding.
- **Margin** — transparent space outside the border, between this element and others.

By default `width` sets the content box only, so padding and border add to the visible width. `box-sizing: border-box` makes `width` include content, padding, and border, which makes layout math predictable — most stylesheets set it globally.

## Modern layout

- **Floats (legacy)** — originally for wrapping text around images and faking columns. Fine for text wrap; not recommended for page layout now (clearfix hacks, fragile).
- **Positioning** — `position: static | relative | absolute | fixed | sticky` for precise placement, often outside normal flow.
- **Flexbox** — **one-dimensional** layout, a row *or* a column. Distributes space and aligns items along a main axis and cross axis: `display: flex`, `flex-direction`, `justify-content`, `align-items`, `flex-wrap`, `flex-grow`, `flex-shrink`, `flex-basis`.
- **CSS Grid** — **two-dimensional** layout, rows *and* columns at once: `display: grid`, `grid-template-columns`, `grid-template-rows`, `grid-gap`, `grid-column`, `grid-row`, `grid-area`.
- **Responsive design** — fluid (percentage) grids, flexible images, and **media queries** (`@media`) that apply different styles based on viewport width, height, orientation, or resolution.

## Managing large stylesheets

- **Preprocessors** (Sass, Less, Stylus) — scripting layers compiled to plain CSS, adding variables, nesting, mixins, and functions for more maintainable, DRY styles.
- **Methodologies** (BEM, SMACSS, OOCSS) — naming and structure conventions to reduce conflicts and specificity wars on large teams. BEM names classes `block__element--modifier` (`.card__title--highlighted`), making relationships explicit and selectors flat.
- **Modern native CSS:**
  - **Custom properties** — `--brand: #06c;` used as `var(--brand)`; dynamic (readable and settable from [JavaScript](/citadel/coding-languages/javascript)) and good for theming.
  - **Functions** — `calc()`, `min()`, `max()`, `clamp()` for responsive sizing; colour functions.
  - **Selectors** — `:is()`, `:where()` (grouping with different specificity effects), `:has()` (the parent selector).
  - **Transitions and animations** without JavaScript.
  - **Frameworks** — Bootstrap (components and a grid), Tailwind CSS (utility classes), Materialize.

## Key takeaways

- The browser parses CSS into the **CSSOM**, merges it with the DOM into the render tree, then lays out and paints.
- Selectors range from element to `:has()`; when several apply, the **cascade** resolves them by origin/importance, then **specificity**, then source order.
- The **box model** (content, padding, border, margin) plus `box-sizing: border-box` governs sizing.
- Layout is **Flexbox** for one dimension and **Grid** for two, with media queries for responsiveness.
- Preprocessors, BEM, and native custom properties are how large stylesheets stay maintainable.
