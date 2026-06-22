# Figmadeck: Design Rules + Adaptive Layout

## Overview

Four changes to the figmadeck skill:
1. New `references/design-rules.md` — full universal design rules from research
2. Expanded QA checks in `qa-cycle.md` — text density, spacing, overlap, hierarchy, contrast
3. Adaptive layout rules in `generation-pipeline.md` — prevent overflow when content is longer than Figma placeholder
4. Background-aware deduplication + context-based variant selection in `figma-extraction.md`

## Problem

Generated slides break when real content is longer than Figma template placeholders:
- Footer text overlaps when Russian labels are longer than English
- Column text overflows into footer zone
- Cards with long text exceed boundaries

Additionally, the skill lacks universal design knowledge (readability, density, hierarchy) and merges visually different slides (same structure, different backgrounds) into one archetype.

---

## Change 1: `references/design-rules.md`

**Create** new reference file with universal design rules extracted from:
- `slidev/docs/research/ugly-presentations-anti-patterns-ru.md`
- `slidev/docs/research/beautiful-presentations-guide-ru.md`
- `slidev/docs/research/html-css-presentation-techniques.md`
- `slidev/docs/research/slidev-design-best-practices.md`

**NOT a summary** — full content of each universal rule with explanation and thresholds. Only rules that don't conflict with Figma templates (no icon rules, no color rules, no archetype preferences).

### Sections:

1. **Text Density & Content Limits** — max 75 words, 6x6 bullet rule, max 2 nesting levels, 3-5 info units per slide (Cowan 2001), 1 idea per slide (3-second test), redundancy principle
2. **Typography & Readability** — font size minimums, max 2 families, left-align body text, line-height 1.2-1.45x, max-width 65ch, bold <= 20%, italic only in quotes
3. **Spacing & Layout** — min 30% whitespace, 8px grid, consistent margins (5-8% of slide width), no floating elements, overlap = failure
4. **Visual Hierarchy** — min 3:1 heading/body ratio, max 3 font sizes per slide, max 2 large elements, squint test, title = conclusion not label
5. **Contrast & Accessibility** — WCAG AA 4.5:1 body, 3:1 large text, 7:1+ for projection, no color-only differentiation, no pure black/white
6. **Element Spacing & Overflow Prevention** — no overlap ever, footer text width check, min 8px gap between adjacent elements, text shortening priority (rephrase → speaker notes), space redistribution
7. **Data Visualization** — no 3D charts, max 5-6 pie segments, bar axis at zero, remove gridlines, hero metric >= 2x supporting
8. **Animation** — only transform + opacity, 200-400ms transitions, max 5-8 clicks per slide

Add reference link in SKILL.md: `references/design-rules.md` — Universal design rules (readability, spacing, hierarchy).

---

## Change 2: QA checks in `qa-cycle.md`

**Modify** `references/qa-cycle.md` Phase A — add design rules checks AFTER basic checks (font floor, layout:none, hex scan) and BEFORE Figma comparison.

### New checks:

**Text density:**
- Word count per slide > 75 → FAIL. Auto-fix: shorten, move to speaker notes.
- Bullet count > 6 → FAIL. Auto-fix: merge or speaker notes.
- Bullet nesting > 2 levels → FAIL. Auto-fix: flatten.

**Typography:**
- Multi-line body text centered → FAIL. Auto-fix: text-align:left + max-width:65ch + margin:0 auto.
- Bold > 20% of text → WARNING. Log for report.
- More than 3 distinct font-sizes → WARNING.

**Spacing & overlap:**
- Any element bounding box intersects another → CRITICAL. Auto-fix: redistribute or shorten.
- Footer total text width > container → FAIL. Auto-fix: shorten labels or convert to flex.
- Content < 5% from slide edge → WARNING.

**Hierarchy:**
- Heading/body ratio < 2:1 → WARNING.
- Largest element < 1.5x second-largest → WARNING "no clear focal point".

**Contrast:**
- Body text contrast < 4.5:1 → FAIL.
- Large text contrast < 3:1 → FAIL.

Severity: CRITICAL → auto-fix + re-check. FAIL → auto-fix + re-check. WARNING → log only.

---

## Change 3: Adaptive layout in `generation-pipeline.md`

**Modify** `references/generation-pipeline.md` — add Step 5b after slot filling, before finalization.

### Step 5b: Adaptive Layout

After filling {{SLOT}} markers, verify content fits. Content in Russian/other languages is often longer than English placeholders.

**Footer adaptation:**
- Replace fixed positioning with flex: `display:flex; gap:24px; align-items:center`
- Add `white-space:nowrap; overflow:hidden; text-overflow:ellipsis` on spans
- Last element: `margin-left:auto`
- If still overflows: shorten labels

**Two-column adaptation:**
- Check text height vs available space (slide minus padding minus footer)
- If overflows: shorten/rephrase first, then reduce gap (min 24px)
- Never let text overlap with footer

**Card/grid adaptation:**
- Verify text doesn't overflow card boundary
- If overflows: shorten text
- Maintain min 8px padding inside cards

**General spacing:**
- Min gap between adjacent elements: 8px (0.5rem)
- No element beyond parent container
- Footer zone: bottom 44-56px reserved — no content may enter
- Convert position:absolute to flex where possible for natural flow

These adaptations are the ONLY permitted deviations from verbatim copy rule.

Add reference link at top: "Read `references/design-rules.md` for universal spacing, density, and hierarchy rules."

---

## Change 4: Background-aware dedup + context selection in `figma-extraction.md`

### 4a: Expand deduplication (Step 4-pre)

Add 6th criterion to the existing 5:

**6. Same background treatment:**
- Compare background fills (solid, gradient, image)
- Both solid: same color (deltaE < 10) → same
- Both gradient: same type AND similar color stops → same
- One solid + one gradient → DIFFERENT
- Dark bg (luminance < 30%) vs light (> 70%) → ALWAYS different (require different text colors)

Example: 4 section-dividers with identical structure but different gradients → 4 separate archetypes:
- `figmadeck-section-teal-gold`
- `figmadeck-section-teal-green`
- `figmadeck-section-blue-purple`
- `figmadeck-section-magenta-gold`

### 4b: Context-based variant selection (Step 4)

When multiple archetypes share the same contentType, select by context:

1. Analyze slide content: topic, mood, emotional valence
2. Analyze variant backgrounds: color temperature, energy
3. Match rules:
   - Warm (gold, amber, magenta) → positive outcomes, achievements, CTA
   - Cool (teal, blue, green) → analysis, data, process, sustainability
   - Neutral/dark → transitions, pauses, serious topics
4. No clear match → distribute evenly (no two adjacent slides use same variant)
5. Cover → first variant (matches Figma opening)
6. CTA/closing → last variant (matches Figma ending)

Composition table extended: `[slide_number, content_type, archetype_id, variant_id, figma_nodeId]`
