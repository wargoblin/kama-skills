# Figmadeck v3: Figma-Native Pipeline

## Overview

Complete rewrite of figmadeck. Instead of extracting from Figma → generating HTML/CSS → comparing back, work **directly inside Figma**: clone the template page, adapt the clone by replacing text and removing/reordering slides, run QA cycles comparing adapted slides against originals.

**Why:** All previous GAPs (gradients, images, coordinates, fonts, shadows, auto-layout) existed because we converted between rendering engines. Clone in Figma = pixel-perfect by definition. Zero conversion loss.

**Pipeline:**
```
Figma template (Page 1) → clone page → analyze slides → match outline → fill content → QA cycle → export via Playwright
```

---

## Commands

```
/figmadeck <figma-url> <outline>          Create presentation in Figma
/figmadeck <figma-url> --learn=N          Calibration + learning
/figmadeck --export pdf|png [figma-url]   Export via Playwright
/figmadeck --edit <figma-url> <comment>   Edit existing generated page
/figmadeck --help                         Show help
```

**Removed** (no longer needed):
- `--dev` — no Slidev, no dev server
- `--preset` — no presets, work directly with Figma file
- Auto-match — no presets to match

---

## Authentication (Playwright)

### First Run
1. Check `~/.claude/figmadeck-auth.json` — exists?
2. If no → ask user: email + password (+ 2FA code if needed)
3. Playwright: open `figma.com/login` → fill form → submit
4. If 2FA form appears → ask user for code → submit
5. If login succeeds → save cookies/session to `~/.claude/figmadeck-auth.json`
6. If login fails → show error, ask for new credentials

### Subsequent Runs
1. Read `~/.claude/figmadeck-auth.json`
2. Playwright: load saved cookies → open `figma.com` → check not redirected to login
3. If session alive → continue
4. If session expired → ask for new credentials

### Security
- `~/.claude/figmadeck-auth.json` stored outside repository (never committed)
- Store only cookies/session token, not plain-text password
- Auth flow documented in `references/auth.md`

---

## Pipeline: Steps 1-7

### Step 1: Auth Check

Verify Playwright can access Figma (see Authentication section). If not → auth flow → retry.

### Step 2: Clone Template Page

One `use_figma` call:
- Find Page 1 (template) via `figma.root.children[0]`
- Create new page: `figma.createPage()` with name `"Generated: <outline-name>"`
- Clone all children: `child.clone()` → `newPage.appendChild(clone)` → preserve x/y positions
- Return `{ pageId, slides: [{id, name, origId, x}] }`

Clone preserves EVERYTHING: gradients, images, shadows, auto-layout, variable bindings, effects. Pixel-perfect by definition.

### Step 3: Analyze Template Slides

One `use_figma` call on the cloned page. For each frame:

**Collect slide map:**
- Frame id, name, index
- Content type (auto-detected via heuristics)
- All TEXT nodes with: id, name, characters, fontSize, position (x/y), width, height, role

**Text node role detection (hybrid: name + heuristics):**

| Priority | Method | Result |
|---|---|---|
| 1 | Node name contains keyword ("Title", "Heading") | title |
| 2 | Node name contains pattern ("description", "body", "desc") | description |
| 3 | Largest fontSize on slide (≥ 36px) | title |
| 4 | Second largest, long text (fontSize 16-24px, > 50 chars) | description |
| 5 | Bottom of slide, small text (y > 90% height, fontSize < 14px) | footer |
| 6 | Short text, uppercase (< 20 chars, all caps) | label/eyebrow |
| 7 | Everything else | body |

**Content type detection** — same heuristics as before:
- First slide + large heading → `intro`
- Last slide + CTA text → `cta`
- Large standalone number → `metric`
- 3+ similar child frames → `cards`
- Mostly images, little text → `visual-break`
- Default → `content`

### Step 4: Match Outline → Template Slides

1. For each outline slide → determine content type
2. Find best matching template slide by content type
3. If multiple templates of same type → **context-based variant selection** (warm colors → positive themes, cool → analytical, per existing design)
4. If outline has MORE slides than templates of that type → clone the template slide within the page (additional `use_figma` call: `node.clone()`)
5. Unused template slides → `node.remove()`
6. Reorder remaining slides by adjusting `node.x` positions (left to right = presentation order)

### Step 5: Fill Content

One `use_figma` call PER SLIDE (incremental, not batch):

**Font handling:**
1. For each TEXT node, check font availability: `listAvailableFontsAsync()`
2. If font available → `loadFontAsync(font)` → proceed
3. If NOT available → find closest fallback:
   - Same style (Regular/Bold) in similar family
   - Known substitution table: Whyte→Inter, Darker Grotesque→Manrope, custom serif→DM Serif Display
   - Last resort: Inter Regular
   - Log every substitution
4. `node.fontName = fallback` (if needed) → `node.characters = newText`

**Text overflow handling:**
1. After text replacement: `node.textAutoResize = "HEIGHT"` (text grows down, not truncated)
2. Check: `node.y + node.height > parentFrame.height`?
3. If overflow:
   - First: shorten/rephrase text (preserve core meaning)
   - Then: reduce fontSize (minimum = current × 0.85)
   - Then: expand container if space available
   - Last resort: move overflow to a note (separate text node below slide)

**Special characters:**
- After clone, Unicode characters may differ — use `includes()` not `===` for text matching
- Footer breadcrumbs: find by position (bottom of frame) + small fontSize, not by exact text match

### Step 6: QA Cycle (Figma-native)

Runs after Step 5. Loops until Fidelity Score ≥ 9/10.

#### Phase A: Structural Check (programmatic)

`use_figma` call to check every slide on the adapted page:

**Overlap detection:**
- For each pair of nodes on a slide → check bounding box intersection
- `nodeA` rect intersects `nodeB` rect → **CRITICAL**

**Boundary check:**
- For each TEXT node: `node.y + node.height > parent.height` → **CRITICAL**

**Gap check:**
- Distance between adjacent elements < 8px → **FAIL**

**Footer zone:**
- Content enters bottom 44-56px of frame → **FAIL**

**Text truncation:**
- `node.textAutoResize === "TRUNCATE"` or text visually clipped → **CRITICAL**

#### Phase B: Visual Comparison

For each slide:
1. `get_screenshot(originalSlideNodeId, fileKey)` → original template
2. `get_screenshot(adaptedSlideNodeId, fileKey)` → adapted version
3. Compare: same style? same spacing feel? same mood? (content differs, so compare STYLE not content)

#### Phase C: Design Critique

Per slide, evaluate:

**Element Integrity (highest priority):**
- Text overlaps another element → **CRITICAL** — shorten or reposition
- Text extends beyond container → **CRITICAL** — textAutoResize + check
- Elements too close (< 8px gap) → **FAIL** — increase gap
- Footer zone occupied by content → **FAIL** — push content up

**Visual Hierarchy:**
- Focal point preserved? Heading is largest element?
- Logical reading order maintained?

**Consistency:**
- Slide looks like part of the same presentation?
- Fonts, colors, spacing uniform?

#### Phase D: Score + Fix

**Scoring:**
- Structural (40%): from Phase A — 10 if zero CRITICAL/FAIL, deduct per issue
- Visual (30%): from Phase B — style correspondence to original
- Design Critique (30%): from Phase C — element integrity + hierarchy

**Fixes via `use_figma`:**
- Overlap → `node.resize(smallerWidth, height)` or shorten `node.characters`
- Boundary breach → `textAutoResize = "HEIGHT"`, reduce fontSize
- Gap too small → adjust `node.y` or `node.x`
- Footer invaded → shift all content elements up

Each fix = one `use_figma` call (incremental).

**Safety stop:** 5 iterations with score delta < 0.3.

**Iteration report:**
```
━━━ QA Iteration <N> ━━━
Fidelity: X.X/10
  Structural:       X/10 (CRITICAL: X, FAIL: X)
  Visual:           X/10
  Design Critique:  X/10

Fixed: <list>
Remaining: <list>
```

### Step 7: Export (Playwright)

1. Auth check
2. Playwright: open Figma file → navigate to generated page
3. For PDF: File → Export → PDF, or Presentation Mode → Print to PDF
4. For PNG: `get_screenshot` per slide → save locally
5. Fallback: export each frame via `get_screenshot` → combine to PDF via Python/Pillow

---

## Learn Mode

### learn_0: Calibration

Clone page → do NOT change any text → QA compare clone vs original → should be 100% match (clone = copy). This validates that the pipeline mechanics work (page creation, screenshot comparison, scoring).

If learn_0 fails → something is broken in the tooling, not in content adaptation.

### learn_1..N: Adaptation Learning

For each cycle:
1. Clone template → fill with diverse outline (Russian, varied industries/formats)
2. Run QA cycle until ≥ 9/10
3. Record which text replacements caused problems:
   - Which roles overflow most? (title vs description vs cards)
   - Which font fallbacks work well / poorly?
   - Which content types need more aggressive text shortening?
4. Update skill references: font fallback table, text shortening heuristics, role detection improvements

Fixes go to **skill files** (mapping rules, font table), not to Figma file.

**Convergence:** ≥ 9/10 for two consecutive cycles → early stop.

---

## Edit Mode

```
/figmadeck --edit <figma-url> <comment>
```

1. Open Figma file, find the generated page (by name prefix "Generated:")
2. Read user's edit comment
3. Apply changes via `use_figma` (modify text, reorder slides, swap templates)
4. Run QA cycle to validate changes

---

## Edge Cases

| Situation | Behavior |
|---|---|
| URL not `figma.com/design/` | Error, stop |
| No access to file | Error, stop |
| No frames on page 1 | Error: "No slides found on page 1" |
| Font unavailable | Auto-fallback to closest match, log substitution |
| Text overflow after replacement | Shorten → textAutoResize → reduce fontSize (min 85%) |
| Outline has more slides than template | Clone template slides within page to fill |
| Outline has fewer slides than template | Remove unused template slides |
| Playwright auth failed | Ask for new credentials |
| Figma rate limit | Retry with backoff (5s, 15s, 30s) |
| `use_figma` error | Stop, read error, fix script, retry (per figma-use skill rules) |

---

## File Structure (rewrite existing)

```
figmadeck/.claude/skills/figmadeck/
  SKILL.md                              # REWRITE: new commands, Figma-native dispatch
  references/
    figma-generation.md                 # NEW: clone → analyze → match → fill (replaces generation-pipeline + figma-extraction)
    qa-cycle.md                         # REWRITE: Figma-native QA (structural + visual + critique)
    figma-learning.md                   # REWRITE: simplified learn with Figma-native pipeline
    design-rules.md                     # UNCHANGED — universal rules stay
    auth.md                             # NEW: Playwright auth flow
    slidev-syntax.md                    # DELETE — no longer needed
  assets/
    demo-outline.md                     # UNCHANGED
```

---

## What's Eliminated

| Old (v1/v2) | New (v3) | Why |
|---|---|---|
| blueprint.json extraction | Clone in Figma | No conversion needed |
| React+Tailwind transpilation | Clone in Figma | No conversion needed |
| CSS variable mapping | Clone in Figma | Variables preserved by clone |
| Font Size Floor rules | Clone preserves exact sizes | No conversion artifacts |
| Gradient serialization | Clone preserves gradients | No conversion loss |
| Image asset download | Clone preserves images | Already in Figma |
| Slidev theme generation | Not needed | No Slidev |
| pixelmatch pixel diff | get_screenshot comparison | Same rendering engine |
| preset.md files | Not needed | Template IS the Figma file |
| Slidev export pipeline | Playwright export | Direct from Figma |
