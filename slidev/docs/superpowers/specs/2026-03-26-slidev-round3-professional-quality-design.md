# Slidev Skill: Round 3 — Professional Quality Design Spec

**Date:** 2026-03-26
**Status:** Draft
**Scope:** 8 problems (+ 2 sub-problems) identified from visual analysis of 3 presentations generated in stitch modes (full/design/improve). Focus: transform output from "basic minimum" to professional presentation quality. Plus new `--stitch=learn` mode.

## Background

Analysis of innosmart-tech-pitch (--stitch=full), den-rozhdeniye-malchika-10-let (--stitch=design), and cloudkitchen-pitch (--stitch=improve) revealed that CloudKitchen is significantly better than the other two — multi-layer gradients, visible decoration, bento cards, strong color. The gap points to 8 systemic problems in the generation pipeline.

## Problems

| # | Problem | Severity | Current state | Target state |
|---|---------|----------|---------------|--------------|
| 8a | Font weights not loaded | Critical | Slidev loads only 400+600; CSS uses 800 → synthetic bold | Explicit @import with 400-800 |
| 8b | Fonts without Cyrillic | Critical | Outfit/Sora = Latin-only → mixed fonts in one line | Cyrillic Whitelist + Blacklist |
| 1 | Flat backgrounds | Critical | One solid color per slide | 2-3 layers: base + atmosphere + texture |
| 2 | No visual anchors | Critical | 100% text+icons, zero charts/SVG | Ghost typography + SVG data viz |
| 3 | Invisible decoration | Major | Opacity 0.08 on light backgrounds | Calibrated opacity by theme luminance |
| 4 | Card monotony | Major | 3 identical cards in a row | Bento grid, mixed styles, hierarchy |
| 5 | No focal point | Major | 5+ elements equal weight | One 2x dominant element per slide |
| 6 | Template structure | Major | 70%+ slides: label+heading+grid | Pattern break every 2-3 slides |
| 7 | No personality | Major | Presentations indistinguishable | Combinatorial signature system |

## Files Modified

| File | Changes |
|------|---------|
| SKILL.md | Step 4 (@import rule), Font Selection (Cyrillic whitelist/blacklist, updated Good pairs), Step 5 (background layers, ghost typography, card diversity, focal point, pattern break), QA-0b (decoration visibility blocking check, focal point check), --stitch=learn procedure |
| design-principles.md | Principle 6 (decoration calibration), new background layer principle |
| preset-format.md | Remove static `signature` field, add reference to decoration-library.md |
| scoring-subroutine.md | Axis 4 (visual anchors), Axis 9 (decoration visibility) |
| NEW: references/decoration-library.md | Combinatorial decoration component library |
| NEW: docs/research/stitch-learned-patterns.md | Auto-populated by --stitch=learn |

---

## Section 1: Font Fixes (8a + 8b)

### 1.1 Mandatory @import with all weights (SKILL.md Step 4)

Add to Step 4 (Write styles/index.css), at the top of the "Include:" list:

```
FONT LOADING RULE — styles/index.css MUST begin with an explicit @import:

@import url('https://fonts.googleapis.com/css2?
  family=<HEADING_FONT>:wght@400;500;600;700;800
  &family=<BODY_FONT>:wght@400;500;600;700
  &display=swap');

Do NOT rely on Slidev's automatic font loading from frontmatter — it loads
only weights 400+600 by default. The @import ensures ALL weights used in
inline styles are available, preventing synthetic bold rendering artifacts.
```

### 1.2 Cyrillic Whitelist + Blacklist (SKILL.md Font Selection)

Add after the existing Serif Blacklist:

```
CYRILLIC FONT RULES — For Russian-language presentations (detected by Cyrillic
characters in the outline):

CYRILLIC WHITELIST (confirmed full Cyrillic coverage at all weights):
  Heading: Manrope, Plus Jakarta Sans, Montserrat, Rubik, Noto Sans, Geist,
           Hanken Grotesk, IBM Plex Sans
  Body:    DM Sans, Nunito, Source Sans Pro, IBM Plex Sans, Noto Sans,
           Rubik, Manrope, Arimo

CYRILLIC BLACKLIST (Latin-only — NO Cyrillic glyphs):
  Outfit, Sora, Barlow, Urbanist, Epilogue, Lexend, Spline Sans, Work Sans,
  Be Vietnam Pro, Public Sans

If outline contains Cyrillic and a font from the Cyrillic Blacklist is selected →
replace with the nearest Cyrillic-safe alternative. Log: "Font [X] has no Cyrillic,
replaced with [Y]."
```

### 1.3 Updated Good sans+sans pairs

Replace the current Good pairs table:

| Heading (display) | Body (text) | Contrast type |
|-------------------|-------------|---------------|
| Manrope | DM Sans | Semi-rounded vs Humanist |
| Plus Jakarta Sans | Nunito | Geometric vs Rounded friendly |
| Montserrat | Source Sans Pro | Geometric bold vs Neutral clean |
| Rubik | IBM Plex Sans | Rounded warm vs Corporate humanist |
| Geist | Noto Sans | Modern tech vs Universal neutral |

All pairs verified for full Cyrillic coverage at weights 400-800.

---

## Section 2: Multi-layer Background System (Problem 1)

### 2.1 SKILL.md Step 5 — Background Layer Rule

Add as HARD RULE in Step 5, after the Font Size Floor:

```
BACKGROUND LAYER SYSTEM — Every slide MUST have minimum 2 background layers
implemented as real HTML div elements (z-index:0, pointer-events:none):

Layer 1 (base): solid color — var(--bg-base), var(--bg-alt), or var(--bg-accent)

Layer 2 (atmosphere): ONE of these, as a positioned div inside the background:
  - Radial glow: radial-gradient(ellipse at 80% 20%, rgba(var(--accent-rgb),OPACITY), transparent 60%)
  - Ambient gradient: linear-gradient(135deg, rgba(var(--accent-rgb),OPACITY), transparent)
  - Vignette: radial-gradient(ellipse at center, transparent 50%, rgba(0,0,0,0.06) 100%)

Layer 3 (texture, on 30-40% of slides): dot-grid, noise, or diagonal stripes
from references/decoration-library.md

CRITICAL: use real HTML <div> elements, NOT CSS pseudo-elements.
::after/::before do NOT render in Slidev headless PNG export.

OPACITY CALIBRATION:
  Light backgrounds (luminance > 70%): atmosphere 0.15-0.30, texture 0.06-0.12
  Dark backgrounds (luminance < 30%): atmosphere 0.08-0.15, texture 0.03-0.08
```

### 2.2 composition-archetypes.md update

Add `<!-- ATMOSPHERE LAYER -->` placeholder comment inside the background div of every archetype HTML skeleton, reminding the generator to insert atmosphere/texture layers.

---

## Section 3: Visual Anchors (Problem 2)

### 3.1 Ghost Typography (SKILL.md Step 5)

```
GHOST TYPOGRAPHY — On stat-hero and breathing slides, add a decorative ghost
element: the hero number or key symbol rendered at 12-20rem, opacity 0.04-0.08,
positioned absolute behind content.

Example implementation:
<div style="position:absolute;top:50%;right:5%;transform:translateY(-50%);
  font-family:var(--font-heading);font-size:18rem;font-weight:800;
  color:rgba(var(--accent-rgb),0.06);line-height:1;pointer-events:none;z-index:0;">
  ₽
</div>

Use on 2-3 slides per deck maximum. Never on every slide.
Good candidates: currency symbols (₽, $), percentages, key metric numbers.
```

### 3.2 SVG Data Visualization enforcement (SKILL.md Step 5)

```
DATA VIZ RULE — If a slide contains 3+ numeric metrics, at least ONE must be
visualized as inline SVG, not text-in-card:

  Proportions/share → donut chart (stroke-dasharray technique)
  Comparison → horizontal bar chart (rect elements)
  Growth/timeline → line with circle dots
  Parts of whole → stacked bar

SVG requirements:
  - Use var(--color-accent) for primary fill/stroke
  - Keep simple: max 5 data points
  - Labels inline on/near elements, no separate legend
  - viewBox-based for scaling
  - Include <title> for accessibility
```

### 3.3 scoring-subroutine.md — Axis 4 (Visual impact)

Append to Low descriptor: "; deck has zero visual anchors — no SVG charts, ghost typography, or decorative data viz"

---

## Section 4: Decoration + Cards + Focal Point + Pattern Break (Problems 3-6)

### 4.1 Decoration Visibility — BLOCKING QA (Problem 3)

Add to QA-0b:

```
Decoration visibility (BLOCKING):
- Scan all decorative div elements (z-index:0, pointer-events:none).
- If slide has light background AND decorative opacity < 0.12 → STOP.
  Auto-fix: raise opacity to 0.15.
- If ANY decorative element < 200px in width or height → STOP.
  Auto-fix: raise to 250px minimum.
```

### 4.2 Card Diversity — HARD RULE (Problem 4)

Add to Step 5:

```
CARD DIVERSITY — BANNED: 3+ cards with identical styling on one slide.
When showing 3 items, MUST use one of:
  a) Bento grid: 1 large (grid-row:span 2 or 1.4fr) + 2 smaller
  b) Mixed styles: card-solid + card-ghost + card-accent
  c) Size hierarchy: first card accent-bordered + larger, rest surface
  d) One card = metric hero (big number), rest = supporting text
```

### 4.3 Focal Point Rule (Problem 5)

Add to Step 5:

```
FOCAL POINT — Every content slide MUST have ONE dominant element
visually 2x+ larger than the next largest:
  Stat slides: hero number 4-8rem, supporting 1-1.5rem
  Card slides: featured card 40-60% of content area
  Data slides: chart is dominant, text supports
  Quote slides: quote 2-3rem, attribution 0.85rem

QA-0b check (WARNING): if largest and second-largest elements differ
by < 1.5x in font-size → "no clear focal point"
```

### 4.4 Pattern Break (Problem 6)

Update the existing Structural Break Rule in Step 5:

```
PATTERN BREAK — After generating Composition Plan, count slides using
"label-top + heading + grid/cards" structure.

If > 40% of content slides use this pattern → MUST replace excess with:
  a) Full-bleed centered: stat-hero, quote-pull — no label, no grid
  b) Visual-dominant: one large element left/right, text secondary
  c) Minimal: heading + single sentence, generous whitespace
  d) Inverted: content top, heading bottom
  e) Asymmetric split: 30/70 or 70/30, not 50/50
```

---

## Section 5: Combinatorial Decoration Library (Problem 7)

### 5.1 New file: references/decoration-library.md

A library of **pre-tested CSS components** that the generator combines like LEGO — never invents CSS on the fly. Each component is a ready-to-use HTML snippet with `{{PLACEHOLDER}}` slots.

Structure:

```markdown
# Decoration Component Library

Generator picks: 1 shape + 1 behavior + 1 position + 1 scale → assembles HTML.
All CSS is pre-tested. Never invent decoration CSS — only use components from this library.

## Shapes

### circle
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border-radius:50%;{{BEHAVIOR}};{{POSITION}};pointer-events:none;z-index:0;"></div>

### arc
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border:2.5px solid rgba(var(--accent-rgb),{{OPACITY}});border-radius:50%;{{POSITION}};pointer-events:none;z-index:0;"></div>

### blob
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border-radius:40% 60% 55% 45%;{{BEHAVIOR}};{{POSITION}};pointer-events:none;z-index:0;"></div>

### dot-grid
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};background-image:radial-gradient(circle,rgba(var(--accent-rgb),{{OPACITY}}) 1.2px,transparent 1.2px);background-size:20px 20px;{{POSITION}};pointer-events:none;z-index:0;"></div>

### line-horizontal
<div style="position:absolute;width:{{SIZE}};height:2px;background:rgba(var(--accent-rgb),{{OPACITY}});{{POSITION}};pointer-events:none;z-index:0;"></div>

### ring
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};border:1.5px solid rgba(var(--accent-rgb),{{OPACITY}});border-radius:50%;{{POSITION}};pointer-events:none;z-index:0;"></div>

### diamond
<div style="position:absolute;width:{{SIZE}};height:{{SIZE}};transform:rotate(45deg);border:2px solid rgba(var(--accent-rgb),{{OPACITY}});{{POSITION}};pointer-events:none;z-index:0;"></div>

## Behaviors (replaces {{BEHAVIOR}} in shapes)

### gradient-fill
background:radial-gradient(circle,rgba(var(--accent-rgb),{{OPACITY}}),transparent 65%)

### solid-fill
background:rgba(var(--accent-rgb),{{OPACITY}})

### glow
background:radial-gradient(circle,rgba(var(--accent-rgb),{{OPACITY}}) 0%,transparent 70%);filter:blur(30px)

### border-only
(empty — shape already has border defined)

## Positions (replaces {{POSITION}})

### corner-TR
top:-80px;right:-80px

### corner-BL
bottom:-60px;left:-60px

### corner-TL
top:-100px;left:-100px

### corner-BR
bottom:-80px;right:-60px

### center-behind
top:50%;left:50%;transform:translate(-50%,-50%)

### edge-right
top:20%;right:-40px

### edge-left
top:30%;left:-60px

### edge-bottom
bottom:-40px;left:30%

## Scales (replaces {{SIZE}})

### subtle
200px

### medium
400px

### bold
600px

### hero
800px

## Opacity Presets (replaces {{OPACITY}})

### light-theme-atmosphere: 0.20
### light-theme-texture: 0.08
### light-theme-border: 0.18
### dark-theme-atmosphere: 0.10
### dark-theme-texture: 0.04
### dark-theme-border: 0.15
```

### 5.2 How the generator uses the library

In SKILL.md Step 5, point 8 (Apply Decorative Layer):

```
DECORATION FROM LIBRARY — When adding decorative elements:
1. Open references/decoration-library.md
2. Pick a shape (circle, arc, blob, dot-grid, ring, diamond, line-horizontal)
3. Pick a behavior (gradient-fill, solid-fill, glow, border-only)
4. Pick a position (corner-TR, corner-BL, center-behind, edge-right, etc.)
5. Pick a scale (subtle/medium/bold/hero) based on slide importance
6. Pick opacity from the preset table (light-theme-* or dark-theme-*)
7. Assemble HTML by substituting {{PLACEHOLDERS}}
8. Insert into the slide's background div

DIVERSITY RULES:
- No two adjacent slides use the same shape+position combination
- Per deck: use minimum 3 different shapes
- Per deck: use minimum 2 different positions
- Rotate: if slide N used corner-TR, slide N+1 uses corner-BL or edge-right

NEVER invent decoration CSS outside the library. If a new pattern is needed,
it must first be added to decoration-library.md (via --stitch=learn).
```

### 5.3 Library growth via --stitch=learn

When `--stitch=learn` extracts a new decorative pattern from Stitch:
1. Classify it: which shape? which behavior? (or new?)
2. If new shape/behavior → add to decoration-library.md with pre-tested CSS
3. Format as the standard template with {{PLACEHOLDERS}}
4. Commit with: "feat(slidev): add [shape/behavior] to decoration library from stitch learn cycle [N]"

---

## Section 6: --stitch=learn Mode

### 6.1 Help text addition

```
/slidev --stitch=learn=N              Learn from Stitch as design teacher (N cycles)
```

### 6.2 Procedure (STL-1 through STL-3)

**STL-1: Generate N diverse outlines** — Same as L-2 in --learn: Russian language, varied industries/formats/sizes.

**STL-2: For each cycle i = 1 to N:**

**STL-2.1: Stitch generates** — Create project → generate full presentation as HTML via `generate_screen_from_text` with modelId `GEMINI_3_1_PRO`. Prompt includes all slides content. Save to `<edu_dir>/stitch_<i>/stitch-output.html`.

**STL-2.2: Our generator makes the same presentation** — Standard pipeline with current skill rules, same outline. Save to `<edu_dir>/stitch_<i>/our-output/`. Export PNG.

**STL-2.3: Comparative analysis** — Agent compares both HTML outputs across 6 dimensions:

```
For each dimension, extract concrete CSS values:

BACKGROUNDS:
  Stitch: [layers count, gradient types, opacity values]
  Ours:   [layers count, gradient types, opacity values]
  Gap:    [what Stitch does better]
  → RULE: [specific CSS addition to skill]

LAYOUT:
  Stitch: [grid templates, proportions, asymmetry]
  Ours:   [grid templates]
  Gap:    [...]
  → RULE: [...]

TYPOGRAPHY:
  Stitch: [size hierarchy, weight distribution]
  Ours:   [...]
  → RULE: [...]

DECORATION:
  Stitch: [shapes, opacity, sizes, positions]
  Ours:   [...]
  → New decoration-library.md components if found

CARDS:
  Stitch: [card variety, bento ratios, styles]
  Ours:   [...]
  → RULE: [...]

HIERARCHY:
  Stitch: [focal point ratio, dominant element size]
  Ours:   [...]
  → RULE: [...]
```

**STL-2.4: Extract patterns** → append to `docs/research/stitch-learned-patterns.md`:
```markdown
## Pattern: [name]
- Source: Stitch cycle [i], slide [N]
- CSS: [concrete CSS code]
- When: [which slide types benefit]
- Added to: [which skill file was updated]
```

**STL-2.5: Apply improvements** via Python script (same as L-3e in --learn). Update SKILL.md, design-principles.md, decoration-library.md.

**STL-2.6: Interim report** to user:
```
━━━ Stitch Learn <i>/<N> — СРАВНЕНИЕ ━━━

Оценка Stitch: X/10
Наша оценка:   Y/10
Разница:       [+/-Z]

Что Stitch делает лучше:
  - [аспект] — извлечено правило: "[краткое описание]"
  - [аспект] — новый компонент в decoration-library
  ...

Что мы делаем лучше:
  + [аспект] (сохраняем)

Применено правил: [кол-во]
Новых компонентов в библиотеке: [кол-во]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**STL-2.7: Git commit + tag**

**STL-3: Final report** with progress.md table (same format as --learn).

### 6.3 Stitch API handling

Same retry strategy as other --stitch modes: blocking call → on error poll list_screens 60s x3 → if still missing → STOP (no fallback).

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Explicit @import, not Slidev fonts.weights | @import is more reliable and visible; Slidev auto-loader has unpredictable weight selection |
| Cyrillic Whitelist approach (not auto-detect) | Auto-detecting Cyrillic support at runtime is unreliable; curated list is safer |
| Real HTML divs for backgrounds, not pseudo-elements | Confirmed in round 1 learn + research: ::after doesn't render in Playwright export |
| Combinatorial decoration, not static list | Static list → recognizable AI patterns after N presentations; combinatorics → thousands of unique combinations |
| Decoration library as reference file, not inline rules | Keeps SKILL.md focused on logic; library is extensible via --stitch=learn |
| BLOCKING QA for decoration visibility | Non-blocking warnings proved ineffective in rounds 1-2 |
| --stitch=learn as comparison mode | Self-critique (--learn) improves rules; Stitch comparison teaches new CSS patterns |

## Feedback Traceability

| Finding | Source | Section |
|---------|--------|---------|
| Font synthetic bold (weights not loaded) | innosmart-tech-pitch analysis | 1.1 |
| Outfit has no Cyrillic | cloudkitchen-pitch analysis | 1.2 |
| Flat backgrounds (1 color) | All 3 presentations | 2 |
| No charts, diagrams, ghost type | innosmart-tech-pitch, den-rozhdeniye | 3 |
| Decoration invisible on light bg | innosmart, den-rozhdeniye | 4.1 |
| 3 identical cards | All 3 presentations | 4.2 |
| No focal point | innosmart-tech-pitch slides 2, 5 | 4.3 |
| 70%+ same label+heading+grid | innosmart-tech-pitch | 4.4 |
| Presentations indistinguishable | Comparing all 3 side by side | 5 |
