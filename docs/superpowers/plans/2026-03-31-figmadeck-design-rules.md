# Figmadeck Design Rules + Adaptive Layout Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add universal design knowledge, adaptive layout rules, and background-aware deduplication to the figmadeck skill.

**Architecture:** Four independent changes, each a separate commit: (1) new reference file design-rules.md, (2) expanded QA checks in qa-cycle.md, (3) adaptive layout Step 5b in generation-pipeline.md, (4) background dedup + variant selection in figma-extraction.md.

**Tech Stack:** Markdown instruction files for the figmadeck skill.

**Spec:** `docs/superpowers/specs/2026-03-31-figmadeck-design-rules-adaptive-layout.md`

---

## File Structure

```
figmadeck/.claude/skills/figmadeck/
  SKILL.md                            # Modify: add reference link (1 line)
  references/
    design-rules.md                   # CREATE: full universal design rules
    qa-cycle.md                       # Modify: add design rules checks to Phase A
    generation-pipeline.md            # Modify: add Step 5b + variant selection in Step 4
    figma-extraction.md               # Modify: add 6th dedup criterion
```

---

### Task 1: Create `references/design-rules.md`

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/design-rules.md`
- Modify: `figmadeck/.claude/skills/figmadeck/SKILL.md:14` (add reference link)

- [ ] **Step 1: Write design-rules.md**

Create `figmadeck/.claude/skills/figmadeck/references/design-rules.md`. This is a full (NOT summary) reference of universal presentation design rules that apply regardless of Figma template style.

The file must contain 8 sections with detailed rules, thresholds, and explanations sourced from the research corpus in `slidev/docs/research/`. Read all 4 research files (Russian versions where available) and extract every universal rule:

- `slidev/docs/research/ugly-presentations-anti-patterns-ru.md`
- `slidev/docs/research/beautiful-presentations-guide-ru.md`
- `slidev/docs/research/html-css-presentation-techniques.md`
- `slidev/docs/research/slidev-design-best-practices.md`

**Required sections with key rules (include ALL of these + any additional universal rules found):**

**Section 1: Text Density & Content Limits**
- Max 75 words on a content slide (150+ = document, not slide)
- Max 6 bullets × 6 words per bullet (stricter: 5×5 or 4×4)
- Max 2 levels of bullet nesting (3+ = structural confusion)
- 3-5 new information units per slide (Cowan 2001, NOT Miller's 7±2)
- 1 idea per slide — 3-second test (Nancy Duarte "instant media")
- Slide ≠ speaker notes (Mayer's Redundancy Principle — worse retention)
- More than 6 lines of text = cognitive overload zone

**Section 2: Typography & Readability**
- Font size minimums: body ≥ 1.25rem (24px projected), heading ≥ 2.2rem, label ≥ 0.65rem
- Max 2 typeface families (3+ = Font Salad)
- Body text: ONLY left-align. Center acceptable ONLY for single-line headings, section breaks, standalone quotes
- Multi-line centered body text = readability failure
- Line-height: 1.2-1.45x for body, 0.9-1.1x for headings
- Max line length for body: 65ch (optimal reading)
- Bold ≤ 20-25% of text — overemphasis kills emphasis
- Use ONE emphasis technique per element (bold OR color OR size, not all three)
- Italic only in blockquotes, attributions, captions — never headings
- `tabular-nums` on metric figures

**Section 3: Spacing & Layout**
- Min 30-40% whitespace on each slide (< 20% = explicit overload)
- All elements align to 8px grid (arbitrary spacing = anti-pattern)
- Consistent margins across ALL slides: min 5-8% of slide width on each side
- No floating/misaligned elements — alignment is non-negotiable
- Max 7 separate visual elements on a slide (beyond = cognitive overload)
- More whitespace around element = greater visual weight
- Whitespace is an active design element, not wasted space

**Section 4: Visual Hierarchy**
- Min 3:1 ratio between heading and body font size
- Minimum 3 hierarchy levels (flat = no focus)
- Max 3 distinct font sizes on one slide
- Max 2 "large" elements per slide
- Squint test: most important element must be obvious with blurred vision
- Title = conclusion, not label ("Revenue grew 23% despite headwinds" NOT "Q3 Revenue")
- The most important element must be obvious even when squinting

**Section 5: Contrast & Accessibility**
- WCAG AA: 4.5:1 for body text, 3:1 for large text (≥ 24px or ≥ 18.66px bold)
- Target 7:1+ for projected presentations (projector washes out contrast)
- #777 on white = 4.47:1 — does NOT pass (no rounding up)
- Never use color as sole information differentiator
- Test designs in grayscale
- No pure #000000 or #FFFFFF — use tinted off-black/off-white variants
- Colorblind safety: avoid red+green, green+brown, blue+purple as only differentiators

**Section 6: Element Spacing & Overflow Prevention**
- Elements must NEVER overlap — overlap = critical failure
- Footer text: if total label width > container width → shorten or redistribute
- Text in fixed-width slots: shorten content to fit, NEVER shrink font below minimum
- Min gap between any two adjacent elements: 8px (0.5rem)
- If content pushes element beyond container boundary → adapt:
  1. Shorten/rephrase text (preserve meaning)
  2. Redistribute space between elements
  3. Move overflow to speaker notes
- Footer zone: reserve bottom 44-56px — no content may enter
- `position: absolute` only for decorative overlays, never main content

**Section 7: Data Visualization**
- Never 3D charts — always 2D
- Max 5-6 pie/donut chart segments (beyond = use bar chart)
- Bar chart axis must start at zero
- Max 5-6 lines on line chart
- Remove gridlines or reduce to 10-20% opacity
- Remove all chart shadows, decorative backgrounds
- Hero metric must be ≥ 2x size of supporting metrics
- Statistics should cite sources

**Section 8: Animation & Transitions**
- Animate ONLY `transform` and `opacity` (never layout properties: width, height, top, left, margin, padding)
- Slide transitions: 200-400ms. Element entrance: 150-300ms. Micro-interactions: 100-200ms
- `v-click` for genuine pacing only, not decoration
- Max 5-8 clicks per slide
- `v-click` on every bullet point = anti-pattern (loses pacing value)
- Purposeful animation only — 2026 experts explicitly warn against "fussy animations that distract"

For each rule, include: the rule itself, the threshold/number, WHY it matters (cognitive science reference where applicable), and what to do when violated (auto-fix strategy).

- [ ] **Step 2: Add reference link in SKILL.md**

In `figmadeck/.claude/skills/figmadeck/SKILL.md`, find the References section (around line 13-17). Add this line after the existing references:

```markdown
- `references/design-rules.md` — **Universal design rules** (readability, spacing, hierarchy, contrast — applied always)
```

- [ ] **Step 3: Verify**

```bash
grep -c "^## " figmadeck/.claude/skills/figmadeck/references/design-rules.md
```
Expected: 8 (one per section)

```bash
grep "design-rules" figmadeck/.claude/skills/figmadeck/SKILL.md
```
Expected: 1 match

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/design-rules.md .claude/skills/figmadeck/SKILL.md
git commit -m "feat(figmadeck): add universal design rules reference"
```

---

### Task 2: Expand `qa-cycle.md` with design rules checks

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md:36-37` (insert after Basic Checks, before Figma Structural Comparison)

- [ ] **Step 1: Read current qa-cycle.md**

Read `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md`. The insertion point is after line 36 ("Fix any violations directly in slides.md before proceeding to Figma structural comparison.") and before line 38 ("### Figma Structural Comparison").

- [ ] **Step 2: Insert design rules checks**

Insert the following block between "Basic Checks" and "Figma Structural Comparison":

```markdown

### Design Rules Checks

Read `references/design-rules.md` before running these checks. Apply AFTER basic checks and BEFORE Figma comparison — fix foundational problems first.

**Text density (per slide):**
- Count words (excluding HTML tags and CSS). If > 75 → **FAIL**. Auto-fix: shorten text, move secondary details to speaker notes. Keep core message.
- Count bullet items (`<li>` or `-` items). If > 6 → **FAIL**. Auto-fix: merge related bullets or move least critical to speaker notes.
- Check bullet nesting depth. If > 2 levels → **FAIL**. Auto-fix: flatten to 2 levels max.

**Typography:**
- Scan for multi-line body text (`<p>`, `<li>` with > 60 chars) with `text-align: center` → **FAIL**. Auto-fix: set `text-align: left; max-width: 65ch; margin: 0 auto`.
- Estimate bold percentage: count characters inside `<strong>`, `<b>`, `font-weight: 700/800/900` vs total text. If > 20% → **WARNING**. Log: "Overemphasis — bold exceeds 20% on slide N".
- Count distinct `font-size` values in one slide's `<style>` + inline styles. If > 3 → **WARNING**. Log: "Too many font sizes on slide N".

**Spacing & overlap:**
- For each slide, check whether any content element's bottom edge + its margin enters the footer zone (bottom 44-56px of slide). If yes → **CRITICAL**. Auto-fix: shorten text or reduce top padding to push content up.
- For footer specifically: if footer uses fixed positioning (`position: absolute; left: Xpx`) or fixed `margin-left` values, and total text width of all footer spans > 900px (slide width minus margins) → **FAIL**. Auto-fix: convert footer to `display: flex; gap: 24px; align-items: center` with `white-space: nowrap; overflow: hidden; text-overflow: ellipsis` on each span.
- Check margins: if any content element is < 5% from any slide edge (48px on 960px width) → **WARNING**. Log: "Content too close to edge on slide N".

**Hierarchy:**
- Calculate heading-to-body font size ratio on each slide. If < 2:1 → **WARNING**. Log: "Weak hierarchy — heading/body ratio X:1 on slide N".
- Compare largest and second-largest elements by font-size. If ratio < 1.5:1 → **WARNING**. Log: "No clear focal point on slide N".

**Contrast:**
- For each text-on-background pair, calculate contrast ratio. Body text (< 24px) with ratio < 4.5:1 → **FAIL**. Large text (≥ 24px) with ratio < 3:1 → **FAIL**. Auto-fix: adjust text color to meet minimum. Log computed ratios.

**Severity handling:**
- **CRITICAL** (overlap into footer zone): auto-fix immediately, re-check after fix
- **FAIL** (density, alignment, contrast, footer overflow): auto-fix immediately, re-check after fix
- **WARNING** (hierarchy, bold, font-size count, edge proximity): log to iteration report only, do not block
```

- [ ] **Step 3: Verify**

```bash
grep -c "CRITICAL\|FAIL\|WARNING" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md
```
Expected: significant increase from baseline (15+ new occurrences)

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/qa-cycle.md
git commit -m "feat(figmadeck): add design rules checks to QA cycle Phase A"
```

---

### Task 3: Add Step 5b (Adaptive Layout) to `generation-pipeline.md`

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md:181` (insert after PROHIBITED section)
- Modify: `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md:1` (add design-rules reference link near top)

- [ ] **Step 1: Read current generation-pipeline.md**

Read `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md`. The insertion point for Step 5b is after line 181 (end of PROHIBITED section: "7. **No out-of-template visuals**..."). The reference link goes near the top of the file (after the title/intro).

- [ ] **Step 2: Add reference link at top**

Find the first paragraph or intro section of `generation-pipeline.md` and add:

```markdown
Read `references/design-rules.md` for universal spacing, density, and hierarchy rules that apply during generation.
```

- [ ] **Step 3: Insert Step 5b after PROHIBITED section**

Insert after the PROHIBITED section (after line 181):

```markdown

---

## Step 5b: Adaptive Layout

After filling `{{SLOT}}` markers (Step 5) and before QA cycle, verify that content fits the archetype's layout. Content in Russian and other languages is often 20-40% longer than the English placeholder text in Figma templates.

**These adaptations are the ONLY permitted deviations from the verbatim copy rule.** They preserve visual identity while preventing layout breakage.

### Footer Adaptation

If the archetype has a footer with multiple text spans and the generated text is longer than placeholders:

1. Replace any fixed positioning (`position: absolute; left: Xpx`, fixed `margin-left`) with flex layout:
   ```css
   display: flex; gap: 24px; align-items: center;
   ```
2. Add overflow protection on each span:
   ```css
   white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
   ```
3. Last element (page number): `margin-left: auto` to push right
4. If total footer text still overflows after flex: shorten labels (e.g., "Стратегическое ревью — Q2 2026" → "Q2 2026")

### Two-Column Text Adaptation

If heading (left column) or description (right column) is longer than Figma placeholder:

1. Estimate text height: (character count / chars-per-line) × line-height × font-size
2. Available height = slide height (540px) − top padding − footer zone (56px)
3. If text height > available height:
   - First: shorten/rephrase content (preserve core meaning)
   - Then: reduce column gap (minimum 24px / 1.5rem)
   - Last resort: move overflow to speaker notes
4. **NEVER** let text overlap with footer or adjacent column

### Card/Grid Element Adaptation

If card text is longer than placeholder:

1. Verify text doesn't overflow card boundary (card height × line count)
2. If overflows: shorten text to fit within card
3. Maintain minimum 8px (0.5rem) padding inside all cards
4. If card content cannot be shortened further: move detail to speaker notes, keep title + key metric only

### General Spacing Rules

- **Min gap**: 8px (0.5rem) between ANY two adjacent elements — no exceptions
- **Container bounds**: no element may extend beyond its parent container boundaries
- **Footer zone**: bottom 44-56px of the slide is RESERVED — no content element may enter this zone
- **Absolute-to-flex**: if the archetype uses `position: absolute` on content elements (not decorative), convert to flex layout where possible to allow natural content flow
- **Overflow cascade**: if one element's overflow pushes adjacent elements → redistribute ALL spacing on the slide, not just the overflowing element
```

- [ ] **Step 4: Add variant selection to Step 4**

Find the Step 4 "Composition Table Output" section (around line 98). Insert the following block BEFORE the composition table, after "Visual Rhythm Check" (after line 96):

```markdown

### Variant Selection

When the preset has multiple archetypes for the same `contentType` (e.g., 4 section-dividers with different gradient backgrounds), select by context:

1. **Analyze content**: What is this slide's topic? What is its emotional valence? (positive/neutral/negative)
2. **Analyze variants**: What color temperature does each variant's background have?
3. **Match by mood**:
   - Warm colors (gold, amber, magenta, rose) → positive outcomes, achievements, CTA, celebration
   - Cool colors (teal, blue, green, cyan) → analysis, data, process, sustainability, technology
   - Neutral/dark → transitions, pauses, serious topics, challenges
4. **If no clear mood match** → distribute evenly across the presentation (no two adjacent slides use the same variant)
5. **Cover slide** → always use the FIRST variant by Figma order (matches the template's opening)
6. **CTA/closing slide** → always use the LAST variant by Figma order (matches the template's ending)
```

Update the Composition Table to include `variant_id`:

Replace the existing table format with:

```
Slide | Content Type | Archetype ID                    | Variant | Figma Node ID
------|--------------|---------------------------------|---------|---------------
1     | intro        | figmadeck-<name>-cover          | —       | 12:345
3     | vision       | figmadeck-<name>-section        | teal    | 12:348
7     | vision       | figmadeck-<name>-section        | magenta | 12:360
```

- [ ] **Step 5: Verify**

```bash
grep -c "Step 5b\|Adaptive Layout\|Variant Selection" figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md
```
Expected: 3+

- [ ] **Step 6: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/generation-pipeline.md
git commit -m "feat(figmadeck): add adaptive layout (Step 5b) + variant selection"
```

---

### Task 4: Background-aware deduplication in `figma-extraction.md`

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md:159-167` (expand Step 4-pre dedup criteria)

- [ ] **Step 1: Read current Step 4-pre**

Read `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md` lines 157-169. The current deduplication has 5 criteria.

- [ ] **Step 2: Add 6th criterion**

In `figma-extraction.md`, find the deduplication criteria list (lines 161-165). After criterion 5 ("Same nesting depth"), add:

```markdown
6. **Same background treatment** — compare the root frame's fills:
   - Both solid color: same color (deltaE < 10 using CIEDE2000) → same
   - Both gradient: same gradient type (linear/radial/angular) AND similar color stops (deltaE < 15 for each stop) → same
   - One solid + one gradient → **DIFFERENT** (always separate archetypes)
   - Dark background (luminance < 30%) vs light background (luminance > 70%) → **ALWAYS DIFFERENT** (they require different text colors, contrast treatment, and overlay behavior)
```

- [ ] **Step 3: Update the paragraph after the criteria list**

Find the paragraph that starts "If ANY detail differs" (around line 167). Replace it with:

```markdown
If ANY criterion differs — structure, layout, element sequence, nesting, OR background — they are **separate archetypes**. Err on the side of keeping more archetypes.

**Naming convention for background variants**: When slides share structure but differ only in background, append a color descriptor to the archetype name:
- `figmadeck-section-teal-gold` (dark gradient with teal + gold stops)
- `figmadeck-section-blue-purple` (dark gradient with blue + purple stops)
- `figmadeck-two-col-light` (light bg-base background)
- `figmadeck-two-col-dark` (dark bg-accent background)

The color descriptor comes from the dominant gradient stop colors or the background luminance category.
```

- [ ] **Step 4: Verify**

```bash
grep -c "background treatment\|luminance\|deltaE\|gradient" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md
```
Expected: 5+ new occurrences

- [ ] **Step 5: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-extraction.md
git commit -m "feat(figmadeck): background-aware deduplication (6th criterion)"
```

---

### Task 5: Final verification + push

**Files:**
- All modified files from Tasks 1-4

- [ ] **Step 1: Verify all changes are present**

```bash
# design-rules.md exists and has 8 sections
grep -c "^## " figmadeck/.claude/skills/figmadeck/references/design-rules.md

# SKILL.md references design-rules.md
grep "design-rules" figmadeck/.claude/skills/figmadeck/SKILL.md

# qa-cycle.md has new checks
grep -c "Design Rules Checks" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md

# generation-pipeline.md has Step 5b and Variant Selection
grep "Step 5b\|Variant Selection" figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md

# figma-extraction.md has 6th criterion
grep "background treatment" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md
```

Expected: all greps return matches.

- [ ] **Step 2: Cross-reference consistency**

```bash
# qa-cycle references design-rules.md
grep "design-rules" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md

# generation-pipeline references design-rules.md
grep "design-rules" figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md

# No references to slidev-specific files (design-principles, composition-archetypes)
grep -r "design-principles\|composition-archetypes\|scoring-subroutine" figmadeck/.claude/skills/figmadeck/
```

Expected: first 2 return matches, last returns nothing.

- [ ] **Step 3: Push**

```bash
git push
```
