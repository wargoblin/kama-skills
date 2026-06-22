# Slidev QA & Generation Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix 10 systematic issues in the Slidev skill — ban serif fonts, add theme generation during learning, fix bg-level distribution, enforce icon variety, harden QA pipeline (6 new checks), and improve composition variety + cost-of-inaction.

**Architecture:** All changes are markdown edits to skill instruction files (SKILL.md + references). No code compilation. Verification is done by reading files and running the skill to generate test presentations.

**Tech Stack:** Markdown (Claude Code skill files), CSS (design patterns), Vue SFC (theme layouts)

**Spec:** `docs/superpowers/specs/2026-03-23-slidev-qa-and-generation-improvements-design.md`

---

## File Map

| File | Path | Changes |
|------|------|---------|
| SKILL.md | `.claude/skills/slidev/SKILL.md` | Font Selection, Steps 1/1.5/3/4.5/5, QA-0a/0b, create-preset wizard, deep_learn/learn |
| design-principles.md | `.claude/skills/slidev/references/design-principles.md` | Principles 3, 4 |
| preset-format.md | `.claude/skills/slidev/references/preset-format.md` | fonts.body rename, icon_container array, theme directory section |
| scoring-subroutine.md | `.claude/skills/slidev/references/scoring-subroutine.md` | Axes 1, 7 descriptors |
| content-review-subroutine.md | `.claude/skills/slidev/references/content-review-subroutine.md` | Cost-of-inaction severity |
| composition-archetypes.md | `.claude/skills/slidev/references/composition-archetypes.md` | Remove per-archetype bg-level comments |
| layout-css-patterns.md | `.claude/skills/slidev/references/layout-css-patterns.md` | Icon container CSS patterns |

All paths are relative to `slidev/` project root.

---

### Task 1: design-principles.md — Principle 3 (Typography) serif removal

**Files:**
- Modify: `.claude/skills/slidev/references/design-principles.md` — Principle 3 section

- [ ] **Step 1: Read Principle 3 section**

Read the "Principle 3: Typographic Drama" section to locate all serif references.

- [ ] **Step 2: Update font pairing guidance**

Replace any text describing "serif body text" or "sans heading + serif body" pairing with:

```
Contrast through character (geometric vs humanist, condensed vs proportional, angular vs rounded), not through serif/sans category split. Both heading and body fonts MUST be sans-serif.
```

In the type scale descriptions, replace references to "serif" body with "sans-text" body. For example:
- "Body (serif)" → "Body (sans-text)"
- "serif body text" → "sans-text body"

- [ ] **Step 3: Update Type Treatment Limit section**

In the "Type Treatment Limit" sub-section, update the italic restriction note if it references serif italic — italic is still allowed only in quotes, regardless of serif/sans.

- [ ] **Step 4: Verify changes**

Read the modified Principle 3 section. Confirm: zero occurrences of "serif" as a font category recommendation (the word "serif" may remain in blacklist or historical context, but never as "use serif for body").

- [ ] **Step 5: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/design-principles.md
git commit -m "refactor(slidev): remove serif font references from Principle 3 typography"
```

---

### Task 2: design-principles.md — Principle 4 (Icon Container Variation)

**Files:**
- Modify: `.claude/skills/slidev/references/design-principles.md` — Principle 4 section

- [ ] **Step 1: Read Principle 4 current icon container section**

Locate the `.icon-container` CSS class definition and the "Icon Container Variation" rule within Principle 4.

- [ ] **Step 2: Replace single icon-container with three variants**

Replace the existing `.icon-container` definition with three named shapes:

```css
/* Circle — for hero numbers, stat-hero, single accent icons */
.icon-circle {
  width: 56px; height: 56px;
  border-radius: 50%;
  background: var(--color-accent-bg);
  border: 1.5px solid var(--color-accent-dim);
  display: flex; align-items: center; justify-content: center;
}

/* Rounded square — for cards, grid elements, comparison-table rows */
.icon-rounded {
  width: 48px; height: 48px;
  border-radius: 12px;
  background: var(--color-accent-bg);
  border: 1.5px solid var(--color-accent-dim);
  display: flex; align-items: center; justify-content: center;
}

/* Ghost (no container) — for icon-trio, timeline, minimal slides */
.icon-ghost {
  display: inline-flex;
  color: var(--color-accent);
}
```

- [ ] **Step 3: Update the Icon Container Variation rule**

Replace the existing rule text with:

```markdown
**Icon Container Variation**: A deck MUST use at least 2 different icon container shapes. On a single slide (e.g., icon-trio), containers should be identical (Gestalt similarity). Between slides, containers must vary. The three available shapes are: `icon-circle`, `icon-rounded`, and `icon-ghost`.
```

- [ ] **Step 4: Verify changes**

Read Principle 4. Confirm: three icon container classes defined, deck-level diversity rule present.

- [ ] **Step 5: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/design-principles.md
git commit -m "feat(slidev): add three icon container shapes to Principle 4"
```

---

### Task 3: layout-css-patterns.md — Icon container CSS patterns

**Files:**
- Modify: `.claude/skills/slidev/references/layout-css-patterns.md`

- [ ] **Step 1: Read current file**

Read layout-css-patterns.md to find the existing pill/badge pattern section and determine where to add icon container patterns.

- [ ] **Step 2: Add icon container patterns section**

After the existing "Pill / Badge / Chip Pattern" section, add:

```markdown
## Icon Container Patterns

Three icon container shapes. Use at least 2 per deck (see Principle 4).

**Identifier mapping** (preset value → CSS class):

| Preset value | CSS class | Use when |
|-------------|-----------|----------|
| `circle` | `.icon-circle` | stat-hero, cover-hero, single accent icons |
| `rounded-square` | `.icon-rounded` | card-mosaic, comparison-table, bento-grid |
| `ghost` | `.icon-ghost` | icon-trio, timeline, minimal slides |

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
```

- [ ] **Step 3: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/layout-css-patterns.md
git commit -m "feat(slidev): add icon container CSS patterns to layout reference"
```

---

### Task 4: preset-format.md — fonts.body + icon_container + theme directory

**Files:**
- Modify: `.claude/skills/slidev/references/preset-format.md`

- [ ] **Step 1: Read current preset-format.md**

Read the full file to locate: fonts field table, shapes section, and end of file.

- [ ] **Step 2: Rename fonts.serif → fonts.body in the field table**

In the frontmatter fields table, find `fonts.serif` and replace with:

```
| fonts.body | String | Body font family (sans-serif). **Renamed from `fonts.serif`.** If parser encounters `fonts.serif`, treat as `fonts.body`. Must be a sans-serif font — serif fonts are banned. |
```

- [ ] **Step 3: Expand icon_container in shapes section**

Find the `icon_container` field description and replace with:

```markdown
| icon_container | String or Array | Icon container shape(s). Single value: `circle`, `rounded-square`, or `ghost`. Array: `[circle, rounded-square]`. If single value, treated as `[value, ghost]`. Mapping: `circle`→`.icon-circle`, `rounded-square`→`.icon-rounded`, `ghost`→`.icon-ghost`. |
```

- [ ] **Step 4: Add Theme Directory section at end of file**

Append before any final blank lines:

```markdown
## Theme Directory

Presets may include a `<preset-name>-theme/` directory alongside the .preset.md file (e.g., `boardroom.preset.md` → `boardroom-theme/`). When present, the theme is copied to the presentation output and referenced via `theme: ./theme` in slides.md frontmatter.

The theme encapsulates:
- CSS variables and base styles (from the preset CSS block)
- Layout Vue components with correct positioning
- Shape vocabulary CSS classes

The CSS block in the preset body remains the source of truth. The theme directory is generated from it and should not be hand-edited — regenerate the theme directory if changes are needed.
```

- [ ] **Step 5: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/preset-format.md
git commit -m "feat(slidev): rename fonts.serif to fonts.body, expand icon_container, add theme directory spec"
```

---

### Task 5: scoring-subroutine.md + content-review-subroutine.md + composition-archetypes.md

**Files:**
- Modify: `.claude/skills/slidev/references/scoring-subroutine.md` — Axes 1, 7
- Modify: `.claude/skills/slidev/references/content-review-subroutine.md` — Cost-of-inaction
- Modify: `.claude/skills/slidev/references/composition-archetypes.md` — Remove bg-level comments

- [ ] **Step 1: Update scoring-subroutine.md Axis 7 (Color Conviction)**

Find the Axis 7 row in the scoring table. The table uses pipe-delimited cells with Low/Mid/High descriptors. **Append** the new text to the existing cell content (do not create new rows):

- **Low (1-3):** add "bg-levels form a strict alternating checkerboard pattern; bg-base used on fewer than half the slides"
- **Mid (4-6):** add "bg-levels show some variation but distribution is unbalanced (base and alt roughly equal)"
- **High (7-10):** add "bg-base dominates (≥55%), bg-alt used sparingly for rhythm, bg-accent reserved for bookend slides"

- [ ] **Step 2: Update scoring-subroutine.md Axis 1 (Composition Variety)**

Find the Axis 1 row in the scoring table. **Append** to existing cell content:

- **Low (1-3):** add "One layout-group dominates >40% of content slides; split layouts used on >30% of slides; 3+ consecutive same-group"
- **Mid (4-6):** add "Some variety but layout-groups are unbalanced; occasional same-group streaks"
- **High (7-10):** add "No group exceeds 40%; at least 3 different groups represented; no same-group streaks of 3+"

- [ ] **Step 3: Update content-review-subroutine.md Cost-of-Inaction**

Find the "Cost-of-Inaction Check" section. Update severity to **MAJOR**. Add applicability clarification:

```
Applies to outlines classified as: pitch, investor, sales, proposal, fundraising, business case. If no cost-of-inaction slide is present → flag as MAJOR.
```

- [ ] **Step 4: Update composition-archetypes.md — remove inline bg-level descriptors**

Find all `**Bg-level:**` inline descriptors in archetype records (e.g., `**Bg-level:** \`--bg-accent\``). Remove these lines entirely — bg-level assignment is now handled by the position-based algorithm in SKILL.md Step 4.5, not per-archetype. The archetype records should describe layout/density/content-type only, not background level.

- [ ] **Step 5: Verify and commit**

```bash
git add -f slidev/.claude/skills/slidev/references/scoring-subroutine.md \
         slidev/.claude/skills/slidev/references/content-review-subroutine.md \
         slidev/.claude/skills/slidev/references/composition-archetypes.md
git commit -m "feat(slidev): update scoring axes, cost-of-inaction severity, remove bg-level annotations"
```

---

### Task 6: SKILL.md — Font Selection overhaul (serif ban)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md` — lines ~1139-1157

- [ ] **Step 1: Read Font Selection section (lines 1139-1157)**

Read the exact current text of the Font Selection section.

- [ ] **Step 2: Add Serif Blacklist after the Good pairs line**

**Apply this step BEFORE Step 3.** Find the line starting with `- Good pairs:` (line ~1144). IMMEDIATELY BEFORE this line, insert the serif blacklist. This anchor is stable because we haven't touched it yet:



```markdown
- **CRITICAL: Never use a serif font.** All fonts must be sans-serif. Blacklisted serif families include (non-exhaustive): Source Serif 4, Newsreader, Merriweather, Playfair Display, DM Serif Display, Garamond, Georgia, Lora, Noto Serif, Crimson Text, EB Garamond, Libre Baskerville, Cormorant, Spectral, Bitter, Zilla Slab, Roboto Slab, Rockwell. If a preset specifies a serif font, replace it with the nearest sans-serif equivalent before generation.
```

- [ ] **Step 3: Update NEVER-use list AND replace Good pairs**

**Apply AFTER Step 2.** In the NEVER-use list (line ~1141), add Space Grotesk:

```
- NEVER use generic fonts: Inter, Roboto, Arial, Open Sans, Space Grotesk, or system fonts
```

Then replace the `Good pairs:` line (now preceded by the serif blacklist from Step 2):

Old: `Good pairs: Outfit + Source Serif 4, Syne + Literata, ...`

New:
```
- Good sans+sans pairs (contrast through character, not category):
  | Heading (display) | Body (sans-text) | Contrast type |
  |------------------------|------------------|---------------|
  | Outfit | DM Sans | Geometric vs Humanist |
  | Manrope | Source Sans Pro | Semi-rounded vs Neutral |
  | Plus Jakarta Sans | Nunito | Modern geometric vs Rounded friendly |
  | Barlow | Lato | Slightly condensed vs Classic humanist |
  | Sora | IBM Plex Sans | Technical vs Corporate humanist |
  | Urbanist | Noto Sans | Modern geometric vs Universal neutral |
```

- [ ] **Step 5: Update Strict 2-Font Rule (line 1153-1154)**

Change:
- "Heading font (sans)" → "Heading font (sans-display)"
- "Body font (serif) for: descriptions, bullets, supporting text" → "Body font (sans-text) for: descriptions, bullets, supporting text"

- [ ] **Step 6: Verify changes**

Read lines 1139-1170. Confirm: no serif recommended, serif blacklist present, sans+sans pairs table present, Space Grotesk in NEVER-use list.

- [ ] **Step 7: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): ban serif fonts, add sans+sans pairing table"
```

---

### Task 7: SKILL.md — Steps 1, 1.5, Mode:Preset (fonts.body + theme resolution)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md` — lines ~1095, ~1235, and insert Step 1.5

- [ ] **Step 1: Update Step 1 font resolution (line 1235)**

Change:
```
- `fonts`: `{ sans, serif, mono }` from Google Fonts
```
To:
```
- `fonts`: `{ sans, body, mono }` from Google Fonts (all must be sans-serif)
```

- [ ] **Step 2: Update Mode:Preset — fonts.serif → fonts.body + validation (around line 1104)**

Find the text that maps preset frontmatter fields to Slidev headmatter. Update any `fonts.serif` reference to `fonts.body`. Add validation rule and mapping note:

```
Note: Slidev's headmatter uses `fonts.serif` as the key for the second font slot. When writing slides.md, map `fonts.body` from the preset to `fonts.serif` in headmatter — the value is a sans-serif font but Slidev requires this key name.

Validation: if the `fonts.body` (or legacy `fonts.serif`) value matches any font in the Serif Blacklist → replace with the nearest sans-serif equivalent before proceeding. Log: "Preset font [name] is serif — replaced with [replacement]."
```

- [ ] **Step 3: Insert Step 1.5 — Theme Resolution**

After Step 1 (Resolve Design Parameters), insert:

```markdown
### Step 1.5: Theme Resolution

1. Check if the resolved preset has a co-located theme directory (`<preset-name>-theme/` sibling to the .preset.md file)
2. If theme directory exists:
   a. Copy it to the output presentation directory as `theme/`
   b. Set slides.md frontmatter: `theme: ./theme`
   c. Skip `styles/index.css` generation (styles live in theme/)
   d. Skip `components/Icon.vue` generation IF theme provides it
3. If theme directory does not exist:
   a. Set slides.md frontmatter: `theme: default`
   b. Generate `styles/index.css` as before (current behavior)
   c. Generate `components/Icon.vue` as before
```

- [ ] **Step 4: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add Step 1.5 theme resolution, rename fonts.serif to fonts.body"
```

---

### Task 8: SKILL.md — Generation pipeline (Steps 3, 4.5, 5)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md` — Steps 4.5 and 5, and after Step 2 for content planning

- [ ] **Step 1: Replace bg-level assignment table in Step 4.5 (lines ~1358-1370)**

Replace the entire per-archetype bg-level table with the new distribution algorithm:

```markdown
**bg-level Assignment Algorithm** (replaces per-archetype table):

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
```

- [ ] **Step 2: Add Layout Budget Rule to Step 4.5**

After the archetype selection algorithm (before bg-level assignment), add:

```markdown
**Layout Budget Rule** — After assigning archetypes, validate composition variety:

Mandatory archetypes (EXEMPT from budget): cover-hero (slide 1), cta-warm (last slide).

For non-exempt content slides:
1. No single layout-group may exceed 40% of content slides
2. Split-group (two-col-text, asymmetric-split) specifically: max 30%
3. No more than 2 consecutive slides from the same layout-group

Layout groups: hero (section-divider, stat-hero, quote-pull), grid (icon-trio, bento-grid, card-mosaic, profile-grid), split (two-col-text, asymmetric-split), timeline (timeline-horizontal, timeline-zigzag), table (data-spotlight, comparison-table).

If violated: replace excess slides with archetypes from under-represented groups. Prefer breathing slides (stat-hero, quote-pull) to break monotony.
```

- [ ] **Step 3: Add Icon Container Selection to Step 5 (after line ~1436)**

After the Icon component creation instruction (Step 6a), add:

```markdown
**Icon Container Selection** — For each slide with icons:
1. Check the deck's icon container history so far
2. If all previous icons used the same shape → pick a different one
3. Match shape to archetype:
   - stat-hero, cover-hero → icon-circle
   - card-mosaic, comparison-table, bento-grid → icon-rounded
   - icon-trio, timeline-horizontal, timeline-zigzag → icon-ghost OR icon-circle (alternate)
4. A deck MUST use at least 2 different icon container shapes.
```

- [ ] **Step 4: Add Cost-of-Inaction auto-insert to Step 4.5 (Composition Planning)**

In SKILL.md, the content planning happens in Step 4.5 (Composition Planning, line ~1317), NOT in Step 3 (which is `Write uno.config.ts`). Insert the following at the TOP of Step 4.5, before archetype assignment begins:

```markdown
**Cost-of-Inaction Check** — After parsing the outline, classify the presentation type. If type is one of [pitch, investor, sales, proposal, fundraising, business case]:
1. Check if any slide addresses cost of inaction (keywords: "risk of inaction", "what happens if", "without this", "status quo cost", "стоимость бездействия", "если ничего не делать", "текущие потери")
2. If no such slide → insert a cost-of-inaction slide after the problem/pain slide (or after slide 2 if no pain slide)
3. Title pattern: "Стоимость бездействия: [quantified impact]"
4. Use numbers from outline if available; otherwise use directional language. Never hallucinate company-specific statistics.
5. Log insertion in generation report: "Added cost-of-inaction slide not present in original outline."

**Hard Constraint exemption:** cost-of-inaction auto-insert is an explicit exception to the "same number of slides as outline" constraint.
```

- [ ] **Step 5: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): new bg-level algorithm, layout budget, icon selection, cost-of-inaction"
```

---

### Task 9: SKILL.md — QA-0a hardening (layout:none, #FFF scan, pill line-height)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md` — QA-0a section (lines ~805-809)

- [ ] **Step 1: Read current QA-0a section**

Read lines 805-809 to see the existing 4 checks.

- [ ] **Step 2: Add layout:none enforcement check**

Append to QA-0a:

```markdown
- **layout:none enforcement (Rule 25)**: Global frontmatter must contain `layout: none`. Every slide separator followed by raw HTML with `position:absolute;inset:0` must have `layout: none` in per-slide frontmatter. Every slide with `layout: none` must have a `<style>` block containing `.slidev-layout { padding: 0 !important; overflow: hidden; }`. Auto-fix: add if missing. (This check enforces Rule 25.)
```

- [ ] **Step 3: Add pure #FFFFFF/#000000 scan**

Append to QA-0a:

```markdown
- **Pure #FFFFFF/#000000 scan**: Scan all inline `style="..."` for `#FFFFFF`, `#fff`, `#FFF`, `color:white`, `#000000`, `#000`, `color:black`. Exclude `rgba(255,255,255,...)` and `rgba(0,0,0,...)` (transparency layers OK). Auto-fix: replace `#FFFFFF`/`#fff` with preset's warm off-white (e.g., `var(--bg-base)` or `#FAFAF8`); replace `#000000`/`#000` with `var(--color-text)` or warm off-black. (QA-0c retains its existing CRITICAL flag as defense-in-depth fallback.)
```

- [ ] **Step 4: Add pill line-height:1 check**

Append to QA-0a:

```markdown
- **Pill line-height:1**: Find all elements where the SAME `style` attribute contains ALL THREE of: (1) `border-radius` in 16-24px range, (2) `padding` with horizontal values, (3) `display:inline-flex`. Check for `line-height:1`. Auto-fix: insert `line-height:1;` if missing. **Theme-class adaptation**: if pills use `class="label-pill"`, verify theme CSS `.label-pill` includes `line-height:1`.
```

- [ ] **Step 5: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add layout:none, #FFF scan, pill line-height to QA-0a"
```

---

### Task 10: SKILL.md — QA-0b hardening (CSS vars, body font, bg dist, icon diversity)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md` — QA-0b section (lines ~811-839)

- [ ] **Step 1: Read current QA-0b section**

Read lines 811-839 to see existing checks.

- [ ] **Step 2: Add CSS Variables enforcement**

Append to QA-0b:

```markdown
- **CSS Variables enforcement**: Parse CSS variable source (`styles/index.css` or `theme/styles/index.css`). Scan inline styles for hardcoded hex values. If hardcoded hex matches a defined CSS variable → auto-replace with `var(--name)`. Warn if >30% of color values are hardcoded. QA-only warning (does not affect Axis 7 score directly).
```

- [ ] **Step 3: Add Body Font Size minimum**

Append to QA-0b:

```markdown
- **Body font size minimum**: Find all `font-size:` in inline styles. Classify: label/eyebrow (has `text-transform:uppercase` or `letter-spacing:0.1em+` or inside pill) → min 0.65rem OK. Body text → min 1.25rem required. Auto-fix: raise to 1.25rem. **Overflow caveat (Rule 20):** if raising to 1.25rem would overflow → do NOT auto-fix, flag as MAJOR: "Slide has too much content at 1.25rem — split or move to speaker notes."
```

- [ ] **Step 4: Add bg-level distribution check**

Append to QA-0b:

```markdown
- **bg-level distribution** (non-blocking): Count slides per bg-level. Warn if bg-base < 50%, bg-alt > 40%, or strict alternating pattern across 4+ slides. No auto-fix — logged to QA-11 report.
```

- [ ] **Step 5: Add icon diversity check**

Append to QA-0b:

```markdown
- **Icon diversity**: Collect all icon containers across the deck. If 100% identical shape → warn: "All icon containers identical. Use at least 2 shapes (icon-circle, icon-rounded, icon-ghost)."
```

- [ ] **Step 6: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add CSS vars, body font, bg dist, icon diversity to QA-0b"
```

---

### Task 11: SKILL.md — --create-preset wizard + deep_learn theme generation

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md` — --create-preset (line ~70), --deep_learn (lines ~654-795)

- [ ] **Step 1: Update --create-preset typography question (line 70)**

Change:
```
- **Typography**: "Font personality? (geometric & modern, classic & refined, rounded & friendly, sharp & technical)"
```
To:
```
- **Typography**: "Font personality? (geometric & modern, rounded & friendly, sharp & technical, humanist & classic, condensed & bold)"
```

- [ ] **Step 2: Add theme generation to --create-preset (after step 4, before step 5)**

After the preset file is written (step 4 in --create-preset), insert:

```markdown
4b. **Generate theme directory**: Based on the preset's CSS block and font/color choices, generate a `<name>-theme/` directory alongside the preset file with:
   - `package.json` (Slidev theme metadata with font defaults)
   - `styles/index.css` (CSS variables, base layout, shape vocabulary from the preset CSS block)
   - `styles/layouts.css` (layout-specific text alignment, background overlay support)
   - `layouts/none.vue`, `layouts/default.vue` (passthrough with `padding:0;overflow:hidden`)
   - `layouts/cover.vue` (bg-accent background), `layouts/section.vue` (bg-alt), `layouts/end.vue` (bg-accent)

   Each layout Vue SFC uses `<template><div class="slidev-layout [name]" style="padding:0;overflow:hidden;"><slot /></div></template>`.

   Note: In theme `package.json`, use Slidev's native `fonts.serif` key for the body font slot (even though the value is sans-serif) — this is Slidev's API requirement.
```

- [ ] **Step 3: Add theme generation to --deep_learn PDL-2 cycle**

In the Preset Deep Learning Procedure (PDL-2), after the preset is finalized in each cycle, add:

```markdown
After applying changes to the preset in PDL-2.4, regenerate the theme directory from the updated CSS block. Test presentations in PDL-2.2 must use `theme: ./theme` (not `theme: default`).
```

- [ ] **Step 4: Add theme generation to --learn procedure**

In the Preset Learning Procedure, find PL-4 (the step where CSS/font improvements are applied to the preset after critic feedback). After the line that saves changes to the preset file, insert:

```markdown
After saving preset changes, regenerate the `<name>-theme/` directory from the updated CSS block. Subsequent test presentations use `theme: ./theme`.
```

- [ ] **Step 5: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add theme generation to create-preset, deep_learn, learn"
```

---

### Task 12: Existing presets — Replace serif fonts

**Files:**
- Modify: Any `.preset.md` files that reference serif body fonts

- [ ] **Step 1: Find all preset files**

Search for `*.preset.md` files in the repository and in `~/.claude/slidev-presets/`. If the global presets directory doesn't exist or no preset files are found, skip this task entirely — it only applies to existing presets.

- [ ] **Step 2: Check each preset for serif fonts**

For each preset, read the `fonts.serif` field and check if the value is a serif font (from the blacklist in spec §1.2).

- [ ] **Step 3: Replace serif → sans-serif in each affected preset**

Known replacements:
- `boardroom.preset.md`: `Source Serif 4` → `DM Sans`
- `kids-party-kp` related presets: `Nunito` is already sans-serif ✅
- Any preset with `Newsreader` → `Nunito` or `Lato`

Also rename field: `fonts.serif` → `fonts.body` in each preset.

- [ ] **Step 4: Regenerate theme directories for updated presets**

For each updated preset that has a `<name>-theme/` directory, regenerate it with the new body font.

- [ ] **Step 5: Commit**

```bash
git add -f <preset files>
git commit -m "feat(slidev): replace serif fonts with sans-serif in existing presets"
```

---

### Task 13: Verification — Generate test presentation

**Files:**
- None modified — validation only

- [ ] **Step 1: Generate a test presentation using boardroom preset**

Run: `/slidev --preset boardroom` with a test outline (if boardroom preset exists; otherwise use `--no-preset` mode) to verify:
- Sans-serif body font used (DM Sans, not Source Serif 4)
- `theme: ./theme` in slides.md (if theme exists) or `theme: default`
- bg-levels follow 60/30/10 distribution
- Icon containers vary across slides
- No `#FFFFFF` or `#000000` in inline styles
- All pills have `line-height:1`
- `layout: none` on all slides
- No split-layout monotony

- [ ] **Step 2: Export and visually verify**

Run: `npx slidev export --format png --output verify-export`

Check first and last slides for correct positioning, font rendering, and color warmth.

- [ ] **Step 3: Run QA checks mentally**

Walk through QA-0a and QA-0b checklists against the generated slides.md to confirm new checks would pass.

- [ ] **Step 4: Clean up test files**

Remove the test presentation directory.
