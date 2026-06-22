# Gamma-Level Design Principles

These principles define what separates a professional, memorable presentation from a generic template. Apply them in ALL modes: generation, editing, preset creation, and Visual QA. They work alongside `/frontend-design` thinking — these are the presentation-specific layer on top of general design excellence.

## Principle 1: Visual Rhythm (Slide Pacing)

A presentation is a sequence, not a collection. Like music, it needs tempo variation.

**Rule**: Never have more than 2-3 dense information slides in a row without a "breathing" slide — a section divider, a full-screen statement, a dramatic single metric, or a quote slide.

**Implementation**:
- After cover: 1-2 content slides, then a section/statement break
- Before the climax (ask/CTA): insert a dramatic statement or fact slide
- Between major topic changes: always use `section` or `statement` layout
- If the outline has 10+ slides, it MUST have at least 2-3 breathing slides interspersed

**Pattern for breathing slides**:
```
layout: statement  (or fact, or section)
```
With ONE big idea — a quote, a key number, a provocative question. Minimal text, maximum visual impact.

**Section divider minimum weight**: Section divider slides MUST use a heading at minimum 3.5rem (the 3-5em minimum from Principle 3 applies to ALL breathing slide types: `fact`, `statement`, AND `section`). The background MUST be visually distinct from both the preceding and following content slides — at minimum a 10% luminance shift, a different hue treatment, or a color inversion. Decorative elements MUST be at Principle 6 "PASS minimum" opacity or higher. A section slide that looks like a dimmer content slide has failed.

**Section divider minimum contrast delta**: Section slides must differ from adjacent slides by at least one of: (A) background luminance shift of 8%+; (B) color inversion or hue shift; (C) a dominant decorative element (large orb, pattern overlay) that does not appear on adjacent content slides. If the section divider uses the same background family as surrounding slides, it MUST compensate with a heading at 4rem+ to create typographic contrast.

**Section background verification — dark theme anchor values**: Abstract luminance percentages are hard to evaluate. Use these concrete rules for dark presentations:
- For a base background of `#0C0C14`: VALID section backgrounds = `#111828` (lifted navy), `#0a1420` (blue-tint), `radial-gradient(ellipse 80% 80% at 50% 50%, #1a2040 0%, #0C0C14 75%)` (centered glow)
- INVALID: `#0e0e1c` or `#0a0e1a` — less than 5 luminance points difference from `#0C0C14`, indistinguishable at export resolution
- When in doubt: add `radial-gradient(ellipse 80% 80% at 50% 50%, <lifted-color> 0%, <base> 75%)` as the background. Even 8% opacity center brightening creates perceptible differentiation.
- **Multiple section slides rule**: If a presentation has 2+ section slides, they MUST use different color temperatures from each other. Example: Section 01 = lifted navy (`#0e1624`), Section 02 = deeper teal (`#080e14`) — NOT both the same dark navy. The color temperature shift between section slides signals structural progression in the narrative.

### Background Level System

Every presentation defines exactly 3 background levels in `styles/index.css`:

| Variable | Usage | Coverage |
|----------|-------|----------|
| `--bg-base` | Primary background for content slides | ~60% of slides |
| `--bg-alt` | Alternative background for section dividers, alternating content | ~30% of slides |
| `--bg-accent` | Cover and CTA/closing slides | ~10% of slides |

**Rules:**
1. All three levels must share the **same color temperature** (all warm OR all cool — never mixed)
2. Luminance delta between `--bg-base` and `--bg-accent`: maximum 40%
3. Pure `#000000` and `#FFFFFF` are **banned** — always use tinted variants (e.g., `#FAF9F6`, `#1A2332`, `#0C0E14`)
4. If `--bg-accent` luminance < 30% (dark) and `--bg-base` luminance > 70% (light), the **pre-CTA slide** (immediately before the CTA/closing slide) must use `--bg-alt` as a visual bridge

**Slide-type to bg-level mapping:**

| Slide type | Background level |
|------------|-----------------|
| Cover | `--bg-accent` |
| Content (odd) | `--bg-base` |
| Content (even) | `--bg-alt` |
| Section divider | `--bg-alt` |
| Stat/fact hero | `--bg-base` |
| Problem/Solution | `--bg-base` or `--bg-alt` (alternating) |
| Ask/CTA | `--bg-accent` |
| Pre-CTA | `--bg-alt` (bridge) |

Archetypes reference `var(--bg-base)` / `var(--bg-alt)` / `var(--bg-accent)` — no hardcoded background colors.

**Visual QA must verify**: export PNGs of consecutive slides, compare backgrounds. If two adjacent slides have visually identical backgrounds (same hex within ±3 luminance points), flag as CRITICAL and fix before proceeding. Also verify: no pure #000/#FFF, consistent color temperature across all slides.

**Anti-pattern**: 8 consecutive `default` layout slides with the same structure. Also: 12 slides with the exact same `background-color` hex value.

---

## Principle 2: Layout Diversity

Every slide should be visually distinguishable from its neighbors.

**Rule**: No more than 2 consecutive slides may share the same visual structure (same heading position + same content arrangement). Vary across these dimensions:
- Heading position: top-left, centered, large/overlaid
- Content structure: single column, two columns, cards grid, timeline, centered hero, asymmetric split
- Visual weight: text-heavy ↔ visual-heavy ↔ minimal/statement

**Layout rotation toolkit** (use at least 4 different patterns per 10-slide deck):
1. **Hero left, content right**: big number/title on left, details on right
2. **Cards grid**: 3-4 equal cards in a row
3. **Timeline/stepper**: numbered phases flowing left-to-right or top-to-bottom
4. **Centered hero**: one big statement/number centered, supporting text below
5. **Split asymmetric**: 60/40 or 70/30 column split
6. **Full-bleed visual**: background image/gradient with overlaid text
7. **Comparison table**: structured comparison grid
8. **Quote/callout**: large quote with attribution

**Enforcement rule — mandatory heading position rotation:**
In any 10+ slide deck, the following MUST appear at least once each:
- Centered heading (statement/fact slide — doesn't count as "content slide" credit)
- Overlaid/hero title: heading at 3em+ that dominates >40% of slide height (no label above it)
- Heading on RIGHT side of content (not left-aligned), OR heading BELOW a key visual
- At least one content slide with NO label above the heading — the heading speaks for itself

The "LABEL → H1 → content" three-layer pattern may appear on at most 3 consecutive content slides before forcing a structural break (different heading position, a breathing slide, or a layout where the heading is not top-left).

**Mandatory structural variation on every 3rd content slide**: On the 3rd consecutive content slide (and every 3rd thereafter), FORCE one of: (A) centered hero layout — heading centered at 2.8rem+, content in a single centered column below; (B) visual-dominant layout — a large metric or icon occupies the left 40%, heading is placed below or beside it; (C) inverted layout — content precedes heading (key insight or data visualization shown first, title appears at bottom or right). The generator MUST track consecutive content slide count and rotate the structure.

**Anti-pattern**: Every slide following "LABEL top-left → accent line → 2-column grid".

### Rule of Thirds

On non-breathing, non-hero slides, place the dominant element at a thirds-grid intersection rather than dead center. Center alignment is reserved for breathing slides (stat-hero, quote-pull) and cover/CTA slides. At least 30% of content slides should use off-center focal points.

---

## Principle 3: Typographic Drama

Typography creates visual hierarchy. Size contrast is the #1 tool for directing attention.

**Rule**: Each presentation must use at least 3 distinct type scales:
- **Hero scale** (4-8rem): for key numbers, dramatic statements, slide titles on statement/fact slides
- **Heading scale** (1.8-2.5rem): for regular slide titles
- **Body scale** (1.0-1.25rem): for descriptions, bullets, labels

**Implementation**:
- Key metrics ($1.5M, 45 min, +20%) → hero scale, accent color, bold weight
- On fact/statement slides, the main text should be 3-5rem minimum
- Never make everything the same size — if everything is emphasized, nothing is
- Pair font weights dramatically: 700 heading next to 300 body text
- Both heading and body fonts MUST be sans-serif. Contrast through character (geometric vs humanist, condensed vs proportional, angular vs rounded), not through serif/sans category split.

**When a slide has 3-4 key metrics**: DON'T put them all in equal-sized cards at the same scale. Pick the MOST impactful metric and make it hero-sized (centered, 3-6rem), then place the remaining metrics below in a row of smaller supporting cards. If ALL metrics are equally important, use a 2x2 grid with each number at 2.5-3rem minimum — never smaller than the heading text.

**Non-duplication rule for consecutive slides:**
If a fact/statement slide immediately follows a data/traction slide, the two slides MUST NOT feature the same headline metric. Either:
- The data slide shows the full dashboard; the fact slide zooms into the MOST surprising/impressive single number (which may differ from the "biggest")
- Or combine them: use only the fact slide if the metric is strong enough to stand alone

**Supporting metric minimum size:**
On a slide with a hero metric + supporting metrics:
- Hero number: 3.5em+
- Supporting numbers: 2.2em minimum (NEVER smaller than the slide heading)
- Size ratio hero:supporting must be at least 1.6:1 to create clear hierarchy

### Type Treatment Limit

Maximum **3 visually distinct type treatments** per slide:
1. Heading font (regular or bold) — titles, labels
2. Body font (regular) — descriptions, bullets
3. One accent treatment — bold OR italic OR uppercase, but **never all three simultaneously**

FAIL if a single slide contains bold + italic + uppercase + multiple font families all at once ("font salad").

### Italic Restriction

Italic is permitted **only** in:
- `<blockquote>` elements and direct quotations
- Attribution lines ("— Author Name")
- Image captions / footnotes

Italic in headings, subheadings, or body text = **FAIL**.

**Anti-pattern**: All headings at 1.8em, all body at 1em, across every slide. Or: 4 identical metric cards with numbers at 1.5em that are smaller than the heading.

---

## Principle 4: Icon System (No Emoji)

Emoji are inconsistent across platforms and look unprofessional in business presentations.

**Rule**: NEVER use emoji (📱💳⭐🧠 etc.) as visual elements in slides. Instead, use inline SVG icons that match the presentation's visual language.

**Implementation**: Create a `components/Icon.vue` component for each presentation:

```vue
<template>
  <svg :width="size" :height="size" viewBox="0 0 24 24" fill="none"
       :stroke="color" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
    <!-- phone -->
    <g v-if="name === 'phone'">
      <rect x="5" y="2" width="14" height="20" rx="2" />
      <line x1="12" y1="18" x2="12" y2="18.01" />
    </g>
    <!-- map-pin -->
    <g v-if="name === 'map-pin'">
      <path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7z" />
      <circle cx="12" cy="9" r="2.5" />
    </g>
    <!-- credit-card -->
    <g v-if="name === 'credit-card'">
      <rect x="1" y="4" width="22" height="16" rx="2" />
      <line x1="1" y1="10" x2="23" y2="10" />
    </g>
    <!-- star -->
    <g v-if="name === 'star'">
      <polygon points="12,2 15.09,8.26 22,9.27 17,14.14 18.18,21.02 12,17.77 5.82,21.02 7,14.14 2,9.27 8.91,8.26" />
    </g>
    <!-- check -->
    <g v-if="name === 'check'">
      <polyline points="20,6 9,17 4,12" />
    </g>
    <!-- x-mark -->
    <g v-if="name === 'x-mark'">
      <line x1="18" y1="6" x2="6" y2="18" /><line x1="6" y1="6" x2="18" y2="18" />
    </g>
    <!-- arrow-right -->
    <g v-if="name === 'arrow-right'">
      <line x1="5" y1="12" x2="19" y2="12" /><polyline points="12,5 19,12 12,19" />
    </g>
    <!-- arrow-up-right (trend) -->
    <g v-if="name === 'trend-up'">
      <polyline points="23,6 13.5,15.5 8.5,10.5 1,18" />
      <polyline points="17,6 23,6 23,12" />
    </g>
    <!-- users / people -->
    <g v-if="name === 'users'">
      <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
      <circle cx="9" cy="7" r="4" />
      <path d="M23 21v-2a4 4 0 0 0-3-3.87" /><path d="M16 3.13a4 4 0 0 1 0 7.75" />
    </g>
    <!-- dollar-sign -->
    <g v-if="name === 'dollar'">
      <line x1="12" y1="1" x2="12" y2="23" />
      <path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6" />
    </g>
    <!-- target -->
    <g v-if="name === 'target'">
      <circle cx="12" cy="12" r="10" /><circle cx="12" cy="12" r="6" /><circle cx="12" cy="12" r="2" />
    </g>
    <!-- globe -->
    <g v-if="name === 'globe'">
      <circle cx="12" cy="12" r="10" />
      <line x1="2" y1="12" x2="22" y2="12" />
      <path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z" />
    </g>
    <!-- chart-bar -->
    <g v-if="name === 'chart'">
      <line x1="18" y1="20" x2="18" y2="10" /><line x1="12" y1="20" x2="12" y2="4" /><line x1="6" y1="20" x2="6" y2="14" />
    </g>
    <!-- shield -->
    <g v-if="name === 'shield'">
      <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z" />
    </g>
    <!-- zap / lightning -->
    <g v-if="name === 'zap'">
      <polygon points="13,2 3,14 12,14 11,22 21,10 12,10" />
    </g>
    <!-- brain / cpu -->
    <g v-if="name === 'brain'">
      <rect x="4" y="4" width="16" height="16" rx="2" />
      <line x1="9" y1="1" x2="9" y2="4" /><line x1="15" y1="1" x2="15" y2="4" />
      <line x1="9" y1="20" x2="9" y2="23" /><line x1="15" y1="20" x2="15" y2="23" />
      <line x1="20" y1="9" x2="23" y2="9" /><line x1="20" y1="14" x2="23" y2="14" />
      <line x1="1" y1="9" x2="4" y2="9" /><line x1="1" y1="14" x2="4" y2="14" />
    </g>
    <!-- rocket -->
    <g v-if="name === 'rocket'">
      <path d="M4.5 16.5c-1.5 1.26-2 5-2 5s3.74-.5 5-2c.71-.84.7-2.13-.09-2.91a2.18 2.18 0 0 0-2.91-.09z" />
      <path d="M12 15l-3-3a22 22 0 0 1 2-3.95A12.88 12.88 0 0 1 22 2c0 2.72-.78 7.5-6 11a22.35 22.35 0 0 1-4 2z" />
      <path d="M9 12H4s.55-3.03 2-4c1.62-1.08 5 0 5 0" />
      <path d="M12 15v5s3.03-.55 4-2c1.08-1.62 0-5 0-5" />
    </g>
    <!-- car -->
    <g v-if="name === 'car'">
      <path d="M5 17h14M5 17a2 2 0 0 1-2-2V9a2 2 0 0 1 2-2h1l2-3h8l2 3h1a2 2 0 0 1 2 2v6a2 2 0 0 1-2 2" />
      <circle cx="7.5" cy="17" r="1.5" /><circle cx="16.5" cy="17" r="1.5" />
    </g>
    <!-- clock -->
    <g v-if="name === 'clock'">
      <circle cx="12" cy="12" r="10" /><polyline points="12,6 12,12 16,14" />
    </g>
    <!-- message -->
    <g v-if="name === 'message'">
      <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z" />
    </g>
    <!-- lock -->
    <g v-if="name === 'lock'">
      <rect x="3" y="11" width="18" height="11" rx="2" />
      <path d="M7 11V7a5 5 0 0 1 10 0v4" />
    </g>
    <!-- diamond / gem -->
    <g v-if="name === 'gem'">
      <polygon points="12,2 22,8.5 12,22 2,8.5" />
      <line x1="2" y1="8.5" x2="22" y2="8.5" />
      <polyline points="7,2 12,8.5 17,2" />
    </g>
    <!-- data / database -->
    <g v-if="name === 'data'">
      <ellipse cx="12" cy="5" rx="9" ry="3" />
      <path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3" />
      <path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5" />
    </g>
  </svg>
</template>

<script setup>
defineProps({
  name: { type: String, required: true },
  size: { type: [Number, String], default: 24 },
  color: { type: String, default: 'currentColor' },
})
</script>
```

**Icon Style Selection**: The preset format supports an `iconStyle` field: `"outlined"` (default — stroke icons as above), `"filled"` (solid fill, no stroke), `"duotone"` (two-tone with fill + lighter overlay).

- If preset defines `iconStyle: filled`, generate Icon.vue with `fill="currentColor"` and `stroke="none"` instead of the stroke-based template above. Adjust SVG paths accordingly for filled rendering.
- If preset defines `iconStyle: duotone`, generate Icon.vue with primary shapes using `fill` at full opacity and secondary shapes using `fill` at 40% opacity.
- **Thematic specificity**: Icons must be thematically relevant to the presentation topic. A construction proposal needs building, crane, helmet, blueprint icons — not abstract geometric shapes. A tech startup needs rocket, chart, code, cloud icons. Generic Lucide-style icons signal "AI-generated."

**Usage in slides**: `<Icon name="phone" :size="32" color="var(--color-accent)" />`

When the aesthetic calls for filled icons instead of outlined, adjust the component to use `fill` instead of `stroke`.

### Icon Container Variation

**CSS definitions** (include in `styles/index.css` for every deck):

```css
.icon-circle {
  width: 56px; height: 56px;
  border-radius: 50%;
  background: var(--color-accent-bg);
  border: 1.5px solid var(--color-accent-dim);
  display: flex; align-items: center; justify-content: center;
}
.icon-rounded {
  width: 48px; height: 48px;
  border-radius: 12px;
  background: var(--color-accent-bg);
  border: 1.5px solid var(--color-accent-dim);
  display: flex; align-items: center; justify-content: center;
}
.icon-ghost {
  display: inline-flex;
  color: var(--color-accent);
}
```

A deck MUST use at least 2 different icon container shapes. On a single slide (e.g., icon-trio), containers should be identical (Gestalt similarity). Between slides, containers must vary. The three available shapes are: icon-circle, icon-rounded, and icon-ghost.

---

## Principle 5: Card Variation

Different types of information should look visually distinct.

**Rule**: Define 3-4 card styles per presentation, and assign them semantically:

| Card Style | Visual Treatment | Use For |
|---|---|---|
| **Solid surface** | `background: var(--color-surface)` + border | Default cards, team members, features |
| **Ghost/outlined** | `background: transparent` + `border: 1px solid accent` | Secondary info, options, light emphasis |
| **Gradient accent** | `background: linear-gradient(...)` with accent | Key metrics, CTAs, highlights |
| **Glass/frosted** | `backdrop-filter: blur(20px)` + semi-transparent bg | Overlaid on images, special callouts |

**Implementation**: Define these as CSS classes in `styles/index.css`:
```css
.card-solid { background: var(--color-surface); border: 1px solid var(--color-surface-border); border-radius: 16px; }
.card-ghost { background: transparent; border: 1px solid var(--color-accent-dim); border-radius: 16px; }
.card-accent { background: linear-gradient(135deg, var(--color-accent-bg), var(--color-surface)); border: 1px solid var(--color-accent-dim); border-radius: 16px; }
.card-glass { background: rgba(255,255,255,0.04); backdrop-filter: blur(20px); border: 1px solid rgba(255,255,255,0.08); border-radius: 16px; }
```

**CRITICAL — Card differentiation must be OBVIOUS, not subtle.** On dark themes especially, vary cards across multiple dimensions simultaneously — not just border color. Use combinations of: different border-radius (8px vs 16px vs 0), different border-width (1px vs 2px), different background treatments (solid surface vs gradient vs transparent), different shadow (none vs glow vs drop-shadow), different internal padding (compact vs spacious). The goal: a viewer should instantly see "these cards are different types" without squinting. On dark backgrounds, a border-color change from `#2A2A38` to `#2A3A4A` is invisible — you need stronger signals like background gradients, glowing borders, or entirely different shapes.

**Glass card fallback rule:**
`card-glass` ONLY works visually when used over a textured or image background (cover photo, dot grid, diagonal lines, or gradient with visible variation). On a flat dark background, `card-glass` is INVISIBLE — use `card-ghost` instead, or add a visible background feature to the slide before using glass cards.

**Minimum multi-dimension differentiation test:**
When mixing card types on a single slide, they must differ on at least 2 of:
- Background fill (flat vs gradient vs transparent)
- Border color/weight (none, 1px muted, 2px accent, glow)
- Border radius (8px compact vs 16px standard vs 24px rounded)
- Shadow (none vs glow: `0 0 20px rgba(accent, 0.15)`)

If only border color differs → they look the same → redesign.

**Light theme card differentiation**: On light/warm backgrounds, card differentiation requires stronger signals than on dark themes. Minimum requirements: (1) The "accent" card variant MUST use a border at full `var(--color-accent)` color (not dim), at least 1.5px wide; (2) At least one card type must use a visible box-shadow (`0 2px 12px rgba(accent, 0.12)`); (3) The background gradient on accent cards must shift at least 8% luminance across the gradient. A card that only differs from its neighbor by 4% background tint has failed differentiation.

**Anti-pattern**: Every card on every slide using the exact same solid surface style. Or: 4 card "variants" that all look like slightly different shades of dark blue.

---

## Principle 6: Decorative Layer

Decoration creates visual personality. Without it, slides look like documents.

**Rule**: Each presentation must define 1-2 decorative motifs that appear on 30-50% of slides as background or accent elements.

**Motif types** (choose based on aesthetic direction):
- **Geometric**: circles, triangles, grid dots, diagonal lines
- **Organic**: gradient blobs, soft radial glows, cloud-like shapes
- **Linear**: parallel lines, cross-hatching, border accents
- **Pattern**: dot grids, subtle noise, isometric shapes

**Implementation**: Define motifs as CSS pseudo-elements or background layers:
```css
/* Geometric accent — corner circles */
.decor-circles::after {
  content: '';
  position: absolute;
  width: 300px; height: 300px;
  border-radius: 50%;
  background: radial-gradient(circle, var(--color-accent-dim) 0%, transparent 70%);
  top: -100px; right: -100px;
  pointer-events: none;
  z-index: 0;
}

/* Dot grid pattern */
.decor-dots {
  background-image: radial-gradient(circle, var(--color-surface-border) 1px, transparent 1px);
  background-size: 24px 24px;
}

/* Gradient blob */
.decor-blob::before {
  content: '';
  position: absolute;
  width: 400px; height: 400px;
  background: radial-gradient(circle, var(--color-accent) 0%, transparent 60%);
  opacity: 0.06;
  bottom: -150px; left: -100px;
  filter: blur(60px);
  pointer-events: none;
}

/* Diagonal lines */
.decor-lines::after {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    -45deg,
    transparent,
    transparent 40px,
    var(--color-surface-border) 40px,
    var(--color-surface-border) 41px
  );
  opacity: 0.3;
  pointer-events: none;
}
```

**CRITICAL — Decorative elements must be VISIBLE in exported PNGs, not just technically present in CSS.** Minimum opacity for decorative patterns: 0.08-0.15 (not 0.03-0.05 which is invisible after rendering). For film grain, scanlines, or noise textures — if they're not noticeable in a static PNG screenshot, increase their opacity or size until they visibly contribute to the presentation's personality. Better to have 2-3 clearly visible decorative touches than 10 invisible ones.

**Mandatory visibility test for each motif:**

FAIL conditions — immediately increase opacity/size if ANY of these:
- Diagonal/scan lines: opacity below 0.08 → invisible at export resolution
- Radial glow: positioned more than 100px outside slide boundary → clipped away
- Corner arc rings: thinner than 2px at opacity below 0.20 → too subtle to register
- Dot grid: dot size below 1.5px → renders as noise, not pattern

PASS minimums:
- Diagonal lines: `rgba(accent, 0.08-0.12)`, 1px wide minimum
- Glow orbs: must have at least 30% of their radius inside the slide boundary
- Corner arcs: 2px stroke, `rgba(accent, 0.20-0.30)`, or skip entirely
- Dot grid: 1.5px dots, `rgba(accent, 0.18+)`, 24-32px grid spacing

**Background hue proximity rule**: When the decorative element color is within 60° hue distance of the background color, apply a 1.6× opacity multiplier above the stated minimum. When within 30° (same hue family, e.g., green decorations on green background), the minimum opacity for ANY decorative element is 0.18 regardless of other calculations. Warm ecru backgrounds with similarly-warm dots require at minimum 0.28 opacity at 1.5px to register as a pattern. For corner arc/glow elements: the arc center MUST be no more than `radius * 0.5` outside the slide boundary to guarantee at least 40% visibility.

**Opacity calibration by background luminance:**
- Light backgrounds (luminance > 70%): atmosphere 0.15-0.30, texture 0.06-0.12, borders 0.15-0.25
- Dark backgrounds (luminance < 30%): atmosphere 0.08-0.15, texture 0.03-0.08, borders 0.10-0.18
- Minimum decorative element size: 200px on any dimension. Elements < 200px are invisible at presentation scale.

See `references/decoration-library.md` for the full library of pre-tested decoration components.

**BACKGROUND LAYER SYSTEM**: Every slide must have minimum 2 background layers. A single solid color is not professional-grade. Layer atmosphere gradients and texture patterns on top of the base color. See Step 5 BACKGROUND LAYER SYSTEM for implementation details.

**Anti-pattern**: Every slide is just text on a flat/gradient background with zero decorative elements. Or: decorative CSS at 0.03 opacity that is technically present but invisible to the human eye.

### Pill/Badge Rendering

All pill/badge/chip elements **must** use this CSS pattern:
```css
display: inline-flex;
align-items: center;
justify-content: center;
line-height: 1;
padding: 6px 18px;
```

`line-height: 1` is **CRITICAL** — without it, uppercase + letter-spacing text shifts visually upward.

Pill background must be **opaque** (solid `var(--bg-base)` or `var(--bg-alt)`), not semi-transparent `rgba(...)`, to prevent "mud" effect when overlapping decorative gradient spots. **Alternative:** if a pill must be semi-transparent by design, radial-gradient spots must **not** be placed in the top 20% of the slide (where pills typically appear).

### Decorative Gradient Exclusion Zones

Decorative gradient spots (radial-gradient blobs, glows):
- Must **not** overlap with text-heavy zones (headings, pills, body text)
- Place in corners, edges, or outside the content area
- If overlap with content is unavoidable: opacity ≤ 0.20

---

## Principle 7: Visual Arc (Narrative Progression)

A presentation tells a story. The visuals should support the narrative arc.

**Rule**: The visual intensity should build toward the climax (typically the Ask/CTA slide):

```
Cover (high impact) → Intro (calm) → Problem (tension builds) →
Solution (bright) → Data (steady) → ASK (peak intensity) → Close (resolution)
```

**Implementation**:
- **Cover**: dramatic — large type, imagery, strong atmosphere
- **Early slides** (problem/context): slightly muted, building tension
- **Middle slides** (solution/product): brighter, more colorful, optimistic
- **Data slides**: clean and structured, professional
- **Ask/CTA slide**: MOST visually distinct — different background treatment, larger type, warmer/more urgent color accent, unique decorative elements
- **Closing slide**: resolution — return to cover's atmosphere but calmer

**Techniques for building the arc**:
- Shift background intensity: darker early → lighter at solution → dramatic at ask
- Introduce a secondary accent color only at the ask slide (e.g., amber/gold for financial ask)
- Increase decorative element density approaching the climax
- Use the largest type scale on the ask slide

**Concrete implementation matrix for arc (dark theme example):**

| Slide type       | Background base   | Glow intensity | Accent coverage |
|------------------|-------------------|----------------|-----------------|
| Problem slides   | darkest base      | minimal/none   | 15% — sparing   |
| Solution slides  | slightly lifted   | moderate glow  | 25% — present   |
| Traction slides  | lifted further    | strong glow    | 30% — confident |
| Ask slide        | deep + warm shift | gold/warm glow | 40% — peak      |
| Close slide      | return to base    | soft glow      | 20% — resolved  |

Minimum implementation: problem slides MUST use a darker/more constrained background treatment than solution slides. Even a 5% difference in background lightness across the deck creates narrative momentum.

**Light theme arc implementation:**

| Slide type       | Background treatment                        | Accent coverage |
|------------------|---------------------------------------------|-----------------|
| Problem/context  | Base background (cream/off-white)           | 15% — sparing   |
| Solution slides  | 3-5% luminance lift via white gradient      | 25% — present   |
| Traction slides  | Stronger warm gradient or accent surface tint| 30% — confident |
| Ask/CTA slide    | Maximum contrast: invert to dark OR full-saturation warm gradient | 40% — peak |
| Close slide      | Return to base or maintain dark inversion   | 20% — resolved  |

Minimum difference between problem and solution slides: 5% luminance change measurable in the rendered PNG.

**Monochromatic lock prevention**: Decks using a single-hue palette (e.g., all greens, all blues) MUST introduce a secondary hue temperature shift on at least 2 slides to break monotony. The ask/CTA slide MUST use a different hue temperature than the surrounding slides (e.g., warm amber/gold on a cool green deck, cool blue on a warm red/orange deck). Luminance changes within the same hue family alone are insufficient for a dramatic visual arc — the audience needs a color temperature surprise at the climax.

**Hue shift requirement**: At minimum, the ask/CTA slide and one other key slide (cover or closing) should introduce a distinctly different color temperature from the dominant palette.

**Same-hue section-break → climax differentiation rule**: If a section break slide (layout: `section`) uses the primary accent color as its full-bleed background, the climax/CTA slide MUST differentiate with one of:
- (A) A warmer/darker directional gradient on the same hue — e.g., `linear-gradient(145deg, #1F6B3A 0%, #2D8B4E 50%, #3A9E5C 100%)` vs. flat `#2D8B4E`. This creates depth and drama vs. the flat section color.
- (B) The secondary accent color (warm tone) as the dominant background if a secondary accent is defined — e.g., transition from green section break to orange/amber climax.
- (C) A fully inverted treatment with a unique compositional anchor (centered oversized number, radial burst SVG, large decorative icon) that does not appear on the section slide.

Rule of thumb: **a viewer who has seen the section break should feel escalation, not déjà-vu, when reaching the climax.** If both use the same background color, the presentation loses its narrative momentum in the final stretch.

**CTA slide color transition smoothness**: The CTA/closing slide background MUST share at least one color channel with the preceding slide. Rules:
- If the deck is dark, CTA may use accent color as background but with base color overlay (30-40% opacity) — creating "warm dark" instead of full color inversion.
- The penultimate slide MUST begin the color transition: slightly warmer background, brighter accent usage, ~5% luminance shift toward the CTA's target color.
- No full color inversion between adjacent slides. The audience should feel a gradual warm-up, not a jarring switch.
- Acceptable CTA backgrounds on a dark deck: `linear-gradient(145deg, accent 0%, darken(accent, 30%) 100%)` with `rgba(base, 0.30)` overlay. This reads as "warm and inviting" rather than "shocking change."

**Anti-pattern**: Every slide at the same visual intensity from start to finish.

---

## Principle 8: Data Visualization Quality

Charts and graphs should look like analytical dashboards, not colored divs.

**Rule**: All data visualizations must include:
- **Value labels** on key data points (not just shapes)
- **Clear axis context** (labels or implied scale)
- **Visual emphasis** on the most important data point
- **Gradient or multi-tone fills** (not flat single-color)

**Bar chart requirements**:
```html
<div class="bar-group">
  <div class="bar-value">$150M</div>  <!-- REQUIRED: label above bar -->
  <div class="bar" style="height: 60%"></div>
  <div class="bar-label">Year 2</div>
</div>
```
```css
.bar {
  background: linear-gradient(180deg, var(--color-accent), var(--color-accent-dim));
  border-radius: 6px 6px 0 0;
  position: relative;
}
.bar-value {
  font-size: 0.75em;
  font-weight: 600;
  color: var(--color-accent);
  text-align: center;
  margin-bottom: 4px;
}
```

**Table requirements**:
- Highlight column for the "hero" entity (e.g., your product) with subtle accent background
- Alternating row shading OR clear border separation
- Bold headers with accent underline
- Use <Icon> components for check/cross instead of text symbols

**SVG charts** (donut, pie): always include text labels inside or next to segments.

**Anti-pattern**: Bar charts without value labels. Tables that are just plain text grids.

**Chart-Specific Rules (from research):**

**Bar charts:**
- Y-axis MUST start at zero — bar length encodes value, truncated axis = visual lie
- Maximum 7-8 bars per chart
- Value labels mandatory on every bar
- Title = insight: "Revenue grew 23%" not "Revenue chart"

**Line charts:**
- Do NOT require zero baseline (acceptable to truncate for trend detail)
- Maximum 5 lines per chart
- Labels directly on lines — no separate legend

**Pie/Donut:**
- Maximum 5-6 segments, remainder → "Other"
- Largest segment at 12 o'clock, clockwise descending by size
- Donut preferred over pie (better readability, center can hold label)
- 3D effects FORBIDDEN

**General chart rules:**
- No legends — label data directly on the chart element
- Chart title = finding/insight, not chart type name
- Dual Y-axes FORBIDDEN (implies false correlation)
- Every statistic needs source citation: `(source: Name, Year)`

**Colorblind-safe data encoding:**
- Prohibited pairs: red+green, green+brown, blue+purple, green+black
- Recommended binary pair: blue+orange
- Never use color as ONLY differentiator — add shape, pattern, or text label
- **Hue ranges:** Red (hue 0-30) + Green (hue 90-165) on same slide = **FAIL**. Brown (hue 20-40) + Green (hue 90-165) = **WARNING**.

### Chart Type Selection Matrix

| Goal | Correct type | Avoid |
|------|-------------|-------|
| Category comparison | Horizontal bar | 3D, exploded pie |
| Change over time | Line chart | Bar with many periods |
| Parts of whole | Stacked bar / Donut (≤5 segments) | Pie >6 segments |
| Correlation | Scatter + trend line | Tables |
| Simple proportion | Donut (3-5 segments) | 3D pie |

---

## Principle 9: No Placeholder Mockups

Empty device frames signal "this is fake." Even basic wireframes look 10x better.

**Rule**: If showing device mockups (phone, laptop, tablet), they MUST contain stylized UI content — at minimum a wireframe level with colored blocks, a status bar, navigation, and 2-3 UI elements.

**Phone wireframe pattern**:
```html
<div class="phone-mock">
  <div class="phone-frame">
    <div class="phone-statusbar">
      <span class="status-time">9:41</span>
      <span class="status-icons">●●●</span>
    </div>
    <div class="phone-content">
      <div class="wire-header" style="height:20px; background:var(--color-accent); opacity:0.3; border-radius:4px; margin-bottom:8px"></div>
      <div class="wire-map" style="height:60%; background:var(--color-surface); border-radius:8px; margin-bottom:8px; display:flex; align-items:center; justify-content:center">
        <div style="width:8px;height:8px;background:var(--color-accent);border-radius:50%"></div>
      </div>
      <div class="wire-button" style="height:32px; background:var(--color-accent); border-radius:8px; display:flex; align-items:center; justify-content:center; color:var(--color-bg); font-size:0.6em; font-weight:600">
        Вызвать
      </div>
    </div>
  </div>
</div>
```

**Anti-pattern**: Gray rectangles with a single word like "Map" or "Payment".

---

## Principle 10: SVG Diagrams Over CSS Positioning

Complex diagrams (flywheels, flow charts, cycles) need proper vector graphics.

**Rule**: For any diagram with arrows, cycles, or complex relationships — use inline SVG with proper `<path>` elements for arrows and `<line>` or `<path>` for connections. NEVER use text arrows (→ ↓ ← ↑) or CSS absolute positioning to simulate a diagram.

**Flywheel pattern** (SVG):
```html
<svg viewBox="0 0 400 400" class="flywheel-svg">
  <!-- Center circle -->
  <circle cx="200" cy="200" r="50" fill="var(--color-accent-bg)" stroke="var(--color-accent)" stroke-width="2"/>
  <text x="200" y="195" text-anchor="middle" fill="var(--color-accent)" font-size="12" font-weight="600">Network</text>
  <text x="200" y="210" text-anchor="middle" fill="var(--color-accent)" font-size="12" font-weight="600">Effect</text>

  <!-- Outer nodes -->
  <rect x="145" y="20" width="110" height="44" rx="10" fill="var(--color-surface)" stroke="var(--color-surface-border)"/>
  <text x="200" y="47" text-anchor="middle" fill="var(--color-text)" font-size="12">More Drivers</text>

  <!-- Curved arrows connecting nodes -->
  <defs>
    <marker id="arrowhead" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto">
      <polygon points="0 0, 8 3, 0 6" fill="var(--color-accent)" opacity="0.6"/>
    </marker>
  </defs>
  <path d="M 270 55 Q 340 120 310 180" fill="none" stroke="var(--color-accent)" stroke-width="1.5"
        opacity="0.5" marker-end="url(#arrowhead)"/>
  <!-- ... more nodes and arrows ... -->
</svg>
```

**Flow chart pattern**: Use `<line>` or `<path>` with `marker-end` for proper arrowheads.

**Anti-pattern**: Using `position: absolute` with text characters `→ ↓` as arrows.

### Text Arrow Character Ban

The following characters are **banned** in slides.md content (outside `<code>`, backticks, and `<!-- -->` speaker notes):

`→` `←` `↑` `↓` `⟶` `➜` `▶`

**Replacements:**
- Sequences ("A → B → C"): rephrase textually ("from A to B, then to C") or use SVG `<path>` arrows
- Pipelines: use `timeline-horizontal` / `timeline-zigzag` archetype with numbered steps
- Navigation cues: use `Icon.vue` with `arrow-right` icon inside layout

Any match in visible content = **FAIL** in QA-0c.

---

## Principle 11: Vertical Spacing & Content Centering

Content should feel balanced on the slide — not pushed to the top.

**Rule**:
- Content should be vertically centered or distributed with generous padding
- Minimum 48px padding-top, 40px padding-bottom on content slides
- Use `display: flex; flex-direction: column; justify-content: center; min-height: 100%` on content wrappers when appropriate
- Headings should not stick to the very top of the slide

**Implementation**: On slides with moderate content, wrap everything in a container:
```css
.slide-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  min-height: calc(100vh - 100px);
  padding: 48px 0 40px;
}
```

For content-heavy slides where vertical centering would compress things, use top-aligned layout but with generous top padding.

**`space-evenly` vs `center` for lists**: When a content slide has 4+ items AND the heading above consumes more than 20% of slide height, use `justify-content:space-evenly` instead of `justify-content:center` on the item container. `space-evenly` distributes equal spacing above the first item and below the last, filling the full available height. `justify-content:center` clusters items in the upper portion of the remaining space, producing a bottom-heavy void.

**`layout: none` for precise visual control**: For presentations requiring exact visual control over every pixel (healthcare, enterprise, premium aesthetics), using `layout: none` with `position:absolute;inset:0` content layers and 100% inline styles is the most reliable approach. It completely bypasses Slidev theme CSS specificity — all font sizes, colors, text alignments, and backgrounds apply exactly as specified without fighting theme selectors. Reserve named layouts (`cover`, `section`, `fact`, `statement`) only for slides where the theme default rendering is acceptable. The tradeoff: `layout: none` requires more verbose HTML but produces zero CSS surprises.

**Anti-pattern**: Content starting at the default Slidev top padding with a huge empty gap at the bottom.

---

## Principle 12: Accent Hierarchy (Color Weight System)

When accent color is everywhere at the same intensity, nothing stands out.

**Rule**: Define 3 levels of accent color and use them purposefully:

| Level | Opacity / Treatment | Use For | Coverage |
|---|---|---|---|
| **Primary** | Full saturation (`var(--color-accent)`) | Key metrics, CTA buttons, active highlights | 10-15% of slide |
| **Secondary** | 40-60% opacity or tinted (`var(--color-accent-dim)`) | Subheadings, icons, borders of active cards | 20-30% |
| **Ambient** | 8-15% opacity (`var(--color-accent-bg)`) | Card backgrounds, subtle fills, hover states | Background layer |

**Implementation in CSS**:
```css
:root {
  --color-accent: #06C1A7;          /* Primary: full strength */
  --color-accent-dim: rgba(6, 193, 167, 0.4);  /* Secondary: subdued */
  --color-accent-bg: rgba(6, 193, 167, 0.08);  /* Ambient: background tint */
}
```

**Anti-pattern**: Using `#06C1A7` at full saturation for everything — big numbers, small labels, borders, icons, backgrounds.

**60-30-10 Color Distribution Rule:**

- **60%** — dominant color (background). One color family across the deck.
- **30%** — secondary (text, card surfaces, borders).
- **10%** — accent (CTA buttons, key metrics, highlights, active states).
- **Maximum 4 distinct colors** in the entire presentation palette (dominant + secondary + accent + optional warm/cool variant).

**AI Color Blacklist:**
These colors are statistically the most common AI-default choices (Tailwind training bias). BANNED as primary/dominant accent:
- Hue range 240-290 (purple-indigo-violet) at saturation >50%
- Specific: `#6366F1`, `#8B5CF6`, `#A855F7`, `#06B6D4`
- Acceptable as secondary/ambient (≤10% coverage), never as primary.

**Pure Black/White Ban:**
Never use pure `#000000` for text or pure `#FFFFFF` for backgrounds. Replace with tinted alternatives:
- Black text → `#1A2332` (dark navy) or `#1C1917` (warm charcoal)
- White background → `#FAF9F6` (cream), `#F5F5F3` (warm gray), or `#EFF5F4` (cool mint)

**Weakest Segment First (Muted Text Contrast):**
When defining `--color-muted` (secondary/gray text), test its contrast ratio against **every** surface it will appear on: `--bg-base`, `--bg-alt`, `--color-accent-bg`, `--color-surface`. The **lowest contrast ratio** must pass WCAG AA (≥ 4.5:1). If it fails on any surface (typically accent cards), darken `--color-muted` **globally** until the weakest segment passes. Do not vary muted color per surface — one muted color everywhere.

---

## Quick Reference: Principle Checklist

Use this checklist during generation, editing, and Visual QA:

- [ ] **Rhythm**: Are there breathing slides between dense content? (min 2-3 per 10+ slide deck)
- [ ] **Diversity**: Do consecutive slides look visually different?
- [ ] **Typography**: Are there 3+ distinct type scales? Is the hero number BIG?
- [ ] **Icons**: Zero emoji? All icons are SVG via `<Icon>` component?
- [ ] **Cards**: Are different card types visually distinct?
- [ ] **Decoration**: Are there decorative motifs on 30-50% of slides?
- [ ] **Arc**: Does visual intensity build toward the climax slide?
- [ ] **Data viz**: Do charts have value labels? Tables have highlights?
- [ ] **Mockups**: Do device frames contain wireframe UI, not empty boxes?
- [ ] **Diagrams**: Are diagrams SVG with proper arrows, not CSS+text?
- [ ] **Spacing**: Is content vertically balanced, not crammed to top?
- [ ] **Accent levels**: Is accent color used at 3 different intensities?
- [ ] **Bg-system**: 3-level bg-system (bg-base/alt/accent), same temperature, no pure #000/#FFF?
- [ ] **Type treatments**: ≤3 per slide; italic only in quotes/attribution?
- [ ] **Icon containers**: Varied (size, shape, or style) when 3+ on one slide?
- [ ] **Pills**: Opaque bg + line-height:1; no mud overlap with decorative gradients?
- [ ] **Charts**: Type matches data goal (see matrix); colorblind-safe hue pairs?
- [ ] **Arrow chars**: No text arrows (→←↑↓) in visible content?
- [ ] **Muted text**: Passes WCAG 4.5:1 on ALL surfaces (weakest segment first)?

---

## Gestalt Compliance Notes

**Proximity:** Labels and captions must be visually adjacent to their referent elements. A label more than ~1/4 slide height away from its element = violation.

**Similarity:** Same-level elements in a grid (e.g., all cards in a card-mosaic) must share the same padding, border-radius, and font-size. Inconsistent styling within a group = violation.

**Speaker Note Redundancy (Mayer Principle):** Speaker notes must EXTEND slide content, not repeat it verbatim. If slide text and speaker note convey the same information = redundancy violation.

---

## Anti-Patterns Quick Reference

What makes a presentation look AI-generated — avoid ALL of these:

1. **Same layout on >30% of slides** — every slide must be structurally distinct from neighbors
2. **Purple/indigo/cyan as primary color** — AI-default from Tailwind training data
3. **Three-column icon grid repeated** — `icon-trio` max once per deck
4. **All-caps label on every slide** — max 30%, and labels must be specific not generic
5. **Generic titles** — "Overview", "Our Approach", "Key Takeaways" = AI tell. Use action titles.
6. **"Thank You" ending** — last slide must be CTA with specific action, not gratitude
7. **Recommendation at the end** — main conclusion by slide 2-3, not buried in final third
8. **Body text centered on multiple lines** — multi-line body MUST be left-aligned
9. **Font below 20px (1.25rem) for body** — audience can't read small text on projected slides
10. **>40 words on a content slide** — that's a document, not a slide
11. **>15% of text bold** — over-emphasis kills emphasis
12. **Bar charts not starting at zero** — truncated Y-axis is a visual lie
13. **Statistics without sources** — unsourced numbers signal hallucination
14. **Sub-bullets 3+ levels deep** — max 2 levels, then restructure
15. **Pure `#000000` or `#FFFFFF`** — always use tinted variants for text and backgrounds
16. **Inconsistent background temperature** — mixing warm and cool backgrounds in the same deck
17. **More than 3 background colors** — only `--bg-base`, `--bg-alt`, `--bg-accent` allowed
18. **Text arrow characters** (→ ← ↑ ↓) in visible content — use SVG arrows or textual rephrasing
19. **3+ identical icon containers** on one slide (same size, shape, border) — vary at least one dimension
20. **Multiple decorative gradient types** (radial + linear as decoration) in same presentation — pick one

**The test:** Would a professional presentation designer approve this slide? Would a McKinsey partner present it? If not — fix it.
