# x-kom desktop design reference

Source: [x-kom homepage](https://www.x-kom.pl/), inspected on 2026-11-05 in visible Chrome through Playwright CLI. Viewport: 1440 x 1000 CSS pixels. This is an extracted implementation reference, not an official x-kom brand manual.

## Assets

| Asset | Local file | Source / purpose |
| --- | --- | --- |
| Design tokens | [design-tokens.json](../assets/design-tokens.json) | Semantic tokens, component measurements and aggregate observations |
| Homepage screenshot | [homepage.png](../assets/homepage.png) | Full page, optional cookies rejected, carousel paused, lazy images loaded |
| Logo | [logo.svg](../assets/logo.svg) | Visible header SVG with tagline; inherited color variables embedded for standalone rendering |
| Favicon | [favicon.ico](../assets/favicon.ico) | Site favicon fetched in the browser context |
| Instrument Sans variable font | [InstrumentSans-VariableFont_wdth,wght.woff2](../assets/fonts/instrument-sans/InstrumentSans-VariableFont_wdth,wght.woff2) | Site-hosted variable font, normal style, weight 400-700 |

Logo and font rights must be verified before use or redistribution. Downloading these assets does not establish a license. Campaign artwork and product photography in the screenshot are visual references, not separately supplied reusable assets.

## Colors

Names below are inferred from measured usage. They do not establish official brand semantics.

| Token | Value | Observed usage |
| --- | --- | --- |
| `brand.primary` | `#222222` | Purchase/search buttons and the campaign information bar |
| `brand.accent` | `#FA0064` | Bright pink in the logo; do not apply it to every primary action |
| `brand.success` | `#007D00` | Discount code labels and green promotional price text; form success states were not inspected |
| `background.default` | `#FFFFFF` | Header, content panels and product surfaces |
| `background.light` | `#F7F7F7` | Main page surface around white sections |
| `background.subtle` | `#F2F2F2` | Search field and neutral badges |
| `background.success` | `#EEF7ED` | Discount code badge fill |
| `text.primary` | `#1A1A1A` | Body copy, navigation and section headings |
| `text.secondary` | `#444444` | Secondary labels and utility controls |
| `text.muted` | `#6E6E6E` | Supporting metadata |
| `text.onDark` | `#FFFFFF` | Filled actions and dark campaign bar |
| `text.link` | `#1C73E7` | Link-colored descendants; top-level category links use primary text |
| `border.default` | `#CCCCCC` | Observed neutral borders |
| `border.onDark` | `rgba(255, 255, 255, 0.48)` | Campaign outline action |

Logo-specific colors: cyan `#0FD2FF`, blue `#0082FA`, violet `#8250FA`, pale violet `#C382FA`, magenta `#BE0064`, pink `#FA0064`. Keep these within the multicolor mark unless another application is justified by the reference.

`brand.error` and `background.overlay` are `null`: no relevant visible state was inspected. Do not substitute campaign pink for an error token. Colors embedded in seasonal banners are campaign artwork rather than global UI tokens.

## Typography

Primary stack: `"Instrument Sans", sans-serif`. The site declares a variable `@font-face` in normal style with weight range `400 700`, `font-display: swap`, and `font-variation-settings: "wdth" 100`.

Downloaded file: `assets/fonts/instrument-sans/InstrumentSans-VariableFont_wdth,wght.woff2` (88,476 bytes). Source: `https://assets.x-kom.pl/public/xkom/assets/fonts/InstrumentSans-VariableFont_wdth,wght.woff2`.

Verify its license before using or redistributing it.

Example CSS, with the URL resolved relative to the consuming stylesheet or its asset pipeline:

```css
@font-face {
  font-family: "Instrument Sans";
  src: url("../assets/fonts/instrument-sans/InstrumentSans-VariableFont_wdth,wght.woff2") format("woff2");
  font-style: normal;
  font-weight: 400 700;
  font-display: swap;
}

body {
  font-family: "Instrument Sans", sans-serif;
  font-variation-settings: "wdth" 100;
}
```

| Weight | Role / evidence |
| --- | --- |
| 400 | Body, navigation, product names, buttons |
| 500 | Supported by the variable font; not observed in sampled visible text |
| 600 | Section headings and emphasis |
| 700 | Supported by the font and measured on the logo's H1 wrapper; not a verified content-heading rule |

| Size | Role / evidence | Measured line height |
| --- | --- | --- |
| 10px | Small auxiliary labels | Not assigned a global line height |
| 12px | Fine print | Not assigned a global line height |
| 13px | Badges and compact actions | 16px |
| 14px | Body, product names, search input | 20px |
| 16px | Category navigation and purchase action | 22px |
| 18px | Emphasized text | Varies by component |
| 20px | Larger emphasis | Varies by component |
| 24px | Large text in sampled elements | Varies by component |
| 26px | Section headings such as "Hity tygodnia" | 32px, weight 600 |

The logo's H1 wrapper computes to 28px/700 but contains the SVG rather than a text title. Do not derive a text H1 style from that wrapper. The body itself reports `line-height: normal`; the 20px base line height comes from actual text components.

## Spacing

A 4px base is a useful inferred organization of the observed measurements. Common values: 4, 8, 12, 16, 20, 24 and 32px; 48px also appears as a gap. Compact exceptions include 2px and 6px gaps. The tokens retain observed exceptions rather than forcing everything onto a single grid.

- Content section padding: 24px.
- Category menu item padding: 8px 12px.
- Purchase button padding: 12px 24px; icon/text gap: 4px.
- Search input padding: 8px 0 8px 20px.
- Campaign outline action padding: 5px 11px.
- Compact discount label padding: 2.5px 4px 1.5px.

## Border radius

| Token | Value | Usage |
| --- | --- | --- |
| `none` | 0px | Structural containers and ordinary text elements |
| `xs` | 2px | Small details observed in the page; not a universal control radius |
| `sm` | 4px | Compact badges |
| `md` | 8px | Buttons, search container and smaller tiles |
| `lg` | 12px | White section panels and larger surfaces |
| `circle` | 100% | Circular category imagery / details |

Asymmetric 8px and 12px corners also occur on joined components. Use the standalone component's measured radius rather than applying all corner values globally. A 16px or pill radius was not established by this extraction.

## Components

### Header and category navigation

The header combines the logo, wide search field and line-icon account, help, favorites and basket controls on white. The measured main row is 72px high; the separately measured category navigation is 58px high. Navigation labels use 16px/22px, weight 400, with 8px 12px item padding. This measurement is for the inspected desktop layout, not a responsive breakpoint contract.

### Campaign bar and hero

The current Black Weeks information bar is 40px high, with `#222222` fill and white 14px/20px copy. Its outline link uses 13px/16px text, 8px radius and a translucent white border. Keep the full-width photographic campaign carousel distinct from the neutral interface around it. Its artwork changes over time; the screenshot records one paused slide.

### Buttons

The visible "Kup teraz" action uses `#222222` background, white text, 8px corners, 12px 24px padding and 16px/22px regular text. It has no border or box shadow. Compact utility buttons use line icons, often with no text line box; their computed `line-height: 0px` is not a text-button typography rule. Hover, disabled, focus and pressed states were not measured.

### Inputs

The search control's outer container uses `#F2F2F2`, 8px corners and a 40px height. Its inner input has a transparent background, no own border and 14px/20px text. The search icon action uses the dark fill. Preserve the distinction between the container and inner input when applying tokens. The newsletter input has 8px corners and 9px 19px inner padding; validation behavior was not tested.

### Content panels and product cards

The "Hity tygodnia" section is a white panel with 12px corners and 24px padding, without a drop shadow. Product imagery sits on light surfaces, followed by compact names, metadata and price text. Section headings use 26px/32px semibold text. The design relies on surface separation, spacing and typography; use borders where observed instead of adding strong shadows.

### Badges

Discount-code badges use `#EEF7ED` fill, `#007D00` text, 4px corners and 13px/16px typography. Neutral labels use `#F2F2F2` with `#222222` text. Do not infer validation semantics from these promotional treatments.

## Logo usage

Use [the saved SVG](../assets/logo.svg) as an image with accessible alternative text such as "x-kom - Technologia po prostu". The inspected desktop mark, including its tagline, occupies approximately 218 x 40 CSS pixels; preserve its intrinsic `217.77 38` viewBox and aspect ratio. The SVG includes the inherited logo colors so it renders independently of the source site's stylesheet.

Place this saved black-lettered variant on a light surface. No inverted variant was captured or verified. Clear-space and minimum-size rules were not published by this extraction; keep enough surrounding space for legibility without claiming an official brand requirement.

## Visual style summary

The interface feels practical and technology focused, with compact product information and clear purchase actions. Neutral white and pale gray surfaces give the multicolor logo and seasonal campaign artwork room to stand out. Rounded panels soften a dense retail layout, while dark controls and restrained line icons maintain a clear hierarchy. Instrument Sans connects the small labels, navigation and larger section headings through a consistent typographic voice.

## Verification and limits

- The live homepage loaded successfully in headed Chrome. No challenge or access-denied page was used.
- The delayed cookie dialog was dismissed with "Odrzucam niewymagane"; the final check found no visible consent dialog. The normal campaign strip and collapsed chat launcher remain part of the page.
- The carousel was paused. Scrolling through the page triggered lazy images before the full-page screenshot was saved and visually checked.
- Core styles, visible palette/components and asset/font discovery were extracted in separate browser evaluations. Hidden and zero-size elements were excluded from palette/component selection.
- The favicon and site-hosted Instrument Sans file were fetched from the page context and decoded with the skill's guarded helper. The logo was copied from the visible header SVG, with inherited color variables preserved.
- The source page logged pre-existing resource errors, including analytics/ad requests, and a warning. This is a visual extraction, not a claim that the external site passes functional or console QA.
- Mobile layouts, responsive breakpoints, dark mode, form errors, checkout flows and interactive state colors were not inspected. Null or explicitly qualified values must not be treated as measured design rules.
