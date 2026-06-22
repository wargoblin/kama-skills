# Figma Extraction Procedure

Extract a Figma presentation template into a preset with archetypes. Called when `/figmadeck <URL>` is used.

All output is namespaced as `figmadeck-<name>-*` — never use `figma-<name>-*`.

---

## FIG-1: File Structure Reconnaissance

Use `mcp__figma__use_figma` (Plugin API) to run JavaScript that traverses the node tree on the target page:

```javascript
// Pseudocode for the JS to run via use_figma:
// 1. Get the target page (by nodeId or first page)
// 2. Iterate page.children — find all FRAME nodes
// 3. Filter: abs(width/height - 16/9) < 0.05 * (16/9)  (aspect ratio ~16:9 ±5%)
// 4. Sort by y position (top to bottom), then x (left to right)
// 5. Return: [{id, name, width, height, index}]
```

**Result**: ordered list of slide frames.

- If 0 frames found → error: `No slides found (frames with ~16:9 aspect ratio). Check file structure.`
- If > 20 frames → take first 20, warn: `Found <N> slides, using first 20 (maximum for extraction).`

---

## FIG-2: Global Style Extraction

For ALL slide frames, collect design statistics via `mcp__figma__use_figma` (Plugin API). Run a single JS call that traverses all children recursively and collects:

### Colors

Group all `fills` by frequency:

- Frame background fills (direct children fills covering full area) → most frequent = `--bg-base`, second = `--bg-alt`
- Small bright-colored elements (used in <20% of nodes) → `--color-accent`
- Text node colors by frequency → most frequent = `--color-text`, second = `--color-muted`
- Derive `--color-accent-rgb` (R,G,B triple), `--bg-accent` (accent at 12% opacity over bg-base)
- **AI Color Blacklist check**: if accent hue 240–290 at saturation >50% → shift to nearest safe hue (teal #0D9488, amber #D97706, emerald #059669, rose #E11D48). Log replacement.

### Fonts

Collect all `fontFamily`, `fontSize`, `fontWeight` from text nodes:

- Heading font: most frequent fontFamily in nodes with fontSize ≥ 24px → `fonts.sans`
- Body font: most frequent fontFamily in nodes with fontSize < 24px → `fonts.body`
- **Serif check**: if either font is serif → find nearest sans-serif equivalent. Log: `Font "<name>" (serif) → "<replacement>" (sans-serif)`

### Spacing

Collect from auto-layout frames:

- Median `itemSpacing` across all auto-layout frames → standard gap
- Median `paddingLeft` / `paddingTop` → base padding

### Shapes

Collect `cornerRadius` values:

- Median cornerRadius of frames with width 100–400px (card-sized) → `card_radius`
- Nodes with cornerRadius ≥ 50% of min(width, height) → `icon_container: circle`
- Nodes with cornerRadius 8–16px → `icon_container: rounded-square`

### Effects

Collect `effects` array:

- Drop shadows → extract offset, blur, color for aesthetic description
- Background blurs → note in aesthetic section

**Output**: convert all collected data into `.preset.md` format per `references/preset-format.md` (YAML frontmatter + aesthetic description + CSS block with `:root` variables).

---

## FIG-3: Per-Slide Extraction

For each slide frame from FIG-1, extract three levels of data using `get_design_context` as the PRIMARY source.

### Level 1 — Code from `get_design_context` (PRIMARY)

Call `mcp__figma__get_design_context(nodeId, fileKey, clientLanguages: "html,css", clientFrameworks: "unknown")` for each slide frame.

Returns: React+Tailwind code + screenshot + contextual metadata (design tokens, annotations, Code Connect mappings).

The React+Tailwind code contains:
- Correct flex/grid from Figma's own auto-layout interpretation
- Correct absolute positioning for fixed-position elements
- Gradient backgrounds as CSS values
- Image references via Figma CDN URLs (`https://figma-*.amazonaws.com/...`)
- Shadow and blur effects as CSS properties
- Auto-layout constraints mapped to flex

Save the code output to `figmadeck-<name>-figma/slide-<N>-<name>/design-context.html`.

### Level 2 — Screenshot (visual reference)

Call `mcp__figma__get_screenshot(nodeId, fileKey)` → download via WebFetch and save to `figmadeck-<name>-figma/slide-<N>-<name>/reference.png`.

This is the pixel-perfect reference used for QA comparison. It is the ground truth for all visual fidelity scoring.

### Level 3 — Metadata (for QA only)

Call `mcp__figma__get_metadata(nodeId, fileKey)` → XML with node IDs, layer types, names, positions, sizes.

Save as `figmadeck-<name>-figma/slide-<N>-<name>/metadata.xml`.

**CRITICAL:** This is used ONLY for QA structural comparison in qa-cycle.md Phase A — NOT for archetype generation. Do not parse or use metadata.xml when building archetypes.

### Asset Download (NEW)

After obtaining Level 1 output, scan `design-context.html` for image URLs:
- Figma CDN pattern: `https://figma-*.amazonaws.com/...`
- Localhost pattern: `http://localhost:...`

For each image URL found:
1. Download via WebFetch
2. Save to `figmadeck-<name>-figma/slide-<N>-<name>/assets/image-<N>.png` (incrementing N per slide)
3. Replace the URL in `design-context.html` with the relative path (e.g., `assets/image-1.png`)

For image fills that appear as background on frames: download and note as `background-image: url(assets/...)` in the archetype during FIG-4 transpilation.

**Note on blueprint.json:** REMOVED as archetype generation source. The `use_figma` Plugin API JS extraction (old Level 3 / Level 4) is no longer used for building archetypes. If detailed property values are needed for QA style comparison, `use_figma` JS calls are made during QA Phase A, not during extraction.

---

## FIG-4: Archetype Conversion

From FIG-3 data, for each slide, build a composition archetype.

### Step 4-pre: Deduplication

Before converting, group slides by layout similarity. Two slides are **duplicates** (merged into one archetype) ONLY if ALL of the following match exactly:

1. Same number of direct children at the top level
2. Same `layoutMode` (HORIZONTAL/VERTICAL/NONE) on the root frame
3. Same grid structure (column/row count, same fr ratios ±5%)
4. Same element type sequence (TEXT, FRAME, FRAME, TEXT vs TEXT, FRAME, TEXT, FRAME = different)
5. Same nesting depth
6. **Same background treatment** — compare the root frame's fills:
   - Both solid color: same color (deltaE < 10 using CIEDE2000) → same
   - Both gradient: same gradient type (linear/radial/angular) AND similar color stops (deltaE < 15 for each stop) → same
   - One solid + one gradient → **DIFFERENT** (always separate archetypes)
   - Dark background (luminance < 30%) vs light background (luminance > 70%) → **ALWAYS DIFFERENT** (they require different text colors, contrast treatment, and overlay behavior)

If ANY criterion differs — structure, layout, element sequence, nesting, OR background — they are **separate archetypes**. Err on the side of keeping more archetypes.

**Naming convention for background variants**: When slides share structure but differ only in background, append a color descriptor to the archetype name:
- `figmadeck-section-teal-gold` (dark gradient with teal + gold stops)
- `figmadeck-section-blue-purple` (dark gradient with blue + purple stops)
- `figmadeck-two-col-light` (light bg-base background)
- `figmadeck-two-col-dark` (dark bg-accent background)

The color descriptor comes from the dominant gradient stop colors or the background luminance category.

Name duplicates: first occurrence becomes the archetype, others reference it via `duplicateOf` in `source.json`.

**Image-only and decoration-only slides** (no text children, only images/shapes/fills): these are NOT skipped. Create archetypes with `{{IMAGE}}` or `{{BACKGROUND}}` slots. Content type: `visual-break`. Slots: `{{IMAGE_URL}}` for the primary image area, optionally `{{CAPTION}}` if small text is present.

### Step 4a: Content Type Detection

Analyze elements in the blueprint using these heuristics:

| Pattern | Content type |
|---------|-------------|
| First slide, large heading (fontSize ≥ 36px) + subtitle + minimal other content | `intro` |
| Last slide with button-like element or CTA text | `cta` |
| 1–2 very large numbers (fontSize ≥ 48px) + small label | `metric` |
| 3+ similarly-sized child frames in a row/grid | `scope` or `credentials` |
| Circular image containers + text blocks | `team` |
| Ordered steps or numbered items | `process` |
| One large heading + one paragraph, minimal decoration | `vision` |
| No text, only images/shapes/fills | `visual-break` |
| Unrecognized pattern | `context` |

### Step 4b: Transpile React+Tailwind → HTML+inline CSS

Take `design-context.html` from FIG-3 Level 1 and post-process it into a static HTML archetype:

**1. Strip React syntax:**
- `<div className="..."` → `<div style="...">` (convert all Tailwind classes to inline style in step 2)
- Remove React-specific attributes: `key`, `ref`, `onClick`
- Convert `className` to `class` where CSS class names are needed (e.g., slot class markers)

**2. Convert Tailwind classes to inline CSS** — replace every utility class with its CSS equivalent as inline `style=""`:

| Tailwind | CSS |
|----------|-----|
| `flex` | `display:flex` |
| `flex-col` | `flex-direction:column` |
| `flex-row` | `flex-direction:row` |
| `items-center` | `align-items:center` |
| `items-start` | `align-items:flex-start` |
| `items-end` | `align-items:flex-end` |
| `justify-between` | `justify-content:space-between` |
| `justify-center` | `justify-content:center` |
| `justify-end` | `justify-content:flex-end` |
| `gap-<N>` | `gap:<N*0.25>rem` |
| `gap-[Xpx]` | `gap:Xpx` |
| `w-full` | `width:100%` |
| `w-[X%]` | `width:X%` |
| `w-[Xpx]` | `width:Xpx` |
| `h-full` | `height:100%` |
| `h-[Xpx]` | `height:Xpx` |
| `flex-1` | `flex:1` |
| `p-<N>` | `padding:<N*0.25>rem` |
| `px-<N>` | `padding-left:<N*0.25>rem;padding-right:<N*0.25>rem` |
| `py-<N>` | `padding-top:<N*0.25>rem;padding-bottom:<N*0.25>rem` |
| `pt-<N>` | `padding-top:<N*0.25>rem` |
| `pb-<N>` | `padding-bottom:<N*0.25>rem` |
| `mt-<N>` | `margin-top:<N*0.25>rem` |
| `mb-<N>` | `margin-bottom:<N*0.25>rem` |
| `text-xl` | `font-size:1.25rem` |
| `text-2xl` | `font-size:1.5rem` |
| `text-3xl` | `font-size:1.875rem` |
| `text-4xl` | `font-size:2.25rem` |
| `text-[Xpx]` | `font-size:Xpx` |
| `font-bold` | `font-weight:700` |
| `font-semibold` | `font-weight:600` |
| `font-medium` | `font-weight:500` |
| `leading-tight` | `line-height:1.25` |
| `leading-snug` | `line-height:1.375` |
| `leading-normal` | `line-height:1.5` |
| `leading-[X]` | `line-height:X` |
| `tracking-wide` | `letter-spacing:0.025em` |
| `tracking-wider` | `letter-spacing:0.05em` |
| `tracking-[X]` | `letter-spacing:X` |
| `text-[#XXXXXX]` | `color:#XXXXXX` (then step 3 converts to var) |
| `bg-[#XXXXXX]` | `background:#XXXXXX` (then step 3 converts to var) |
| `rounded-xl` | `border-radius:0.75rem` |
| `rounded-2xl` | `border-radius:1rem` |
| `rounded-full` | `border-radius:9999px` |
| `rounded-[Xpx]` | `border-radius:Xpx` |
| `border` | `border:1px solid` |
| `absolute` | `position:absolute` |
| `relative` | `position:relative` |
| `inset-0` | `inset:0` |
| `z-<N>` | `z-index:N` |
| `overflow-hidden` | `overflow:hidden` |
| `object-cover` | `object-fit:cover` |
| `object-contain` | `object-fit:contain` |

For any Tailwind class not in this table, derive its CSS equivalent from its name. When uncertain, log: `Slide <N>: unmapped class "<class>" — inlined as best-effort`.

**3. Replace hardcoded hex with CSS variables** — scan all `color`, `background`, `background-color`, `border-color`, `fill`, `stroke` values:
- Match each hex/rgb value to the preset's `var(--*)` by color proximity (deltaE < 3 using CIEDE2000)
- `#1a1a2e` ≈ `var(--bg-accent)`, `#f1f1f1` ≈ `var(--bg-base)`, `#ffe614` ≈ `var(--color-accent)`
- For gradient stops: replace each stop's color if it matches a preset variable
- Leave `rgba(...)` transparency values intact — only replace the color portion if deltaE < 3
- If no preset variable matches within deltaE 3 → keep the original hex (it may be a one-off gradient stop or decoration)

**4. Replace image URLs with local paths** — Figma CDN URLs (`https://figma-*.amazonaws.com/...`) → relative `assets/image-<N>.png` paths (already downloaded in FIG-3 Asset Download step). For `{{IMAGE}}` slots in visual-break archetypes: keep as slot marker instead of an asset path.

**CRITICAL — Font Size Floor exception:** During FIG-4 transpilation, do NOT apply Font Size Floor (body ≥ 1.25rem, heading ≥ 2.2rem, label ≥ 0.65rem). Preserve exact Figma values. The Font Size Floor applies ONLY during content generation (learn_1..N and regular generation), never during archetype extraction.

Wrap the entire transpiled output in:

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:...">
  <!-- archetype content here -->
</div>
```

All slides use `layout: none` + `.slidev-layout { padding: 0 !important; overflow: hidden; }`.

### Step 4c: Insert {{SLOT}} markers

Identify all text content in the transpiled HTML and replace with slot markers:

1. Find all text nodes: content of `<h1>`, `<h2>`, `<h3>`, `<p>`, `<span>`, `<div>` elements that contain only readable text (not structural containers)
2. Replace with `{{SLOT_NAME}}`:
   - First large heading → `{{TITLE}}`
   - Subtitle or description paragraph → `{{SUBTITLE}}` or `{{DESCRIPTION}}`
   - Repeated items in grid/flex row → `{{CARD_1_TITLE}}`, `{{CARD_1_DESC}}`, `{{CARD_2_TITLE}}`, `{{CARD_2_DESC}}`, etc.
   - Large standalone number → `{{METRIC_VALUE}}`
   - Small label above/below metric → `{{METRIC_LABEL}}`
   - Footer breadcrumb / section indicator → `{{BREADCRUMB}}`, `{{SECTION}}`, `{{PAGE_NUM}}`
   - Primary image area → `{{IMAGE_URL}}`
   - Optional caption → `{{CAPTION}}`
3. Record the original Figma text for each slot in `flexibility.yaml` as `original_text` (used during learn_0 QA to validate slot sizing and overflow behavior)

### Step 4d: CSS rendering fixes

Apply these corrections to every archetype during transpilation:

- **Letter-spacing**: if Figma value is a percentage → convert to `em` (e.g., `2%` → `letter-spacing: 0.02em`). If in `px` → convert: `letterSpacing_px / fontSize_px` → `em`
- **Line-height**: always use unitless ratio — `lineHeight_px / fontSize_px` → e.g., `line-height: 1.43`
- **Coordinates**: `Math.round()` all `top`, `left`, `width`, `height`, `gap`, `padding` values to integers to avoid sub-pixel rendering glitches
- **Box-shadow**: if Figma shadow specifies `blur: Xpx` → CSS `box-shadow` blur = `X * 2`px (Figma's blur is half of CSS blur)
- **Font feature settings**: preserve any `font-feature-settings` property returned by `get_design_context` — do not strip it

### Step 4e: Deduplication

Same 6 criteria as Step 4-pre — applied again after transpilation to confirm deduplication decisions were correct. If transpiled output reveals structural differences not caught in Step 4-pre, create separate archetypes and update `source.json`.

### Step 4f: Flexibility Rules

For each archetype, generate a `flexibility.yaml` stored alongside the archetype:

```yaml
archetype: figmadeck-<name>
content_type: <detected type from step 4a>
figma_source:
  nodeId: "<nodeId>"
  name: "<frame name from Figma>"

slots:
  title:
    original_text: "<verbatim text from Figma>"
    max_length: <estimated from text box width: width_px / avg_char_width>
    overflow: shrink-font     # shrink-font | truncate | wrap
  description:
    original_text: "<verbatim text from Figma>"
    max_length: <estimated>
    overflow: wrap
  cards:                       # only if repeated elements detected
    count_in_figma: <N>
    min: <max(2, N-1)>
    max: <N+2>
    scaling: grid-auto         # grid-auto (change columns) | wrap (overflow row) | stack (vertical)
    items:
      - slot: card_1_title
        original_text: "<verbatim text from Figma>"
      - slot: card_1_desc
        original_text: "<verbatim text from Figma>"

layout:
  grid_columns: flexible       # flexible (can change col count) | fixed (exact match required)
  preserve:                    # IMMUTABLE — must match Figma
    - spacing-ratio
    - font-size-hierarchy
    - color-usage
    - border-radius
  flexible:                    # ADAPTABLE — can change to fit content
    - column-count
    - card-count
    - text-length
```

---

## FIG-5: Preset Assembly

Assemble all outputs into a complete preset.

### Directory Structure

```
.slidev-presets/
  figmadeck-<name>.preset.md               # global style + CSS (from FIG-2)
  figmadeck-<name>-theme/                   # Slidev theme directory
    package.json
    styles/index.css
    layouts/none.vue
    layouts/default.vue
    layouts/cover.vue
    layouts/section.vue
    layouts/end.vue
  figmadeck-<name>-figma/                   # Figma reference data
    source.json                             # file metadata + node index
    slide-1-<name>/
      design-context.html                   # React+Tailwind code from get_design_context (FIG-3 Level 1)
      reference.png                         # pixel-perfect screenshot (FIG-3 Level 2)
      metadata.xml                          # structural XML for QA only (FIG-3 Level 3)
      assets/                               # downloaded image assets (FIG-3 Asset Download)
      archetype.html                        # transpiled HTML+inline CSS with {{SLOT}} (FIG-4b/4c)
      flexibility.yaml                      # scaling rules + original_text (FIG-4f)
    slide-2-<name>/
      design-context.html
      reference.png
      metadata.xml
      assets/
      archetype.html
      flexibility.yaml
    ...
```

### Preset Frontmatter

Add `figma` section to the standard preset YAML:

```yaml
---
name: figmadeck-<name>
figma:
  fileKey: "<fileKey>"
  sourceUrl: "<full Figma URL>"
  extractedAt: "<ISO 8601 timestamp>"
  slideCount: <N>
  archetypes:
    - id: figmadeck-<slide-1-name>
      nodeId: "<nodeId>"
      contentType: <detected type>
    - id: figmadeck-<slide-2-name>
      nodeId: "<nodeId>"
      contentType: <detected type>
# ... rest of standard preset fields (fonts, theme, colorSchema, accentColor, etc.)
---
```

### Archetypes Block

```archetypes
preferred: [figmadeck-<slide-1-name>, figmadeck-<slide-2-name>, ...]
avoid: []
cta_style: figmadeck-<cta-slide-name>
cover_style: figmadeck-<cover-slide-name>
```

### Theme Directory

Generate from the CSS block (same as `--create-preset` step 4b): `package.json`, `styles/index.css`, layout Vue files (`none.vue`, `default.vue`, `cover.vue`, `section.vue`, `end.vue`).

Every generated `styles/index.css` **MUST** include these rendering alignment rules:

```css
/* Figma rendering alignment */
.slidev-layout * {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: geometricPrecision;
}

/* Consistent box model */
*, *::before, *::after {
  box-sizing: border-box;
}
```

These rules ensure that text rendering and element sizing in Slidev match Figma's rendering as closely as possible.

### source.json

```json
{
  "fileKey": "<key>",
  "sourceUrl": "<URL>",
  "extractedAt": "<ISO 8601>",
  "figmaFrameWidth": 1440,
  "figmaFrameHeight": 810,
  "slideNodes": [
    {"index": 1, "nodeId": "1:2", "name": "Cover", "contentType": "intro"},
    {"index": 2, "nodeId": "1:3", "name": "Scope", "contentType": "scope"},
    {"index": 3, "nodeId": "1:4", "name": "Metric", "contentType": "metric"}
  ]
}
```

### Extraction Summary

Print after completing FIG-5:

```
━━━ Figma Extraction Complete ━━━
Source: <URL>
Slides extracted: <N>
Preset: .slidev-presets/figmadeck-<name>.preset.md

Archetypes:
  1. figmadeck-<name>-cover (intro) — 3 slots, absolute layout
  2. figmadeck-<name>-scope (scope) — 5 slots, 3-col grid
  3. figmadeck-<name>-metric (metric) — 2 slots, centered
  ...

Fonts: <heading font> + <body font>
Palette: <bg-base> / <accent> / <text>
```

---

## Error Handling

| Situation | Behavior |
|-----------|----------|
| URL not `figma.com/design/` | Error, stop |
| No file access | Error, stop |
| No 16:9 frames found | Error: `No slides found (frames with ~16:9 aspect ratio). Check file structure.` |
| > 20 slides | Take first 20, warn user |
| Nesting > 3 levels | Flatten during extraction, log |
| Serif font detected | Replace with sans-serif, log |
| Accent hue in blacklist (240–290, sat >50%) | Shift hue, log replacement |
| Figma API unavailable during later learning cycles | Use cached `design-context.html` + `reference.png` from `figmadeck-<name>-figma/` |
