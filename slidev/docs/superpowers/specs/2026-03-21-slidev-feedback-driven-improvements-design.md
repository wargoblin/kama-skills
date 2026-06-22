# Slidev Skill: Feedback-Driven Improvements

**Date:** 2026-03-21
**Status:** Approved (brainstorming)
**Scope:** Improve `/slidev` skill generation quality based on human feedback + research gaps

## Context

Real-world feedback on a generated presentation ("Prezentatsiya kotoraya ubezhdaet") revealed systematic problems that the skill's existing rules and QA pipeline did not prevent. Three research documents (AI Detection Guide, Beautiful Presentations Guide, Ugly Presentations Anti-Patterns) contain additional principles not yet incorporated.

### Feedback Sources

| Participant | Role | Weight |
|-------------|------|--------|
| Anna Pozdnyakova (Anya) | Designer | Highest |
| Alina | Detailed visual analysis | High |
| Vladimir Sivaev | AI-pattern detection | Medium |
| Maka (Maxim) | General perception | Medium |
| Dmitrii (author) | Additional visual observations from screenshots | Direct |

### Approach

**Hybrid (C):** Feedback items (F1-F13) as targeted additions with full pipeline integration. Research items (R1-R9) as systematic expansions. All changes flow through the complete pipeline: rules -> generation -> code QA -> visual QA -> scoring.

**Note on R5 (Executive sequence):** R5 — "recommendation on slides 1-2, not buried at the end" — is already partially covered by the existing Anti-Pattern #7 in design-principles.md ("Recommendation buried after slide 5") and QA-0c's "recommendation position check." No additional changes needed; existing coverage is sufficient.

---

## Section 1: Systematic Background Palette

**Solves:** F3 (inconsistent background temperature), F6 (pure #000/#FFF), F10 (pill-on-gradient mud), F13 (jarring CTA slide)

### Design

Generate three background levels in `styles/index.css`:

```css
--bg-base    /* primary background, 60% of slides */
--bg-alt     /* alternative background, 30% of slides */
--bg-accent  /* cover/CTA background, 10% of slides */
```

### Rules

1. All three levels share the **same color temperature** (all warm or all cool)
2. Luminance delta between bg-base and bg-accent: **max 40%**
3. **Never** pure `#000000` or `#FFFFFF` — always tinted (e.g., `#FAF9F6`, `#1A2332`)
4. If bg-accent luminance < 30% (dark) and bg-base luminance > 70% (light), the **pre-CTA slide** (the slide immediately before the CTA/closing slide, regardless of deck position) uses bg-alt as a visual bridge. This **replaces** the existing Principle 1 "Penultimate" row.
5. Archetypes reference `var(--bg-base)` / `var(--bg-alt)` / `var(--bg-accent)` — no hardcoded colors

### Relationship to existing Principle 1

The existing Principle 1 defines a per-slide-type luminance-delta recipe table (Cover +3-5%, Content even +3%, etc.). The new bg-system **replaces** that table. Mapping:

| Existing Principle 1 slide type | New bg-level |
|---------------------------------|-------------|
| Cover | `--bg-accent` |
| Content (odd) | `--bg-base` |
| Content (even) | `--bg-alt` |
| Section divider | `--bg-alt` |
| Stat/fact hero | `--bg-base` |
| Problem/Solution | `--bg-base` or `--bg-alt` (alternating) |
| Ask/CTA | `--bg-accent` |
| Pre-CTA | `--bg-alt` (bridge) |

The luminance-delta values within each level are no longer needed — bg-base, bg-alt, and bg-accent are defined as concrete hex values in `styles/index.css`, not as relative deltas.

### Files to Change

| File | Change |
|------|--------|
| `design-principles.md` | New principle or extend Principle 1/12: Background System |
| `SKILL.md` Step 4 | Generate 3 bg-level CSS variables with temperature consistency |
| `SKILL.md` Step 4.5 | Map archetypes to bg-levels (cover->accent, content->base/alt, section-divider->alt, CTA->accent) |
| `composition-archetypes.md` | Replace hardcoded background colors with `var(--bg-*)` tokens |
| `SKILL.md` QA-0c | New checks: all backgrounds use only 3 levels; temperature consistency; no #000/#FFF |
| `scoring-subroutine.md` | Axis 7 (Color conviction): verify bg-system compliance |

---

## Section 2: Typographic Discipline

**Solves:** F2 (too many typeface variations per slide), F7 (italic in non-quote context), F9 (font digit rendering)

### 2a. Max Variations Per Slide

**Rule:** Maximum **3 visually distinct type treatments** on one slide:
- Heading font (regular or bold) — for titles
- Body font (regular) — for text
- One accent treatment — bold OR italic OR uppercase, not all simultaneously

### 2b. Italic = Quotes Only

**Rule:** Italic permitted only in:
- `<blockquote>` elements and quotations
- Attribution lines ("— Garr Reynolds")
- Image captions

Italic in headings, subheadings, body text = **FAIL**.

### 2c. Font Digit Rendering Check

**Rule:** When a presentation contains prominent numbers (stat-hero, metrics), the heading font must have **visually distinguishable bold for digits**. Add to Font Number Blacklist: **Bricolage Grotesque** (bold digits visually indistinguishable from regular weight at display sizes — confirmed via feedback). If a number-heavy presentation selects a font not on the blacklist, the generator must verify during Step 1 that bold digit rendering is distinct from regular by checking the font's weight range (fonts with weight range < 300 between regular and bold = WARNING). Blacklisted font in number-heavy presentation = FAIL.

### Files to Change

| File | Change |
|------|--------|
| `design-principles.md` | Extend Principle 3 (Typographic Drama): max 3 treatments/slide, italic restriction |
| `SKILL.md` QA-0c | New code checks: count font-family+weight+style combos per slide; grep italic outside blockquote |
| `SKILL.md` Font Selection | Expand Font Number Blacklist; add digit-bold-check instruction |
| `scoring-subroutine.md` | Axis 3 (Font discipline): penalize >3 treatments/slide, italic misuse |

---

## Section 3: Pill/Badge and Decorative Elements

**Solves:** F8 (pill vertical centering), F10/F11 (pill-on-gradient mud)

### 3a. Pill Vertical Centering — Mandatory CSS Template

All pill/badge elements must use:
```css
display: inline-flex;
align-items: center;
justify-content: center;
line-height: 1;
padding: 6px 18px; /* vertical >= horizontal * 0.33 */
```

`line-height: 1` is critical — without it, uppercase + letter-spacing text shifts upward.

### 3b. Pill Background — "No Mud" Rule

Pill elements must not visually merge with decorative gradient spots beneath them.

**Preferred solution:** pill has **opaque background** (`background: var(--bg-base)` or `var(--bg-alt)`), not semi-transparent `rgba(...)`. Gradient beneath does not bleed through.

**Alternative:** if pill is semi-transparent by design, radial-gradient spots must **not be placed in the top 20%** of the slide (where pills typically appear).

### 3c. Decorative Gradient Spots — Exclusion Zone

Radial-gradient decorative elements (Principle 6):
- Must not overlap text-heavy zones (headings, pills, body text)
- Place in corners or outside content area
- Opacity <= 0.20 if content overlap is unavoidable

### Verification (dual: code + visual)

| Check | Type | Rule |
|-------|------|------|
| QA-0b | Code | Grep `border-radius.*20px` (or pill/badge/chip class) — verify `line-height: 1`, `display:.*flex`, `align-items: center` present |
| QA-0b | Code | Grep `rgba(` in pill background + `radial-gradient` in same slide = WARNING "potential mud overlap" |
| QA-0c | Code | Structured pill checklist: centering CSS, opacity, gradient exclusion |
| QA-4 | Visual | Verify pill text centered; verify pill boundaries crisp on PNG |

### Files to Change

| File | Change |
|------|--------|
| `design-principles.md` | Extend Principle 6 (Decorative Layer): exclusion zones. New sub-principle for pill rendering |
| `layout-css-patterns.md` | Add pill CSS template with mandatory properties |
| `composition-archetypes.md` | Add `line-height:1` to all archetypes with pills |
| `SKILL.md` QA-0b, QA-0c | New grep checks and structured checklist |
| `SKILL.md` QA-4 | Visual check instructions for pill clarity |
| `scoring-subroutine.md` | Axis 9 (Decorative quality): penalize mud overlap |

---

## Section 4: Text Contrast — "Weakest Segment First"

**Solves:** F12 (muted text fades on accent surfaces)

### Design

When defining `--color-muted` in `styles/index.css`:

1. Enumerate **all surfaces** where muted text appears: `--bg-base`, `--bg-alt`, `--color-accent-bg`, `--color-surface`
2. Calculate contrast ratio of `--color-muted` against **each** surface
3. The **lowest ratio** must pass WCAG AA (>= 4.5:1)
4. If it fails on accent surface, darken `--color-muted` **globally** until the weakest segment passes
5. Result: one `--color-muted` everywhere, guaranteed readable on the hardest surface

### Verification (dual: code + visual)

| Check | Type | Rule |
|-------|------|------|
| QA-0c | Code | New checkpoint "Muted text on all surfaces": list all CSS variable pairs (`--color-muted` vs `--bg-base`, `--bg-alt`, `--color-accent-bg`, `--color-surface`) from `styles/index.css`. Verify each pair's contrast ratio >= 4.5:1. This is a variable-level check, not a per-element DOM traversal. FAIL if any pair < 4.5:1 |
| QA-4 | Visual | Find palest text on each exported PNG. If any text is hard to read against its background = FAIL. This is the definitive PASS/FAIL gate for contrast |

### Files to Change

| File | Change |
|------|--------|
| `design-principles.md` | Extend Principle 12 (Accent Hierarchy): "weakest segment first" rule |
| `SKILL.md` Step 4 | When generating CSS, add comment with contrast ratios for all pairs |
| `SKILL.md` QA-0c | New contrast check: CSS variable pairs from styles/index.css |
| `SKILL.md` QA-4 | Visual contrast verification on exported PNGs (definitive gate) |
| `scoring-subroutine.md` | Axis 7 (Color conviction): "all text-on-surface pairs pass WCAG" |

---

## Section 5: Anti-AI Pattern Reinforcement

**Solves:** F1 (text arrows), F4 (uniform icon containers), F5 (multiple gradient types)

### 5a. Text Arrow Ban

**Rule:** Characters `→`, `←`, `↑`, `↓`, `⟶`, `➜`, `▶` are **banned** in slides.md except inside `<code>` / backticks / speaker notes (`<!-- -->`).

**Replacements:**
- Sequences ("A → B → C"): rephrase textually or use SVG arrow figures
- Pipelines: timeline-horizontal/timeline-zigzag archetype with numbered steps
- Navigation: Icon.vue `arrow-right` inside layout

**Code check:** QA-0c grep `[→←↑↓⟶➜▶]` in slides.md (excluding `<!-- -->` and backticks). Any match = FAIL.

### 5b. Icon Container Variation — "No Uniform Circles"

**Rule:** When 3+ icons appear on one slide, their containers must **differ** on at least one dimension:
- Size (e.g., 48px, 56px, 64px), OR
- Shape (circle, rounded-square, no container), OR
- Style (solid fill, ghost outline, accent bg), OR
- Placement (not strictly equal-gap row)

All containers same width+height+border-radius+border = **FAIL**.

**Exception:** numbered steps (1, 2, 3) may be uniform — uniformity is semantically justified.

**Code check:** QA-0b grep all `border-radius:50%` with icons on same slide. If 3+ with identical width+height+border = WARNING.

### 5c. Gradient Type Limit — Max 1 Decorative Gradient Type

**Rule:** One presentation uses **only 1 type of decorative gradient**:
- Either `radial-gradient` for decorative blobs/orbs (Principle 6)
- Or `linear-gradient` for decorative backgrounds/overlays
- Not both as decorative elements

**Clarification — what counts as "decorative":** Gradients used purely for visual embellishment (blobs, glows, background washes). Gradients that serve functional purposes are exempt:
- Card `.card-accent` with a subtle `linear-gradient` tint = **functional** (differentiates card type) — exempt
- `bg-accent` slide with `linear-gradient` background = **structural** (bg-system level) — exempt
- A `radial-gradient` blob floating in a corner = **decorative** — counted

**Worked example:** If you choose `radial-gradient` as decorative type: decorative blobs use radial-gradient, card backgrounds use flat solid colors (no linear-gradient decoration), and the CTA bg-accent slide may still use linear-gradient (structural exempt). If you choose `linear-gradient`: decorative overlays use linear-gradient washes, no radial-gradient blobs anywhere.

**Code check:** QA-0c — grep `radial-gradient` and `linear-gradient` in slides.md. For each match, classify as decorative or functional/structural. If both types appear as decorative = WARNING "multiple decorative gradient types".

### Files to Change

| File | Change |
|------|--------|
| `design-principles.md` | Strengthen Principle 10 (SVG): explicit arrow char ban. Extend Principle 4 (Icons): container variation. New Anti-Pattern: mixed gradient types |
| `SKILL.md` QA-0c | 3 new grep checks: arrow chars, uniform icons, mixed gradients |
| `SKILL.md` Step 4 | Choose gradient type during CSS generation |
| `SKILL.md` Step 5 | Enforce chosen gradient type during slide writing |
| `composition-archetypes.md` | Remove hardcoded decorative gradient types. Add comment to each archetype specifying which gradient slots are decorative vs structural. Generator enforces chosen decorative type as a behavioral rule during Step 5 |
| `scoring-subroutine.md` | Axis 1: penalize uniform icons. Axis 7: penalize mixed gradients |

---

## Section 6: Research-Driven Expansions

### 6a. Rule of Thirds (R1)

**Source:** Beautiful Presentations Guide

**Rule:** Place dominant element at thirds-grid intersection, not dead center, except on breathing/stat-hero slides.

**Verification:**
- **QA-4 (visual, primary):** On each non-breathing, non-hero PNG, check if the heading/dominant element is dead center with no asymmetric counterweight (no offset column, no side image, no left-aligned body text). If every content slide is symmetrically centered = WARNING "consider Rule of Thirds for at least 30% of content slides."
- **QA-0b (code, heuristic):** If a non-breathing, non-cover, non-CTA slide has its heading at `text-align:center` AND the slide contains no grid/flex split (no `grid-template-columns` with unequal fractions, no asymmetric width splits) = WARNING. This is a heuristic — visual QA-4 is the definitive check.

**Files:** `design-principles.md` Principle 2; `SKILL.md` QA-4, QA-0b; `composition-archetypes.md` add focal-point comments per archetype.

### 6b. Chart Type Selection Matrix (R2)

**Source:** Beautiful Presentations Guide

**Rule:** Add to Principle 8:

| Goal | Correct type | Avoid |
|------|-------------|-------|
| Category comparison | Horizontal bar | 3D, exploded pie |
| Change over time | Line chart | Bar with many periods |
| Parts of whole | Stacked bar / Donut (<=5) | Pie >6 segments |
| Correlation | Scatter + trend line | Tables |
| Simple proportion | Donut (3-5 segments) | 3D pie |

**Code check:** QA-0b — pie chart with >5 items = FAIL.

**Files:** `design-principles.md` Principle 8.

### 6c. Gestalt Principles Checks (R6)

**Source:** Ugly Presentations Anti-Patterns

**Rules:**
- **Proximity:** label/caption must be visually adjacent to its referent element. **QA-4 (visual, primary):** on PNG, if a label appears more than ~1/4 slide height away from its element = WARNING. **QA-0b (code, heuristic):** within a grid or flex container, check that caption elements are in the same or adjacent cell as their referent.
- **Similarity:** same-level elements in a grid must share padding, border-radius, font-size. **QA-0b (code):** grep border-radius values within one grid container — if different values = WARNING.
- **Figure-ground:** text on background-image without overlay = FAIL (partially exists, no change needed).

**Files:** `SKILL.md` QA-4, QA-0b; `design-principles.md` new sub-section in layout principles.

### 6d. Colorblind Enforcement (R9)

**Source:** Beautiful + Ugly guides

**Rule:** If accent is red (hue 0-30) and used alongside green (hue 90-165) on same slide = FAIL. Replacement: blue/orange.

This **augments** the existing QA-0c colorblind check ("No colorblind-unsafe data pairs (red+green, green+brown, blue+purple)") by adding concrete hue-range thresholds. Do not create a separate check — expand the existing one with: "Red hue 0-30 + green hue 90-165 on same slide = FAIL. Brown hue 20-40 + green hue 90-165 = WARNING."

**Files:** `design-principles.md` Principle 8 (add hue ranges to existing colorblind rule); `SKILL.md` QA-0c (expand existing checkpoint, not new one).

### 6e. Text Burstiness (R8)

**Source:** AI Detection Guide

**Rule:** If a slide has 3 or more bullets AND the word count of every bullet is within +-3 words of the arithmetic mean of all bullet word counts on that slide = WARNING "monotonous text length, vary sentence structure." The +-3 threshold reduces false positives on naturally concise slides.

**Files:** `content-review-subroutine.md`.

### 6f. Cost-of-Inaction Slide (R4)

**Source:** AI Detection Guide

**Rule:** For pitch/business presentations, check for a slide answering "what happens if we don't act?" If absent = SUGGESTION (not FAIL — not all presentations are business pitches).

**Files:** `content-review-subroutine.md`.

### 6g. Cognitive Overload Prevention (R7)

**Source:** Ugly Presentations Anti-Patterns (Mayer Redundancy Principle)

**Rule:** Speaker notes must EXTEND slide content, not repeat it. If slide text and speaker note say the same thing = redundancy violation.

**Files:** `content-review-subroutine.md`; `design-principles.md`.

### 6h. Modern Trends Vocabulary for Presets (R3)

**Source:** Beautiful Presentations Guide

**Addition:** In `preset-format.md`, add "Trend vocabulary" section for `--create-preset`:
- `bold-minimalism` — white text on dark, max 1-2 elements/slide
- `editorial` — wide margins, asymmetric grids, magazine feel
- `human-centered` — organic shapes, natural palette, intentional imperfection
- `broken-grid` — overlapping text/images, angular sections

**Scope:** This addition is purely for `--create-preset` mode. The trend vocabulary is only surfaced when a preset is being created or the user explicitly references a trend. It does not affect default generation design thinking.

**Files:** `preset-format.md`.

---

## Complete File Change Matrix

| File | Sections |
|------|----------|
| `design-principles.md` | 1, 2, 3, 4, 5, 6a, 6b, 6c, 6d, 6g |
| `SKILL.md` (Step 4) | 1, 4, 5c |
| `SKILL.md` (Step 4.5) | 1 |
| `SKILL.md` (Step 5) | 5c |
| `SKILL.md` (QA-0b) | 1, 2, 3, 5b, 6a, 6c |
| `SKILL.md` (QA-0c) | 1, 2, 3, 4, 5a, 5c, 6d |
| `SKILL.md` (QA-4) | 3, 4, 6a, 6c |
| `SKILL.md` (Font Selection) | 2c |
| `composition-archetypes.md` | 1, 3, 5c, 6a |
| `layout-css-patterns.md` | 3 |
| `scoring-subroutine.md` | 1, 2, 3, 4, 5, 6 — see Scoring Guidance below |
| `content-review-subroutine.md` | 6e, 6f, 6g |
| `preset-format.md` | 6h |

## Scoring Guidance

Concrete text to add/modify in `scoring-subroutine.md` axis descriptions:

| Axis | Add to Low (1-3) | Add to High (7-10) |
|------|-------------------|---------------------|
| 1 (Composition variety) | "3+ uniform icon containers on one slide" | "Icon containers vary in size, shape, or style" |
| 2 (Shape diversity) | No changes | No changes |
| 3 (Font discipline) | ">3 type treatments on one slide; italic outside quotes; blacklisted font for number-heavy deck" | "Max 3 treatments/slide; italic only in quotes; font digit rendering verified" |
| 4 (Visual impact) | No changes | "Rule of Thirds applied on 30%+ of content slides" |
| 7 (Color conviction) | "Pure #000/#FFF; inconsistent bg temperature; bg-system not used; mixed decorative gradient types; muted text fails WCAG on any surface" | "3-level bg-system; single color temperature; all text-on-surface pairs pass WCAG 4.5:1; single decorative gradient type" |
| 8 (Content clarity) | "Monotonous bullet length (all within +-3 words)" | "Varied sentence structure; cost-of-inaction slide present (pitch decks)" |
| 9 (Decorative quality) | "Pill/badge mud overlap with gradient; gradient exclusion zone violated" | "Pills crisp on all backgrounds; decorative elements in corners/edges only" |

---

## Feedback Traceability

| Feedback | Section | Status |
|----------|---------|--------|
| F1 — text arrows → | 5a | Covered |
| F2 — too many typeface treatments/slide | 2a | Covered |
| F3 — inconsistent background temperature | 1 | Covered |
| F4 — uniform icon containers | 5b | Covered |
| F5 — 3+ gradient types | 5c | Covered |
| F6 — pure #000/#FFF | 1 | Covered |
| F7 — italic in non-quote context | 2b | Covered |
| F8 — pill vertical centering | 3a | Covered |
| F9 — font digit bold indistinguishable | 2c | Covered |
| F10 — pill-on-gradient mud | 3b, 3c | Covered |
| F11 — pill on clean bg = good (control) | 3b | Covered |
| F12 — muted text fades on accent surface | 4 | Covered |
| F13 — jarring CTA slide transition | 1 | Covered |
| R1 — Rule of Thirds | 6a | Covered |
| R2 — Chart type matrix | 6b | Covered |
| R3 — Modern trends vocabulary | 6h | Covered |
| R4 — Cost-of-inaction slide | 6f | Covered |
| R5 — Executive sequence (recommendation first) | — | Already covered by existing Anti-Pattern #7 + QA-0c |
| R6 — Gestalt principles | 6c | Covered |
| R7 — Cognitive overload | 6g | Covered |
| R8 — Text burstiness | 6e | Covered |
| R9 — Colorblind enforcement | 6d | Covered |
