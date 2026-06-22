# QA Cycle — Iterative Figma Comparison

Runs after slides.md is generated (Step 5 of generation-pipeline.md). Loops until Fidelity Score >= 9/10.

## Overview

The cycle runs three phases per iteration:

```
repeat:
  Phase A: CSS + Figma structural/style comparison → fix issues
  Phase B: Visual export + Figma screenshot comparison → fix issues
  Phase C: Score → if >= 9/10, DONE; else next iteration
  Safety stop: 5 iterations with score delta < 0.3 → stop with report
```

Every slide is compared on every iteration. Fixes are applied to slides.md before moving to the next phase.

## Comparison Scope

EVERY slide is compared — including those with adapted archetypes. Each slide is compared against the closest Figma original from the SAME preset. Read `nodeId` from `source.json` in the figmadeck-`<name>`-figma/ directory.

Adapted archetypes (content type had no exact match) are compared against the archetype they were adapted from, with a relaxed content-fit criterion: the slide must visually belong to the same presentation family.

## Phase A: CSS + Figma Structural/Style Comparison

### Basic Checks

Run these on every slide before calling Figma APIs:

- **Font-size floor**: body >= 1.25rem, heading >= 2.2rem, label >= 0.65rem
- **layout: none enforcement**: every slide that uses raw HTML absolute positioning must declare `layout: none`
- **Hardcoded hex scan**: find any `#rrggbb` / `rgb(...)` literals → replace with `var(--*)` CSS variables
- **Content overflow**: text fits within slot boundaries, no element overlap between content blocks

Fix any violations directly in slides.md before proceeding to Figma structural comparison.

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

### Figma Structural Comparison

For each slide, call using its archetype's `nodeId`:

```
mcp__figma__get_metadata(nodeId, fileKey)
```

Returns positions and sizes of all elements. Compare:

- **Element positions**: tolerance +-5% of viewport width/height
- **Element proportions** (width/height ratios): tolerance +-10%
- **Nesting hierarchy**: exact match — wrapper → section → element structure must be preserved

### Figma Style Comparison

For each slide, call:

```
mcp__figma__use_figma  (JS extraction: fonts, colors, spacing, radius)
```

Compare extracted values against slides.md CSS:

| Property | Tolerance |
|----------|-----------|
| font-size | +-0.15rem |
| font-weight | exact |
| color | deltaE < 5 (CIEDE2000) |
| spacing (gap) | +-0.25rem |
| padding | +-0.5rem |
| border-radius | +-2px |
| position | +-5% viewport |

Fix all issues found → modify slides.md before Phase B.

## Phase B: Visual Export + Figma Screenshot Comparison

### Step B1: Export + Figma screenshot

```bash
npm install && npx playwright install chromium   # only if deps missing
npx slidev export --format png --output slides-qa
```

For each slide: call `mcp__figma__get_screenshot(nodeId, fileKey)` — fresh live screenshot, not cached. Save as `figmadeck-<name>-figma/slide-<N>-<name>/reference.png`.

### Step B2: Quantitative pixel diff

For each slide, compare `slides-qa/<N>.png` against `figmadeck-<name>-figma/slide-<N>-<name>/reference.png`.

Compute pixel diff using a Node.js script with pixelmatch:

```bash
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

**If `pixelmatch` and `pngjs` are not available:** install them: `npm install pixelmatch pngjs`. Alternatively, if this is impractical in the Slidev project, perform the comparison visually (fall back to LLM visual assessment), but MUST log that quantitative diff was unavailable.

Map `diffRatio` to Visual sub-score:
- `< 0.03` → 10
- `< 0.05` → 9
- `< 0.08` → 8
- `< 0.12` → 7
- `>= 0.12` → 6 or lower

Save the diff image for targeted fixes — regions with pink/red highlights show where pixels differ.

### Step B3: Design critique

After pixel diff, run a design critique on each exported slide PNG. This is "designer thinking" that catches issues pixel diff misses (correct pixels but wrong visual hierarchy, or text technically within bounds but visually cramped).

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
1. **CRITICAL** (overlap, cut-off, boundary breach) → auto-fix immediately: shorten text, redistribute spacing, increase container size
2. **FAIL** (spacing gaps, footer zone) → auto-fix immediately: adjust padding/margin
3. Hierarchy/consistency issues → log to report, fix only if total score < 9

Fix all issues found → modify slides.md before Phase C.

### Cleanup

```bash
rm -rf slides-qa
```

Run after each iteration (re-export fresh on next iteration).

## Phase C: Score

**Fidelity Score = Pixel Diff (40%) + Structural (25%) + Style (25%) + Design Critique (10%)**

Each sub-score is 0–10. Weighted sum gives the final score out of 10.

- **Pixel Diff** (40%): from Step B2 — `diffRatio < 0.03` → 10, `< 0.05` → 9, `< 0.08` → 8, `< 0.12` → 7, `>= 0.12` → 6 or lower.
- **Structural** (25%): from Phase A Figma structural comparison — positions within ±5%, proportions within ±10%, hierarchy exact match. Score: 10 if all within tolerance, deduct 1 per out-of-tolerance element (min 0).
- **Style** (25%): from Phase A Figma style comparison (tolerance table: font-size ±0.15rem, color ΔE<5, etc.). Score: 10 if all within tolerance, deduct 0.5 per out-of-tolerance property (min 0).
- **Design Critique** (10%): from Step B3 — 10 if zero CRITICAL and zero FAIL. Deduct 3 per CRITICAL, 1 per FAIL (min 0).

**Targets:**
- Regular generation: ≥ 9/10
- learn_0: zero FAIL/CRITICAL AND diffRatio < 0.05

**Safety stop:** 5 iterations with score delta < 0.3, OR pixel diff plateaus (diffRatio change < 0.005 between two consecutive iterations).

## Figma API Usage

- Make **live calls on EACH iteration** — do not cache get_screenshot, get_metadata, or use_figma results across iterations.
- If Figma API is unavailable: retry 2x with a 10-second pause between attempts.
- After 2 failed retries: fall back to cached `reference.png` from the figmadeck-`<name>`-figma/ directory. Note the fallback in the iteration report.

## Iteration Report

Print after each Phase C:

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

## Final Summary (on DONE)

Print when score reaches >= 9/10:

```
━━━ QA Complete ━━━
Final Fidelity: X.X/10
Iterations: N
Total fixes applied: X
Presentation: <project path>
```
