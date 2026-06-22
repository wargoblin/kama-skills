# Slidev Figma Learning Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add `--figma=<URL>` flag to the slidev skill that extracts a Figma presentation template into a preset with archetypes, and provides a learning mode with live Figma comparison.

**Architecture:** All changes go into SKILL.md (inline, following the existing stitch pattern) and preset-format.md. The Figma procedure is ~370 lines of markdown instructions added after the existing `--deep_learn` handler. No new files created — the skill is a single SKILL.md with references/.

**Tech Stack:** Figma MCP tools (`use_figma`, `get_screenshot`, `get_metadata`, `get_design_context`), existing Slidev skill infrastructure.

**Spec:** `docs/superpowers/specs/2026-03-30-slidev-figma-learning-design.md`

---

### Task 1: Update SKILL.md metadata and help text

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:2-4` (frontmatter)
- Modify: `slidev/.claude/skills/slidev/SKILL.md:35-66` (help text)

- [ ] **Step 1: Update the frontmatter description and argument-hint**

In `slidev/.claude/skills/slidev/SKILL.md`, edit line 3 — append `, --figma` to the description:

```
description: Use when generating a Slidev presentation from a slide outline. Supports preset styles, unique designs, custom style descriptions, and image placement. Scaffolds a complete Slidev project. Also handles --help, --dev, --export, --edit, --picture, --create-preset, --learn, and --figma subcommands.
```

Edit line 4 — add `--figma=<URL>` to the argument-hint:

```
argument-hint: "[--help | --dev [dir] | --export <format> [dir] | --edit [dir] <comment> | --picture [auto|paths...] [dir] | --create-preset <name> | --learn=N | --deep_learn=N | --figma=<URL> | --no-preset | --preset <name> | style: <desc>] <outline or file path>"
```

- [ ] **Step 2: Add --figma lines to the help text block**

In the help text block (inside the ``` code fence, after the `--stitch=learn=N` line around line 63), add these lines:

```
  /slidev --figma=<URL>                         Create preset from Figma template
  /slidev --figma=<URL> --learn=N               Figma learning (create preset + refine with Figma reference)
  /slidev --figma=<URL> --deep_learn=N           Same as --figma --learn (both trigger FDL)
  /slidev --figma=<URL> <outline>               Create preset from Figma + generate presentation
```

- [ ] **Step 3: Verify changes**

Run: `grep -c "figma" slidev/.claude/skills/slidev/SKILL.md`
Expected: at least 4 new occurrences (description, argument-hint, 4 help lines)

- [ ] **Step 4: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add --figma flag to help text and metadata"
```

---

### Task 2: Add --figma subcommand handler to SKILL.md

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:~423` (after `--deep_learn` handler, before `--polish`)

- [ ] **Step 1: Add the --figma subcommand handler**

Insert the following block after the `--deep_learn` handler (after the line "Follow the Preset Deep Learning Procedure (PDL-1 through PDL-7). Stop here — do not proceed to generation.") and before `**\`--polish=N [dir]\`**`:

```markdown
**`--figma=<URL>`**: Create a preset from a Figma presentation template. Optionally run a learning loop with live Figma comparison.

**URL Parsing**: Extract `fileKey` and optional `nodeId` from the Figma URL:
- `https://figma.com/design/:fileKey/:fileName?node-id=:nodeId` → convert `-` to `:` in nodeId
- `https://figma.com/design/:fileKey/branch/:branchKey/:fileName` → use branchKey as fileKey
- If no `node-id` in URL → use `0:1` (first page)
- If URL is not `figma.com/design/...` → error: `--figma only supports Figma Design files (figma.com/design/...)`

**Compatibility checks**:
- `--figma` is **incompatible** with `--stitch`, `--preset`, `--no-preset`, `style:`. If any present → error: `--figma cannot be combined with --stitch/--preset/--no-preset/style:. It creates its own preset.`
- `--figma` is **compatible** with `--learn=N`, `--deep_learn=N`, `<outline>`.

**Dispatch logic**:
1. Always run the **Figma Extraction Procedure (FIG-1 through FIG-5)** first → creates preset + archetypes + reference data.
2. If `--learn=N` or `--deep_learn=N` present → run **Figma Learning Loop (FDL-1 through FDL-5)**. Both flags trigger the same FDL procedure (since fixes never go to SKILL.md, the distinction is irrelevant). Maximum N: 10. Stop after FDL.
3. If `<outline>` present (no --learn) → generate presentation using the created Figma preset. Continue to Generation Modes with the Figma preset loaded.
4. If neither `--learn` nor `<outline>` → generate demo presentation using `assets/demo-outline.md` with the Figma preset, run Visual QA, print summary. Stop.

### Figma Extraction Procedure
```

- [ ] **Step 2: Verify insertion point**

Run: `grep -n "Figma Extraction Procedure" slidev/.claude/skills/slidev/SKILL.md`
Expected: one match at the new insertion point

- [ ] **Step 3: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add --figma subcommand handler and dispatch logic"
```

---

### Task 3: Add FIG-Extract procedure (FIG-1 through FIG-3)

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md` (append after "### Figma Extraction Procedure" heading from Task 2)

- [ ] **Step 1: Add FIG-1 (File Structure Reconnaissance)**

Append directly after the `### Figma Extraction Procedure` heading:

```markdown

**FIG-1: File Structure Reconnaissance** — Use `mcp__figma__use_figma` (Plugin API) to run JavaScript that traverses the node tree on the target page:

```javascript
// Pseudocode for the JS to run via use_figma:
// 1. Get the target page (by nodeId or first page)
// 2. Iterate page.children — find all FRAME nodes
// 3. Filter: abs(width/height - 16/9) < 0.05*16/9 (aspect ratio ~16:9 ±5%)
// 4. Sort by y position (top to bottom) then x (left to right)
// 5. Return: [{id, name, width, height, index}]
```

Result: ordered list of slide frames. If 0 frames found → error: `No slides found (frames with ~16:9 aspect ratio). Check file structure.` If > 20 frames → take first 20, warn: `Found <N> slides, using first 20 (maximum for extraction).`

**FIG-2: Global Style Extraction** — For ALL slide frames, collect design statistics via `mcp__figma__use_figma` (Plugin API). Run a single JS call that traverses all children recursively and collects:

**Colors** — group all `fills` by frequency:
- Frame background fills (direct children fills with full area) → most frequent = `--bg-base`, second = `--bg-alt`
- Small bright-colored elements (used in <20% of nodes) → `--color-accent`
- Text node colors by frequency → most frequent = `--color-text`, second = `--color-muted`
- Derive `--color-accent-rgb` (R,G,B triple), `--bg-accent` (accent at 12% opacity over bg-base)
- **AI Color Blacklist check**: if accent hue 240-290 at saturation >50% → shift to nearest safe hue (teal #0D9488, amber #D97706, emerald #059669, rose #E11D48). Log replacement.

**Fonts** — collect all `fontFamily`, `fontSize`, `fontWeight` from text nodes:
- Heading font: most frequent fontFamily in nodes with fontSize ≥ 24px → `fonts.sans`
- Body font: most frequent fontFamily in nodes with fontSize < 24px → `fonts.body`
- **Serif check**: if either font is serif → find nearest sans-serif (per skill rule: serif banned). Log: `Font "<name>" (serif) → "<replacement>" (sans-serif)`

**Spacing** — collect from auto-layout frames:
- Median `itemSpacing` across all auto-layout frames → standard gap
- Median `paddingLeft`/`paddingTop` → base padding

**Shapes** — collect `cornerRadius`:
- Median cornerRadius of frames with width 100-400px (card-sized) → `card_radius`
- Nodes with cornerRadius ≥ 50% of min(width,height) → `icon_container: circle`
- Nodes with cornerRadius 8-16px → `icon_container: rounded-square`

**Effects** — collect `effects` array:
- Drop shadows → extract offset, blur, color for aesthetic description
- Background blurs → note in aesthetic section

Convert all collected data into `.preset.md` format per `references/preset-format.md` (YAML frontmatter + aesthetic description + CSS block with `:root` variables).

**FIG-3: Per-Slide Extraction** — For each slide frame from FIG-1, extract four levels of data:

**Level 1 — Screenshot (visual reference):**
- Call `mcp__figma__get_design_context(nodeId, fileKey)` → returns screenshot inline + React+Tailwind code
- The screenshot is the visual reference for this slide
- Save React+Tailwind code as code hint for layout understanding

**Level 2 — Structure (positions and sizes):**
- Call `mcp__figma__get_metadata(nodeId, fileKey)` → XML with node IDs, layer types, names, positions, sizes
- Parse the XML into a structured list: `[{name, type, x, y, width, height, children: [...]}]`

**Level 3 — Detailed properties (exact CSS values):**
- Call `mcp__figma__use_figma` with JS that for each child element (recursive, max depth 3) extracts:
  - `fills` (array of paint objects), `strokes`, `effects` (shadow, blur)
  - `fontSize`, `fontFamily`, `fontWeight`, `lineHeight`, `letterSpacing`
  - `layoutMode` (HORIZONTAL/VERTICAL/NONE), `itemSpacing`, `paddingLeft/Right/Top/Bottom`
  - `constraints`, `layoutAlign`, `layoutGrow`, `layoutSizingHorizontal/Vertical`
  - `cornerRadius`, `opacity`
  - `characters` (text content — for determining which elements become slots)
  - `type` (TEXT, FRAME, RECTANGLE, ELLIPSE, etc.)
- Save as `blueprint.json` in the figma-ref directory:

```json
{
  "slideIndex": 1,
  "slideName": "Cover",
  "frameWidth": 1440,
  "frameHeight": 810,
  "elements": [
    {
      "name": "Title",
      "type": "TEXT",
      "x": 120, "y": 280, "width": 1200, "height": 80,
      "fontSize": 48, "fontFamily": "Inter", "fontWeight": 700,
      "lineHeight": 1.1, "letterSpacing": -0.02,
      "fills": [{"type": "SOLID", "color": "#1A1A2E"}],
      "characters": "Presentation Title Here",
      "slot": "TITLE"
    },
    {
      "name": "Card Container",
      "type": "FRAME",
      "x": 120, "y": 400, "width": 1200, "height": 300,
      "layoutMode": "HORIZONTAL", "itemSpacing": 24,
      "paddingLeft": 0, "paddingTop": 0,
      "cornerRadius": 0,
      "children": [
        {
          "name": "Card 1",
          "type": "FRAME",
          "x": 0, "y": 0, "width": 384, "height": 300,
          "cornerRadius": 12,
          "fills": [{"type": "SOLID", "color": "#F5F5F0"}],
          "layoutMode": "VERTICAL", "itemSpacing": 12,
          "paddingLeft": 24, "paddingTop": 24,
          "children": [...]
        }
      ]
    }
  ]
}
```

**Level 4 — Code hint:**
- Already obtained in Level 1 via `get_design_context` — the React+Tailwind code
- Used as supplementary hint for understanding grid structure (flex vs grid, column ratios)
```

- [ ] **Step 2: Verify FIG-1 through FIG-3 are present**

Run: `grep -c "FIG-1\|FIG-2\|FIG-3" slidev/.claude/skills/slidev/SKILL.md`
Expected: at least 3 (one per section heading)

- [ ] **Step 3: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add FIG-1 through FIG-3 (Figma extraction procedure)"
```

---

### Task 4: Add FIG-4 (Archetype Conversion) and FIG-5 (Preset Assembly)

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md` (append after FIG-3 from Task 3)

- [ ] **Step 1: Add FIG-4 and FIG-5**

Append after the FIG-3 section:

```markdown

**FIG-4: Archetype Conversion** — From FIG-3 data, for each slide, build a composition archetype:

**Step 4a: Determine content type** — Analyze elements in the blueprint:
- First slide with one large heading (fontSize ≥ 36px) + subtitle + minimal other content → `intro` (cover)
- Last slide with button-like element or CTA text → `cta`
- Slide with 1-2 very large numbers (fontSize ≥ 48px) + small label → `metric`
- Slide with 3+ similarly-sized child frames in a row/grid → `credentials` or `scope` (cards/grid)
- Slide with circular image containers + text blocks → `team`
- Slide with ordered steps or numbered items → `process`
- Slide with one large heading + one paragraph, minimal decoration → `vision` (section divider)
- Default for unrecognized patterns → `context` (two-col-text)

**Step 4b: Build HTML skeleton** — Convert Figma layout to HTML with `{{SLOT}}` markers:

- **Auto-layout HORIZONTAL** → `display:flex; flex-direction:row; gap:<itemSpacing>px`
- **Auto-layout VERTICAL** → `display:flex; flex-direction:column; gap:<itemSpacing>px`
- **No auto-layout (absolute positioning)** → `position:relative` container with `position:absolute` children
- **Nested frames** → nested `<div>` elements. **Max 3 levels** — if deeper, flatten (merge intermediate frames' styles into parent or child). Log: `Slide <N>: nesting flattened from <X> to 3 levels`
- **Text elements** → `{{SLOT_NAME}}` markers. Naming convention:
  - Single heading → `{{TITLE}}`
  - Single subtitle/paragraph → `{{SUBTITLE}}` or `{{DESCRIPTION}}`
  - Items in repeated grid → `{{CARD_1_TITLE}}`, `{{CARD_1_DESC}}`, `{{CARD_2_TITLE}}`, etc.
  - Large number → `{{METRIC_VALUE}}`
  - Small label above metric → `{{METRIC_LABEL}}`
- **Viewport conversion** — Figma uses arbitrary coords (e.g., 1440x810), Slidev renders at 960x540:
  - Position: `figma_x / figma_frame_width * 100` → percentage
  - Font size: `figma_fontSize / 16` → rem, then apply Font Size Floor: body ≥ 1.25rem, heading ≥ 2.2rem, label ≥ 0.65rem
  - Spacing (gap, padding): `figma_px / 16` → rem
  - Proportions (width ratios between siblings) → preserved as fr units or percentages

Wrap entire skeleton in:
```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;...padding:...">
  <!-- archetype content here -->
</div>
```

All slides use `layout: none` + `.slidev-layout { padding: 0 !important; overflow: hidden; }` (per Rule 25).

**Step 4c: Create flexibility rules** — For each archetype, generate `flexibility.yaml`:

```yaml
archetype: figma-<name>
content_type: <detected type from step 4a>
figma_source:
  nodeId: "<nodeId>"
  name: "<frame name from Figma>"

slots:
  title:
    max_length: <estimated from text box width in Figma: width_px / avg_char_width>
    overflow: shrink-font     # shrink-font | truncate | wrap
  cards:                       # only if repeated elements detected
    count_in_figma: <N>
    min: <max(2, N-1)>
    max: <N+2>
    scaling: grid-auto         # grid-auto (change columns) | wrap (overflow row) | stack (vertical)
  # ... other slots as detected

layout:
  grid_columns: flexible       # flexible (can change col count) | fixed (exact match required)
  preserve:                    # IMMUTABLE — these must match Figma
    - spacing-ratio            # proportions between gaps
    - font-size-hierarchy      # ratio of heading to body size
    - color-usage              # which elements use accent, muted, text colors
    - border-radius            # corner radius values
  flexible:                    # ADAPTABLE — can change to fit content
    - column-count
    - card-count
    - text-length
```

**FIG-5: Preset Assembly** — Assemble all outputs into a complete preset:

**Directory structure:**
```
.slidev-presets/
  figma-<name>.preset.md               # global style + CSS (from FIG-2)
  figma-<name>-theme/                   # Slidev theme directory (generated from CSS block)
    package.json
    styles/index.css
    layouts/none.vue, default.vue
    layouts/cover.vue, section.vue, end.vue
  figma-<name>-figma/                   # Figma reference data
    source.json                         # {fileKey, sourceUrl, nodeIds, extractedAt}
    slide-1-<name>/
      blueprint.json                    # element properties (from FIG-3 Level 3)
      archetype.html                    # HTML skeleton with {{SLOT}} (from FIG-4b)
      flexibility.yaml                  # scaling rules (from FIG-4c)
    slide-2-<name>/
      ...
```

**Preset frontmatter** — add `figma` section:
```yaml
---
name: figma-<name>
figma:
  fileKey: "<fileKey>"
  sourceUrl: "<full Figma URL>"
  extractedAt: "<ISO 8601 timestamp>"
  slideCount: <N>
  archetypes:
    - id: figma-<slide-1-name>
      nodeId: "<nodeId>"
      contentType: <detected type>
    - id: figma-<slide-2-name>
      nodeId: "<nodeId>"
      contentType: <detected type>
    # ... one per slide
# ... rest of standard preset fields (fonts, theme, colorSchema, etc.)
---
```

**Archetypes block:**
```archetypes
preferred: [figma-<slide-1-name>, figma-<slide-2-name>, ...]
avoid: []
cta_style: figma-<cta-slide-name>
cover_style: figma-<cover-slide-name>
```

**Theme directory** — generate from the CSS block (same as `--create-preset` step 4b): `package.json`, `styles/index.css`, layout Vue files.

**source.json:**
```json
{
  "fileKey": "<fileKey>",
  "sourceUrl": "<URL>",
  "extractedAt": "<ISO 8601>",
  "figmaFrameWidth": <original frame width>,
  "figmaFrameHeight": <original frame height>,
  "slideNodes": [
    {"index": 1, "nodeId": "<id>", "name": "<name>", "contentType": "<type>"}
  ]
}
```

Print extraction summary:
```
━━━ Figma Extraction Complete ━━━
Source: <URL>
Slides extracted: <N>
Preset: .slidev-presets/figma-<name>.preset.md

Archetypes:
  1. figma-<name> (intro) — 3 slots, flexible grid
  2. figma-<name> (scope) — 5 slots, 3-col grid
  ...

Fonts: <heading> + <body>
Palette: <bg-base> / <accent> / <text>
```
```

- [ ] **Step 2: Verify FIG-4 and FIG-5 are present**

Run: `grep -c "FIG-4\|FIG-5" slidev/.claude/skills/slidev/SKILL.md`
Expected: at least 2

- [ ] **Step 3: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add FIG-4 and FIG-5 (archetype conversion + preset assembly)"
```

---

### Task 5: Add Figma Learning Loop (FDL-1 through FDL-3)

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md` (append after FIG-5 from Task 4)

- [ ] **Step 1: Add FDL procedure header and FDL-1, FDL-2**

Append after the FIG-5 section:

```markdown

### Figma Learning Loop (FDL)

Triggered when `--figma=<URL>` is combined with `--learn=N` or `--deep_learn=N`. Both flags trigger the same FDL loop (since fixes are always scoped to the preset, not SKILL.md).

**Maximum N**: 10. If N > 10, cap at 10 and notify.

**FDL-1: Initialization** — Preset is already created by FIG-Extract. Create the next sequential `edu_XX/` directory (same as standard `--learn` L-1). Store `fileKey` and the slide `nodeId` list from `source.json` for live queries during comparison.

**FDL-2: Generate N diverse outlines** — Same as L-2: create all outlines upfront. Save each as `<edu_dir>/learn_<i>/outline.md`. **All outlines MUST be in Russian.** Vary across industries, formats, sizes, tone (same diversity rules as standard --learn).

**FDL-3: Learning cycle** — For i = 1 to N:

**FDL-3a: Generate presentation** — Launch a subagent (Agent tool) to run the full `/slidev` pipeline:
- Uses the Figma preset from FIG-5 (re-reads `.preset.md` from disk — it may have been modified by prior cycle's fixes)
- Uses the outline from `<edu_dir>/learn_<i>/outline.md`
- Outputs to `<edu_dir>/learn_<i>/`
- During Composition Planning: match content types to Figma archetypes (see "Generation with Figma Preset" section below)
- During slot filling: apply `flexibility.yaml` rules

**FDL-3b: Export** — After creation:
```bash
cd <edu_dir>/learn_<i> && npm install && npx playwright install chromium
npx slidev export --format png --output slides
```
If export fails, log error in critique.md and skip to FDL-3g. Do NOT abort the entire learning loop.

**FDL-3c: Figma Live Comparison** — For each generated slide that used a Figma archetype, run three comparison levels. Read the archetype's `nodeId` from `source.json`.

**Level 1 — Visual comparison (screenshot vs screenshot):**
- Get fresh Figma screenshot: `mcp__figma__get_screenshot(nodeId, fileKey)` → returns image
- Critic agent sees BOTH: our exported PNG from `slides/<N>.png` + Figma screenshot
- Evaluates: overall visual impression, color correspondence, typography proportions, layout balance

**Level 2 — Structural comparison (positions and sizes):**
- Get fresh metadata: `mcp__figma__get_metadata(nodeId, fileKey)` → XML with positions/sizes
- Parse our `slides.md` — extract CSS values (position, width, height, grid-template, gap, padding)
- Compare with tolerances:
  - Element positions (heading, cards, icons): ±5% of viewport
  - Element proportions (width/height ratio): ±10%
  - Nesting hierarchy (level count, element order): exact match

**Level 3 — Style comparison (exact CSS values):**
- Get fresh properties: `mcp__figma__use_figma` with JS → extract fonts, colors, spacing, radius, effects for the slide's elements
- Compare against our CSS values:

| Property | Figma Source | Our CSS | Tolerance |
|----------|-------------|---------|-----------|
| font-size | `fontSize` (px → rem) | `font-size` | ±0.15rem |
| font-weight | `fontWeight` | `font-weight` | exact |
| color | `fills[0].color` (hex) | `color` / `var(--*)` | ΔE < 5 (CIEDE2000) |
| spacing (gap) | `itemSpacing` (px → rem) | `gap` | ±0.25rem |
| padding | `padding*` (px → rem) | `padding` | ±0.5rem |
| border-radius | `cornerRadius` (px) | `border-radius` | ±2px |
| position | `x, y` (normalized %) | position/margin/grid | ±5% viewport |
```

- [ ] **Step 2: Add FDL-3d (Critic Report) and FDL-3e (Apply Fixes)**

Continue appending:

```markdown

**FDL-3d: Critic Report** — Launch a subagent (Agent tool) as a Figma fidelity critic. Provide all three comparison levels. The critic writes to `<edu_dir>/learn_<i>/critique.md`:

```markdown
# Figma Fidelity Report: Iteration <i>

## Overall Fidelity Score: X/10

## Per-Slide Comparison

### Slide N: <name> (archetype: figma-<name>, nodeId: <id>)
- **Visual fidelity**: X/10 — <brief note>
- **Structural fidelity**: X/10 — <brief note>
- **Style fidelity**: X/10 — <brief note>

Issues:
1. [CRITICAL|MAJOR|MINOR] <description> — Figma: <value>, Ours: <value>, tolerance: <threshold>
   → Fix: <target file> — <exact change description>

### Slide M: <name> ...
(repeat for each slide with a Figma archetype)

## Adaptation Quality
- Slides where flexibility rules worked well: [list]
- Slides where flexibility rules failed: [list + reason]
  → Fix: update flexibility.yaml — <specific change>

## What Matched Well
- [list of archetype parts that matched — do NOT modify these]
```

Critic scoring guide:
- **Visual fidelity**: Does our slide LOOK like the Figma original? Color mood, typography weight, whitespace distribution.
- **Structural fidelity**: Are elements in the right positions? Grid structure, hierarchy depth, element ordering.
- **Style fidelity**: Do exact CSS values match within tolerance? Font sizes, colors, spacing, border-radius.

Overall score = weighted average: Visual 40% + Structural 30% + Style 30%.

**FDL-3e: Apply fixes** — Automatic (same policy as PDL-2.4):
- `critical` severity: always apply
- `major` severity: always apply
- `minor` severity: auto-apply only if fix touches a single CSS property; otherwise defer

Fixes go to:
- **Archetype HTML** (`figma-<name>-figma/slide-<N>/archetype.html`) — layout, positioning, grid structure
- **Preset CSS** (`.preset.md` CSS block) — colors, fonts, spacing variables
- **Flexibility rules** (`figma-<name>-figma/slide-<N>/flexibility.yaml`) — if content adaptation failed

**NEVER** modify SKILL.md, design-principles.md, or any other skill file.

After applying changes to the preset CSS, regenerate the `<name>-theme/` directory from the updated CSS block (same as PDL-2.4).

Log all changes to `<edu_dir>/learn_<i>/changes-applied.md`:
```
# Figma Learning Cycle <i> Changes

## Applied
1. [archetype: figma-cover] gap: 2rem → 1.5rem — severity: major
   Figma value: 24px (1.5rem), was: 2rem

## Skipped
1. [archetype: figma-cards] shadow blur: 8px → 6px — severity: minor, reason: multi-property shadow change
```
```

- [ ] **Step 3: Verify FDL sections are present**

Run: `grep -c "FDL-1\|FDL-2\|FDL-3a\|FDL-3b\|FDL-3c\|FDL-3d\|FDL-3e" slidev/.claude/skills/slidev/SKILL.md`
Expected: 7

- [ ] **Step 4: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add Figma Learning Loop FDL-1 through FDL-3e"
```

---

### Task 6: Add FDL-3f through FDL-5 and edge cases

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md` (append after FDL-3e from Task 5)

- [ ] **Step 1: Add FDL-3f, FDL-3g, FDL-4, FDL-5**

Append after FDL-3e:

```markdown

**FDL-3f: Intermediate report** — Print to user:
```
━━━ Figma Learning: Итерация <i>/<N> ━━━

Figma Fidelity: X/10
  Визуальное совпадение:   X/10
  Структурное совпадение:  X/10
  Стилевое совпадение:     X/10

Правки: <X> applied (critical: <C>, major: <M>), <Y> skipped (minor)
Идеальные слайды: <count>/<total>
Проблемные: <slide names with scores below 7>
```

**FDL-3g: Git commit** — Stage and commit all artifacts for this iteration:
```bash
git add <edu_dir>/learn_<i>/
git add .slidev-presets/figma-<name>.preset.md
git add .slidev-presets/figma-<name>-figma/
git commit -m "learn(slidev-figma): iteration <i>/<N> — fidelity <score>/10"
git tag figma-learn-<edu_dir>-iter-<i>
```

**FDL-4: Convergence detection** — After each cycle's report, check:
- If Fidelity Score ≥ 9/10 for **two consecutive cycles** → stop early:
  ```
  Раннее завершение: fidelity стабилизировался на <score>/10 в циклах <c-1> и <c>.
  ```
  Skip remaining cycles, proceed to FDL-5.
- If score decreased vs previous cycle (regression) → warn but do NOT stop. Next cycle will attempt to fix it.

**FDL-5: Final report** — Write to `<edu_dir>/figma-learning-report.md` and print:
```markdown
# Figma Learning Report

## Source: <Figma URL>
## Cycles: <actual> / <N planned>
## Early stop: yes/no (reason)
## Final Fidelity: X/10

## Fidelity Progression
Cycle 1: ██████░░░░ 6.2
Cycle 2: ████████░░ 7.8
Cycle 3: █████████░ 9.1

## Archetype Changes Summary
- figma-<cover>: <N> fixes (<list>)
- figma-<cards>: <N> fixes (<list>)
...

## Flexibility Rules Updated
- figma-<cards>: max changed 5 → 6, scaling changed grid-auto → wrap
...

## Final Preset State
- Path: .slidev-presets/figma-<name>.preset.md
- Archetypes: <N>
- Theme: .slidev-presets/figma-<name>-theme/
```

Stop here — do not proceed to generation.
```

- [ ] **Step 2: Add edge cases and error handling section**

Append after FDL-5:

```markdown

### Figma Procedure: Edge Cases

| Situation | Behavior |
|-----------|----------|
| URL is not `figma.com/design/...` | Error: `--figma only supports Figma Design files (figma.com/design/...)` Stop. |
| No access to Figma file (API error) | Error: `No access to Figma file. Ensure the file is shared or you are authorized.` Stop. |
| No 16:9 frames found on the page | Error: `No slides found (frames with ~16:9 aspect ratio). Check file structure.` Stop. |
| Frame nesting > 3 levels | Flatten to 3 levels during archetype creation. Log warning. |
| Figma API unavailable during FDL learning cycle | Retry 2× with 10s pause. If still unavailable → use cached blueprint.json and last-known screenshots from `figma-ref/` directory. Log: `⚠ Figma API unavailable, using cached reference for slide <N>` |
| Slide frame contains only image fill (no child elements) | Skip archetype creation for this slide. Log: `Slide <N> "<name>": image-only, no archetype created` |
| Serif font detected in Figma | Replace with nearest sans-serif. Log replacement. |
| Accent color in AI blacklist (hue 240-290, sat >50%) | Shift hue to nearest safe color. Log replacement. |
| > 20 slide frames in file | Take first 20. Warn: `Found <N> slides, using first 20 (maximum for extraction).` |

### Figma Procedure: Limitations (v1)

- **Images not extracted** — background photos, illustrations become placeholders in archetypes. Use `--picture` separately after generation.
- **Figma components not resolved** — we see expanded instances, not component masters.
- **Animations not extracted** — Figma does not store slide transitions. Defaults from preset are used.
- **One file = one preset** — cannot merge slides from multiple Figma files into one preset.
```

- [ ] **Step 3: Verify completeness**

Run: `grep -c "FDL-4\|FDL-5\|Edge Cases\|Limitations" slidev/.claude/skills/slidev/SKILL.md`
Expected: 4

- [ ] **Step 4: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add FDL-4/5 convergence, final report, edge cases"
```

---

### Task 7: Update generation pipeline for Figma archetypes

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:~1661` (Mode: Preset section, after Preset Resolution)

- [ ] **Step 1: Find the Composition Planning section**

The generation pipeline is in SKILL.md under "## Mode: Preset" and subsequent sections. Search for the section that handles archetype selection / composition planning. This is typically in the generation steps (Step 4 or Step 4.5). Read the relevant section to find the exact insertion point.

Run: `grep -n "Composition Planning\|Step 4.5\|archetype.*select\|content.*type.*mapping" slidev/.claude/skills/slidev/SKILL.md | head -10`

- [ ] **Step 2: Add Figma archetype priority rule**

At the beginning of the Composition Planning step (or wherever archetype selection happens), insert this block:

```markdown

**Figma Archetype Priority** — When using a Figma-sourced preset (detected by `figma:` section in preset frontmatter):
1. For each slide, determine content type from outline (as usual)
2. Look for a Figma archetype with matching `contentType` in `figma.archetypes[]`
3. If found → use that Figma archetype. Load its HTML skeleton from `figma-<name>-figma/slide-<N>/archetype.html`.
4. If no matching `contentType` → fallback to standard archetype from `references/composition-archetypes.md`
5. If outline has more slides than Figma archetypes of that type → reuse the same Figma archetype for multiple slides (different content, same layout)

**Slot Filling with Flexibility Rules** — When filling `{{SLOT}}` markers in a Figma archetype:
1. Read `flexibility.yaml` from the archetype's figma-ref directory
2. For **repeating elements** (cards, list items):
   - Content count < `slots.<name>.min` → merge slots or add subtle placeholder
   - Content count within `min..max` → apply `scaling` strategy:
     - `grid-auto`: adjust `grid-template-columns` (e.g., `repeat(3,1fr)` → `repeat(4,1fr)`)
     - `wrap`: keep grid definition, elements flow naturally with `flex-wrap:wrap`
     - `stack`: switch from horizontal to vertical layout
   - Content count > `slots.<name>.max` → truncate, move overflow to speaker notes
3. For **text slots**:
   - Text length > `slots.<name>.max_length` → apply `overflow` strategy:
     - `shrink-font`: reduce font-size by 0.1rem steps (but never below Font Size Floor)
     - `truncate`: cut text with ellipsis
     - `wrap`: allow text wrapping (may increase element height)
4. **Preserved properties** (from `layout.preserve`) are NEVER changed:
   - `spacing-ratio`: gap proportions between elements stay as extracted from Figma
   - `font-size-hierarchy`: ratio of heading-to-body font sizes stays fixed
   - `color-usage`: accent/muted/text color assignment per element stays fixed
   - `border-radius`: corner radius values stay as extracted
```

- [ ] **Step 3: Verify insertion**

Run: `grep -n "Figma Archetype Priority" slidev/.claude/skills/slidev/SKILL.md`
Expected: one match at the inserted location

- [ ] **Step 4: Commit**

```bash
cd slidev && git add .claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add Figma archetype priority in generation pipeline"
```

---

### Task 8: Update preset-format.md with Figma extensions

**Files:**
- Modify: `slidev/.claude/skills/slidev/references/preset-format.md:~112` (after Frontmatter Fields table)

- [ ] **Step 1: Add figma frontmatter fields to the table**

In `references/preset-format.md`, find the Frontmatter Fields table (around line 112). Add these rows to the table:

```markdown
| `figma` | No | Object. Present only for Figma-sourced presets. Contains `fileKey`, `sourceUrl`, `extractedAt`, `slideCount`, `archetypes[]` |
| `figma.fileKey` | — | Figma file key for live API queries during learning |
| `figma.sourceUrl` | — | Full Figma URL for reference |
| `figma.extractedAt` | — | ISO 8601 timestamp of extraction |
| `figma.slideCount` | — | Number of slides extracted |
| `figma.archetypes` | — | Array of `{id, nodeId, contentType}` — one per extracted slide |
```

- [ ] **Step 2: Add Figma Reference Directory section**

After the "Theme Directory" section (around line 217), add:

```markdown

## Figma Reference Directory

Figma-sourced presets include a `<preset-name>-figma/` directory alongside the `.preset.md` file. This directory stores per-slide reference data used during generation and learning.

Structure:
```
<preset-name>-figma/
  source.json                           # Extraction metadata: fileKey, URL, timestamp, slide list
  slide-<N>-<kebab-name>/
    blueprint.json                      # Exact element properties from Figma Plugin API
    archetype.html                      # HTML skeleton with {{SLOT}} markers
    flexibility.yaml                    # Slot scaling rules (min/max, overflow strategy)
```

**blueprint.json** — Contains the element tree with exact CSS-equivalent values: fills, fonts, spacing, effects, positions, sizes. Used by the Figma Learning Loop (FDL) for style comparison.

**archetype.html** — HTML skeleton using `{{SLOT}}` markers for dynamic content. Built from Figma layout data (auto-layout → flex, nested frames → nested divs). Modified by FDL when fixes are applied.

**flexibility.yaml** — Defines how archetypes adapt to different content:
- `slots.<name>.count_in_figma`: original element count in Figma
- `slots.<name>.min / max`: allowed range
- `slots.<name>.scaling`: adaptation strategy (grid-auto, wrap, stack)
- `slots.<name>.max_length`: max text characters before overflow
- `layout.preserve`: immutable properties (spacing-ratio, font-size-hierarchy, color-usage, border-radius)
- `layout.flexible`: adaptable properties (column-count, card-count, text-length)

**source.json** — Metadata for live Figma API queries:
```json
{
  "fileKey": "<key>",
  "sourceUrl": "<URL>",
  "extractedAt": "<ISO 8601>",
  "figmaFrameWidth": 1440,
  "figmaFrameHeight": 810,
  "slideNodes": [
    {"index": 1, "nodeId": "1:2", "name": "Cover", "contentType": "intro"}
  ]
}
```
```

- [ ] **Step 3: Verify**

Run: `grep -c "figma" slidev/.claude/skills/slidev/references/preset-format.md`
Expected: significant increase from baseline (20+ new occurrences)

- [ ] **Step 4: Commit**

```bash
cd slidev && git add .claude/skills/slidev/references/preset-format.md
git commit -m "docs(slidev): add Figma preset extensions to preset-format.md"
```

---

### Task 9: Add reference to Figma procedure in SKILL.md header

**Files:**
- Modify: `slidev/.claude/skills/slidev/SKILL.md:11-28` (References section)

- [ ] **Step 1: Verify no reference file needed**

The Figma procedure is inline in SKILL.md (following the Stitch pattern). No separate reference file to link. However, verify that the References section doesn't need a note about Figma MCP tools.

Since the Figma procedure references Figma MCP tools directly (`mcp__figma__use_figma`, `mcp__figma__get_screenshot`, etc.), no reference file is needed — the MCP tools are available in the environment.

- [ ] **Step 2: Final consistency check**

Verify all cross-references are consistent:

```bash
# Check that all FIG/FDL steps are present
grep -c "^\\*\\*FIG-[0-9]" slidev/.claude/skills/slidev/SKILL.md
# Expected: 5 (FIG-1 through FIG-5)

grep -c "^\\*\\*FDL-[0-9]" slidev/.claude/skills/slidev/SKILL.md
# Expected: 5+ (FDL-1, FDL-2, FDL-3a-g, FDL-4, FDL-5)

# Check figma is in help text
grep "figma" slidev/.claude/skills/slidev/SKILL.md | head -5

# Check preset-format has figma section
grep "figma" slidev/.claude/skills/slidev/references/preset-format.md | head -5

# Check the --figma handler dispatches to FIG and FDL
grep -A2 "Dispatch logic" slidev/.claude/skills/slidev/SKILL.md
```

- [ ] **Step 3: Final commit (if any fixes needed)**

```bash
cd slidev && git add .claude/skills/slidev/
git commit -m "feat(slidev): complete --figma implementation — all procedures and references"
```
