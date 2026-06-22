# Preset Format Specification

Presets are `.preset.md` files that define reusable design configurations for Slidev presentations.

## File Structure

A preset file consists of YAML frontmatter followed by an optional body.

```markdown
---
name: preset-name
theme: default
colorSchema: dark
fonts:
  sans: Space Grotesk
  serif: Playfair Display
  mono: JetBrains Mono
aspectRatio: 16/9
transition: slide-left
accentColor: "#ff6b35"
---

Free-text aesthetic description guiding design decisions.
Describe the mood, visual metaphors, textures, and overall feel.

```css
:root {
  --slidev-theme-primary: #ff6b35;
  --slidev-theme-background: #1a1a2e;
  --slidev-theme-code-background: #16213e;
}

.slidev-layout {
  /* Base layout overrides (background, color, texture) */
}

/* === LAYOUT-SPECIFIC STYLES (REQUIRED) === */

/* Cover layout — centered text, overlay-ready */
.slidev-layout.cover {
  text-align: center;
}
.slidev-layout.cover h1,
.slidev-layout.cover p {
  text-align: center;
}

/* Section layout — centered text, overlay-ready */
.slidev-layout.section {
  text-align: center;
}
.slidev-layout.section h1,
.slidev-layout.section p {
  text-align: center;
}

/* Fact layout — centered stat */
.slidev-layout.fact {
  text-align: center;
}
.slidev-layout.fact h1,
.slidev-layout.fact p {
  text-align: center;
}

/* End layout — centered closing */
.slidev-layout.end {
  text-align: center;
}
.slidev-layout.end h1,
.slidev-layout.end p,
.slidev-layout.end div {
  text-align: center;
}

/* Statement / Center layouts */
.slidev-layout.statement h1,
.slidev-layout.statement p,
.slidev-layout.center h1,
.slidev-layout.center p {
  text-align: center;
}

/* Background image overlay support */
.slidev-layout.cover,
.slidev-layout.section {
  position: relative;
}
.slidev-layout.cover::before,
.slidev-layout.section::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  z-index: 0;
  pointer-events: none;
}
.slidev-layout.cover > *,
.slidev-layout.section > * {
  position: relative;
  z-index: 1;
}

/* Hide overlay when no background image is used (per-slide <style> overrides this) */
.slidev-layout.cover:not([style*="background"]) .slidev-layout.cover::before,
.slidev-layout.section:not([style*="background"]) .slidev-layout.section::before {
  display: none;
}
```
```

## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique preset identifier (kebab-case) |
| `theme` | No | Slidev theme package name. Default: `default` |
| `colorSchema` | No | `light`, `dark`, or `auto`. Default: `auto` |
| `fonts.sans` | No | Primary sans-serif font (Google Fonts) |
| `fonts.body` | No | Body font family (sans-serif). **Renamed from `fonts.serif`.** If parser encounters `fonts.serif`, treat as `fonts.body`. Must be sans-serif — serif fonts are banned. |
| `fonts.mono` | No | Monospace font for code blocks |
| `aspectRatio` | No | Slide aspect ratio. Default: `16/9` |
| `transition` | No | Default slide transition |
| `accentColor` | No | Primary accent color (hex) |
| `iconStyle` | No | Icon rendering style: `outlined` (default), `filled`, `duotone` |
| `densityProfile` | No | Archetype density preference: `minimal`, `balanced` (default), `dense` |
| `maxFonts` | No | Max visual font identities. Default: `2` |
| `figma` | No | Object. Present only for Figma-sourced presets. Contains `fileKey`, `sourceUrl`, `extractedAt`, `slideCount`, `archetypes[]` |
| `figma.fileKey` | — | Figma file key for live API queries during learning |
| `figma.sourceUrl` | — | Full Figma URL for reference |
| `figma.extractedAt` | — | ISO 8601 timestamp of extraction |
| `figma.slideCount` | — | Number of slides extracted |
| `figma.archetypes` | — | Array of `{id, nodeId, contentType}` — one per extracted slide |

## Body

The body has two parts:

1. **Aesthetic description** (plain text): Free-form text describing the visual direction. The skill uses this to make design decisions not covered by frontmatter fields (e.g., background textures, illustration style, spacing density).

2. **Archetypes block** (optional fenced `archetypes` block): Configures composition archetype preferences for this preset.

   ```archetypes
   preferred: [bento-grid, asymmetric-split, icon-trio, stat-hero]
   avoid: [timeline-zigzag]
   cta_style: cta-warm
   cover_style: cover-hero
   ```

   - `preferred` — acts as a tie-breaker within the candidate set from content-type mapping. Does NOT override the mapping.
   - `avoid` — archetypes removed from candidate set before filtering. Takes absolute priority.
   - `cta_style` / `cover_style` — fixed archetypes for first and last slide.

3. **Shapes block** (optional fenced `shapes` block): Configures sub-slide visual element shapes.

   ```shapes
   card_radius: 14px
   icon_container: [circle, rounded-square]  # Single value or array. Values: circle, rounded-square, ghost. Single → [value, ghost]. Mapping: circle→.icon-circle, rounded-square→.icon-rounded, ghost→.icon-ghost
   stat_display: typographic     # typographic | pill-badge | card-inset
   label_style: pill             # pill | uppercase-text | accent-line
   divider_style: diagonal       # diagonal | horizontal | gradient-fade | none
   photo_mask: circle            # circle | rounded-rect | hexagon
   ```

   When generating slides, archetype HTML skeletons use these shape values to render elements. Defaults when absent: `icon_container: none`, `stat_display: card-inset`, `label_style: uppercase-text`, `divider_style: horizontal`, `photo_mask: rounded-rect`.

For decoration elements, the generator uses the combinatorial library in references/decoration-library.md — pre-tested CSS components assembled from shape + behavior + position + scale. The library grows via --stitch=learn.

4. **CSS block** (REQUIRED fenced `css` block): Written verbatim to `styles/index.css` in the generated project. When a `shapes` block is present, the CSS block should include corresponding utility classes for the shape vocabulary. This block MUST include:

   **Required sections in the CSS block:**
   - **CSS variables** (`:root` block with `--slidev-theme-*` and `--color-*` variables)
   - **Base layout styles** (`.slidev-layout` — background, color, texture)
   - **Layout-specific text alignment** — explicit `text-align: center` on `h1`, `p`, `div`, `li` for ALL centered layouts: `.slidev-layout.cover`, `.slidev-layout.section`, `.slidev-layout.fact`, `.slidev-layout.end`, `.slidev-layout.statement`, `.slidev-layout.center`
   - **Background image overlay support** — `::before` pseudo-element on `.slidev-layout.cover` and `.slidev-layout.section` for dark overlay when background images are used

   **Why this is critical:** Slidev themes set `text-align: left` on child elements with high CSS specificity. Without explicit `text-align: center` on child elements in centered layouts, text will appear left-aligned even though the parent is centered. See `references/layout-css-patterns.md` for the full explanation.

## Preset Lookup Order

When resolving `--preset <name>`:

1. **Direct path**: If `<name>` is a file path (contains `/` or `\` or ends in `.md`), read directly
2. **Local**: Check `./.slidev-presets/<name>.preset.md` (current project directory)
3. **Registry**: Check `~/.claude/slidev-presets.json` for a `{ "name": "path" }` mapping
4. **Global convention**: Look for `~/.claude/slidev-presets/<name>.preset.md`

Local presets take priority over global ones, allowing project-specific overrides.

If none found, emit an error listing all paths tried.

## Storage Locations

Presets can be stored in two locations:

- **Global**: `~/.claude/slidev-presets/<name>.preset.md` — available across all projects
- **Local**: `./.slidev-presets/<name>.preset.md` — scoped to the current project, committed with the repo

The optional registry file `~/.claude/slidev-presets.json` maps preset names to absolute paths for presets stored elsewhere.

## Trend Vocabulary (for --create-preset)

When creating presets via `--create-preset`, these named aesthetic trends can be referenced in the preset body to guide design decisions. This vocabulary is **only for preset creation** — it does not affect default generation.

| Trend | Description |
|-------|-------------|
| `bold-minimalism` | White text on dark backgrounds, max 1-2 elements per slide, large typography as primary design element |
| `editorial` | Wide margins, asymmetric grids, magazine-like layout, strong photo hierarchy, pull quotes in contrasting type |
| `human-centered` | Organic shapes, natural color palettes (terracotta, sage, cream), tactile imagery, intentional imperfection |
| `broken-grid` | Overlapping text and images, angular sections, diagonal dividers, dynamic and editorial feel |

Reference in preset body as: "This preset follows the `bold-minimalism` trend..."

## Theme Directory

Presets may include a `<preset-name>-theme/` directory alongside the .preset.md file (e.g., `boardroom.preset.md` → `boardroom-theme/`). When present, the theme is copied to the presentation output and referenced via `theme: ./theme` in slides.md frontmatter.

The theme encapsulates:
- CSS variables and base styles (from the preset CSS block)
- Layout Vue components with correct positioning
- Shape vocabulary CSS classes

The CSS block in the preset body remains the source of truth. The theme directory is generated from it and should not be hand-edited — regenerate the theme directory if changes are needed.

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
