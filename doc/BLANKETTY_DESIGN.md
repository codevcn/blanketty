---
version: 1.0
name: Blanketty-editorial-minimalism
description: An editorial, premium, and minimalist eCommerce design system tailored for Blanketty. It emphasizes extreme whitespace, large-scale typography, and staggered micro-animations to create a high-end fashion/lifestyle boutique feel. The design relies heavily on off-white canvas backgrounds with stark black ink text and muted accents, completely avoiding generic shadows and clunky borders in favor of a clean, structured grid system.

colors:
  primary: "#0a0a0a"
  primary-active: "#1f1f1f"
  primary-disabled: "#e5e5e5"
  ink: "#0a0a0a"
  body: "#3a3a3a"
  body-strong: "#1a1a1a"
  muted: "#6a6a6a"
  muted-soft: "#9a9a9a"
  hairline: "#e5e5e5"
  hairline-soft: "#f0f0f0"
  canvas: "#fffaf0"
  surface-soft: "#faf5e8"
  surface-card: "#f5f0e0"
  surface-strong: "#ebe6d6"
  surface-dark: "#0a1a1a"
  surface-dark-elevated: "#1a2a2a"
  on-primary: "#ffffff"
  on-dark: "#ffffff"
  on-dark-soft: "#a0a0a0"
  success: "#22c55e"
  warning: "#f59e0b"
  error: "#ef4444"

typography:
  display-xl:
    fontFamily: "\"Plain Black\", Inter, -apple-system, BlinkMacSystemFont, \"Segoe UI\", Roboto, sans-serif"
    fontSize: 72px
    fontWeight: 500
    lineHeight: 1
    letterSpacing: -2.5px
  display-lg:
    fontFamily: "\"Plain Black\", Inter, sans-serif"
    fontSize: 56px
    fontWeight: 500
    lineHeight: 1.05
    letterSpacing: -2px
  display-md:
    fontFamily: "\"Plain Black\", Inter, sans-serif"
    fontSize: 40px
    fontWeight: 500
    lineHeight: 1.1
    letterSpacing: -1px
  title-lg:
    fontFamily: "Inter, sans-serif"
    fontSize: 24px
    fontWeight: 600
    lineHeight: 1.3
    letterSpacing: -0.3px
  title-md:
    fontFamily: "Inter, sans-serif"
    fontSize: 18px
    fontWeight: 600
    lineHeight: 1.4
    letterSpacing: 0
  body-md:
    fontFamily: "Inter, sans-serif"
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.55
    letterSpacing: 0
  body-sm:
    fontFamily: "Inter, sans-serif"
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.55
    letterSpacing: 0
  caption-uppercase:
    fontFamily: "Inter, sans-serif"
    fontSize: 12px
    fontWeight: 600
    lineHeight: 1.4
    letterSpacing: 1.5px
  button:
    fontFamily: "Inter, sans-serif"
    fontSize: 14px
    fontWeight: 600
    lineHeight: 1
    letterSpacing: 0

rounded:
  xs: 6px
  sm: 8px
  md: 12px
  lg: 16px
  xl: 24px
  pill: 9999px
  full: 50%

spacing:
  xxs: 4px
  xs: 8px
  sm: 12px
  md: 16px
  lg: 24px
  xl: 32px
  xxl: 48px
  section: 96px

components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.button}"
    rounded: "{rounded.pill}"
    padding: 18px 32px
    transition: "transform 0.3s cubic-bezier(0.16, 1, 0.3, 1), background-color 0.2s"
    hover: "Scale up slightly (1.02), background dims to primary-active"
  
  feature-card-border:
    backgroundColor: "{colors.surface-card}"
    textColor: "{colors.ink}"
    typography: "{typography.body-sm}"
    rounded: "{rounded.md}"
    border: "1px solid {colors.hairline}"
    padding: "{spacing.xl}"

  color-swatch-variant:
    shape: "Circle ({rounded.full})"
    size: "32px x 32px"
    border: "1px solid {colors.hairline}"
    active-state: "2px solid {colors.ink} wrapping the outer edge with padding"
    
  quantity-selector:
    backgroundColor: "transparent"
    textColor: "{colors.ink}"
    border: "1px solid {colors.hairline}"
    rounded: "{rounded.pill}"
    padding: "4px 8px"
    display: "flex / inline-flex with +/- buttons and center input"

---

## Overview

**Blanketty** adopts a high-end, premium editorial aesthetic often found in luxury lifestyle and fashion brands. The foundation relies on a warm, off-white **cream canvas** (`{colors.canvas}` — #fffaf0) contrasted sharply with deep black ink (`{colors.ink}`). It consciously avoids generic e-commerce tropes such as heavy drop shadows, boxed layouts, and overly bright, multiple accent colors.

Instead, Blanketty builds "voltage" (visual impact) through:
1. **Extreme Whitespace:** Huge padding blocks (`{spacing.section}`) between sections and content grids.
2. **Editorial Typography:** Strikingly large display fonts with tight negative letter-spacing paired with clean `Inter` running text.
3. **Staggered Micro-Animations:** Content blocks fade and slide up sequentially (staggered delay) as they enter the viewport, creating a cinematic, breathable rhythm.

**Key Characteristics:**
- Warm canvas background (`#fffaf0`) instead of stark `#ffffff`.
- Primary buttons are solid black (`#0a0a0a`), fully rounded (pill-shaped), and scale up elegantly on hover.
- Variant selectors emphasize visual swatches (circular colors) over clunky dropdowns or rectangular text boxes.
- "Add to Cart" logic uses seamless AJAX without reloading the page, offering immediate feedback through button text changes ("Đang xử lý...", "Đã thêm ✓").
- Sections are cleanly divided by 1px hairlines (`{colors.hairline}`) rather than drop shadows.

## Colors

### Surface & Canvas
- **Canvas** (`{colors.canvas}` — #fffaf0): The foundational background color. It provides a soft, organic feel appropriate for lifestyle/home goods.
- **Surface Strong/Card** (`{colors.surface-card}`): Used for cards or highlighted blocks (like the full-width description box) to gently separate them from the canvas without relying on aggressive borders.
- **Hairline** (`{colors.hairline}` — #e5e5e5): The absolute standard for borders, dividers, and card outlines.

### Text
- **Ink** (`{colors.ink}` — #0a0a0a): Main headings, product titles, prices.
- **Muted** (`{colors.muted}` — #6a6a6a): Secondary text, product descriptions, shipping details.
- **Caption Uppercase**: Often used for meta-labels (like "TỪ KHÓA", "VẬN CHUYỂN", variant option names) to provide structural hierarchy.

## Typography

The system utilizes a split-font strategy:
- **Display Headings:** "Plain Black" (or equivalent elegant sans-serif) at heavy sizes (40px to 72px) with negative letter spacing (`-1px` to `-2.5px`). This gives the site its bold, magazine-like header presence.
- **Body & UI:** "Inter" at weight 400 for descriptions, and weight 600 for buttons/labels.

## Layout & Whitespace

Whitespace is not empty space; it is the core structural element of Blanketty.
- **Product Split Layout:** Desktop utilizes a `1fr 1fr` grid. The left side (Gallery) can be sticky or heavily padded. The right side (Product Info) is padded generously (`padding-left: {spacing.section}`).
- **Spacing Scale:** Blanketty avoids tight margins. The gap between a product title and price might be `{spacing.md}`, but the gap between the Add to Cart block and the description is `{spacing.xl}` or greater.

## Components & UI Patterns

### Buttons
Primary buttons (`.btn-primary`) are non-negotiable solid ink (`#0a0a0a`). They feature a pill shape (`border-radius: 9999px`) and rely on a smooth CSS transform (`scale(1.02)`) on hover rather than box-shadows.

### Variant Selectors (Colors)
Color options are rendered as perfect circles. When selected, they gain an outer black ring (via `box-shadow` or `border` with padding), mimicking high-end cosmetic or Apple-style hardware selectors.

### Info Cards (Shipping / Tags)
Information that supports the buying decision (like Shipping or Keywords) is neatly organized into a 2-column grid (`.pdp-info-cards`). Each card has a minimal hairline border, rounded corners (`{rounded.md}`), an elegant SVG icon, and uppercase caption titles.

### Full-Width Description
Instead of cramping long paragraphs into the right-hand column, Blanketty moves extensive descriptions into a dedicated full-width section below the fold. The content is constrained to a readable `max-width: 800px`, padded generously, and wrapped in a hairline border to form a distinct "Editorial Card".

## Do's and Don'ts

### Do
- Use the cream canvas (`#fffaf0`) as the default background.
- Rely on `{colors.hairline}` for borders and dividers.
- Implement staggered animations (`.staggered-item`) for any list of elements or product info blocks.
- Use `caption-uppercase` (12px, 600 weight, 1.5px tracking) for small labels and metadata titles.
- Implement AJAX for all cart interactions.

### Don't
- Don't use heavy drop-shadows on cards or buttons.
- Don't use standard generic blue/green colors for CTAs; stick to Ink black.
- Don't cram text; if a description is too long, move it to a dedicated wide layout.
- Don't use square buttons; maintain the pill (`9999px`) or slightly rounded (`12px`) shapes.
