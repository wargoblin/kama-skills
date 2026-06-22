# Slidev Figma Learning Design

## Overview

Extend the slidev skill with `--figma=<URL>` flag that:
1. Extracts a complete presentation template from a Figma design file into a preset with archetypes
2. Provides a new learning mode where generated slides are compared against the live Figma reference (visual + structural + style comparison)
3. Fixes are applied to archetypes and preset CSS only — SKILL.md and design-principles.md are never modified

## Non-Goals

- Replacing Slidev with raw Figma export (Slidev provides navigation, transitions, export, speaker notes, hot reload)
- Modifying core skill rules (experiment is isolated to preset)
- Extracting images/illustrations from Figma (placeholder only)
- Extracting animations/transitions from Figma

---

## Section 1: Flag Parsing & Entry Points

### Syntax

```
/slidev --figma=<URL>                        Create preset from Figma template
/slidev --figma=<URL> --learn=N              Create preset + learning with Live Figma reference
/slidev --figma=<URL> --deep_learn=N         Create preset + deep learn with Live Figma reference
/slidev --figma=<URL> <outline>              Create preset + generate presentation immediately
```

URL is a Figma design file link: `https://figma.com/design/:fileKey/:fileName?node-id=...`

If `node-id` is specified — take that specific page/frame. If not — take the first page of the file.

### URL Parsing

Extract `fileKey` and optional `nodeId` (converting `-` to `:`):
- `https://figma.com/design/abc123/MyPres?node-id=0-1` → `fileKey=abc123`, `nodeId=0:1`
- `https://figma.com/design/abc123/branch/br456/MyPres` → `fileKey=br456` (branch)

### Compatibility

- `--figma` is **incompatible** with `--stitch`, `--preset`, `--no-preset`, `style:` (conflicting style sources)
- `--figma` is **compatible** with `--learn=N`, `--deep_learn=N`, `<outline>` (additional actions after preset creation)

### Behavioral Note: `--learn` vs `--deep_learn` with `--figma`

Since `--figma` learning fixes are **always** scoped to the preset (never SKILL.md), both `--learn=N` and `--deep_learn=N` trigger the **same** Figma Learning Loop (FDL) described in Section 3. The distinction between the two flags is irrelevant in `--figma` context.

### `--figma=<URL>` alone (no `--learn`, no outline)

When `--figma` is used without `--learn`/`--deep_learn`/`<outline>`:
1. Run FIG-Extract (Section 2) — create preset + archetypes + reference data
2. Generate demo presentation using `assets/demo-outline.md` with the new preset
3. Run Visual QA on the demo
4. Print summary: preset path, demo path, how to preview, how to use
5. Stop. No learning loop.

---

## Section 2: Figma Extraction Phase (FIG-Extract)

### FIG-1: File Structure Reconnaissance

Use `use_figma` (Plugin API) to run JS that traverses the node tree:
- Find all top-level frames on the target page
- Filter by aspect ratio ~16:9 (+-5%) — these are slides
- Return: `[{nodeId, name, width, height, index}]`

Result: ordered list of slide frames with their `nodeId` and names.

### FIG-2: Global Style Extraction

For ALL slides, collect statistics via `use_figma` (Plugin API):

**Colors** — traverse fills of all elements, group:
- Most frequent background → `--bg-base`
- Second background → `--bg-alt`
- Accent color (bright, used in <20% of elements) → `--color-accent`
- Primary text color → `--color-text`
- Secondary text color → `--color-muted`

**Fonts** — collect all fontFamily + fontSize + fontWeight:
- Most frequent heading font → `fonts.sans`
- Most frequent body font → `fonts.body`
- If serif — find nearest sans-serif (skill rule: serif banned)

**Spacing** — collect auto-layout `itemSpacing` and `padding`:
- Median padding → base spacing
- Median gap → standard gap

**Shapes** — analyze `cornerRadius`:
- Median for cards → `card_radius`
- Circular elements (radius >= 50%) → `icon_container: circle`

**Effects** — collect shadows, blurs:
- Characteristic shadows → description in aesthetic section of preset

Convert to `.preset.md` with standard format (YAML frontmatter + CSS block).

### FIG-3: Per-Slide Extraction (three levels of data)

For each slide frame:

**Level 1 — Screenshot (visual reference):**
- `get_screenshot(nodeId, fileKey)` → returns screenshot URL
- Download the image via `WebFetch` or Playwright and save as `figma-ref/slide-<N>-<name>/reference.png`
- Alternative: use `get_design_context` which returns a screenshot inline (can be viewed by the agent directly)

**Level 2 — Structure (positions and sizes):**
- `get_metadata(nodeId, fileKey)` → XML with positions, layer types, names
- Parse into structured data: which elements, where, what size

**Level 3 — Detailed properties (exact CSS values):**
- `use_figma` with JS code that for each child element extracts:
  - `fills`, `strokes`, `effects` (shadow, blur)
  - `fontSize`, `fontFamily`, `fontWeight`, `lineHeight`, `letterSpacing`
  - `layoutMode`, `itemSpacing`, `paddingLeft/Right/Top/Bottom`
  - `constraints`, `layoutAlign`, `layoutGrow`
  - `cornerRadius`, `opacity`
  - `characters` (text content — for slot determination)
- Save as `figma-ref/slide-<N>-<name>/blueprint.json`

**Level 4 — Code hint:**
- `get_design_context(nodeId, fileKey)` → React+Tailwind code
- Used as supplementary hint for grid structure understanding

### FIG-4: Archetype Conversion

From FIG-3 data, for each slide:

1. **Determine content type** — analyze elements:
   - Single large heading + little text → `cover` or `section`
   - Large number + caption → `metric` / `stat-hero`
   - Multiple identical blocks in grid → `cards` / `bento-grid`
   - People photos + text → `team` / `profile-grid`
   - Last slide with button/CTA → `cta`

2. **Build HTML skeleton** — from Figma layout data:
   - Auto-layout horizontal → `display:flex; flex-direction:row`
   - Auto-layout vertical → `display:flex; flex-direction:column`
   - Nested frames → nested divs (max 3 levels — Slidev rule)
   - Element sizes → CSS sizes (conversion from Figma coords to 960x540 Slidev viewport)
   - Text elements → `{{SLOT}}` markers (e.g. `{{TITLE}}`, `{{CARD_1_TITLE}}`, `{{METRIC_VALUE}}`)

3. **Create flexibility rules** — `flexibility.yaml`:

```yaml
archetype: figma-content-grid
content_type: scope
figma_source:
  nodeId: "3:45"
  name: "Content Grid"

slots:
  title:
    max_length: 45           # estimated from text box width in Figma
    overflow: shrink-font     # shrink-font | truncate | wrap
  cards:
    count_in_figma: 3
    min: 2
    max: 5
    scaling: grid-auto        # grid-auto | wrap | stack
  subtitle:
    max_length: 80
    overflow: wrap
  icon:
    required: true

layout:
  grid_columns: flexible      # flexible | fixed
  preserve:                   # IMMUTABLE
    - spacing-ratio
    - font-size-hierarchy
    - color-usage
    - border-radius
  flexible:                   # ADAPTABLE
    - column-count
    - card-count
    - text-length
```

### FIG-5: Preset Assembly

Final structure:

```
.slidev-presets/
  figma-<name>.preset.md              # global style + CSS
  figma-<name>-theme/                 # Slidev theme directory
    package.json
    styles/index.css
    layouts/none.vue, cover.vue, ...
  figma-<name>-figma/                 # Figma reference data
    source.json                       # {fileKey, sourceUrl, extractedAt}
    slide-1-cover/
      reference.png                   # screenshot from Figma
      blueprint.json                  # exact element properties
      archetype.html                  # HTML skeleton with {{SLOT}}
      flexibility.yaml                # scaling rules
    slide-2-content-grid/
      ...
```

Preset frontmatter gets a new `figma` section:

```yaml
figma:
  fileKey: "abc123"
  sourceUrl: "https://figma.com/design/abc123/..."
  extractedAt: "2026-03-30T12:00:00Z"
  slideCount: 10
  archetypes:
    - id: figma-cover
      nodeId: "1:2"
      contentType: intro
    - id: figma-content-grid
      nodeId: "3:45"
      contentType: scope
    ...
```

And in the `archetypes` block:
```archetypes
preferred: [figma-cover, figma-content-grid, figma-stat-hero, figma-team, figma-cta]
avoid: []
cta_style: figma-cta
cover_style: figma-cover
```

---

## Section 3: Figma Learning Loop (FDL)

### FDL-1: Initialization

- Preset already created during FIG-Extract phase
- Create `edu_XX/` directory (same as standard `--learn`)
- Store `fileKey` and slide `nodeId` list for live queries

### FDL-2: Outline Generation

Identical to current `--learn` (L-2): N diverse outlines in Russian, varied industries/formats/sizes. Save to `edu_XX/learn_<i>/outline.md`.

### FDL-3: Learning Cycle (i = 1 to N)

#### FDL-3a: Generate Presentation

Subagent runs standard `/slidev` pipeline with Figma preset. During generation:
- Composition Planning matches content types to Figma archetypes
- Fills `{{SLOT}}` markers with outline data
- Applies `flexibility.yaml`: adapts grid if element count differs from Figma
- Output: `edu_XX/learn_<i>/slides.md` + `styles/index.css`

#### FDL-3b: Export

```bash
npx slidev export --format png --output slides
```
Result: PNG of each slide in `edu_XX/learn_<i>/slides/`

#### FDL-3c: Figma Live Comparison (NEW)

For each generated slide that has a Figma archetype, run **three comparison levels**:

**Level 1 — Visual comparison (screenshot vs screenshot):**
- Get fresh screenshot from Figma: `get_screenshot(nodeId, fileKey)`
- Critic agent sees BOTH images: our exported PNG + Figma PNG
- Evaluates: overall impression, color match, typography, proportions

**Level 2 — Structural comparison (element positions):**
- Get fresh metadata: `get_metadata(nodeId, fileKey)` → positions/sizes
- Parse our `slides.md` — extract CSS values (position, width, height, grid-template, gap)
- Compare:
  - Key element positions (heading, cards, icons) — tolerance +-5% of viewport
  - Element proportions (width/height ratio) — tolerance +-10%
  - Nesting hierarchy (level count, element order)

**Level 3 — Style comparison (exact CSS values):**
- Get fresh properties: `use_figma` with JS → fonts, colors, spacing, radius, effects
- Compare with our CSS values:

| Property | Figma Source | Our CSS | Tolerance |
|----------|-------------|---------|-----------|
| font-size | `fontSize` (px → rem) | `font-size` in style/scoped | +-0.15rem |
| font-weight | `fontWeight` | `font-weight` | exact |
| color | `fills[0].color` (hex) | `color` / `var(--*)` | deltaE < 5 (CIEDE2000) |
| spacing (gap) | `itemSpacing` (px → rem) | `gap` | +-0.25rem |
| padding | `padding*` (px → rem) | `padding` | +-0.5rem |
| border-radius | `cornerRadius` (px) | `border-radius` | +-2px |
| position | `x, y` (normalized %) | position/margin/grid | +-5% viewport |

#### FDL-3d: Critic Report

Critic agent receives all three levels and writes to `edu_XX/learn_<i>/critique.md`:

```markdown
# Figma Fidelity Report: Iteration <i>

## Overall Fidelity Score: X/10

## Per-Slide Comparison

### Slide 1: Cover (archetype: figma-cover, nodeId: 1:2)
- **Visual fidelity**: 8/10
- **Structural fidelity**: 7/10
- **Style fidelity**: 9/10

Issues:
1. [MAJOR] gap between cards 24px (Figma) vs 32px (ours) — exceeds tolerance
   -> Fix: archetype figma-cover, line 5, gap: 24px -> 1.5rem
2. [MINOR] heading line-height 1.1 (Figma) vs 1.2 (ours)
   -> Fix: archetype figma-cover, h1 style, line-height: 1.1

### Slide 2: Content Grid (archetype: figma-content-grid, nodeId: 3:45)
...

## Adaptation Quality
- Slides where content adapted well: [list]
- Slides where flexibility rules failed: [list + reason]
  -> Fix: update flexibility.yaml

## What Matched Well
- [list of things that matched — preserve these archetype parts]
```

#### FDL-3e: Apply Fixes

Automatic (same policy as current PDL-2.4):
- `critical` and `major` — always apply
- `minor` — apply if fix touches a single CSS property

Fixes go to:
- **Archetype HTML** (`archetype.html`) — layout, positioning, grid
- **Preset CSS** (`.preset.md` CSS block) — colors, fonts, spacing
- **Flexibility rules** (`flexibility.yaml`) — if content adaptation failed

**Never** to SKILL.md or design-principles.md.

#### FDL-3f: Intermediate Report

```
--- Figma Learning: Iteration <i>/<N> ---

Figma Fidelity: X/10
  Visual match:      X/10
  Structural match:  X/10
  Style match:       X/10

Fixes applied: X critical, Y major, Z minor
Perfect match slides: [count]/[total]
Problem slides: [list with reasons]
```

#### FDL-3g: Git Commit

Commit after each iteration (same as current `--learn`).

### FDL-4: Convergence

Early stop if Fidelity Score >= 9/10 for **two consecutive cycles**.

### FDL-5: Final Report

```markdown
# Figma Learning Report

## Source: <figma URL>
## Cycles: <actual>/<N>
## Final Fidelity: X/10

## Fidelity Progression
Cycle 1: 6.2/10
Cycle 2: 7.8/10
Cycle 3: 9.1/10

## Archetype Changes Summary
- figma-cover: 3 fixes (gap, line-height, padding)
- figma-content-grid: 5 fixes (column widths, card radius, ...)
- figma-cta: 1 fix (button color)

## Flexibility Rules Updated: 2 archetypes
```

---

## Section 4: Generation with Figma Preset

### Composition Planning Changes

With Figma preset, Figma archetypes have **priority** over standard ones:

1. Determine slide content type from outline (as usual)
2. Look for Figma archetype with matching `contentType` in `figma.archetypes[]`
3. If found — use it. If not — fallback to standard archetype from `composition-archetypes.md`
4. If outline has 12 slides but only 8 Figma templates — use closest matching Figma archetype or standard fallback for "extra" slides

### Slot Filling with Flexibility Rules

When filling `{{SLOT}}` markers:

1. Read `flexibility.yaml` for the chosen archetype
2. Check content element count vs `slots.cards.count_in_figma`:
   - Elements **below** min → add empty placeholders or merge slots
   - Elements **in range** min..max → apply `scaling` strategy:
     - `grid-auto`: change `grid-template-columns` (e.g. `repeat(3, 1fr)` → `repeat(4, 1fr)`)
     - `wrap`: keep grid, elements wrap to next row
     - `stack`: change grid to vertical stack
   - Elements **above** max → truncate, overflow to speaker notes
3. Check text length vs `slots.title.max_length`:
   - If longer → apply `overflow` strategy (`shrink-font`, `truncate`, `wrap`)
4. Values from `layout.preserve` are **never changed** — spacing-ratio, font-hierarchy, color-usage, border-radius stay as in Figma

### Viewport Conversion

Figma uses arbitrary canvas coordinates (typically 1440x810 or 1920x1080), Slidev renders at **960x540**.

Conversion during archetype creation:
- `figma_px / figma_frame_width * 960` → Slidev px (for position)
- `figma_px / 16` → rem (for font-size, padding, gap)
- Apply Font Size Floor from existing rules: body >= 1.25rem, heading >= 2.2rem, label >= 0.65rem
- Proportions (%) preserved without change

---

## Section 5: Edge Cases & Limitations

### Error Handling

| Situation | Behavior |
|-----------|----------|
| URL is not Figma design file | Error: `--figma only supports Figma Design files (figma.com/design/...)` |
| No access to file | Error: `No access to Figma file. Ensure file is shared or you are authorized.` |
| No 16:9 frames found | Error: `No slides found (frames with ~16:9 aspect ratio). Check file structure.` |
| Frame too complex (>3 nesting levels) | Flatten to 3 levels during archetype creation. Log warning. |
| Figma API unavailable during learning | Retry 2x with 10s pause. If still unavailable — fallback to cached data from `figma-ref/`. Log warning. |
| Slide contains only image (no structure) | Skip archetype creation. Save screenshot only as visual reference. |
| Serif font in Figma | Replace with nearest sans-serif (skill rule). Log replacement. |
| Accent color in AI blacklist (hue 240-290, sat >50%) | Shift hue to nearest safe color. Log replacement. |

### First Version Limitations

- **Images not extracted** from Figma — background photos, illustrations become placeholders. Use `--picture` separately.
- **Figma components not resolved** — we see expanded instances, not masters.
- **Animations not extracted** — Figma doesn't store transitions. Defaults from preset used.
- **Maximum 20 slides** — more = too many API calls. If >20 frames, take first 20 and warn.
