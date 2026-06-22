# Generation Pipeline

Generate a Slidev presentation from an outline using a Figma preset. Executes Steps 1–5 in order without stopping for confirmation. All files are namespaced as `figmadeck-<name>-*`.

Read `references/design-rules.md` for universal spacing, density, and hierarchy rules that apply during generation.

---

## Step 1: Parse Outline

Read the outline (inline text or file path). For each slide extract: slide number, title, content, and initial content type.

| Pattern | Content Type |
|---------|-------------|
| First slide, large heading + subtitle | `intro` |
| Last slide, CTA phrasing | `cta` |
| 1–2 large numbers + small label | `metric` |
| 3+ parallel items in a list or grid | `scope` |
| Team members with names/roles | `team` |
| Numbered steps or ordered process | `process` |
| Single impactful statement + paragraph | `vision` |
| Image placeholder, no text | `visual-break` |
| Anything else | `context` |

Count slides. Record the total — this is the contract. The final presentation MUST have exactly this many slides in exactly this order.

---

## Step 2: Ghost Deck Test

### Title Rewrite

Convert label-titles (nouns/categories) into action-titles (specific, data-bearing statements):

| Before | After |
|--------|-------|
| "Our Team" | "15 engineers scaling B2B products since 2019" |
| "Market" | "Market 24B RUB growing +22% YoY" |
| "Results" | "ARR tripled in 18 months without headcount growth" |

Rules: pull specific numbers from content into the title. If no numbers, rewrite as a verb-led claim. Titles must pass the "so what?" test. Never invent facts.

### Content Shortening

Proactively shorten content that is clearly too long for a slide:
- Bullet points > ~10 words → trim to core claim
- More than 5–6 bullets → keep top 4–5, move the rest to speaker notes
- Dense paragraphs → convert to 2–3 key bullets

This is structural simplification. The text fit check in Step 5 handles overflow against slot `max_length`.

---

## Step 3: Scaffolding

### Theme Directory

Copy `figmadeck-<name>-theme/` from the preset directory verbatim into `<project>/theme/`. Do not modify any file.

### package.json

Write `<project>/package.json` with Slidev dependencies:

```json
{
  "name": "figmadeck-<name>-presentation",
  "private": true,
  "scripts": { "dev": "slidev --open", "build": "slidev build", "export": "slidev export" },
  "dependencies": { "@slidev/cli": "^0.49.0", "@slidev/theme-default": "latest" },
  "devDependencies": { "playwright-chromium": "^1.42.0" }
}
```

### uno.config.ts

Write `<project>/uno.config.ts` importing font tokens from the preset's CSS block (`--font-sans`, `--font-body`).

### Icon Component (conditional)

Scan all `figmadeck-<name>-figma/slide-*/archetype.html` for the string `<Icon`.
- If found → create `<project>/components/Icon.vue` using the icon set referenced in the preset.
- If NOT found → skip entirely. Do not create the file. Do not add any icon infrastructure.

---

## Step 4: Composition Planning

### Archetype Matching

For each slide, find a matching archetype **within the current preset only** — never mix presets, never use standard (non-Figma) archetypes:

1. Read `figma.archetypes[]` from the `.preset.md` frontmatter
2. Find archetype with `contentType` matching the slide's content type (exact match)
3. If no exact match → find closest by structure (grid → another grid archetype; prefer archetypes used less frequently to avoid consecutive repetition)

### Visual Rhythm Check

After initial matching: if 4+ consecutive slides use dense archetypes (`context`, `scope`, `credentials`, `team`) → replace one with `figma-section` archetype (if present). Pick the slide with the single most impactful number, quote, or insight. This changes the archetype used for an existing slide — never adds slides.

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

### Composition Table Output

Print before Step 5:

```
Slide | Content Type | Archetype ID                    | Variant | Figma Node ID
------|--------------|---------------------------------|---------|---------------
1     | intro        | figmadeck-<name>-cover          | —       | 12:345
3     | vision       | figmadeck-<name>-section        | teal    | 12:348
7     | vision       | figmadeck-<name>-section        | magenta | 12:360
```

---

## Step 5: Write slides.md

**Archetype format (Pipeline v2):** Archetypes are now generated from `get_design_context` output, transpiled from React+Tailwind to HTML+inline CSS. They contain:
- Inline `style=""` attributes (not Tailwind classes)
- `var(--*)` CSS variable references for colors
- Flex/grid layout from Figma's own auto-layout interpretation
- Preserved gradient backgrounds, shadows, and image assets

The **VERBATIM COPY RULE** still applies — copy the transpiled archetype HTML exactly, replace only `{{SLOT}}` markers with content.

### 5a: VERBATIM Copy

**COPY** each archetype.html **VERBATIM** — literal character-for-character, not "inspired by". Every inline style, every class, every HTML element, every attribute preserved exactly.

### 5b: Slot Replacement

**REPLACE** only `{{SLOT}}` markers. Standard slots:

| Slot | Source |
|------|--------|
| `{{TITLE}}` | Slide title |
| `{{SUBTITLE}}` | Subtitle or first content line |
| `{{DESCRIPTION}}` | Paragraph content |
| `{{CARD_N_TITLE}}` / `{{CARD_N_DESC}}` | Nth grid item |
| `{{METRIC_VALUE}}` / `{{METRIC_LABEL}}` | Large number + label |
| `{{IMAGE_URL}}` | Image path or placeholder |
| `{{CAPTION}}` | Caption text |

Replace slots only. Touch nothing else.

### 5c: Text Fit Check

After slot replacement, check each text element against `max_length` from `flexibility.yaml`:
- Fits → proceed
- Too long → shorten/rephrase to fit elegantly, preserving core meaning
- Still too long → move overflow to speaker notes (`<!-- notes -->` block)
- **Never reduce font below minimum**: body ≥ 1.25rem, heading ≥ 2.2rem, label ≥ 0.65rem

### 5d: Element Count Adaptation

When the outline has a different item count than the archetype's slot count, apply `scaling` from `flexibility.yaml`:

- **More items**: `grid-auto` → add columns; `wrap` → allow flex-wrap; `stack` → switch to column layout
- **Fewer items**: merge slots (e.g., 2 items in 3-card grid → 2 equal columns)
- Prefer reducing item count (move to speaker notes) over applying scaling when content allows
- Never leave empty `{{SLOT}}` markers in the output

### 5e: Per-Slide Frontmatter

Every slide must have:

```md
---
layout: none
---

<!-- VERBATIM archetype HTML with slots filled -->

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
</style>
```

Slides are separated by `---`.

---

## PROHIBITED in Step 5

These are **critical errors** — violating any one invalidates the slide:

1. **No HTML rewrite** — never restructure, reorder, or simplify the archetype's HTML skeleton
2. **No inline-to-class replacement** — never convert `style="..."` attributes into Tailwind/UnoCSS classes
3. **No element addition** — never add any element that does not have a corresponding `{{SLOT}}` in the archetype
4. **No Icon injection** — never add `<Icon>` components to slides whose archetype does not contain `<Icon>` tags
5. **No CSS value changes** — never modify font-size, padding, gap, flex ratios, colors, or any CSS property value from the archetype
6. **No bg-alt substitution** — never swap `bg-base` for `bg-alt` or vice versa; this breaks visual hierarchy
7. **No out-of-template visuals** — never add icons, decorative shapes, dividers, gradients, or any visual element absent from the preset's Figma archetypes

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

---

## flexibility.yaml Reference

```yaml
archetype: figmadeck-<name>-<slidename>
content_type: <type>
figma_source:
  nodeId: "<nodeId>"
  name: "<frame name from Figma>"

slots:
  title:
    max_length: <chars>          # estimated from text box width / avg char width
    overflow: shrink-font        # shrink-font | truncate | wrap
  cards:                         # present only if repeated elements detected
    count_in_figma: <N>
    min: <max(2, N-1)>
    max: <N+2>
    scaling: grid-auto           # grid-auto | wrap | stack

layout:
  grid_columns: flexible         # flexible | fixed
  preserve:                      # IMMUTABLE — must match Figma exactly
    - spacing-ratio
    - font-size-hierarchy
    - color-usage
    - border-radius
  flexible:                      # ADAPTABLE — may change to fit content
    - column-count
    - card-count
    - text-length
```

**Preserved properties** are immutable — they come from the Figma source. Changing them is a Step 5 violation.

**Flexible properties** may be adjusted only to the minimum extent required by content — never speculatively.

**Scaling** applies only when item count falls outside `[min, max]`. If within range, use the archetype's column count unchanged.

**When flexibility.yaml is missing**: treat all properties as `preserve`, apply no scaling, shorten text aggressively.
