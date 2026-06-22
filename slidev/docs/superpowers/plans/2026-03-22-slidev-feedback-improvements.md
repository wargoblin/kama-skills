# Slidev Skill: Feedback-Driven Improvements — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Improve `/slidev` skill to prevent 13 design problems found by human reviewers and incorporate 9 research-backed quality rules.

**Architecture:** All changes are to markdown instruction files (no executable code). The skill is at `slidev/.claude/skills/slidev/`. Changes flow through: design rules (`references/design-principles.md`) -> generation pipeline (`SKILL.md` Steps 4/4.5/5) -> QA checks (`SKILL.md` QA-0b/0c/4) -> scoring (`references/scoring-subroutine.md`).

**Task dependencies:** Tasks 1-3 (design-principles.md) MUST complete before Tasks 4-6 (SKILL.md) because QA checks reference the rules defined in design-principles.md. Within Tasks 1-3, Task 1 MUST complete before Task 2 (Task 2 Step 3 inserts content after Task 1 Step 3's addition). Tasks 7-9 can run after Tasks 1-3 but are independent of Tasks 4-6.

**Tech Stack:** Markdown instruction files, CSS patterns, Slidev presentation framework.

**Spec:** `slidev/docs/superpowers/specs/2026-03-21-slidev-feedback-driven-improvements-design.md`

---

## File Map

| File | Path | Lines | What Changes |
|------|------|-------|-------------|
| design-principles.md | `slidev/.claude/skills/slidev/references/design-principles.md` | ~718 | Principles 1,2,3,4,6,8,10,12 + Anti-Patterns QR |
| SKILL.md | `slidev/.claude/skills/slidev/SKILL.md` | ~1500+ | Steps 4/4.5/5, QA-0b/0c/4, Font Selection |
| composition-archetypes.md | `slidev/.claude/skills/slidev/references/composition-archetypes.md` | ~351 | bg-level tokens, pill line-height, gradient comments, focal-point comments |
| layout-css-patterns.md | `slidev/.claude/skills/slidev/references/layout-css-patterns.md` | ~180 | Pill CSS template |
| scoring-subroutine.md | `slidev/.claude/skills/slidev/references/scoring-subroutine.md` | ~55 | Axes 1,3,4,7,8,9 Low/High text |
| content-review-subroutine.md | `slidev/.claude/skills/slidev/references/content-review-subroutine.md` | ~88 | 3 new aspects: burstiness, cost-of-inaction, redundancy |
| preset-format.md | `slidev/.claude/skills/slidev/references/preset-format.md` | ~192 | Trend vocabulary section |

---

## Task 1: Background System in design-principles.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:5-55` (Principle 1)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:641-677` (Principle 12)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:698-718` (Anti-Patterns QR)

- [ ] **Step 1: Read Principle 1 current content**

Read lines 5-55 of `design-principles.md`. Identify the existing per-slide-type luminance-delta recipe table that will be replaced.

- [ ] **Step 2: Replace Principle 1 background table with bg-system**

In Principle 1, replace the luminance-delta recipe table with the new 3-level background system. Add:

```markdown
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
```

Keep the rest of Principle 1 (breathing slides, pacing rules) unchanged.

- [ ] **Step 3: Add pure #000/#FFF ban to Principle 12**

After the AI Color Blacklist section in Principle 12 (~line 670), add:

```markdown
### Pure Black/White Ban

Never use pure `#000000` for text or pure `#FFFFFF` for backgrounds. Replace with tinted alternatives:
- Black text -> `#1A2332` (dark navy) or `#1C1917` (warm charcoal)
- White background -> `#FAF9F6` (cream), `#F5F5F3` (warm gray), or `#EFF5F4` (cool mint)
```

- [ ] **Step 4: Add anti-pattern #15 to Anti-Patterns Quick Reference**

After item 14 in Anti-Patterns QR (~line 714), add:

```markdown
15. Pure `#000000` or `#FFFFFF` used as background or text color
16. Inconsistent background temperature (mixing warm and cool backgrounds)
17. More than 3 distinct background colors in the deck (only bg-base/alt/accent allowed)
```

- [ ] **Step 5: Verify and commit**

Read the modified sections to verify consistency. Commit:

```bash
git add slidev/.claude/skills/slidev/references/design-principles.md
git commit -m "feat(slidev): add 3-level background system to design principles"
```

---

## Task 2: Typography, Pills, Contrast rules in design-principles.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:91-123` (Principle 3)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:315-397` (Principle 6)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:641-677` (Principle 12)

- [ ] **Step 1: Add type-treatment limit and italic restriction to Principle 3**

After the existing Principle 3 content (~line 122), add:

```markdown
### Type Treatment Limit

Maximum **3 visually distinct type treatments** per slide:
1. Heading font (regular or bold) — titles, labels
2. Body font (regular) — descriptions, bullets
3. One accent treatment — bold OR italic OR uppercase, but **never all three simultaneously**

FAIL if a single slide contains bold + italic + uppercase + multiple font families all at once.

### Italic Restriction

Italic is permitted **only** in:
- `<blockquote>` elements and direct quotations
- Attribution lines ("— Author Name")
- Image captions / footnotes

Italic in headings, subheadings, or body text = **FAIL**.
```

- [ ] **Step 2: Add pill rendering and exclusion zones to Principle 6**

After the existing Principle 6 content (~line 396), add:

```markdown
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

Pill background must be **opaque** (solid `var(--bg-base)` or `var(--bg-alt)`), not semi-transparent `rgba(...)`, to prevent "mud" effect when overlapping decorative gradient spots.

**Alternative:** If a pill must be semi-transparent by design, then radial-gradient decorative spots must **not** be placed in the top 20% of the slide (where pills typically appear).

### Decorative Gradient Exclusion Zones

Decorative gradient spots (radial-gradient blobs, glows):
- Must **not** overlap with text-heavy zones (headings, pills, body text)
- Place in corners, edges, or outside the content area
- If overlap with content is unavoidable: opacity ≤ 0.20
```

- [ ] **Step 3: Add "weakest segment first" to Principle 12**

After the Pure Black/White Ban (added in Task 1), add:

```markdown
### Weakest Segment First (Muted Text Contrast)

When defining `--color-muted` (secondary/gray text), test its contrast ratio against **every** surface it will appear on:
- `--bg-base`
- `--bg-alt`
- `--color-accent-bg` (accent card backgrounds)
- `--color-surface` (card backgrounds)

The **lowest contrast ratio** must pass WCAG AA (≥ 4.5:1). If it fails on any surface (typically accent cards), darken `--color-muted` **globally** until the weakest segment passes. Do not vary muted color per surface — one muted color everywhere.
```

- [ ] **Step 4: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/references/design-principles.md
git commit -m "feat(slidev): add typography discipline, pill rendering, weakest-segment contrast rules"
```

---

## Task 3: Anti-AI Patterns + Research expansions in design-principles.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:57-90` (Principle 2)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:124-273` (Principle 4)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:468-542` (Principle 8)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:574-609` (Principle 10)
- Modify: `slidev/.claude/skills/slidev/references/design-principles.md:698+` (Anti-Patterns QR)

- [ ] **Step 1: Add Rule of Thirds to Principle 2**

After the existing layout rotation rules in Principle 2 (~line 89), add:

```markdown
### Rule of Thirds

On non-breathing, non-hero slides, place the dominant element at a thirds-grid intersection rather than dead center. Center alignment is reserved for breathing slides (stat-hero, quote-pull) and cover/CTA slides. At least 30% of content slides should use off-center focal points.
```

- [ ] **Step 2: Add icon container variation to Principle 4**

After the Icon.vue component template (~line 273), add:

```markdown
### Icon Container Variation

When 3+ icons appear on one slide, their containers must **differ** on at least one dimension:
- Size (e.g., 48px, 56px, 64px), OR
- Shape (circle, rounded-square, no container), OR
- Style (solid fill, ghost outline, accent bg), OR
- Placement (not strictly equal-gap row)

All containers with identical width + height + border-radius + border = **FAIL**.

**Exception:** numbered steps (1, 2, 3) may use uniform containers — uniformity is semantically justified for sequential items.
```

- [ ] **Step 3: Add chart type matrix and colorblind hue ranges to Principle 8**

After the existing chart-type rules (~line 540), add:

```markdown
### Chart Type Selection Matrix

| Goal | Correct type | Avoid |
|------|-------------|-------|
| Category comparison | Horizontal bar | 3D, exploded pie |
| Change over time | Line chart | Bar with many periods |
| Parts of whole | Stacked bar / Donut (≤5 segments) | Pie >6 segments |
| Correlation | Scatter + trend line | Tables |
| Simple proportion | Donut (3-5 segments) | 3D pie |

### Colorblind Safety — Hue Ranges

Expand the existing colorblind rule with concrete thresholds:
- Red (hue 0-30) + Green (hue 90-165) on same slide = **FAIL**
- Brown (hue 20-40) + Green (hue 90-165) on same slide = **WARNING**
- Always substitute: use **blue/orange** pair instead of red/green
```

- [ ] **Step 4: Strengthen text arrow ban in Principle 10**

In Principle 10 (~line 575), find the existing mention of text arrows and expand:

```markdown
### Text Arrow Character Ban

The following characters are **banned** in slides.md content (outside `<code>`, backticks, and `<!-- -->` speaker notes):

`→` `←` `↑` `↓` `⟶` `➜` `▶`

**Replacements:**
- Sequences ("A → B → C"): rephrase textually ("from A to B, then to C") or use SVG `<path>` arrows
- Pipelines: use `timeline-horizontal` / `timeline-zigzag` archetype with numbered steps
- Navigation cues: use `Icon.vue` with `arrow-right` icon inside layout

Any match in visible content = **FAIL** in QA-0c.
```

- [ ] **Step 5: Add items 15-20 to Anti-Patterns Quick Reference**

After item 14 (and items 15-17 added in Task 1), add:

```markdown
18. Text arrow characters (→ ← ↑ ↓) in visible content
19. 3+ identical icon containers (same size, shape, border) on one slide
20. Multiple decorative gradient types (radial + linear) in same presentation
```

- [ ] **Step 6: Add Gestalt and cognitive overload to design-principles.md**

At the end of the file, before the Anti-Patterns QR section, add:

```markdown
## Gestalt Compliance Notes

**Proximity:** Labels and captions must be visually adjacent to their referent elements. A label more than ~1/4 slide height away from its element = violation.

**Similarity:** Same-level elements in a grid (e.g., all cards in a card-mosaic) must share the same padding, border-radius, and font-size. Inconsistent styling within a group = violation.

**Speaker Note Redundancy (Mayer Principle):** Speaker notes must EXTEND slide content, not repeat it verbatim. If slide text and speaker note convey the same information = redundancy violation.
```

- [ ] **Step 7: Update Quick Reference Checklist**

At line 679, add these items to the existing 12-item checklist:

```markdown
- [ ] **Principle 1+**: 3-level bg-system (bg-base/alt/accent), same temperature, no pure #000/#FFF
- [ ] **Principle 3+**: ≤3 type treatments per slide; italic only in quotes/attribution
- [ ] **Principle 4+**: Icon containers vary (size, shape, or style) when 3+ on one slide
- [ ] **Principle 6+**: Pills have opaque bg + line-height:1; decorative gradients in exclusion zones only
- [ ] **Principle 8+**: Chart type matches data goal (see matrix); colorblind-safe hue pairs
- [ ] **Principle 10+**: No text arrow characters (→←↑↓) in visible content
- [ ] **Principle 12+**: Muted text passes WCAG 4.5:1 on ALL surfaces (weakest segment first)
```

- [ ] **Step 8: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/references/design-principles.md
git commit -m "feat(slidev): add anti-AI reinforcement and research-driven design rules"
```

---

## Task 4: Generation Pipeline in SKILL.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:1104-1115` (Font Selection + Number Blacklist)
- Modify: `slidev/.claude/skills/slidev/SKILL.md:1242-1253` (Step 4)
- Modify: `slidev/.claude/skills/slidev/SKILL.md:1254-1305` (Step 4.5)
- Modify: `slidev/.claude/skills/slidev/SKILL.md:1306+` (Step 5)

- [ ] **Step 1: Update Font Number Blacklist**

At line ~1113, add `Bricolage Grotesque` to the Font Number Blacklist:

```markdown
**Font Number Blacklist** — For number-heavy presentations (3+ slides with prominent numbers), these heading fonts are forbidden: `Syne, Playfair Display, Bodoni Moda, Bricolage Grotesque`.

Bricolage Grotesque: bold digits are visually indistinguishable from regular weight at display sizes. Confirmed via human feedback.

For any font not on the blacklist used in number-heavy presentations: verify the font's weight range — if bold-regular weight delta < 300 = WARNING.
```

- [ ] **Step 2: Update Step 4 (styles/index.css generation)**

At line ~1242, expand the Step 4 instructions to include:

```markdown
#### Background Level System (REQUIRED)

Generate these 3 CSS variables:
```css
--bg-base: <hex>;    /* 60% of slides — primary content background */
--bg-alt: <hex>;     /* 30% of slides — section dividers, alternating content */
--bg-accent: <hex>;  /* 10% of slides — cover, CTA */
```

Rules:
- All 3 must share the same color temperature (all warm or all cool)
- Luminance delta between --bg-base and --bg-accent: max 40%
- Never pure #000 or #FFF
- Add a CSS comment with contrast ratios: `--color-muted` vs each bg-level, `--color-text` vs each bg-level

#### Decorative Gradient Type Choice

Choose exactly ONE decorative gradient type for the presentation and note it in a CSS comment:
```css
/* Decorative gradient type: radial-gradient */
```
This is a behavioral constraint — enforce during Step 5 when writing slides.

#### Muted Text — Weakest Segment Check

After defining `--color-muted`, verify its contrast ratio against ALL bg-levels and `--color-accent-bg`. The **lowest ratio** must be ≥ 4.5:1. If not, darken `--color-muted` until it passes. Document ratios in a CSS comment.
```

- [ ] **Step 3: Update Step 4.5 (Composition Planning)**

At line ~1254, add to the archetype-to-slide mapping instructions:

```markdown
#### Background Level Assignment

After selecting archetypes, assign each slide a background level:

| Archetype | Default bg-level |
|-----------|-----------------|
| cover-hero | `--bg-accent` |
| section-divider | `--bg-alt` |
| stat-hero, quote-pull | `--bg-base` |
| icon-trio, bento-grid, two-col-text, card-mosaic, comparison-table | `--bg-base` or `--bg-alt` (alternate) |
| timeline-horizontal, timeline-zigzag | `--bg-base` |
| cta-warm | `--bg-accent` |
| Pre-CTA slide (N-1 before cta-warm) | `--bg-alt` (if bg-accent is dark) |

Record the assignment in the composition table alongside archetype selection. No inline background colors — only `var(--bg-*)` tokens.
```

- [ ] **Step 4: Update Step 5 (slide writing)**

At line ~1306, add to the slide-writing instructions:

```markdown
#### Gradient Type Enforcement

Use ONLY the decorative gradient type chosen in Step 4. If "radial-gradient" was chosen:
- Decorative blobs/glows: `radial-gradient` (allowed)
- Card accent backgrounds: solid colors only (no `linear-gradient` decoration)
- bg-accent slide: may use `linear-gradient` (structural, exempt)

If "linear-gradient" was chosen:
- Decorative washes/overlays: `linear-gradient` (allowed)
- No radial-gradient blobs anywhere

#### Text Arrow Enforcement

Never write `→`, `←`, `↑`, `↓`, `⟶`, `➜`, `▶` in visible slide content. Use textual rephrasing, SVG arrows, or timeline archetypes instead.

#### Italic Enforcement

Never use `font-style: italic` except in `<blockquote>` elements, attribution lines, and image captions.
```

- [ ] **Step 5: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): update generation pipeline with bg-system, gradient enforcement, font blacklist"
```

---

## Task 5: QA Pipeline — QA-0b and QA-0c in SKILL.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:811-831` (QA-0b)
- Modify: `slidev/.claude/skills/slidev/SKILL.md:832-870` (QA-0c)

- [ ] **Step 1: Add new QA-0b checks**

**Before** the closing instruction sentence at ~line 830 ("Fix any issues found before proceeding to visual review"), add these code-grep checks:

```markdown
#### New QA-0b Checks (code grep)

- [ ] **Pill centering:** Grep `border-radius.*20px` (or pill/badge/chip patterns). Verify each has `line-height: 1`, `display:.*flex`, `align-items: center`. Missing = WARNING
- [ ] **Pill-on-gradient mud:** Grep `rgba(` in pill/badge background context + `radial-gradient` on same slide. Both present = WARNING "potential mud overlap — use opaque pill background"
- [ ] **Icon container uniformity:** Find all `border-radius:50%` with icon containers on same slide. If 3+ with identical width+height+border = WARNING "uniform icon containers"
- [ ] **Grid element similarity:** Within each grid container, grep `border-radius` values. If different values among same-level children = WARNING "Gestalt similarity violation"
- [ ] **Rule of Thirds heuristic:** On non-breathing, non-cover, non-CTA slides: if heading has `text-align:center` AND no `grid-template-columns` with unequal fractions AND no asymmetric width splits = WARNING "dead center layout — consider Rule of Thirds"
- [ ] **Pie chart segment count:** If slides.md contains pie/donut chart markup with >5 segments/items = FAIL "pie chart exceeds 5 segments — use bar chart instead"
- [ ] **Caption proximity heuristic:** Within grid/flex containers, check that caption/label elements are in the same or immediately adjacent cell as their referent element. If separated by >1 cell = WARNING "Gestalt proximity violation"
```

- [ ] **Step 2: Add new QA-0c checks**

In QA-0c (lines 832-870), add these new items to the existing sections. Insert items in **file order** (Typography comes before Color in the file):

After the last item in **Typography:** section (~line 847, after "No centered multi-line body text"), add:

```markdown
- [ ] Type treatments per slide ≤3 distinct combinations of font-family+weight+style (CRITICAL)
- [ ] No `font-style: italic` outside `<blockquote>`, attribution, or caption context (CRITICAL)
- [ ] Heading font not in Font Number Blacklist for number-heavy decks (CRITICAL)
```

After the last item in **Color:** section (~line 852, after "No colorblind-unsafe data pairs"), expand with:

```markdown
- [ ] No pure `#000000` or `#FFFFFF` in any background or text color (CRITICAL)
- [ ] All backgrounds use only `var(--bg-base)`, `var(--bg-alt)`, or `var(--bg-accent)` — no hardcoded background colors (CRITICAL)
- [ ] Background temperature consistency: all 3 bg-levels share same warm/cool temperature (CRITICAL)
- [ ] Muted text contrast: `--color-muted` vs each bg-level and `--color-accent-bg` — all pairs ≥ 4.5:1 (CRITICAL)
- [ ] Colorblind safety: red (hue 0-30) + green (hue 90-165) on same slide = FAIL. Brown (hue 20-40) + green (hue 90-165) = WARNING. (Augments existing colorblind check, do not create separate entry)
```

After the last item in **AI tells:** section (~line 859), add:

```markdown
- [ ] No text arrow characters (→←↑↓⟶➜▶) in visible content outside `<code>` and `<!-- -->` (CRITICAL)
- [ ] No 3+ identical icon containers (same width+height+border-radius+border) on one slide — exception: numbered steps (CRITICAL)
- [ ] Only 1 decorative gradient type used — classify each gradient as decorative vs functional/structural. Both radial and linear as decorative = WARNING
```

- [ ] **Step 3: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add new QA-0b and QA-0c checks for feedback-driven rules"
```

---

## Task 6: QA-4 Visual Checks in SKILL.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:880-931` (QA-4)

- [ ] **Step 1: Add new QA-4 visual check items**

In the QA-4 per-slide visual review checklist (~line 930), add:

```markdown
#### Feedback-Driven Visual Checks (per slide)

- [ ] **Pill centering:** Text inside pill/badge elements is vertically centered (not shifted up)
- [ ] **Pill clarity:** Pill boundaries are crisp — no "mud" effect from gradient bleed-through
- [ ] **Muted text readability:** Palest text on this slide is readable against its background (especially on accent-colored surfaces)
- [ ] **Gestalt proximity:** Labels/captions are adjacent to their referent elements (not >1/4 slide height away)

#### Feedback-Driven Visual Checks (deck-level, after all slides reviewed)

- [ ] **Rule of Thirds:** At least 30% of content slides use off-center focal points (not all dead-center)
- [ ] **Background consistency:** No unexpected color temperature shifts between adjacent slides
- [ ] **Gradient consistency:** Decorative gradients all use the same type (all radial OR all linear)
- [ ] **Icon container variation:** No slide has 3+ identical icon circles in a row (unless numbered steps)
```

- [ ] **Step 2: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add feedback-driven visual checks to QA-4"
```

---

## Task 7: composition-archetypes.md + layout-css-patterns.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/composition-archetypes.md` (all archetypes)
- Modify: `slidev/.claude/skills/slidev/references/layout-css-patterns.md:151+`

- [ ] **Step 1: Add bg-level token comments to each archetype**

For each archetype in composition-archetypes.md, add a `bg-level:` line to the metadata block. Examples:

- `cover-hero` (line 24): add `bg-level: --bg-accent`
- `section-divider` (line 45): add `bg-level: --bg-alt`
- `stat-hero` (line 61): add `bg-level: --bg-base`
- `quote-pull` (line 80): add `bg-level: --bg-base`
- `cta-warm` (line 95): add `bg-level: --bg-accent`
- `icon-trio` (line 117): add `bg-level: --bg-base | --bg-alt`
- All remaining content archetypes: add `bg-level: --bg-base | --bg-alt`

Replace any hardcoded background colors in archetype HTML skeletons with `var(--bg-base)`, `var(--bg-alt)`, or `var(--bg-accent)`.

- [ ] **Step 2: Add line-height:1 to all pill elements in archetypes**

Search all archetypes for pill/badge/chip patterns (rounded corners with uppercase text). Add `line-height:1;` to each pill's inline style.

- [ ] **Step 3: Add focal-point comments to content archetypes**

For each non-breathing archetype, add a comment indicating the recommended focal point per Rule of Thirds:

```markdown
<!-- Focal point: left-third intersection (heading + primary content) -->
```

- [ ] **Step 4: Add decorative gradient type annotations**

For each archetype that uses gradients, add a comment classifying each gradient:

```markdown
<!-- Gradient: DECORATIVE (use chosen type) -->
<!-- Gradient: STRUCTURAL (bg-accent exempt) -->
```

- [ ] **Step 5: Add pill CSS template to layout-css-patterns.md**

At the end of layout-css-patterns.md, first add a `---` separator after the last line of existing content (~line 179), then add a new section:

```markdown
## Pill / Badge / Chip Pattern

All pill-style elements (eyebrow labels, tags, category badges) must use:

```css
.pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  padding: 6px 18px;
  border-radius: 20px;
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.16em;
  font-weight: 700;
  /* Use opaque background — never rgba() over decorative gradients */
  background: var(--bg-base);
  border: 1.5px solid var(--color-accent-dim);
  color: var(--color-accent);
}
```

**Critical:** `line-height: 1` prevents vertical text shift. Opaque background prevents "mud" bleed-through from decorative gradient spots beneath.
```

- [ ] **Step 6: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/references/composition-archetypes.md slidev/.claude/skills/slidev/references/layout-css-patterns.md
git commit -m "feat(slidev): update archetypes with bg-tokens, pill fixes, focal-point comments"
```

---

## Task 8: Scoring Subroutine

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/scoring-subroutine.md:13-23`

- [ ] **Step 1: Read current 9-axis table**

Read lines 13-23 of scoring-subroutine.md to see current Low/High descriptions.

- [ ] **Step 2: Update axis descriptions**

Modify the 9-axis table to add feedback-driven criteria:

| Axis | Add to Low (1-3) | Add to High (7-10) |
|------|-------------------|---------------------|
| 1 (Composition variety) | "3+ uniform icon containers on one slide" | "Icon containers vary in size, shape, or style" |
| 3 (Font discipline) | ">3 type treatments on one slide; italic outside quotes; blacklisted font for number-heavy deck" | "Max 3 treatments/slide; italic only in quotes; font digit rendering verified" |
| 4 (Visual impact) | *(no change)* | "Rule of Thirds applied on 30%+ of content slides" |
| 7 (Color conviction) | "Pure #000/#FFF; inconsistent bg temperature; bg-system not used; mixed decorative gradient types; muted text fails WCAG on any surface" | "3-level bg-system; single color temperature; all text-on-surface pairs pass WCAG 4.5:1; single decorative gradient type" |
| 8 (Content clarity) | "Monotonous bullet length (all within +-3 words)" | "Varied sentence structure; cost-of-inaction slide present (pitch decks)" |
| 9 (Decorative quality) | "Pill/badge mud overlap with gradient; gradient exclusion zone violated" | "Pills crisp on all backgrounds; decorative elements in corners/edges only" |

Append these criteria to the existing Low/High text for each axis — do not replace the existing text.

- [ ] **Step 3: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/references/scoring-subroutine.md
git commit -m "feat(slidev): update scoring axes with feedback-driven criteria"
```

---

## Task 9: Content Review + Preset Format

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/content-review-subroutine.md:50+`
- Modify: `slidev/.claude/skills/slidev/references/preset-format.md:129+`

- [ ] **Step 1: Add 3 new aspects to content-review-subroutine.md**

After aspect 5 (Information Hierarchy, ~line 57), add:

```markdown
### 6. Text Burstiness

If a slide has 3 or more bullets AND the word count of every bullet is within +-3 words of the arithmetic mean of all bullet word counts on that slide = WARNING "monotonous text length, vary sentence structure."

Human writing varies sentence length naturally. Uniform-length bullets signal AI generation.

### 7. Cost-of-Inaction Check

For pitch/business presentations only: check for at least one slide that answers "what happens if we don't act?" or "what is the cost of inaction?"

If absent = SUGGESTION (not FAIL — not all presentations are business pitches). This is the most persuasive element in business presentations and is almost never generated by AI.

### 8. Speaker Note Redundancy (Mayer Principle)

Speaker notes must EXTEND slide content, not repeat it. If a speaker note contains substantially the same words/message as the visible slide text = redundancy violation.

The audience reads the slide faster than the speaker talks. Identical text in both channels causes cognitive interference (Mayer Redundancy Principle) — worse retention than either channel alone.
```

- [ ] **Step 2: Update Output section**

In the Output section (~line 59), add the 3 new aspect names (Burstiness, Cost-of-Inaction, Speaker Note Redundancy) to the severity-based issue list format so reviewers know to check and report on them. If the Output section contains a numbered aspect list, extend it to include aspects 6-8.

- [ ] **Step 3: Add Trend Vocabulary to preset-format.md**

At the **end of the file** (~line 191, after all existing content), add as a new top-level section:

```markdown
## Trend Vocabulary (for --create-preset)

When creating presets via `--create-preset`, these named aesthetic trends can be referenced in the preset body to guide design decisions. This vocabulary is **only for preset creation** — it does not affect default generation.

| Trend | Description |
|-------|-------------|
| `bold-minimalism` | White text on dark backgrounds, max 1-2 elements per slide, large typography as primary design element |
| `editorial` | Wide margins, asymmetric grids, magazine-like layout, strong photo hierarchy, pull quotes in contrasting type |
| `human-centered` | Organic shapes, natural color palettes (terracotta, sage, cream), tactile imagery, intentional imperfection |
| `broken-grid` | Overlapping text and images, angular sections, diagonal dividers, dynamic and editorial feel |

Reference in preset body as: "This preset follows the `bold-minimalism` trend..."
```

- [ ] **Step 4: Verify and commit**

```bash
git add slidev/.claude/skills/slidev/references/content-review-subroutine.md slidev/.claude/skills/slidev/references/preset-format.md
git commit -m "feat(slidev): add burstiness, cost-of-inaction, redundancy checks and trend vocabulary"
```

---

## Summary

| Task | Files | Sections from Spec |
|------|-------|-------------------|
| 1 | design-principles.md | Section 1 (bg-system) |
| 2 | design-principles.md | Sections 2, 3, 4 (typography, pills, contrast) |
| 3 | design-principles.md | Sections 5, 6 (anti-AI, research) |
| 4 | SKILL.md | Generation pipeline (Steps 4, 4.5, 5, Font Selection) |
| 5 | SKILL.md | QA-0b, QA-0c |
| 6 | SKILL.md | QA-4 |
| 7 | composition-archetypes.md, layout-css-patterns.md | Sections 1, 3, 5c, 6a |
| 8 | scoring-subroutine.md | All sections |
| 9 | content-review-subroutine.md, preset-format.md | Sections 6e, 6f, 6g, 6h |
