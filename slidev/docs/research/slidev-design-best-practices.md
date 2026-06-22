# Research: Beautiful Presentations in Slidev

*Compiled from 18 sources across Slidev official docs, expert practitioner blogs, and 2026 design trend reports. March 2026.*

---

## 1. Slidev's Design Architecture

### The 960x540 Canvas

Slidev's default canvas is **960x540 px** (16:9). All sizing decisions reference this space. `1rem = 16px`. A heading at `2rem = 32px` — usually too small for visual impact. Breathing slides need `3.5-6rem` headings. `canvasWidth` in headmatter controls this.

### UnoCSS as the Styling Engine (Since v0.42.0)

Four presets ship by default:
- `@unocss/preset-wind3` — Tailwind-compatible utilities
- `@unocss/preset-attributify` — attribute-mode styling (`bg="blue-500"`)
- `@unocss/preset-icons` — icon class system (`i-carbon-mountain`)
- `@unocss/transformer-directives` — enables `@apply` in CSS

**Extend via `uno.config.ts`:**
```typescript
import { defineConfig } from 'unocss'
export default defineConfig({
  shortcuts: {
    'card-base': 'rounded-xl border border-white/10 bg-white/5 p-6',
    'label-pill': 'text-xs uppercase tracking-widest font-bold opacity-60',
    'metric-hero': 'text-7xl font-black tabular-nums leading-none',
  },
})
```

Arbitrary values work: `class="grid grid-cols-[100px,1fr] gap-4"`

---

## 2. Layout System

### Complete Built-in Layouts

| Layout | Best Use |
|--------|----------|
| `cover` / `end` | Opening, closing |
| `default` / `full` / `none` | Content, full-bleed, zero-styling |
| `center` / `statement` / `fact` | Centered hero, dramatic statement, metric |
| `section` / `quote` / `intro` | Divider, pull quote, speaker intro |
| `two-cols` | Side-by-side via `::right::` slot |
| `two-cols-header` | Header spanning + two columns via `::left::` / `::right::` |
| `image-left` / `image-right` / `image` | Photo layouts with `image:` param |
| `iframe-left` / `iframe-right` / `iframe` | Web demo with `url:` param |

**Slot sugar for two-column:**
```markdown
---
layout: two-cols-header
---
# Spanning Title
::left::
Left content
::right::
Right content
```

### CSS Grid Patterns That Work

```html
<!-- 3 equal cards -->
<div class="grid grid-cols-3 gap-6 mt-8">

<!-- Custom widths -->
<div class="grid grid-cols-[2fr,1fr] gap-8">

<!-- Bento: large + 4 small -->
<div class="grid grid-cols-3 grid-rows-2 gap-4 h-full">
  <div class="row-span-2 ...">Big card</div>
  <div>Small 1</div><div>Small 2</div>
  <div>Small 3</div><div>Small 4</div>
</div>
```

**Custom layout Vue component** in `layouts/my-layout.vue`:
```vue
<template>
  <div class="slidev-layout my-layout">
    <div class="header"><slot name="header" /></div>
    <div class="grid grid-cols-3 gap-6"><slot /></div>
  </div>
</template>
```

Layout loading priority: built-in -> theme -> addon -> your `layouts/` (later overrides earlier).

---

## 3. Critical CSS Rules (Bugs You WILL Hit)

### Specificity Override Bug (Most Common)
Theme CSS sets `text-align: left` with high specificity — it **overrides inherited alignment**:
```css
/* Wrong — inheritance is blocked by theme */
.slidev-layout { text-align: center; }

/* Correct — explicit on every element */
.slidev-layout { text-align: center; }
h1, h2, p { text-align: center; }
```

### Background Image Blockage Bug
Using `background:` in frontmatter puts the image on a **parent container above** `.slidev-layout`. The layout's own `background-color` blocks it:
```css
/* Fix: make layout transparent, set image directly */
.slidev-layout {
  background: url('/images/hero.jpg') center/cover no-repeat;
  position: relative;
}
.slidev-layout::before {
  content: ''; position: absolute; inset: 0;
  background: rgba(0, 0, 0, 0.55); z-index: 0;
}
.slidev-layout > * { position: relative; z-index: 1; }
h1 { color: #ffffff !important; text-align: center; }
```

### Child Combinator Broken in Scoped CSS
```css
/* Breaks — Vue scoping incompatible with > combinator */
.card > h2 { ... }
/* Use descendant instead */
.card h2 { ... }
```

### Global Style Leakage into Presenter Mode
```css
/* Bad — leaks into presenter mode */
.grid { gap: 1.5rem; }
/* Good — scoped to slides */
.slidev-layout .grid { gap: 1.5rem; }
```

---

## 4. Visual Effects Toolkit

**Gradient text:**
```css
h1 {
  background: linear-gradient(135deg, #ffffff 30%, var(--color-accent));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

**Glassmorphism cards:**
```css
.card {
  background: rgba(255,255,255,0.05);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 16px;
}
```

**Atmospheric background orb (no image needed):**
```css
.slidev-layout::before {
  content: ''; position: absolute;
  width: 400px; height: 400px;
  background: radial-gradient(circle, rgba(56,189,248,0.15) 0%, transparent 70%);
  top: -100px; right: -100px;
  pointer-events: none; z-index: 0;
}
```

**Dot grid pattern:**
```css
.slidev-layout {
  background-image: radial-gradient(circle, rgba(255,255,255,0.04) 1px, transparent 1px);
  background-size: 32px 32px;
}
```

**Accent border on headings:**
```css
h1 { border-left: 4px solid var(--color-accent); padding-left: 1rem; }
```

---

## 5. Typography

### Font Configuration
```yaml
---
fonts:
  sans: 'Plus Jakarta Sans'
  mono: 'JetBrains Mono'
  weights: '300,400,600,700,800'
  italic: true
---
```

### Recommended Pairings for Developer Presentations
- **Technical/Modern**: Inter + JetBrains Mono
- **Bold/Impact**: Plus Jakarta Sans + Fira Code
- **Elegant**: DM Sans + DM Mono
- **Editorial**: Sora (800w headings) + IBM Plex Mono
- **Unique character**: Space Grotesk + Source Code Pro

### Type Scale (960x540 Canvas)
| Scale | Size | Usage |
|-------|------|-------|
| Hero | 4-8rem | Key metrics, statement slides |
| Display | 2.8-3.5rem | Cover, section dividers |
| Heading | 1.8-2.5rem | Regular slide titles |
| Body | 1.0-1.1rem | Bullets, descriptions |
| Caption | 0.75-0.85rem | Labels, footnotes |

**Rules**: Both heading + body must be sans-serif (contrast via character — geometric vs humanist, not serif vs sans). Italic only in blockquotes/attributions/captions. Never italic in headings. Use `tabular-nums` on metric figures.

---

## 6. Animation System

### Click Reveals
```html
<v-click>Reveals on click 1</v-click>
<div v-after>Appears with previous</div>
<v-clicks>
- Item A
- Item B
</v-clicks>
<div v-click="3">After exactly 3 clicks</div>
<div v-click.hide>Hides on click</div>
```

**Custom animation CSS:**
```css
.slidev-vclick-target { transition: all 400ms cubic-bezier(0.16, 1, 0.3, 1); }
.slidev-vclick-hidden { transform: translateY(20px); opacity: 0; }
```

### v-motion (Spring Physics)
```html
<div v-motion
  :initial="{ x: -80, opacity: 0 }"
  :enter="{ x: 0, opacity: 1, transition: { duration: 500 } }"
  :click-1="{ scale: 1.05, color: '#38bdf8' }">
  Content
</div>
```

### Rough Markers
```markdown
<span v-mark.circle.red>Critical concept</span>
<span v-mark="{ at: 3, color: '#38bdf8', type: 'underline' }">Term</span>
```
Types: `underline`, `circle`, `box`, `highlight`, `strike-through`, `bracket`

### Slide Transitions
```yaml
transition: slide-left          # Global in headmatter
transition: slide-left | slide-right  # Forward | backward
transition: fade                # For section breaks
```

**Best practice**: `slide-left` for forward flow, `fade` for section breaks. Avoid `slide-up` (feels like mobile app).

### Shiki Magic Move (Code Morphing)
````markdown
````md magic-move
```js
function greet(name) { return 'Hello ' + name }
```
```js
const greet = (name = 'World') => `Hello ${name}`
```
````
````
Options: `magicMoveDuration: 600`, `magicMoveCopy: 'final'`

---

## 7. 2026 Design Trends Applicable to Slidev

**Dark mode is default** for tech presentations. Base colors: `#0c0e16` to `#0f1724` (never pure `#000000`). Accent: teal `#0d9488`, amber `#d97706`, sky `#38bdf8`.

**AI color trap**: Purple/violet (hue 240-290 at saturation >50%) is the AI default — audiences recognize it immediately as low-effort. Avoid.

**Bento grid layouts**: Modular cards of different sizes — the 2026 alternative to bullet lists.

**Typography as hero**: Massive bold headings (5-8rem) occupying 50%+ of the slide for key statements.

**Purposeful animation only**: 2026 experts explicitly warn against "fussy animations that distract." `v-click` for pacing, `v-mark` for emphasis — not decoration on every element.

**Rule of thirds**: On content slides, place dominant element at a thirds-grid intersection — not dead center (center is for breathing slides only).

---

## 8. CSS Variable System (Professional Foundation)

Every serious Slidev deck defines this in `styles/index.css`:
```css
:root {
  --color-heading: #f0f4ff;
  --color-body: rgba(240, 244, 255, 0.72);
  --color-muted: rgba(240, 244, 255, 0.40);
  --color-accent: #38bdf8;
  --color-accent-dim: rgba(56, 189, 248, 0.25);
  --color-accent-bg: rgba(56, 189, 248, 0.08);
  --bg-base: #0c0e16;    /* 60% of slides */
  --bg-alt: #111828;     /* 30% — section dividers */
  --bg-accent: #0a1428;  /* 10% — cover, CTA */
}
```

Three background levels, **same color temperature** (all cool OR all warm — never mixed). Luminance delta between `--bg-base` and `--bg-accent`: maximum 40%.

---

## 9. Anti-Patterns to Avoid

| Anti-Pattern | Fix |
|-------------|-----|
| All slides identical `LABEL -> H1 -> grid` structure | Rotate layouts: centered hero, visual-dominant, inverted on every 3rd content slide |
| Emoji as icons | UnoCSS icon classes or custom `Icon.vue` SVG component |
| Pure `#000`/`#fff` backgrounds | Tinted variants: `#0c0e16`, `#faf9f6` |
| All metrics same size in identical cards | Pick a hero metric at 3.5rem+; supporting at 2.2rem min |
| 12 consecutive slides same background | Three-level background system, alternate `--bg-base` / `--bg-alt` |
| `v-click` on every bullet point | Reserve for genuine pacing needs; 5-8 clicks max per slide |
| Purple/violet accent | Safe: teal, amber, emerald, sky, rose |
| Inter/Roboto/Arial fonts | Character fonts: Plus Jakarta Sans, Sora, DM Sans |

---

## Sources

- [Slidev Built-in Layouts](https://sli.dev/builtin/layouts)
- [Slidev Animations Guide](https://sli.dev/guide/animations)
- [Slidev UnoCSS Config](https://sli.dev/custom/config-unocss)
- [Slidev Font Config](https://sli.dev/custom/config-fonts)
- [Slidev Slide-Scope Style](https://sli.dev/features/slide-scope-style)
- [Slidev Shiki Magic Move](https://sli.dev/features/shiki-magic-move)
- [Slidev Rough Marker](https://sli.dev/features/rough-marker)
- [Technical Presentations with Slidev — Wim Deblauwe](https://www.wimdeblauwe.com/blog/2024/11/05/technical-presentations-with-slidev/)
- [Advanced Slides with Slidev — Nir Tamir](https://www.nirtamir.com/articles/advanced-slides-with-slidev)
- [Neversink Theme Styling](https://gureckis.github.io/slidev-theme-neversink/styling.html)
- [Presentation Design Trends 2026 — SlideRabbit](https://sliderabbit.com/blog/presentation-design-trends-shaping-modern-slides-in-2026/)
- [Core Design Principles 2026 — PitchWorx](https://pitchworx.com/the-ultimate-guide-to-presentation-design-core-principles-for-2026/)
- [Animation & Transitions Deep Dive — DeepWiki](https://deepwiki.com/slidevjs/slidev/4.2-animation-and-transitions)
- [10 Design Principles — Flashdocs](https://www.flashdocs.com/post/10-design-principles-every-slide-creator-should-know)
