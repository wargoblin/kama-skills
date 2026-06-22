# Slidev Skill: QA & Generation Improvements Design Spec

**Date:** 2026-03-23
**Status:** Draft
**Scope:** 10 improvements across 6 areas based on cross-analysis of 5 generated presentations against skill rules and research documents.

## Background

Cross-analysis of 5 presentations (innosmart-tech, investor-q1-boardroom, investor-q1-report, preset-deep-learn-boardroom, preset-deep-learn-kids-party-kp) against SKILL.md rules, research documents (ai-presentation-detection-guide, beautiful-presentations-guide, ugly-presentations-anti-patterns), and superpowers change history revealed 10 systematic issues where documented rules are not enforced during generation or QA.

## Improvement Summary

| # | Area | Severity | Description |
|---|------|----------|-------------|
| 1 | Serif ban | New requirement | Ban all serif fonts, switch to sans+sans pairing |
| 2 | Theme generation | New feature | Generate Slidev themes during learning cycles |
| 3 | bg-system distribution | Critical | Fix checkerboard 50/50 → proper 60/30/10 |
| 4 | Icon container variation | Critical | Break uniform circles → 2+ shapes per deck |
| 5a | QA: layout:none | Critical | Enforce layout:none + style block on all slides |
| 5b | QA: #FFF/#000 scan | Major | Auto-replace pure white/black with warm off-values |
| 5c | QA: pill line-height | Major | Enforce line-height:1 on all pills/badges |
| 5d | QA: CSS vars | Major | Enforce var() over hardcoded hex colors |
| 5e | QA: body font size | Major | Enforce ≥1.25rem on body text |
| 5f | QA: bg distribution | Major | Warn if bg-base < 50% or bg-alt > 40% |
| 6a | Composition variety | Major | Layout budget: max 40% one group, split cap 30% |
| 6b | Cost-of-inaction | Major | Auto-insert for pitch/investor/sales decks |

## Files Modified

| File | Sections Changed |
|------|-----------------|
| SKILL.md | Font Selection, Steps 3/4.5/5, QA-0a/0b, learn/deep_learn/create-preset, Mode Unique/Preset |
| design-principles.md | Principles 3, 4 (Principle 12 unchanged — pure #FFF/#000 ban and AI color blacklist remain as-is; enforcement moves to QA §5.2) |
| preset-format.md | fonts.body rename, icon_container array, theme directory spec |
| scoring-subroutine.md | Axes 1, 7 |
| content-review-subroutine.md | Cost-of-inaction severity |
| composition-archetypes.md | Remove per-archetype bg-level comments (superseded by §3 algorithm) |
| layout-css-patterns.md | Icon container pattern additions |
| Existing presets | Replace serif → sans-serif body fonts |

---

## Section 1: Serif Font Ban

### Problem

The skill currently pairs sans-serif headings with serif body text (e.g., Outfit + Source Serif 4, Plus Jakarta Sans + Newsreader). User requirement: no serif fonts at all. Research supports this — 4 of 6 proven font pairings in beautiful-presentations-guide.md are sans+sans or single-font.

### Design

#### 1.1 SKILL.md — Font Selection (Mode: Unique)

Replace the current `sans (heading) + serif (body)` pairing strategy with `sans-display (heading) + sans-text (body)`:

- **sans-display**: Geometric or condensed sans-serif fonts optimized for large sizes (headings, hero numbers). Examples: Outfit, Manrope, Plus Jakarta Sans, Barlow, Sora, Urbanist.
- **sans-text**: Humanist or rounded sans-serif fonts optimized for readability at body sizes. Examples: DM Sans, Nunito, Lato, Source Sans Pro, Noto Sans, IBM Plex Sans.

Contrast is achieved through **character difference** (geometric vs humanist, condensed vs proportional, angular vs rounded), not through serif/sans category split.

#### 1.2 Serif Blacklist

Add to SKILL.md Font Selection section:

> **CRITICAL: Never use a serif font.** All fonts must be sans-serif. Blacklisted font families include (non-exhaustive): Source Serif 4, Newsreader, Merriweather, Playfair Display, DM Serif Display, Garamond, Georgia, Lora, Noto Serif, Crimson Text, EB Garamond, Libre Baskerville, Cormorant, Spectral, Bitter, Zilla Slab, Roboto Slab, Rockwell.
>
> If a preset specifies a serif font, replace it with the nearest sans-serif equivalent before generation.

Also add Space Grotesk to the NEVER-use font list in SKILL.md Font Selection (Mode: Unique), alongside Inter/Roboto/Arial. Space Grotesk is identified in research as an "elevated-but-safe" AI alternative that signals AI generation.

#### 1.3 Recommended Sans+Sans Pairings

Add to SKILL.md as reference table:

| Heading (display) | Body (text) | Contrast type |
|-------------------|-------------|---------------|
| Outfit | DM Sans | Geometric vs Humanist |
| Manrope | Source Sans Pro | Semi-rounded vs Neutral |
| Plus Jakarta Sans | Nunito | Modern geometric vs Rounded friendly |
| Barlow | Lato | Slightly condensed vs Classic humanist |
| Sora | IBM Plex Sans | Technical vs Corporate humanist |
| Urbanist | Noto Sans | Modern geometric vs Universal neutral |

#### 1.4 preset-format.md Changes

- Rename field `fonts.serif` → `fonts.body`
- Backward compatibility: if parser encounters `fonts.serif`, treat as `fonts.body`
- Add validation: if the font value matches the Serif Blacklist → error, suggest nearest sans-serif replacement

#### 1.5 design-principles.md — Principle 3 (Typographic Drama)

Update text:
- Remove all references to "serif body text"
- Replace pairing guidance: "Contrast through character (geometric vs humanist, condensed vs proportional, angular vs rounded), not through serif/sans category split."
- Update the type scale descriptions to reference sans-text instead of serif

#### 1.6 --create-preset Wizard (SKILL.md, `--create-preset` section, step 2, typography question)

Update typography question:
- Old: "Font personality? (geometric & modern, classic & refined, rounded & friendly, sharp & technical)"
- New: "Font personality? (geometric & modern, rounded & friendly, sharp & technical, humanist & classic, condensed & bold)"
- Remove "classic & refined" which implied serif. Replace with "humanist & classic" which is sans-serif.

#### 1.7 Existing Preset Updates

| Preset | Current body font | Replacement |
|--------|------------------|-------------|
| boardroom | Source Serif 4 | DM Sans |
| Any preset with Newsreader | Newsreader | Nunito or Lato |
| Any preset with serif body | (varies) | Nearest sans-text match |

#### 1.8 Exhaustive SKILL.md Locations to Update

All locations in SKILL.md that reference serif fonts or the old pairing model:

| Location | Current content | Change |
|----------|----------------|--------|
| Font Selection — NEVER-use list (~line 1141) | Lists generic fonts to avoid | Add Space Grotesk to the list |
| Font Selection — Good pairs (~line 1144) | `Outfit + Source Serif 4, Syne + Literata, Cabinet Grotesk + Newsreader, Plus Jakarta Sans + Fraunces, Instrument Serif + DM Sans, Bricolage Grotesque + Lora, Familjen Grotesk + Crimson Pro` | Replace entire list with sans+sans pairs from §1.3 |
| Strict 2-Font Rule — body font role (~line 1154) | "Body font (serif) for: descriptions, bullets, supporting text" | Change to "Body font (sans-text) for: descriptions, bullets, supporting text" |
| Step 1 — font resolution (~line 1235) | `fonts: { sans, serif, mono }` | Change to `fonts: { sans, body, mono }` — align with preset-format.md rename |
| Mode: Preset — applying preset (~line 1095) | References `fonts.serif` from preset | Change to `fonts.body` |

---

## Section 2: Slidev Theme Generation During Learning

### Problem

All presets use `theme: default`. Visual identity is implemented entirely through inline styles + styles/index.css, which leads to inconsistency: some presentations use CSS variables, others hardcode values. The generator "reinvents" styles each time, producing unstable results.

### Design

#### 2.1 Theme Directory Structure

During `--deep_learn`, `--learn`, or `--create-preset`, generate a local Slidev theme directory:

```
theme/
  package.json
  styles/
    index.css           # CSS variables, base layout, shape vocabulary
    layouts.css         # Layout-specific styles (cover, section, etc.)
  layouts/
    default.vue         # Base layout: padding:0, overflow:hidden
    cover.vue           # Cover: bg-accent, centered flex, decorative layer
    section.vue         # Section divider: bg-alt, centered
    end.vue             # CTA/end: bg-accent, centered, warm transition
    none.vue            # Full HTML control (passthrough)
```

#### 2.2 What the Theme Contains

**package.json:**
```json
{
  "name": "slidev-theme-<preset-name>",
  "version": "1.0.0",
  "slidev": {
    "colorSchema": "light",
    "defaults": {
      "fonts": {
        "sans": "<heading-font>",
        "serif": "<body-font>",
        "mono": "<mono-font>"
      }
    }
  }
}
```

Note: The Slidev engine uses `fonts.serif` internally for the second font slot. Despite our rename of `fonts.serif` → `fonts.body` in the preset format (§1.4), the theme package.json must use Slidev's native field name `serif` — this is Slidev's API, not ours. The value will be a sans-serif font (e.g., "DM Sans"), but the key must remain `serif` for Slidev compatibility.

**styles/index.css** — migrated from current styles/index.css generation:
- All CSS custom properties (--color-accent, --bg-base, --bg-alt, --bg-accent, --font-heading, --font-body, etc.)
- Base `.slidev-layout` rules
- Blockquote reset
- Shape vocabulary classes (.icon-container variants, .label-pill, .stat-hero-num, .card-solid, .card-ghost, .card-accent)

**styles/layouts.css:**
- Layout-specific text alignment (cover, section, fact, end, statement, center)
- Background image overlay support for cover/section

**layouts/*.vue** — Vue SFC components. Each layout enforces correct padding, alignment, overflow. Minimal skeleton for `none.vue` (passthrough):

```vue
<template>
  <div class="slidev-layout" style="padding:0;overflow:hidden;">
    <slot />
  </div>
</template>
```

`default.vue` — identical to `none.vue` (all slides use full HTML control).

`cover.vue` — adds bg-accent background and centered flex:
```vue
<template>
  <div class="slidev-layout cover" style="padding:0;overflow:hidden;background:var(--bg-accent);">
    <slot />
  </div>
</template>
```

`section.vue` and `end.vue` follow the same pattern with their respective bg-levels. The `<slot />` element is required for Slidev content injection.

#### 2.3 Integration with slides.md

Generated slides.md frontmatter changes:
```yaml
theme: ./theme    # instead of theme: default
```

The generator can now reference theme classes:
```html
<span class="label-pill">EYEBROW TEXT</span>
```
instead of:
```html
<span style="display:inline-flex;align-items:center;line-height:1;background:...;border:...;border-radius:20px;padding:6px 18px;font-size:0.7rem;...">EYEBROW TEXT</span>
```

This reduces inline style length and ensures pill CSS (including line-height:1) is always correct.

#### 2.4 Learning Cycle Integration

**--deep_learn=N:**
1. After preset finalization (fonts, colors, shapes) → generate `theme/` directory
2. Test presentations use `theme: ./theme`
3. Critic evaluates — if visual bug is caused by theme CSS, fix the theme (not inline styles)
4. Final theme is saved alongside the preset file

**--create-preset:**
1. After wizard → generate theme
2. Demo presentation uses generated theme
3. Theme directory is saved next to the .preset.md file

**Normal generation with preset:**
1. Preset contains or references a `theme/` directory
2. Theme is copied to the presentation output directory
3. slides.md gets `theme: ./theme`
4. Generator writes HTML using theme classes where available, inline styles for unique per-slide elements only

#### 2.5 Integration with Generation Procedure (SKILL.md Steps 1-8)

Insert a new **Step 1.5 — Theme Resolution** between existing Step 1 (scaffold) and Step 2 (design):

```
Step 1.5 — Theme Resolution:
  1. Check if the resolved preset has a co-located theme/ directory
     (same parent directory as the .preset.md file)
  2. If theme/ exists:
     a. Copy theme/ to the output presentation directory
     b. Set slides.md frontmatter: theme: ./theme
     c. Skip styles/index.css generation (styles live in theme/)
     d. Skip components/Icon.vue generation IF theme provides it
  3. If theme/ does not exist:
     a. Set slides.md frontmatter: theme: default
     b. Generate styles/index.css as before (current behavior)
     c. Generate components/Icon.vue as before
```

The 4-tier preset lookup (SKILL.md ~line 1085) resolves to a `.preset.md` file path. The theme/ directory is located as a sibling directory named `<preset-name>-theme/` — e.g., `~/.claude/slidev-presets/boardroom.preset.md` → `~/.claude/slidev-presets/boardroom-theme/`. For local presets: `./.slidev-presets/my-preset.preset.md` → `./.slidev-presets/my-preset-theme/`.

#### 2.6 Backward Compatibility

- If a preset has no `<name>-theme/` directory → fall back to `theme: default` + styles/index.css (current behavior)
- Old presets continue to work without changes
- New presets get themes automatically

#### 2.7 preset-format.md Addition

Add new section:

```markdown
## Theme Directory

Presets may include a `theme/` directory alongside the .preset.md file.
When present, the theme is copied to the presentation output and
referenced via `theme: ./theme` in slides.md frontmatter.

The theme encapsulates:
- CSS variables and base styles (from the preset CSS block)
- Layout Vue components with correct positioning
- Shape vocabulary CSS classes

The CSS block in the preset body remains the source of truth.
The theme directory is generated from it and should not be hand-edited —
regenerate the theme directory if changes are needed.
```

---

## Section 3: bg-system 60/30/10 Distribution

### Problem

All 5 analyzed presentations use a strict base→alt→base→alt checkerboard pattern, producing ~45/35/20 distribution instead of the prescribed 60/30/10.

### Design

#### 3.1 SKILL.md Step 4.5 — bg-level Assignment Algorithm

**This algorithm REPLACES the existing per-archetype bg-level table in Step 4.5 entirely.** The old table assigned bg-levels per archetype (e.g., "icon-trio → bg-base or bg-alt (alternate)"). The new algorithm assigns bg-levels based on slide position and distribution targets, regardless of archetype:


```
Input: N slides with assigned archetypes
Output: bg-level for each slide

1. Slide 1 (cover) → bg-accent
2. Last slide (CTA/end) → bg-accent
3. Breathing/statement slides (quote-pull, stat-hero with "breathing" annotation) → bg-alt
4. Section dividers → bg-alt
5. Count remaining unassigned slides as R
6. Assign bg-alt to ceil(R * 0.2) additional slides, spaced evenly (every 4th-5th)
7. All remaining slides → bg-base

Validation: bg-base must be ≥50% of total. If not, convert the most recent bg-alt to bg-base.
```

#### 3.2 scoring-subroutine.md — Axis 7 (Color Conviction)

Update the axis descriptor text (not point deductions — the scoring rubric is holistic 1-10):

- **Low (1-3) descriptor addition:** "bg-levels form a strict alternating checkerboard pattern; bg-base used on fewer than half the slides"
- **Mid (4-6) descriptor addition:** "bg-levels show some variation but distribution is unbalanced (base and alt roughly equal)"
- **High (7-10) descriptor addition:** "bg-base dominates (≥55%), bg-alt used sparingly for rhythm, bg-accent reserved for bookend slides"

#### 3.3 QA-0b Addition

New check: bg-level distribution (**non-blocking** — see §5.6 for rationale).
- Count slides per bg-level
- Warn if bg-base < 50%
- Warn if bg-alt > 40%
- Warn if strict alternating pattern detected

---

## Section 4: Icon Container Variation

### Problem

All 5 presentations use identical circular icon containers (same size, border-radius:50%, same background, same border) on every icon. The rule "alternate between at least two container shapes" is not enforced.

### Design

#### 4.1 design-principles.md — Principle 4 (Icon System)

Expand the icon container section. Currently only `.icon-container` (circle). Add:

| Shape name | CSS | Use when |
|------------|-----|----------|
| `icon-circle` | `border-radius:50%; width:56px; height:56px` | Hero numbers, stat-hero, single accent icons |
| `icon-rounded` | `border-radius:12px; width:48px; height:48px` | Cards, grid elements, comparison-table rows |
| `icon-ghost` | `background:none; border:none; width:auto; height:auto` | Icon-trio, timeline, minimal slides |

Deck-level rule:
> A deck MUST use at least 2 different icon container shapes. On a single slide (e.g., icon-trio), containers should be identical (Gestalt similarity). Between slides, containers must vary.

#### 4.2 SKILL.md Step 5 — Icon Container Selection

Add after icon selection:

```
For each slide with icons:
  1. Check current deck icon container history
  2. If all previous icons used the same shape → pick a different one
  3. Match shape to archetype:
     - stat-hero, cover-hero → icon-circle
     - card-mosaic, comparison-table, bento-grid → icon-rounded
     - icon-trio, timeline-horizontal, timeline-zigzag → icon-ghost OR icon-circle (alternate)
```

#### 4.3 preset-format.md — shapes Section

Expand `icon_container` field:
```yaml
# Old format (backward compatible):
icon_container: circle

# New format:
icon_container: [circle, rounded-square]  # array of allowed shapes
```

If single value → treat as `[value, ghost]` (always at least 2 options).

**Identifier mapping** (preset value → CSS class):

| Preset value | CSS class | Description |
|-------------|-----------|-------------|
| `circle` | `.icon-circle` | Round container |
| `rounded-square` | `.icon-rounded` | Rounded-corner square |
| `ghost` | `.icon-ghost` | No container, icon only |

Preset values are the canonical identifiers used in `.preset.md` files. CSS classes are generated in the theme/styles/index.css.

#### 4.4 layout-css-patterns.md

Add CSS patterns for all three icon container shapes:

```css
/* Circle */
.icon-circle {
  width: 56px; height: 56px;
  border-radius: 50%;
  background: var(--color-accent-bg);
  border: 1.5px solid var(--color-accent-dim);
  display: flex; align-items: center; justify-content: center;
}

/* Rounded square */
.icon-rounded {
  width: 48px; height: 48px;
  border-radius: 12px;
  background: var(--color-accent-bg);
  border: 1.5px solid var(--color-accent-dim);
  display: flex; align-items: center; justify-content: center;
}

/* Ghost (no container) */
.icon-ghost {
  display: inline-flex;
  color: var(--color-accent);
}
```

#### 4.5 QA-0b — Icon Diversity Check

New check:
- Collect all icon containers across the deck
- If 100% identical → warn: "All icon containers are identical. Use at least 2 shapes."

---

## Section 5: QA Pipeline Hardening

### Problem

Six classes of bugs pass through QA undetected. All have documented rules but no enforcement.

### Design

All changes go into SKILL.md QA-0a and QA-0b sections.

#### 5.1 QA-0a: layout:none Enforcement

**Check:**
- Global frontmatter must contain `layout: none`
- Every slide separator (`---`) that is followed by raw HTML with `position:absolute;inset:0` must have `layout: none` in its per-slide frontmatter
- Every slide with `layout: none` must have a `<style>` block containing `.slidev-layout { padding: 0 !important; overflow: hidden; }`

**Auto-fix:**
- Add `layout: none` to global frontmatter if missing
- Add `<style>` block to slides that lack it

**Rationale:** The innosmart-tech first slide bug was caused entirely by missing `layout: none`. The cover layout CSS conflicted with inline HTML positioning.

**Relationship to existing Rule 25:** Rule 25 (SKILL.md ~line 1523) already mandates `padding:0 !important; overflow:hidden` on `.slidev-layout` as a safeguard. This QA-0a check is the **enforcement mechanism** for Rule 25 — it verifies what Rule 25 prescribes. Rule 25 remains the source of truth; QA-0a automates its verification.

#### 5.2 QA-0a: Pure #FFFFFF / #000000 Scan

**Check:**
- Scan all inline `style="..."` attributes for:
  - `#FFFFFF`, `#fff`, `#FFF`, `color:white`, `background:white`
  - `#000000`, `#000`, `color:black`, `background:black`
- Exclude `rgba(255,255,255,...)` and `rgba(0,0,0,...)` — transparency layers are acceptable

**Auto-fix:**
- `#FFFFFF` / `#fff` → use the preset's warm off-white (typically the bg-base value, e.g., `#FAFAF8`, `#f8f7f4`)
- `#000000` / `#000` → use the preset's text color variable or a warm off-black (e.g., `#1C1C1A`, `#1a1a1a`)
- Prefer `var(--color-text)` and `var(--bg-base)` when CSS variables are available

**Relationship to existing QA-0c:** QA-0c (~line 867) already flags pure #000/#FFF as CRITICAL. This new QA-0a check runs earlier and auto-fixes. QA-0c remains as defense-in-depth fallback — if QA-0a misses a dynamically constructed value, QA-0c catches it. No changes to QA-0c needed.

#### 5.3 QA-0a: Pill line-height:1

**Check:**
- Find all elements where the SAME `style` attribute contains ALL THREE of: (1) `border-radius` in 16-24px range, (2) `padding` with horizontal values, (3) `display:inline-flex`. This compound condition identifies pills/badges while excluding cards (which have border-radius but not display:inline-flex).
- On matched elements, check for presence of `line-height:1`

**Auto-fix:**
- Insert `line-height:1;` into the style attribute if missing

**Theme-class adaptation:** After Section 2 is implemented, pills may use `<span class="label-pill">` with no inline style. In this case, QA-0a must also verify that the theme's CSS class `.label-pill` includes `line-height:1`. Check by reading the theme's `styles/index.css` for the `.label-pill` rule. If missing, add it to the theme CSS.

#### 5.4 QA-0b: CSS Variables Enforcement

**Check:**
- Parse the CSS variable source: `styles/index.css` OR `theme/styles/index.css` (if theme/ directory exists after Section 2 implementation). Extract all defined CSS variable values (e.g., `--color-accent: #0D9488`).
- Scan all inline styles for hardcoded hex values
- For each hardcoded hex, check if it matches (or is within 1-2 steps of) a defined CSS variable value

**Auto-fix:**
- Replace hardcoded hex with `var(--variable-name)` when an exact match exists
- Warn (don't auto-fix) when approximate match

**Threshold:** If more than 30% of color values in inline styles are hardcoded when a matching CSS variable exists → flag as issue. This is a QA-only warning; it does not directly affect the Axis 7 score (which evaluates color conviction holistically).

#### 5.5 QA-0b: Body Font Size Minimum

**Check:**
- Find all `font-size:` declarations in inline styles
- Classify each as:
  - **Label/eyebrow**: has `text-transform:uppercase` or `letter-spacing:0.1em+` or is inside a pill → minimum 0.65rem allowed
  - **Body text**: everything else → minimum 1.25rem required

**Auto-fix:**
- Increase font-size to 1.25rem for body text that falls below
- **Overflow caveat (Rule 20 compliance):** If raising to 1.25rem would cause the slide content to overflow its container (text exceeds available space), do NOT auto-fix. Instead, flag as MAJOR: "Slide has too much content at 1.25rem — split the slide or move details to speaker notes" (per existing Rule 20).

#### 5.6 QA-0b: bg-level Distribution

**Check:**
- Count slides per background level (bg-base, bg-alt, bg-accent)
- Identify strict alternating patterns (base-alt-base-alt...)

**Warn conditions:**
- bg-base < 50% of total slides
- bg-alt > 40% of total slides
- Strict alternating pattern detected across 4+ consecutive slides

**No auto-fix** — this requires re-planning slide backgrounds, which is a Step 4.5 concern. QA warns; the fix happens in redesign.

**Non-blocking:** bg-distribution warnings do NOT block the QA pipeline. They are logged and surfaced in the QA-11 final report. This is because fixing distribution requires regenerating the entire deck's bg-level assignments, which is impractical post-generation.

---

## Section 6: Composition Variety & Cost-of-Inaction

### 6a. Layout Budget for Composition Variety

#### Problem

innosmart-tech uses split-type layouts on 8 of 10 content slides (80%), creating visual monotony despite technically using different archetypes.

#### Design

**SKILL.md Step 4.5 — Layout Budget Rule:**

After assigning archetypes to slides, validate:

```
Mandatory archetypes (EXEMPT from budget calculation):
  - cover-hero (always slide 1)
  - cta-warm (always last slide)

For each layout-group, count only NON-EXEMPT content slides:
  count = number of content slides using archetypes from this group
  total = total content slides (excluding cover-hero and cta-warm)
  percentage = count / total

Rules:
  1. No single layout-group may exceed 40% of content slides
  2. Split-group specifically: max 30% of content slides
  3. No more than 2 consecutive slides from the same layout-group
     (even if different archetypes within the group)

If violated:
  - Identify the over-represented group
  - Replace excess slides with archetypes from under-represented groups
  - Prefer inserting breathing slides (stat-hero, quote-pull) to break monotony
```

Layout groups for reference (cover-hero and cta-warm excluded from budget):
- **hero**: section-divider, stat-hero, quote-pull
- **grid**: icon-trio, bento-grid, card-mosaic, profile-grid
- **split**: two-col-text, asymmetric-split
- **timeline**: timeline-horizontal, timeline-zigzag
- **table**: data-spotlight, comparison-table

**scoring-subroutine.md — Axis 1 (Composition Variety):**

Update the axis descriptor text (holistic 1-10, not point deductions):

- **Low (1-3) descriptor addition:** "One layout-group dominates >40% of content slides; split layouts used on >30% of slides; 3+ consecutive same-group"
- **Mid (4-6) descriptor addition:** "Some variety but layout-groups are unbalanced; occasional same-group streaks"
- **High (7-10) descriptor addition:** "No group exceeds 40%; at least 3 different groups represented; no same-group streaks of 3+"

### 6b. Cost-of-Inaction Auto-Insert

#### Problem

No business presentation in the analysis includes a cost-of-inaction slide, despite the content-review-subroutine documenting this check.

#### Design

**content-review-subroutine.md:**
- Upgrade "Cost-of-Inaction Check" severity from current level to **MAJOR**
- Clarify applicability: "Applies to outlines classified as: pitch, investor, sales, proposal, fundraising, business case"

**SKILL.md Step 3 (Content Planning):**

Add rule:

```
After parsing the outline, classify the presentation type.
If type is one of [pitch, investor, sales, proposal, fundraising, business case]:
  1. Check if any slide in the outline addresses cost of inaction
     (keywords: "risk of inaction", "what happens if", "without this", "status quo cost",
      "cost of doing nothing", "стоимость бездействия", "если ничего не делать",
      "риски бездействия", "текущие потери")
  2. If no such slide exists → insert a cost-of-inaction slide after the problem/pain slide
  3. Title pattern: "What happens if [audience persona] does nothing"
     or "Стоимость бездействия: [quantified impact]"
  4. Content: quantify the cost — lost revenue, market share erosion,
     competitive disadvantage, or opportunity cost. Use numbers from the outline
     if available; if no data exists, use directional language
     ("companies lose X% market share on average") rather than fabricating
     specific figures. Never hallucinate company-specific statistics.
  5. If no identifiable problem/pain slide exists → insert after slide 2
  6. Log the insertion in the generation report so the user is aware

Hard Constraint exemption: The existing Hard Constraint ("same number of slides as outline")
does NOT apply to cost-of-inaction auto-insert. This is an explicit exception because
the research strongly supports this slide's presence for persuasion effectiveness.
The generation report must note: "Added cost-of-inaction slide not present in original outline."
```

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Single spec for all 10 improvements | Files heavily overlap; separate specs would create conflicting edits |
| Serif blacklist instead of "prefer sans" | User requirement is absolute: no serifs |
| Local Slidev themes (not npm packages) | Simpler, no publishing needed, `theme: ./theme` works in Slidev |
| bg-base ≥50% threshold (not 60%) | Practical — with 2 accent slides, reaching exactly 60% is hard on small decks |
| Icon ghost as third shape | Reduces visual noise on minimal slides; research supports "less decoration" |
| Layout budget 40% (not 33%) | Allows some flexibility while preventing the 80% split-monotony problem |
| Cost-of-inaction auto-insert (not just warn) | QA warning is too late — slide should be planned at Step 3, not patched in QA |

## Breaking Changes

| Change | Impact | Migration |
|--------|--------|-----------|
| `fonts.serif` → `fonts.body` | Existing presets use `fonts.serif` | Parser accepts both; old field maps to new |
| Serif fonts banned | Existing presets with serif body fonts | Auto-replace with nearest sans-serif |
| `icon_container` array format | Old presets use single string | Single string treated as `[value, ghost]` |
| `theme: ./theme` in slides.md | Old presentations use `theme: default` | No migration needed; old presentations unaffected |

## Feedback Traceability

| Finding | Source | Section |
|---------|--------|---------|
| Serif ban | User requirement | 1 |
| Theme generation | User requirement | 2 |
| bg checkerboard | Cross-analysis of 5 presentations | 3 |
| Uniform icon circles | Cross-analysis of 5 presentations | 4 |
| layout:none missing | innosmart-tech first slide bug | 5.1 |
| Pure #FFFFFF usage | innosmart-tech, kids-party-kp | 5.2 |
| Pill line-height:1 missing | innosmart-tech, kids-party-kp | 5.3 |
| Hardcoded CSS values | investor-q1-boardroom, kids-party-kp | 5.4 |
| Body font < 1.25rem | kids-party-kp, innosmart-tech | 5.5 |
| bg distribution unchecked | All 5 presentations | 5.6 |
| Split layout monotony | innosmart-tech (80% splits) | 6a |
| No cost-of-inaction | All business presentations | 6b |
