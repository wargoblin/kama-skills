---
name: figma-untitled
theme: default
colorSchema: light
fonts:
  sans: Manrope
  body: DM Sans
  mono: JetBrains Mono
aspectRatio: 16/9
transition: fade
accentColor: "#1A1A1A"
iconStyle: outlined
densityProfile: minimal
maxFonts: 2
figma:
  fileKey: "wCyEkoqYCW1n8AqwECY5hn"
  sourceUrl: "https://www.figma.com/design/wCyEkoqYCW1n8AqwECY5hn/Untitled?node-id=0-1"
  extractedAt: "2026-03-30T12:00:00Z"
  slideCount: 20
  archetypes:
    - id: figma-cover
      nodeId: "1:40"
      contentType: intro
    - id: figma-agenda
      nodeId: "1:49"
      contentType: context
    - id: figma-section
      nodeId: "1:96"
      contentType: vision
    - id: figma-two-col-intro
      nodeId: "1:107"
      contentType: context
    - id: figma-three-cards
      nodeId: "1:116"
      contentType: scope
    - id: figma-image-text
      nodeId: "1:157"
      contentType: portfolio
    - id: figma-text-grid
      nodeId: "1:177"
      contentType: scope
    - id: figma-phone-features
      nodeId: "1:262"
      contentType: scope
    - id: figma-bento
      nodeId: "1:312"
      contentType: metric
---

Bold minimalism meets editorial clarity. This design follows the `bold-minimalism` trend with a monochrome content palette and dramatic gradient mesh section dividers. CRITICAL FIGMA RULE: ALL content slides use pure white (--bg-base) background — no bg-alt panels, no split coloring, no gray side panels. Only section dividers and cover/CTA use dark gradient mesh backgrounds. Typography is the primary design element — oversized geometric sans headings (5rem+ for hero headings on agenda/title slides, font-weight: 700 not 800 to approximate Whyte's lighter visual weight) dominate the visual hierarchy. Gray (#888) for secondary text creates a subtle two-tone rhythm. No decorative elements on content slides — only generous whitespace and precise typography. Section dividers rotate through different gradient color palettes (teal/gold, teal/green, blue/purple, magenta/gold) to provide visual variety. Every slide features a bottom navigation bar with breadcrumb context. Card elements use #D9D9D9 light gray with 13px border-radius — card titles and descriptions sit ABOVE the gray card area, not inside. Cards themselves are image/visual placeholders. The overall feel is sophisticated, tech-corporate, Apple-inspired.

```archetypes
preferred: [figma-cover, figma-section, figma-two-col-intro, figma-three-cards, figma-bento, figma-image-text]
avoid: []
cta_style: figma-section
cover_style: figma-cover
```

```shapes
card_radius: 13px
icon_container: ghost
stat_display: typographic
label_style: uppercase-text
divider_style: none
photo_mask: rounded-rect
```

```css
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&family=DM+Sans:wght@400;500;600;700&display=swap');

:root {
  /* Core palette — monochrome editorial */
  --color-accent: #1A1A1A;
  --color-accent-dim: rgba(26, 26, 26, 0.50);
  --color-accent-bg: rgba(26, 26, 26, 0.08);
  --color-accent-warm: #D97706;
  --accent-rgb: 26, 26, 26;

  /* Backgrounds — 3-level system */
  --bg-base: #FFFFFF;
  --bg-alt: #F5F5F5;
  --bg-accent: #0A0A0A;
  --bg-base-rgb: 255, 255, 255;
  --bg-accent-rgb: 10, 10, 10;

  /* Text */
  --color-text: #000000;
  --color-muted: #888888;
  --text-rgb: 0, 0, 0;

  /* Surfaces */
  --color-surface: #D9D9D9;
  --color-surface-border: #E5E5E5;

  /* Typography */
  --font-heading: 'Manrope', sans-serif;
  --font-body: 'DM Sans', sans-serif;

  /* Slidev theme mapping */
  --slidev-theme-primary: #1A1A1A;
  --slidev-theme-background: #FFFFFF;
  --slidev-theme-code-background: #F5F5F5;

  /* Contrast ratios:
     --color-muted (#888) vs --bg-base (#FFF): 3.54:1 — WARN (bump for body text)
     --color-muted (#888) vs --bg-alt (#F5F5F5): 3.26:1
     --color-text (#000) vs --bg-base (#FFF): 21:1
     --color-text (#000) vs --bg-alt (#F5F5F5): 18.1:1
     Muted used only for labels/descriptions, headings use --color-text */

  /* Decorative gradient type: radial-gradient */
}

/* === BASE LAYOUT === */
.slidev-layout {
  font-family: var(--font-body);
  color: var(--color-text);
}

/* Blockquote reset */
.slidev-layout blockquote {
  background: transparent !important;
  border-left: none !important;
  border: none !important;
  padding: 0 !important;
  margin: 0 !important;
  color: inherit !important;
}

/* === LAYOUT-SPECIFIC TEXT ALIGNMENT === */
.slidev-layout.cover { text-align: center; }
.slidev-layout.cover h1, .slidev-layout.cover p { text-align: center; }
.slidev-layout.section { text-align: center; }
.slidev-layout.section h1, .slidev-layout.section p { text-align: center; }
.slidev-layout.fact { text-align: center; }
.slidev-layout.fact h1, .slidev-layout.fact p { text-align: center; }
.slidev-layout.end { text-align: center; }
.slidev-layout.end h1, .slidev-layout.end p, .slidev-layout.end div { text-align: center; }
.slidev-layout.statement h1, .slidev-layout.statement p { text-align: center; }
.slidev-layout.center h1, .slidev-layout.center p { text-align: center; }

/* === BACKGROUND IMAGE OVERLAY === */
.slidev-layout.cover, .slidev-layout.section { position: relative; }
.slidev-layout.cover::before, .slidev-layout.section::before {
  content: ''; position: absolute; inset: 0;
  background: linear-gradient(to bottom, rgba(0,0,0,0.88) 0%, rgba(0,0,0,0.75) 70%, rgba(0,0,0,0.65) 100%);
  z-index: 0; pointer-events: none;
}
.slidev-layout.cover > *, .slidev-layout.section > * { position: relative; z-index: 1; }

/* === SHAPE VOCABULARY === */

/* Card — rounded gray surface */
.card-solid {
  background: var(--color-surface);
  border-radius: 13px;
  overflow: hidden;
}

.card-ghost {
  background: transparent;
  border: 1.5px solid var(--color-surface-border);
  border-radius: 13px;
}

.card-accent {
  background: var(--bg-accent);
  border-radius: 13px;
  color: #FFFFFF;
}

/* Icon containers — ghost style (monochrome theme) */
.icon-ghost {
  width: 48px; height: 48px;
  border-radius: 50%;
  border: 1.5px solid var(--color-surface-border);
  display: flex; align-items: center; justify-content: center;
  background: transparent;
}

.icon-circle {
  width: 48px; height: 48px;
  border-radius: 50%;
  background: var(--color-accent-bg);
  display: flex; align-items: center; justify-content: center;
}

/* Stat display — typographic (no card wrapper) */
.stat-hero {
  font-family: var(--font-heading);
  font-size: 5rem;
  font-weight: 800;
  color: var(--color-text);
  line-height: 1;
}

/* Label — uppercase text */
.label-upper {
  font-family: var(--font-heading);
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.15em;
  color: var(--color-muted);
}

/* Bottom navigation bar */
.slide-footer {
  position: absolute;
  bottom: 0; left: 0; right: 0;
  height: 44px;
  display: flex;
  align-items: center;
  padding: 0 48px;
  gap: 48px;
  font-family: var(--font-body);
  font-size: 0.65rem;
  color: var(--color-muted);
  z-index: 10;
}

.slide-footer-dark {
  color: rgba(255,255,255,0.45);
}

.slide-footer-icon {
  width: 12px; height: 12px;
  background: var(--color-surface);
  border-radius: 2px;
}

/* Gradient mesh backgrounds for section dividers */
.gradient-mesh-1 {
  background:
    radial-gradient(ellipse 80% 80% at 70% 60%, rgba(59,130,246,0.5), transparent 60%),
    radial-gradient(ellipse 60% 70% at 85% 25%, rgba(217,180,80,0.6), transparent 50%),
    radial-gradient(ellipse 90% 90% at 30% 80%, rgba(99,60,180,0.3), transparent 60%),
    #0A0A0A;
}

.gradient-mesh-2 {
  background:
    radial-gradient(ellipse 80% 80% at 65% 55%, rgba(16,185,129,0.6), transparent 55%),
    radial-gradient(ellipse 60% 70% at 85% 30%, rgba(180,210,80,0.5), transparent 50%),
    radial-gradient(ellipse 70% 80% at 20% 70%, rgba(6,95,70,0.5), transparent 60%),
    #0A0A0A;
}

.gradient-mesh-3 {
  background:
    radial-gradient(ellipse 80% 80% at 55% 60%, rgba(168,85,247,0.5), transparent 55%),
    radial-gradient(ellipse 60% 70% at 80% 30%, rgba(200,160,230,0.4), transparent 50%),
    radial-gradient(ellipse 70% 80% at 30% 70%, rgba(59,130,246,0.4), transparent 60%),
    #0A0A0A;
}

.gradient-mesh-4 {
  background:
    radial-gradient(ellipse 80% 80% at 60% 65%, rgba(219,39,119,0.6), transparent 55%),
    radial-gradient(ellipse 60% 70% at 85% 25%, rgba(217,180,120,0.5), transparent 50%),
    radial-gradient(ellipse 70% 80% at 25% 60%, rgba(168,85,247,0.4), transparent 60%),
    #0A0A0A;
}
```
