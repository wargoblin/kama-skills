# HTML/CSS Presentation Research: Comprehensive Findings

*Compiled from research across reveal.js, Slidev, Marp, Spectacle docs, expert practitioner blogs, and 2026 design reports. March 2026.*

---

## 1. Framework Landscape

| Framework | Architecture | Canvas | Scaling | Export |
|---|---|---|---|---|
| **reveal.js** | HTML + CSS + JS | Fixed (960x700 default) | CSS transform scale | DeckTape / print-pdf |
| **Slidev** | Markdown + Vue + Vite | Configurable fixed | CSS transform | Playwright (PDF/PNG/PPTX) |
| **Marp/Marpit** | Markdown -> SVG/HTML | 1280x720 or 960x720 | SVG viewBox (CSS-only) | Chromium headless |
| **impress.js** | HTML + CSS3 3D | Infinite (Prezi-style) | CSS 3D transforms | DeckTape |
| **Spectacle** | React/JSX | Configured | React-based | Browser print |

**Key distinctions:**
- **Slidev** is the modern developer choice: Markdown input, Vue components, Vite HMR, UnoCSS atomic CSS, export via `playwright-chromium`.
- **reveal.js** is the most mature with the largest plugin ecosystem.
- **Marp** uses `<svg><foreignObject>` for pixel-perfect CSS-only scaling without JavaScript.

---

## 2. Viewport Scaling: Three Approaches

### Approach A: CSS Transform Scale (reveal.js, Slidev)

Dominant approach. JavaScript resize listener computes scale factor:

```javascript
const scale = Math.min(
  availableWidth  / SLIDE_W,
  availableHeight / SLIDE_H
);
slidesEl.style.transform = `translate(-50%, -50%) scale(${scale})`;
slidesEl.style.transformOrigin = '0 0';
```

**reveal.js internals:** `transform: scale(var(--slide-scale)) translate(-50%, -50%)` on `.slides` container. Defaults: `width: 960, height: 700, margin: 0.04, minScale: 0.2, maxScale: 2.0`.

**Z-index layers in reveal.js:** slides 10-11, controls 11, overlays 99-100, backgrounds z-index 2.

### Approach B: SVG viewBox (Marp, CSS-only)

```html
<svg viewBox="0 0 1280 720" xmlns="http://www.w3.org/2000/svg">
  <foreignObject width="1280" height="720">
    <section><!-- slide content --></section>
  </foreignObject>
</svg>
```

Browser's native SVG scaling handles aspect-ratio-preserving resize without JavaScript.

### Approach C: Pure CSS Viewport Units

```css
.slide {
  width: 100vw;
  height: 56.25vw;     /* 16:9 */
  max-height: 100vh;
  max-width: 177.78vh;  /* inverse 16:9 */
}
:root { font-size: 2vw; } /* 1rem = 2% of viewport width */
```

---

## 3. Layout Systems

### Primary: Flexbox (content-driven)

```css
.slide {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 60px 80px;
}

.slide-two-col {
  flex-direction: row;
  gap: 48px;
  align-items: start;
}
```

### Secondary: CSS Grid (layout-driven)

```css
.feature-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 32px;
}

.slide-editorial {
  display: grid;
  grid-template-columns: 1fr 2fr;
  grid-template-rows: auto 1fr;
  gap: 24px 48px;
}
```

### `position: absolute` -- Only for Overlays

Absolute positioning for main content = **anti-pattern**: breaks with different text lengths. Use only for:
- Slide number badges
- Logo placement
- Decorative shapes (`z-index: 0; pointer-events: none`)
- Backgrounds/textures

```css
.slide { position: relative; }
.slide-badge { position: absolute; top: 24px; right: 32px; }
.slide-bg-accent {
  position: absolute; inset: 0;
  z-index: 0; pointer-events: none;
}
.slide-content { position: relative; z-index: 1; }
```

---

## 4. Typography for Fixed-Canvas Presentations

### The 4 Font Sizes Rule

Research consensus: **at most 4 distinct font sizes**.

| Role | Size | Weight |
|---|---|---|
| Hero title | 48-72px | Bold (700-900) |
| Slide title | 36-48px | Semibold (600) |
| Body / bullets | 20-28px | Regular (400) |
| Caption / label | 14-16px | Regular (400) |

For **accessibility at distance**: body minimum 24-28px, titles 44-64px.

### rem-Based Scale Anchored to Canvas

```css
html { font-size: 0.78125vw; } /* 10/1280 = 0.78125%, so 1rem = 10px at 1280px */

:root {
  --t-xs:   1.2rem;  /* 12px */
  --t-sm:   1.4rem;  /* 14px */
  --t-base: 1.8rem;  /* 18px */
  --t-lg:   2.2rem;  /* 22px */
  --t-xl:   2.8rem;  /* 28px */
  --t-2xl:  3.6rem;  /* 36px */
  --t-3xl:  5.0rem;  /* 50px */
  --t-4xl:  7.2rem;  /* 72px */
}
```

### Fluid Typography (clamp)

```css
.slide-title {
  font-size: clamp(1rem, -0.875rem + 8.333vw, 3.5rem);
}
```

### Heading Spacing

```css
.slide h1, .slide h2 {
  line-height: 1.1;
  letter-spacing: -0.02em;
}
.slide p, .slide li {
  line-height: 1.5;
  max-width: 65ch;
}
```

### Font Loading for Export

```css
@font-face {
  font-family: 'Inter';
  src: url('fonts/inter-regular.woff2') format('woff2');
  font-weight: 400;
  font-display: block; /* CRITICAL: blocks render until font loads -- prevents FOUT in screenshots */
}
```

---

## 5. Background Techniques (No Images)

### Mesh/Aurora Gradients

```css
.bg-aurora {
  background:
    radial-gradient(ellipse 80% 60% at 20% 20%, rgba(120,119,198,0.3) 0%, transparent 60%),
    radial-gradient(ellipse 60% 80% at 80% 80%, rgba(255,119,198,0.3) 0%, transparent 60%),
    radial-gradient(ellipse 100% 100% at 50% 0%, rgba(56,189,248,0.2) 0%, transparent 70%),
    #0f0f23;
}
```

### CSS Patterns

```css
/* Dot grid */
.bg-dots {
  background-color: #0f0f23;
  background-image: radial-gradient(circle, rgba(255,255,255,0.08) 1px, transparent 1px);
  background-size: 24px 24px;
}

/* Diagonal stripes */
.bg-stripes {
  background-image: repeating-linear-gradient(
    45deg, transparent, transparent 10px,
    rgba(255,255,255,0.03) 10px, rgba(255,255,255,0.03) 20px
  );
}
```

### Overlay Technique

```css
.slide-hero::before {
  content: '';
  position: absolute; inset: 0;
  background: linear-gradient(to bottom, rgba(0,0,0,0.1) 0%, rgba(0,0,0,0.6) 100%);
  z-index: 0;
}
```

---

## 6. Glassmorphism

```css
.glass-card {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border-radius: 16px;
}

.glass-dark {
  background: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(12px) saturate(1.5) brightness(1.1);
  -webkit-backdrop-filter: blur(12px) saturate(1.5) brightness(1.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

@supports not (backdrop-filter: blur(1px)) {
  .glass-card { background: rgba(255, 255, 255, 0.85); }
}
```

**Rules:**
- Blur: 8-15px (higher = GPU expensive)
- Opacity: 0.1-0.25 light, 0.15-0.3 dark
- Requires colorful background behind it
- Never `overflow: hidden` on slide container -- breaks backdrop-filter

---

## 7. CSS Animation Best Practices

### Performance Rule

**Animate ONLY:** `transform`, `opacity` -- GPU-accelerated.
**Never animate:** `width`, `height`, `top`, `left`, `margin`, `padding`.

### Timing

- Slide transitions: 200-400ms
- Element entrances: 150-300ms
- Micro-interactions: 100-200ms

### Custom Easing

```css
:root {
  --ease-out:    cubic-bezier(0.22, 1, 0.36, 1);
  --ease-in:     cubic-bezier(0.64, 0, 0.78, 0);
  --ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

### Professional Entrance

```css
@keyframes fade-up {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.animate-in { animation: fade-up 0.35s var(--ease-out) both; }

/* Stagger children */
.stagger > *:nth-child(1) { animation-delay: 0ms; }
.stagger > *:nth-child(2) { animation-delay: 80ms; }
.stagger > *:nth-child(3) { animation-delay: 160ms; }
```

### Per-Property Easing

```css
.element {
  transition:
    opacity linear 0.3s,
    transform cubic-bezier(0.22, 1, 0.36, 1) 0.3s;
}
```

### Accessibility

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 8. CSS Design Token System

### Three-Layer Architecture

```css
/* Layer 1: Primitives */
:root {
  --blue-500: #3b82f6;
  --gray-950: #0a0f1e;
  --spacing-4: 1rem;
  --radius-md: 8px;
}

/* Layer 2: Semantic */
:root {
  --color-brand:     var(--blue-500);
  --color-bg:        var(--gray-950);
  --color-bg-card:   rgba(255,255,255,0.06);
  --color-text:      #ffffff;
  --color-text-muted: rgba(255,255,255,0.55);
  --color-border:    rgba(255,255,255,0.10);
}

/* Layer 3: Component */
.card {
  --card-bg: var(--color-bg-card);
  --card-radius: var(--radius-md);
  background: var(--card-bg);
  border-radius: var(--card-radius);
}
```

### Theme Switching

```css
[data-theme="light"] {
  --color-bg:          #f1f5f9;
  --color-bg-card:     rgba(0,0,0,0.04);
  --color-text:        #0f172a;
  --color-text-muted:  rgba(0,0,0,0.5);
}
```

---

## 9. SVG Charts and Diagrams

### Bar Chart

```html
<svg viewBox="0 0 420 150" aria-labelledby="chart-title" role="img">
  <title id="chart-title">Revenue by Region</title>
  <g class="bar">
    <rect width="40" height="60" x="10" y="90"/>
    <text x="30" y="82" text-anchor="middle">$2.4M</text>
  </g>
</svg>
```

### Donut Chart (stroke-dasharray)

```html
<svg viewBox="0 0 100 100">
  <circle cx="50" cy="50" r="40" fill="transparent"
    stroke="#e2e8f0" stroke-width="12"/>
  <circle cx="50" cy="50" r="40" fill="transparent"
    stroke="var(--color-brand)" stroke-width="12"
    stroke-dasharray="75 25"
    stroke-dashoffset="25"
    transform="rotate(-90 50 50)"/>
</svg>
```

### SVG Path Drawing Animation

```css
.animated-line {
  stroke-dasharray: 500;
  stroke-dashoffset: 500;
  animation: draw 1.5s var(--ease-out) forwards;
}
@keyframes draw { to { stroke-dashoffset: 0; } }
```

---

## 10. Accessibility (WCAG)

| Text Type | WCAG AA Minimum |
|---|---|
| Normal text (< 18px) | 4.5:1 |
| Large text (>= 18px) | 3:1 |
| UI components, icons | 3:1 |

For projected presentations: target 7:1+ body, 4.5:1 minimum titles.

### Semantic HTML

```html
<section aria-label="Slide 3: Revenue Growth">
  <h2>Revenue Growth 2025</h2>
  <figure>
    <svg aria-labelledby="chart-title" role="img">
      <title id="chart-title">Bar chart showing 40% growth</title>
    </svg>
    <figcaption>Source: Internal Q4 Report</figcaption>
  </figure>
</section>
```

---

## 11. Export Pipeline (PDF/PNG)

### Slidev

```bash
slidev export --format pdf --wait 500 --wait-until networkidle
slidev export --format png
slidev export --format pptx
```

### Playwright Manual

```javascript
const page = await browser.newPage();
await page.setViewportSize({ width: 1280, height: 720 });
await page.goto('file:///slides.html', { waitUntil: 'networkidle' });
await page.screenshot({ path: 'slide-01.png',
  clip: { x: 0, y: 0, width: 1280, height: 720 } });
```

### DeckTape (Universal)

```bash
decktape reveal http://localhost:8080/slides.html output.pdf -s 1280x720
decktape reveal slides.html output.pdf --screenshots --screenshots-format png
```

### CSS for Print

```css
@media print {
  .slide { page-break-after: always; }
  * { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
  *, *::before, *::after { animation: none !important; transition: none !important; }
}
```

### Export Pitfalls

1. **Font not loaded:** Use `font-display: block`, await `document.fonts.ready`
2. **Background colors missing:** `printBackground: true` in Puppeteer; `print-color-adjust: exact`
3. **Animations mid-state:** Add `--wait` delay or `waitUntil: networkidle`
4. **Wrong viewport:** Match browser viewport to slide dimensions
5. **CORS failures:** Use inline/embedded assets

---

## 12. Common Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| `position: absolute` for main content | Flexbox/Grid, absolute only for decorative |
| z-index chaos (999, 9999...) | Systematic layers: `--z-base: 0; --z-elevated: 10; --z-overlay: 100` |
| `overflow: hidden` on slide | Breaks backdrop-filter; clip at wrapper |
| Animating layout properties | Only `transform` + `opacity` |
| Text overload (paragraphs) | Max 6 bullets, prefer 3 |
| Too many fonts (3+) | Maximum 2 typefaces |
| Arbitrary spacing | Use 8px grid |
| Low-contrast accents | Darken brand color for text use |

---

## 13. 2026 Design Trends (CSS-Implementable)

### Depth / Soft Shadows

```css
--shadow-sm:  0 1px 3px rgba(0,0,0,0.12);
--shadow-md:  0 4px 12px rgba(0,0,0,0.12);
--shadow-lg:  0 8px 32px rgba(0,0,0,0.16);
--shadow-xl:  0 16px 64px rgba(0,0,0,0.20);
```

### Maximalism (Bold Color + Layered Backgrounds)

```css
.bg-maximalist {
  background:
    radial-gradient(ellipse at 10% 10%, rgba(255,107,107,0.4) 0%, transparent 40%),
    radial-gradient(ellipse at 90% 90%, rgba(78,205,196,0.4) 0%, transparent 40%),
    #1a1a2e;
}

.hero-number {
  font-size: 15rem; font-weight: 900;
  line-height: 0.85; letter-spacing: -0.05em;
  opacity: 0.12; position: absolute;
  pointer-events: none;
}
```

---

## Sources

- reveal.js SCSS source and documentation
- Slidev official docs (sli.dev)
- Marp/Marpit architecture docs
- Wim Deblauwe — Technical Presentations with Slidev
- Nir Tamir — Advanced Slides with Slidev
- SlideRabbit — Presentation Design Trends 2026
- PitchWorx — Core Design Principles 2026
- Neversink Theme documentation
- MDN Web Docs — CSS Grid, Flexbox, backdrop-filter
- WCAG 2.1 specification
- DeckTape documentation
- Playwright documentation
