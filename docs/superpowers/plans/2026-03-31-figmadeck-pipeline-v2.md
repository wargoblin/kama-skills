# Figmadeck Pipeline v2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite figmadeck's extraction and QA pipeline to achieve pixel-perfect Figma reproduction using `get_design_context` as primary source and quantitative pixel-diff scoring.

**Architecture:** Four file rewrites/updates: (1) figma-extraction.md — `get_design_context` primary with transpilation, (2) qa-cycle.md — pixelmatch + design-critique, (3) generation-pipeline.md — new archetype format note, (4) figma-learning.md — learn_0 quantitative criteria.

**Tech Stack:** Figma MCP (`get_design_context`, `get_screenshot`, `get_metadata`, `use_figma`), Playwright/pixelmatch for pixel diff, Slidev.

**Spec:** `docs/superpowers/specs/2026-03-31-figmadeck-pipeline-v2.md`

---

## File Structure

All files **already exist** — this is a rewrite/update, not new creation:

```
figmadeck/.claude/skills/figmadeck/
  references/
    figma-extraction.md       # REWRITE: FIG-3 and FIG-4 (lines 77-275)
    qa-cycle.md               # REWRITE: Phase B and Phase C (lines 105-175)
    generation-pipeline.md    # UPDATE: Step 5 note (line 128)
    figma-learning.md         # UPDATE: FDL-0c and FDL-0d (lines 46-68)
```

---

### Task 1: Rewrite FIG-3 in `figma-extraction.md` — `get_design_context` primary

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md:77-152` (replace FIG-3 section)

- [ ] **Step 1: Read current FIG-3**

Read `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md` lines 77-152 to understand the current 4-level extraction being replaced.

- [ ] **Step 2: Replace FIG-3 section**

Replace the entire `## FIG-3: Per-Slide Extraction` section (from line 77 to the line before `## FIG-4`) with the new `get_design_context`-first approach.

The new FIG-3 must contain:

**New Level 1 — Code from `get_design_context` (PRIMARY):**
- Call `mcp__figma__get_design_context(nodeId, fileKey, clientLanguages: "html,css", clientFrameworks: "unknown")` for each slide frame
- Returns: React+Tailwind code + screenshot + contextual metadata (design tokens, annotations, Code Connect mappings)
- The React+Tailwind code contains: correct flex/grid from Figma's own auto-layout interpretation, correct absolute positioning, gradient backgrounds as CSS values, image references via Figma CDN URLs, shadow/blur effects as CSS properties, auto-layout constraints mapped to flex
- Save the code output to `figmadeck-<name>-figma/slide-<N>-<name>/design-context.html`

**New Level 2 — Screenshot (visual reference):**
- `mcp__figma__get_screenshot(nodeId, fileKey)` → download and save as `figmadeck-<name>-figma/slide-<N>-<name>/reference.png`
- This is the pixel-perfect reference for QA comparison

**New Level 3 — Metadata (structural backup for QA):**
- `mcp__figma__get_metadata(nodeId, fileKey)` → XML with positions/sizes
- Used ONLY for QA structural comparison in qa-cycle.md Phase A — NOT for archetype generation
- Save as `figmadeck-<name>-figma/slide-<N>-<name>/metadata.xml`

**Asset Download (NEW):**
- Scan the `get_design_context` output for image URLs (Figma CDN `https://figma-*.amazonaws.com/...` or localhost `http://localhost:...`)
- Download each image asset via WebFetch
- Save to `figmadeck-<name>-figma/slide-<N>-<name>/assets/`
- Replace URLs in the code with relative paths (e.g., `assets/image-1.png`)
- For image fills that appear as background on frames: download and set as `background-image: url(assets/...)` in the archetype

**Note on blueprint.json:** Removed as archetype generation source. The `use_figma` Plugin API JS extraction (old Level 3) is no longer used for building archetypes. If detailed property values are needed for QA style comparison, `use_figma` JS calls are made during QA Phase A, not during extraction.

- [ ] **Step 3: Verify new FIG-3**

```bash
grep -c "get_design_context\|design-context.html\|reference.png\|metadata.xml\|Asset Download" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md
```
Expected: 5+ matches

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-extraction.md
git commit -m "feat(figmadeck): rewrite FIG-3 — get_design_context as primary source"
```

---

### Task 2: Rewrite FIG-4 in `figma-extraction.md` — transpilation pipeline

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md:153-275` (replace FIG-4 section, update FIG-5)

- [ ] **Step 1: Read current FIG-4**

Read `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md` lines 153-275.

- [ ] **Step 2: Replace FIG-4 section**

Replace the entire `## FIG-4: Archetype Conversion` section with the new transpilation pipeline. Keep Step 4-pre deduplication (with 6th criterion) and Step 4a content type detection — they're unchanged. Replace Steps 4b (HTML Skeleton) and 4c (Flexibility Rules) with the new transpilation steps.

The new FIG-4 must contain:

**Step 4-pre: Deduplication** — KEEP AS-IS (6 criteria including background treatment)

**Step 4a: Content type detection** — KEEP AS-IS (same heuristics table)

**Step 4b: Transpile React+Tailwind → HTML+inline CSS (NEW)**

This is the core conversion step. Take the `design-context.html` from FIG-3 Level 1 and post-process it:

1. **Strip React syntax**: `<div className="..."` → `<div style="..."`. Remove React-specific attributes (`key`, `ref`, `onClick`). Convert `className` to `class` where needed.

2. **Convert Tailwind to inline CSS**: Replace Tailwind utility classes with their CSS equivalents as inline `style` attributes:
   - Layout: `flex` → `display:flex`, `flex-col` → `flex-direction:column`, `items-center` → `align-items:center`, `justify-between` → `justify-content:space-between`, `gap-6` → `gap:1.5rem`
   - Sizing: `w-full` → `width:100%`, `w-[48%]` → `width:48%`, `h-[200px]` → `height:200px`, `flex-1` → `flex:1`
   - Spacing: `p-6` → `padding:1.5rem`, `px-4` → `padding-left:1rem;padding-right:1rem`, `mt-2` → `margin-top:0.5rem`
   - Typography: `text-xl` → `font-size:1.25rem`, `font-bold` → `font-weight:700`, `leading-tight` → `line-height:1.25`
   - Colors: `bg-[#1a1a2e]` → `background:#1a1a2e` (then step 3 converts to var)
   - Borders: `rounded-xl` → `border-radius:0.75rem`, `border` → `border:1px solid`
   - Position: `absolute` → `position:absolute`, `inset-0` → `inset:0`, `z-10` → `z-index:10`
   - Overflow: `overflow-hidden` → `overflow:hidden`

3. **Replace hardcoded hex with CSS variables**: Scan all color values (`#XXXXXX`, `rgb(...)`, `rgba(...)`) and replace with matching `var(--*)` from the preset:
   - Match by proximity (deltaE < 3): `#1a1a2e` ≈ `var(--bg-accent)`, `#f1f1f1` ≈ `var(--bg-base)`, `#ffe614` ≈ `var(--color-accent)`
   - Leave `rgba(...)` transparency values intact — only replace the color portion
   - If no matching variable → keep the hex value (it may be a gradient stop or one-off color)

4. **Replace image URLs with local paths**: Figma CDN URLs → `assets/image-<N>.png` (already downloaded in FIG-3). For `{{IMAGE}}` slots in visual-break archetypes: keep as slot marker.

**Step 4c: Insert `{{SLOT}}` markers (NEW)**

Identify text content and replace with slot markers:
1. Find all text nodes in the transpiled HTML (content of `<h1>`, `<h2>`, `<p>`, `<span>`, `<div>` that contain readable text)
2. Replace with `{{SLOT_NAME}}`:
   - First large heading → `{{TITLE}}`
   - Subtitle/description paragraph → `{{SUBTITLE}}` or `{{DESCRIPTION}}`
   - Repeated items in grid/flex → `{{CARD_1_TITLE}}`, `{{CARD_1_DESC}}`, `{{CARD_2_TITLE}}`, etc.
   - Large standalone number → `{{METRIC_VALUE}}`
   - Small label above metric → `{{METRIC_LABEL}}`
   - Footer breadcrumb text → `{{BREADCRUMB}}`, `{{SECTION}}`, `{{PAGE_NUM}}`
3. Record original text in flexibility.yaml as `original_text` for each slot (used in learn_0)

**Step 4d: CSS rendering fixes (NEW)**

Apply to every archetype during transpilation:
- **Letter-spacing**: if Figma value is percentage → convert to `em` (e.g., `2%` → `letter-spacing: 0.02em`). If already `px` → convert: `px / fontSize_px` → `em`
- **Line-height**: always use unitless ratio: `line-height: calc(lineHeight_px / fontSize_px)` → e.g., `line-height: 1.43`
- **Coordinates**: `Math.round()` all position/size values to integers to avoid sub-pixel rendering issues
- **Box-shadow**: if Figma shadow has `blur: Xpx` → CSS `box-shadow` blur = `X * 2` px (Figma blur ≈ half CSS blur)
- **Font feature settings**: preserve `font-feature-settings` from `get_design_context` if present

**Step 4e: Deduplication** — same 6 criteria (KEEP AS-IS)

**Step 4f: Flexibility rules** — same as before, plus `original_text` field per slot (KEEP + EXTEND)

**Font Size Floor exception:**
During FIG-4 transpilation: do NOT apply Font Size Floor. Preserve exact Figma values. Font Size Floor (body ≥ 1.25rem, heading ≥ 2.2rem) applies ONLY during generation with different content (learn_1..N and regular generation).

- [ ] **Step 3: Update FIG-5 — theme CSS additions**

In the `## FIG-5: Preset Assembly` section, add to the theme CSS generation instructions:

Add these rules to every generated `theme/styles/index.css`:
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

- [ ] **Step 4: Verify**

```bash
grep -c "Transpile\|React+Tailwind\|inline CSS\|SLOT.*marker\|rendering fix\|font-smoothing\|letter-spacing.*em\|box-shadow.*double\|Font Size Floor.*exception" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md
```
Expected: 8+ matches

- [ ] **Step 5: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-extraction.md
git commit -m "feat(figmadeck): rewrite FIG-4 — transpilation pipeline + CSS fixes + font floor exception"
```

---

### Task 3: Rewrite Phase B and Phase C in `qa-cycle.md`

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md:105-175` (replace Phase B and Phase C)

- [ ] **Step 1: Read current Phase B and C**

Read `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md` lines 105-175.

- [ ] **Step 2: Replace Phase B**

Replace `## Phase B: Visual Export + Figma Screenshot Comparison` (line 105 onwards) with:

**Step B1: Export + Figma screenshot** (same as before):
- Install deps if needed: `npm install && npx playwright install chromium`
- Export: `npx slidev export --format png --output slides-qa`
- For each slide: `mcp__figma__get_screenshot(nodeId, fileKey)` → fresh live screenshot

**Step B2: Quantitative pixel diff (NEW):**
- For each slide, compare `slides-qa/<N>.png` against `figmadeck-<name>-figma/slide-<N>-<name>/reference.png`
- Compute pixel diff using Playwright's built-in pixelmatch:
  ```bash
  # Run via Node.js script (Playwright already installed)
  node -e "
  const { PNG } = require('pngjs');
  const pixelmatch = require('pixelmatch');
  const fs = require('fs');
  const img1 = PNG.sync.read(fs.readFileSync('slides-qa/1.png'));
  const img2 = PNG.sync.read(fs.readFileSync('reference.png'));
  const diff = new PNG({width: img1.width, height: img1.height});
  const numDiff = pixelmatch(img1.data, img2.data, diff.data, img1.width, img1.height, {
    threshold: 0.15,
    includeAA: false
  });
  const ratio = numDiff / (img1.width * img1.height);
  fs.writeFileSync('diff.png', PNG.sync.write(diff));
  console.log(JSON.stringify({diffPixels: numDiff, total: img1.width*img1.height, ratio}));
  "
  ```
- Note: if `pixelmatch` and `pngjs` are not available, the agent should install them: `npm install pixelmatch pngjs`. Alternatively, if this is impractical in the Slidev project, the agent can perform the comparison visually (fall back to LLM visual assessment) but MUST log that quantitative diff was unavailable.
- Map `diffRatio` to Visual sub-score: `<0.03` → 10, `<0.05` → 9, `<0.08` → 8, `<0.12` → 7, `>=0.12` → 6 or lower
- Save diff image for targeted fixes — regions with pink/red highlights show where pixels differ

**Step B3: Design critique (NEW):**

After pixel diff, run a design critique on each exported slide PNG. This is the "designer thinking" that catches issues pixel diff misses (correct pixels but wrong visual hierarchy, or text technically within bounds but visually cramped).

Framework (from Anthropic Design plugin `design-critique` skill):

**First Impression (2 seconds):**
- Look at the slide for 2 seconds. What draws the eye first?
- Is that the intended focal point from the Figma template? (Compare against reference.png)
- Is the purpose of the slide immediately clear?

**Visual Hierarchy:**
- Clear reading order? (heading → subheading → body → footer)
- Right elements emphasized? (hero metric should be ≥ 2x size of supporting text)
- Whitespace distributed as in Figma template?
- Any text competing with other text for attention?

**Element Integrity (most critical for overflow detection):**
- Does any text overlap with another element? → **CRITICAL**
- Does any text extend beyond its container boundary? → **CRITICAL**
- Is any text cut off or truncated unintentionally? → **CRITICAL**
- Are all gaps between adjacent elements ≥ 8px (0.5rem)? → **FAIL**
- Is the footer zone (bottom 44-56px) clear of content? → **FAIL**

**Consistency with Template:**
- Does slide look like it belongs in the same presentation as Figma original?
- Same color mood? Same font treatment? Same whitespace density?
- Adapted elements (different content length) still visually balanced?

**Fix priority:**
1. CRITICAL (overlap, cut-off, boundary breach) → auto-fix immediately: shorten text, redistribute spacing, increase container size
2. FAIL (spacing gaps, footer zone) → auto-fix immediately: adjust padding/margin
3. Hierarchy/consistency issues → log to report, fix only if total score < 9

- [ ] **Step 3: Replace Phase C**

Replace `## Phase C: Score` with the enhanced scoring formula:

**Fidelity Score = Pixel Diff (40%) + Structural (25%) + Style (25%) + Design Critique (10%)**

- **Pixel Diff** (40%): from Step B2 — `diffRatio < 0.03` → 10, `< 0.05` → 9, etc.
- **Structural** (25%): from Phase A Figma structural comparison — positions within ±5%, proportions within ±10%, hierarchy exact match. Score: 10 if all within tolerance, deduct 1 per out-of-tolerance element (min 0).
- **Style** (25%): from Phase A Figma style comparison (tolerance table: font-size ±0.15rem, color ΔE<5, etc.). Score: 10 if all within tolerance, deduct 0.5 per out-of-tolerance property (min 0).
- **Design Critique** (10%): from Step B3 — 10 if zero CRITICAL and zero FAIL. Deduct 3 per CRITICAL, 1 per FAIL (min 0).

**Targets:**
- Regular generation: ≥ 9/10
- learn_0: zero FAIL/CRITICAL AND diffRatio < 0.05

**Safety stop:** 5 iterations with score delta < 0.3, OR pixel diff plateaus (diffRatio change < 0.005 between two consecutive iterations).

**Iteration Report:**
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

Clean up: `rm -rf slides-qa` after each iteration (re-export on next).

- [ ] **Step 4: Verify**

```bash
grep -c "pixelmatch\|pixel diff\|diffRatio\|design critique\|First Impression\|Element Integrity\|CRITICAL.*overlap\|Pixel Diff (40%)" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md
```
Expected: 8+ matches

- [ ] **Step 5: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/qa-cycle.md
git commit -m "feat(figmadeck): rewrite QA — pixelmatch scoring + design-critique framework"
```

---

### Task 4: Update `generation-pipeline.md` — new archetype format note

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md:128-135` (update Step 5 intro)

- [ ] **Step 1: Read current Step 5 intro**

Read `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md` lines 128-145.

- [ ] **Step 2: Add archetype format note**

At the beginning of `## Step 5: Write slides.md` (after the heading), add this note:

```markdown
**Archetype format (Pipeline v2):** Archetypes are now generated from `get_design_context` output, transpiled from React+Tailwind to HTML+inline CSS. They contain:
- Inline `style=""` attributes (not Tailwind classes)
- `var(--*)` CSS variable references for colors
- Flex/grid layout from Figma's own auto-layout interpretation
- Preserved gradient backgrounds, shadows, and image assets

The **VERBATIM COPY RULE** still applies — copy the transpiled archetype HTML exactly, replace only `{{SLOT}}` markers with content.
```

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/generation-pipeline.md
git commit -m "feat(figmadeck): add v2 archetype format note to Step 5"
```

---

### Task 5: Update `figma-learning.md` — learn_0 quantitative criteria

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-learning.md:46-68` (update FDL-0c and FDL-0d)

- [ ] **Step 1: Read current FDL-0c and FDL-0d**

Read `figmadeck/.claude/skills/figmadeck/references/figma-learning.md` lines 46-68.

- [ ] **Step 2: Update FDL-0c**

Replace the `### FDL-0c: Pixel-perfect QA cycle` section with:

```markdown
### FDL-0c: Pixel-perfect QA cycle

Run a **strict variant** of `qa-cycle.md` with these overrides:

**What to SKIP:**
- Skip all design-rules checks (text density, hierarchy, contrast) — original Figma content is the truth
- Skip Font Size Floor — reproduce Figma values exactly, even if body < 1.25rem

**What to RUN:**
- Phase A: Figma structural comparison (positions, proportions, hierarchy) — full tolerance table
- Phase A: Figma style comparison (font-size, colors, spacing, radius) — full tolerance table
- Phase B Step B2: **Quantitative pixel diff** — this is the primary pass/fail criterion
- Phase B Step B3: Design critique **Element Integrity only** (overlap, cut-off, boundary) — skip hierarchy/consistency checks

**Pass condition (ALL must be true):**
1. `diffRatio < 0.05` (≤5% pixel difference) for EVERY slide
2. Zero CRITICAL issues from design critique Element Integrity
3. All structural positions within ±5% tolerance
4. All style values within tolerance table thresholds

**If any condition fails:** fix issues → re-generate → re-compare. Fixes go to archetype.html + preset.md CSS + flexibility.yaml.

Export PDF after each iteration for visual review (same FDL-3b2 procedure).
```

- [ ] **Step 3: Update FDL-0d**

Replace the `### FDL-0d: Verification from scratch` section with:

```markdown
### FDL-0d: Verification from scratch

After FDL-0c achieves pass on all slides:

1. **Delete** `<edu_dir>/learn_0/slides.md` and all generated artifacts (keep outline.md and reference PNGs)
2. **Re-generate** the calibration presentation from scratch using the updated archetypes/preset — clean generation, no carried-over fixes
3. **Run the same strict QA** as FDL-0c — quantitative pixel diff + element integrity
4. If `diffRatio < 0.05` for ALL slides AND zero CRITICAL on **first QA iteration** (no fix cycles needed) → **learn_0 PASSED**
5. If not → return to FDL-0c with the new issues, fix archetypes, repeat verification

**Maximum attempts:** If verification fails 3 times → STOP. Report which archetypes/properties cannot be reproduced pixel-perfect. Do NOT proceed to learn_1..N.
```

- [ ] **Step 4: Verify**

```bash
grep -c "diffRatio\|0.05\|pixel diff\|Font Size Floor\|design-rules.*SKIP\|quantitative" figmadeck/.claude/skills/figmadeck/references/figma-learning.md
```
Expected: 5+ matches

- [ ] **Step 5: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-learning.md
git commit -m "feat(figmadeck): update learn_0 — quantitative pixel diff criteria + font floor exception"
```

---

### Task 6: Final verification + push

**Files:**
- All modified files from Tasks 1-5

- [ ] **Step 1: Verify key concepts present in all files**

```bash
# figma-extraction.md: get_design_context primary, transpilation, assets, CSS fixes
grep "get_design_context.*PRIMARY\|Transpile\|Asset Download\|font-smoothing\|Font Size Floor.*exception" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md

# qa-cycle.md: pixelmatch, design critique, new scoring
grep "pixelmatch\|Design critique\|Pixel Diff (40%)\|Element Integrity" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md

# generation-pipeline.md: v2 archetype format
grep "Pipeline v2\|get_design_context" figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md

# figma-learning.md: quantitative learn_0
grep "diffRatio.*0.05\|Font Size Floor\|quantitative" figmadeck/.claude/skills/figmadeck/references/figma-learning.md
```

Expected: all greps return matches.

- [ ] **Step 2: No stale references**

```bash
# No references to blueprint.json as generation source (should only appear in QA context if at all)
grep -n "blueprint.json" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md | grep -v "QA\|comparison\|backup\|optional\|REMOVED"

# No references to old "4 levels" extraction
grep "Level 1.*screenshot\|Level 2.*Structure\|Level 3.*Detailed\|Level 4.*Code hint" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md
```

Expected: both return empty (no matches).

- [ ] **Step 3: Push**

```bash
git push
```
