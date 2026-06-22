# Figmadeck Pipeline v2: Pixel-Perfect + Design Intelligence

## Overview

Major rewrite of the figmadeck extraction and QA pipeline. Three pillars:

1. **`get_design_context` as primary archetype source** — replace blueprint.json manual conversion with Figma MCP's own code output (React+Tailwind → HTML+inline CSS). Fixes gradients, images, coordinates, flex constraints in one move.
2. **CSS rendering fixes** — font-smoothing, unitless line-height, letter-spacing in em, integer pixel rounding, shadow blur doubling. Closes the rendering gap between Figma and browser.
3. **Design-critique-powered QA** — integrate the design-critique framework (First Impression, Visual Hierarchy, Consistency, Accessibility) into the QA cycle for "designer thinking" that catches overflow, intersection, and hierarchy issues.

## Problems Solved

- learn_0 doesn't achieve pixel-perfect despite calibration cycles
- Text overflow and element intersection after content adaptation
- Gradients collapse to flat colors (GAP 1)
- Images absent entirely (GAP 2)
- Nested coordinates miscalculated (GAP 3)
- Font size floor inflates small text during extraction (GAP 4)
- `get_design_context` code is higher-fidelity but unused as primary (GAP 5)
- Auto-layout HUG/FILL constraints lost (GAP 6)
- Shadows not converted to CSS (GAP 8)
- QA comparison is qualitative (LLM opinion) not quantitative (pixel diff score)

---

## Change 1: Rewrite `figma-extraction.md` — `get_design_context` as primary

### FIG-3: Per-Slide Extraction (rewrite)

Replace the current 4-level extraction with a `get_design_context`-first approach:

**New Level 1 — Code from `get_design_context` (PRIMARY):**
- Call `mcp__figma__get_design_context(nodeId, fileKey, clientLanguages: "html,css", clientFrameworks: "unknown")` for each slide frame
- This returns: React+Tailwind code + screenshot + contextual metadata (design tokens, annotations)
- The React+Tailwind code already contains:
  - Correct flex/grid structure as interpreted by Figma's own code-gen
  - Correct absolute positioning in viewport-relative coordinates
  - Gradient backgrounds as CSS values (not flat colors)
  - Image references via Figma CDN URLs
  - Shadow/blur effects as CSS properties
  - Auto-layout constraints mapped to flex

**New Level 2 — Screenshot (visual reference):**
- `get_screenshot(nodeId, fileKey)` → save as `reference.png`
- Also save the inline screenshot from `get_design_context`

**New Level 3 — Metadata (structural backup):**
- `get_metadata(nodeId, fileKey)` → XML positions/sizes
- Used for QA structural comparison, NOT for archetype generation

**Level 4 (blueprint.json) — REMOVED as generation source.** Optional: keep for QA property comparison only.

### FIG-4: Archetype Conversion (rewrite)

Replace the blueprint.json → HTML conversion with a post-processing pipeline on `get_design_context` output:

**Step 4a: Transpile React+Tailwind → HTML+inline CSS**
1. Strip React component syntax (`<div className=...>` → `<div style="...">`)
2. Convert Tailwind classes to inline CSS values (e.g., `w-[48%]` → `width: 48%`, `flex-1` → `flex: 1`, `gap-6` → `gap: 1.5rem`)
3. Replace hardcoded hex colors with `var(--*)` CSS variables from the preset
4. Replace Figma CDN image URLs with local `{{IMAGE}}` slots or download assets

**Step 4b: Insert `{{SLOT}}` markers**
1. Identify text content in the transpiled HTML (text inside elements)
2. Replace text with `{{SLOT_NAME}}` markers using the same naming convention: `{{TITLE}}`, `{{DESCRIPTION}}`, `{{CARD_1_TITLE}}`, etc.
3. Keep the original text in flexibility.yaml as `original_text` for reference

**Step 4c: CSS rendering fixes (apply to every archetype)**
Add to the archetype's `<style>` or to the theme CSS:
```css
/* Font smoothing — match Figma's Skia renderer */
.slidev-layout * {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: geometricPrecision;
}
```

Fix specific properties during transpilation:
- **Letter-spacing**: Figma percentage → CSS `em` (Figma 2% = `0.02em`)
- **Line-height**: use unitless ratio (`line-height: 1.4` not `line-height: 1.4rem`)
- **Coordinates**: round to integer pixels (`Math.round()`) to avoid sub-pixel rendering issues
- **Box-shadow**: double the blur value (Figma blur ≈ CSS blur / 2)

**Step 4d: Deduplication** — same 6 criteria as current (including background treatment)

**Step 4e: Flexibility rules** — same as current

**Step 4f: Content type detection** — same heuristics as current

### FIG-3 Asset Download (NEW)

When `get_design_context` returns image URLs (Figma CDN or localhost):
1. Download each image asset
2. Save to `figmadeck-<name>-figma/slide-<N>-<name>/assets/`
3. Replace URL in archetype HTML with relative path or `{{IMAGE}}` slot

This closes GAP 2 (images absent).

### FIG-5: Theme CSS Additions

Add to every generated `theme/styles/index.css`:

```css
/* Figma rendering alignment */
.slidev-layout * {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: geometricPrecision;
}

/* Ensure consistent rendering */
*, *::before, *::after {
  box-sizing: border-box;
}
```

### Font Size Floor — learn_0 exception

During FIG-4 transpilation and during learn_0 QA:
- **Do NOT apply Font Size Floor.** Reproduce Figma values exactly.
- Font Size Floor (body ≥ 1.25rem, heading ≥ 2.2rem) applies ONLY during learn_1..N and regular generation — when content differs from the Figma original.

---

## Change 2: Rewrite `qa-cycle.md` — design critique + quantitative scoring

### Phase A: CSS + Figma Comparison (enhanced)

**Existing checks remain:** font-size floor (except learn_0), layout:none, hex scan, content overflow.

**Design Rules Checks remain** (from design-rules.md): text density, typography, spacing, hierarchy, contrast.

**Figma structural comparison remains:** `get_metadata` for positions/sizes with tolerance table.

**Figma style comparison remains:** `use_figma` JS for exact CSS values.

### Phase B: Visual Export + Design Critique (NEW approach)

**Step B1: Export + Figma screenshot** (same as before)
- `npx slidev export --format png --output slides-qa`
- `get_screenshot(nodeId, fileKey)` for each slide

**Step B2: Quantitative pixel diff (NEW)**
- For each slide, compute pixel diff between exported PNG and Figma reference PNG
- Use Playwright's pixelmatch (already in the pipeline — `npx playwright install chromium` is standard):
  ```
  threshold: 0.15 (tight for presentation context)
  includeAA: false (ignore anti-aliasing differences — they're rendering engine, not our bug)
  ```
- Compute `diffRatio = diffPixels / totalPixels`
- Map to Visual sub-score:
  - `diffRatio < 0.03` → 10/10
  - `diffRatio < 0.05` → 9/10
  - `diffRatio < 0.08` → 8/10
  - `diffRatio < 0.12` → 7/10
  - `diffRatio >= 0.12` → 6/10 or lower
- Save diff image (highlights where pixels differ) for targeted fixes

**Step B3: Design critique (NEW)**

After pixel diff, run a design critique on each slide using the framework from the Design plugin's `design-critique` skill:

**First Impression (2 seconds):**
- What draws the eye first? Is that the intended focal point from the Figma template?
- Is the purpose of the slide immediately clear?

**Visual Hierarchy:**
- Is there a clear reading order? (heading → subheading → body → footer)
- Are the right elements emphasized? (hero metric ≥ 2x supporting)
- Is whitespace distributed as in the Figma template?
- Is any text competing with other text for attention?

**Element Integrity:**
- Does any text overlap with another element? → **CRITICAL**
- Does any text extend beyond its container boundary? → **CRITICAL**
- Is any text cut off / truncated unintentionally? → **CRITICAL**
- Are all gaps between elements ≥ 8px? → **FAIL**
- Is the footer zone (bottom 44-56px) clear of content? → **FAIL**

**Consistency with Template:**
- Does the slide look like it belongs to the same presentation as the Figma original?
- Same color mood? Same font treatment? Same whitespace density?
- Are adapted elements (different content length) still visually balanced?

**Fix priority for Phase B issues:**
1. CRITICAL (overlap, cut-off, boundary breach) → auto-fix immediately
2. FAIL (spacing, footer zone) → auto-fix immediately
3. Hierarchy/consistency warnings → log, fix if score < 9

### Phase C: Scoring (enhanced)

**New scoring formula:**

```
Fidelity Score = Pixel Diff (40%) + Structural (25%) + Style (25%) + Design Critique (10%)
```

- **Pixel Diff** (40%): quantitative, from Step B2 pixelmatch score
- **Structural** (25%): from Phase A Figma structural comparison
- **Style** (25%): from Phase A Figma style comparison (tolerance table)
- **Design Critique** (10%): from Step B3 — 10 if no CRITICAL/FAIL issues, deduct per issue

**Target:** ≥ 9/10 for regular generation, zero FAIL/CRITICAL for learn_0.

**Safety stop:** 5 iterations with score delta < 0.3, OR pixel diff plateaus (diff change < 0.5% between iterations).

### Iteration Report (enhanced)

```
━━━ QA Iteration <N> ━━━
Fidelity: X.X/10
  Pixel Diff:       X/10 (diffRatio: X.XX%)
  Structural:       X/10
  Style:            X/10
  Design Critique:  X/10 (CRITICAL: X, FAIL: X)

Diff hotspots: [slide N: top-right corner, slide M: footer zone]
Fixed: <list>
Remaining: <list>
```

---

## Change 3: Update `generation-pipeline.md` — use new archetypes

### Step 5: Write slides.md (adjusted for new archetype format)

Archetypes from `get_design_context` may have slightly different structure than blueprint.json-derived ones:
- Inline styles instead of CSS classes (since we transpiled Tailwind → inline)
- `var(--*)` references for colors
- Flex/grid from Figma's own auto-layout interpretation

**VERBATIM COPY RULE still applies** — copy the transpiled archetype exactly, replace only `{{SLOT}}` markers.

**Step 5b: Adaptive Layout still applies** — footer flex conversion, text truncation, spacing rules.

### Step 4: Composition Planning — variant selection

Already implemented (context-based variant selection). No changes needed.

---

## Change 4: Update `figma-learning.md` — learn_0 adjustments

### FDL-0c: Pixel-perfect QA (adjusted)

- **Use quantitative pixel diff** as the primary pass/fail criterion (not LLM visual assessment)
- Pass condition: `diffRatio < 0.05` (≤5% pixel difference) for EVERY slide AND zero CRITICAL from design critique
- **Skip Font Size Floor** — reproduce Figma values exactly
- **Skip design-rules density checks** — original Figma content is the truth

### FDL-0d: Verification from scratch (adjusted)

- After regeneration, pixel diff must be `< 0.05` on FIRST run without QA iterations
- If not → return to FDL-0c

---

## Change 5: Update `SKILL.md` — reference links

Add to References section:
```
- `references/design-rules.md` — Universal design rules (already added)
```

Note: The design-critique framework is embedded directly in qa-cycle.md (Phase B Step B3), not as a separate reference file. This keeps it close to where it's used.

---

## Files Modified

| File | Change |
|------|--------|
| `references/figma-extraction.md` | **Rewrite** FIG-3 and FIG-4: get_design_context primary, transpilation pipeline, asset download, CSS fixes, font floor learn_0 exception |
| `references/qa-cycle.md` | **Rewrite** Phase B: pixelmatch quantitative diff + design-critique framework. Enhanced Phase C scoring formula |
| `references/generation-pipeline.md` | **Minor update** Step 5: note about new archetype format |
| `references/figma-learning.md` | **Update** FDL-0c/0d: quantitative pixel diff pass criteria, font floor exception |
| `SKILL.md` | No changes needed (references already correct) |

---

## Known Limitations

- **Pixel diff requires headless Chrome** — Playwright already in pipeline, no new dependency
- **Figma CDN image URLs may expire** — asset download during FIG-3 mitigates this
- **Font rendering gap** (~3-6%) is platform-inherent — `includeAA: false` in pixelmatch compensates
- **Non-axis-aligned gradients** (not 0°/45°/90°) may still diverge slightly — CSS gradient model differs from Figma's affine transform model
- **`get_design_context` output varies** — Figma MCP may return different code quality for different node complexity. Transpilation must be robust.
