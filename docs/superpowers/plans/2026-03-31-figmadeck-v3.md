# Figmadeck v3: Figma-Native Pipeline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite figmadeck to work directly inside Figma (clone page → adapt → QA) instead of extracting to Slidev.

**Architecture:** Complete rewrite of SKILL.md + 4 reference files. Delete old Slidev-specific files, create new Figma-native generation and auth references. design-rules.md and demo-outline.md stay unchanged.

**Tech Stack:** Figma MCP (`use_figma`, `get_screenshot`, `get_metadata`), Playwright (auth + export), Figma Plugin API.

**Spec:** `docs/superpowers/specs/2026-03-31-figmadeck-v3-figma-native.md`

---

## File Structure

```
figmadeck/.claude/skills/figmadeck/
  SKILL.md                      # REWRITE (110 → ~120 lines)
  references/
    figma-generation.md          # CREATE (replaces generation-pipeline.md + figma-extraction.md)
    qa-cycle.md                  # REWRITE
    figma-learning.md            # REWRITE
    auth.md                      # CREATE
    design-rules.md              # UNCHANGED
    slidev-syntax.md             # DELETE
    generation-pipeline.md       # DELETE
    figma-extraction.md          # DELETE
  assets/
    demo-outline.md              # UNCHANGED
```

---

### Task 1: Delete old files + create auth.md

**Files:**
- Delete: `figmadeck/.claude/skills/figmadeck/references/slidev-syntax.md`
- Delete: `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md`
- Delete: `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md`
- Create: `figmadeck/.claude/skills/figmadeck/references/auth.md`

- [ ] **Step 1: Delete old files**

```bash
cd figmadeck
rm .claude/skills/figmadeck/references/slidev-syntax.md
rm .claude/skills/figmadeck/references/generation-pipeline.md
rm .claude/skills/figmadeck/references/figma-extraction.md
```

- [ ] **Step 2: Write auth.md**

Write `figmadeck/.claude/skills/figmadeck/references/auth.md` with the complete Playwright authentication flow. This file must contain:

```markdown
# Figma Authentication (Playwright)

Figmadeck uses Playwright to authenticate with Figma for export operations. Credentials are stored locally, never in the repository.

## Credential Storage

File: `~/.claude/figmadeck-auth.json`

This file is stored in the user's home directory (outside any git repository). It contains Playwright browser cookies/session tokens — NOT plain-text passwords.

## First Run Flow

1. Check if `~/.claude/figmadeck-auth.json` exists
2. If NO → ask the user:
   ```
   Figma authentication required for export.
   Email: [wait for input]
   Password: [wait for input]
   ```
3. Launch Playwright (chromium):
   ```
   Navigate to https://www.figma.com/login
   Fill email field → submit
   Fill password field → submit
   ```
4. If 2FA form appears:
   ```
   Two-factor authentication detected.
   Enter your 2FA code: [wait for input]
   ```
   Fill 2FA field → submit
5. If login succeeds (redirected to figma.com dashboard or file):
   - Extract cookies: `context.cookies()`
   - Save to `~/.claude/figmadeck-auth.json`:
     ```json
     {
       "cookies": [...],
       "savedAt": "ISO-8601 timestamp"
     }
     ```
   - Print: `✓ Figma authentication successful. Session saved.`
6. If login fails (still on login page, error message visible):
   - Print error: `✗ Login failed: <error text from page>. Please try again.`
   - Return to step 2

## Subsequent Runs

1. Read `~/.claude/figmadeck-auth.json`
2. Launch Playwright, add saved cookies: `context.addCookies(saved.cookies)`
3. Navigate to `https://www.figma.com` (not login page)
4. Check: is the page the dashboard or a file? (Not redirected to /login?)
5. If YES → session is alive → return success
6. If NO (redirected to /login) → session expired:
   - Print: `⚠ Figma session expired. Re-authentication required.`
   - Delete `~/.claude/figmadeck-auth.json`
   - Run First Run Flow

## Usage in Pipeline

Auth check is the FIRST step of any operation that needs Playwright:
- `/figmadeck --export` — always needs auth
- `/figmadeck <url> <outline>` — needs auth only if export is requested after generation

Auth is NOT needed for `use_figma` MCP calls — those use the Figma MCP server's own authentication, not Playwright.

## Security Notes

- NEVER commit `~/.claude/figmadeck-auth.json` — it's outside the repo by design
- NEVER store plain-text passwords — only browser cookies
- NEVER log cookie values — only log success/failure status
- Cookies may expire (typically 30 days) — the subsequent run flow handles re-auth automatically
```

- [ ] **Step 3: Verify**

```bash
ls figmadeck/.claude/skills/figmadeck/references/
```
Expected: `auth.md`, `design-rules.md`, `figma-learning.md`, `qa-cycle.md` — no `slidev-syntax.md`, `generation-pipeline.md`, `figma-extraction.md`

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add -A .claude/skills/figmadeck/references/
git commit -m "feat(figmadeck): v3 — delete Slidev files, add auth.md"
```

---

### Task 2: Create figma-generation.md

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/figma-generation.md`

- [ ] **Step 1: Write figma-generation.md**

Create `figmadeck/.claude/skills/figmadeck/references/figma-generation.md`. This is the core of the new pipeline — replaces both `generation-pipeline.md` and `figma-extraction.md`.

Read the spec `docs/superpowers/specs/2026-03-31-figmadeck-v3-figma-native.md` Steps 2-5 (lines 62-135) for authoritative content.

The file must contain ALL of these sections with full detail:

**## Step 2: Clone Template Page**
- One `use_figma` call
- Find Page 1 via `figma.root.children[0]`
- `await figma.setCurrentPageAsync(page1)` — CRITICAL: page context resets each call
- Create new page: `figma.createPage()`, name = `"Generated: <outline-name>"`
- Clone all children: for loop with `child.clone()` → `newPage.appendChild(clone)` → preserve `clone.x = child.x; clone.y = child.y`
- Return `{ pageId: newPage.id, slides: [{id, name, origId, x}] }`
- Note: clone preserves EVERYTHING (gradients, images, shadows, auto-layout, variable bindings)

**## Step 3: Analyze Template Slides**
- One `use_figma` call on the cloned page
- `await figma.setCurrentPageAsync(generatedPage)` at start
- For each top-level frame: collect slide map with id, name, index, contentType, textNodes[]
- Each text node: id, name, characters, fontSize, absoluteTransform position, width, height, role
- **Text node role detection** — hybrid (name + heuristics):
  - Priority 1: node.name contains "Title"/"Heading" → title
  - Priority 2: node.name contains "description"/"body"/"desc" → description
  - Priority 3: largest fontSize ≥ 36px → title
  - Priority 4: second largest, > 50 chars → description
  - Priority 5: y > 90% frame height, fontSize < 14px → footer
  - Priority 6: < 20 chars, all uppercase → label/eyebrow
  - Priority 7: everything else → body
- **Content type detection**:
  - First slide + large heading → intro
  - Last slide + CTA text → cta
  - Large standalone number → metric
  - 3+ similar child frames → cards
  - Mostly images, little text → visual-break
  - Default → content

**## Step 4: Match Outline → Template Slides**
- Parse outline: count slides, determine content type for each
- For each outline slide → find best matching template slide by content type
- Multiple templates of same type → context-based variant selection (warm colors → positive themes, cool → analytical)
- Outline has MORE slides than templates of that type → additional `use_figma` call: `node.clone()` within the page
- Unused template slides → `node.remove()`
- Reorder remaining by adjusting `node.x` (left to right = presentation order)

**## Step 5: Fill Content**
- One `use_figma` call PER SLIDE (incremental)
- `await figma.setCurrentPageAsync(generatedPage)` at start of EVERY call
- **Font handling:**
  1. Check availability via `listAvailableFontsAsync()`
  2. If available → `loadFontAsync(font)`
  3. If NOT → fallback: same style in similar family. Known table: Whyte→Inter, Darker Grotesque→Manrope, custom serif→DM Serif Display. Last resort: Inter. Log every substitution.
  4. `node.fontName = fallback` (if needed) → `node.characters = newText`
- **Text overflow handling:**
  1. `node.textAutoResize = "HEIGHT"` (text grows down)
  2. Check `node.y + node.height > parentFrame.height`
  3. If overflow: shorten/rephrase → reduce fontSize (min current × 0.85) → expand container → last resort: note below slide
- **Special characters:** use `includes()` not `===` for text matching (Unicode differences after clone)
- **Footer:** find by position (bottom) + small fontSize, not by exact text

**## Important: use_figma Rules**
- MUST `await figma.setCurrentPageAsync(page)` at start of EVERY `use_figma` call — page context resets between calls
- MUST `await figma.loadFontAsync(font)` before ANY text changes
- `layoutSizingHorizontal = "FILL"` MUST be set AFTER `parent.appendChild(child)`
- MUST `return` all created/mutated node IDs
- Use `return` for output (not console.log, not figma.notify)
- Colors are 0-1 range, not 0-255
- On error: STOP, read error, fix, retry

Target: ~200-250 lines.

- [ ] **Step 2: Verify**

```bash
grep -c "Step [2-5]\|clone\|use_figma\|loadFontAsync\|textAutoResize\|setCurrentPageAsync" figmadeck/.claude/skills/figmadeck/references/figma-generation.md
```
Expected: 10+

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-generation.md
git commit -m "feat(figmadeck): v3 — add figma-generation.md (clone → analyze → match → fill)"
```

---

### Task 3: Rewrite qa-cycle.md (Figma-native)

**Files:**
- Rewrite: `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md`

- [ ] **Step 1: Write new qa-cycle.md**

Completely replace `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md` with the Figma-native QA cycle.

Read spec lines 137-213 for authoritative content.

The file must contain:

**## Overview**
- Runs after Step 5 (fill content). Loops until Fidelity Score ≥ 9/10.
- Safety stop: 5 iterations with score delta < 0.3.
- All checks run INSIDE Figma — no Slidev export, no HTML/CSS comparison.

**## Phase A: Structural Check (programmatic)**
- One `use_figma` call per slide (or batch per page)
- `await figma.setCurrentPageAsync(generatedPage)` at start
- **Overlap detection**: for each pair of visible nodes on a slide, check bounding box intersection using `absoluteTransform` + `width`/`height`. Intersection → **CRITICAL**
- **Boundary check**: for each TEXT node, `node.y + node.height > parent.height` → **CRITICAL**
- **Gap check**: distance between adjacent elements < 8px → **FAIL**
- **Footer zone**: content enters bottom 44-56px of frame → **FAIL**
- **Text truncation**: `node.textAutoResize === "TRUNCATE"` or text clipped → **CRITICAL**
- Return: `{ slide: N, issues: [{type: "CRITICAL"|"FAIL", description, nodeIds}] }`

**## Phase B: Visual Comparison**
- For each slide:
  1. `mcp__figma__get_screenshot(originalSlideNodeId, fileKey)` → original template screenshot
  2. `mcp__figma__get_screenshot(adaptedSlideNodeId, fileKey)` → adapted version screenshot
  3. Compare visually (agent sees both images): same style? same spacing? same mood?
  4. Content WILL differ (different text) — compare STYLE not content
  5. Check: fonts look the same? colors unchanged? whitespace balanced? no visual glitches?

**## Phase C: Design Critique**
- Read `references/design-rules.md` before running
- Per slide, evaluate:
- **Element Integrity (highest priority):**
  - Text overlaps another element → **CRITICAL** — shorten or reposition
  - Text extends beyond container → **CRITICAL** — textAutoResize + check
  - Elements too close (< 8px gap) → **FAIL** — increase gap
  - Footer zone occupied → **FAIL** — push content up
- **Visual Hierarchy:**
  - Focal point preserved? Heading is largest element?
  - Logical reading order?
- **Consistency:**
  - Slide looks like same presentation as original?
  - Fonts, colors, spacing uniform?

**## Phase D: Score + Fix**
- Scoring: Structural (40%) + Visual (30%) + Design Critique (30%)
- Structural: 10 if zero CRITICAL/FAIL. Deduct 3 per CRITICAL, 1 per FAIL (min 0).
- Visual: agent score 1-10 based on style correspondence
- Design Critique: 10 if zero integrity issues. Deduct 3 per CRITICAL, 1 per FAIL.
- Target: ≥ 9/10
- **Fixes via `use_figma`** (one call per fix, incremental):
  - Overlap → `node.resize(smallerWidth, height)` or shorten `node.characters`
  - Boundary → `textAutoResize = "HEIGHT"`, reduce fontSize
  - Gap → adjust `node.y` or `node.x`
  - Footer → shift elements up
- Clean up after iteration, proceed to next
- **Iteration report** format (from spec line 204-213)

Target: ~120-150 lines.

- [ ] **Step 2: Verify**

```bash
grep -c "Phase [ABCD]\|use_figma\|get_screenshot\|CRITICAL\|FAIL\|Structural (40%)" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md
```
Expected: 10+

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/qa-cycle.md
git commit -m "feat(figmadeck): v3 — rewrite qa-cycle.md for Figma-native QA"
```

---

### Task 4: Rewrite figma-learning.md

**Files:**
- Rewrite: `figmadeck/.claude/skills/figmadeck/references/figma-learning.md`

- [ ] **Step 1: Write new figma-learning.md**

Completely replace `figmadeck/.claude/skills/figmadeck/references/figma-learning.md` with the simplified Figma-native learning procedure.

Read spec lines 225-246 for authoritative content.

The file must contain:

**## Overview**
- Triggered by `/figmadeck <figma-url> --learn=N`. Max N: 10.
- Uses Figma-native pipeline (clone → fill → QA), NOT Slidev.
- Fixes go to **skill files** (mapping heuristics, font table), NOT to Figma file.

**## FDL-1: Initialization**
- Scan for existing `edu_*` directories. Create next sequential (MANDATORY scan — `ls -d edu_*`).
- Store fileKey and page info for the learning session.

**## FDL-0: Calibration (learn_0)**
- Clone template page → do NOT change any text → run QA comparing clone vs original
- Should be 100% match (clone = perfect copy)
- If learn_0 fails → pipeline tooling is broken, not content adaptation
- This validates: page creation works, screenshot comparison works, scoring works

**## FDL-2: Generate N Diverse Outlines**
- All in Russian. Varied industries, formats, sizes, tone.
- Save to `<edu_dir>/learn_<i>/outline.md`

**## FDL-3: Learning Cycle (i = 1..N)**
- **FDL-3a**: Clone template → fill with outline via `figma-generation.md`
- **FDL-3b**: Run `qa-cycle.md` until ≥ 9/10
- **FDL-3c**: Record which text replacements caused problems:
  - Which roles overflow most? (title/description/cards)
  - Which font fallbacks work well/poorly?
  - Which content types need more aggressive shortening?
- **FDL-3d**: Intermediate report (fidelity, issues found, patterns)
- **FDL-3e**: Git commit + tag `figmadeck-learn-<edu_dir>-iter-<i>`

**## FDL-4: Convergence**
- ≥ 9/10 for two consecutive cycles → early stop

**## FDL-5: Final Report**
- Fidelity progression, adaptation patterns discovered, font fallback effectiveness, recommendations
- Save to `<edu_dir>/figmadeck-learning-report.md`

Target: ~100-120 lines.

- [ ] **Step 2: Verify**

```bash
grep -c "FDL-[0-5]\|learn_0\|clone\|qa-cycle\|convergence" figmadeck/.claude/skills/figmadeck/references/figma-learning.md
```
Expected: 8+

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-learning.md
git commit -m "feat(figmadeck): v3 — rewrite figma-learning.md for Figma-native pipeline"
```

---

### Task 5: Rewrite SKILL.md

**Files:**
- Rewrite: `figmadeck/.claude/skills/figmadeck/SKILL.md`

- [ ] **Step 1: Write new SKILL.md**

Completely replace `figmadeck/.claude/skills/figmadeck/SKILL.md` with the v3 Figma-native entry point.

The file must contain ALL of this (adapt from current SKILL.md structure but with new commands/dispatch):

```markdown
---
name: figmadeck
description: Create presentations directly in Figma. Clones a template page, adapts it to an outline by replacing text and reordering slides, then runs QA cycles for design fidelity. Also handles --learn, --export, --edit subcommands.
argument-hint: "[<figma-url> <outline> | <figma-url> --learn=N | --export pdf|png [figma-url] | --edit <figma-url> <comment> | --help]"
---

# Figmadeck — Figma-Native Presentation Generator

Create presentations by cloning and adapting Figma templates directly. No conversion to HTML/CSS — work inside Figma, preserving every gradient, shadow, image, and font exactly.

## References

Before executing, read the relevant reference for the current mode:
- `references/figma-generation.md` — **Generation mode**: clone → analyze → match → fill
- `references/qa-cycle.md` — **QA**: Figma-native structural + visual + design critique cycle
- `references/figma-learning.md` — **Learn mode**: calibration + iterative improvement
- `references/auth.md` — **Authentication**: Playwright login to Figma
- `references/design-rules.md` — **Universal design rules** (readability, spacing, hierarchy)

**MANDATORY**: Before any `use_figma` call, load the `figma-use` skill (from the Figma MCP plugin). It contains critical Plugin API rules.

## Input Parsing

### Help

**`--help`**: Display usage and stop:

```
Figmadeck — Figma-Native Presentation Generator

Usage:
  /figmadeck <figma-url> <outline>          Create presentation in Figma
  /figmadeck <figma-url> --learn=N          Calibration + learning (N cycles)
  /figmadeck --export pdf|png [figma-url]   Export via Playwright
  /figmadeck --edit <figma-url> <comment>   Edit existing generated page
  /figmadeck --help                         Show this help
```

### Subcommands

**`--export <format> [figma-url]`**: Export presentation.
1. Run auth check (`references/auth.md`)
2. Playwright: open Figma file → navigate to generated page ("Generated: ...")
3. For `pdf`: File → Export PDF, or Presentation Mode → Print to PDF
4. For `png`: `get_screenshot` per slide → save locally
5. Fallback: export each frame via `get_screenshot` → combine to PDF via Python/Pillow

**`--edit <figma-url> <comment>`**: Edit existing presentation.
1. Open Figma file, find generated page (name prefix "Generated:")
2. Read user's edit comment
3. Apply changes via `use_figma`
4. Run QA cycle (`references/qa-cycle.md`) to validate

### Mode Detection

1. **Figma URL + `--learn=N`** → Learning mode: read `references/figma-learning.md`. Max N: 10.
2. **Figma URL + outline** → Generation mode: read `references/figma-generation.md`, then `references/qa-cycle.md`.
3. **`--export`** → Export mode: read `references/auth.md`, execute export.
4. **`--edit`** → Edit mode: find generated page, apply changes, run QA.

### URL Parsing

Extract `fileKey` from Figma URL:
- `figma.com/design/:fileKey/:fileName` → fileKey
- `figma.com/design/:fileKey/branch/:branchKey/:fileName` → use branchKey
- If URL is not `figma.com/design/...` → error

## Hard Constraint

**The generated presentation MUST have exactly the same number of slides, in exactly the same order, with exactly the same content as the outline.** Never add, remove, merge, or reorder slides beyond what the outline specifies.

### Content Adaptation

When outline content is longer than the template placeholder text:
1. **Shorten/rephrase** to fit (preserve core meaning)
2. **textAutoResize = "HEIGHT"** to let text grow
3. **Reduce fontSize** (minimum = current × 0.85)
4. Last resort: move overflow detail to a note

## Autonomous Execution

**CRITICAL:** Execute all steps autonomously without stopping for confirmation. If blocked, attempt to resolve (2-3 attempts). Only stop if truly unresolvable.
```

- [ ] **Step 2: Verify**

```bash
wc -l figmadeck/.claude/skills/figmadeck/SKILL.md
grep -c "figma-generation\|qa-cycle\|figma-learning\|auth\|design-rules" figmadeck/.claude/skills/figmadeck/SKILL.md
```
Expected: < 130 lines, 5+ reference matches

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/SKILL.md
git commit -m "feat(figmadeck): v3 — rewrite SKILL.md for Figma-native pipeline"
```

---

### Task 6: Final verification + push

**Files:**
- All files from Tasks 1-5

- [ ] **Step 1: Verify file structure**

```bash
ls figmadeck/.claude/skills/figmadeck/references/
```
Expected: `auth.md`, `design-rules.md`, `figma-generation.md`, `figma-learning.md`, `qa-cycle.md` — exactly 5 files. NO `slidev-syntax.md`, `generation-pipeline.md`, `figma-extraction.md`.

- [ ] **Step 2: Cross-reference check**

```bash
# SKILL.md references all 5 reference files
grep "references/" figmadeck/.claude/skills/figmadeck/SKILL.md

# figma-generation mentions qa-cycle
grep "qa-cycle" figmadeck/.claude/skills/figmadeck/references/figma-generation.md

# figma-learning mentions figma-generation and qa-cycle
grep "figma-generation\|qa-cycle" figmadeck/.claude/skills/figmadeck/references/figma-learning.md

# No Slidev references anywhere
grep -r "slidev\|Slidev\|slides\.md\|npm install\|npx slidev" figmadeck/.claude/skills/figmadeck/ | grep -v "design-rules"
```

Expected: first 3 return matches, last returns empty (no Slidev references except possibly in design-rules.md which is unchanged).

- [ ] **Step 3: No stale references**

```bash
# No blueprint.json references
grep -r "blueprint.json" figmadeck/.claude/skills/figmadeck/

# No preset.md references
grep -r "preset\.md\|\.slidev-presets" figmadeck/.claude/skills/figmadeck/

# No pixelmatch references (old v2)
grep -r "pixelmatch\|diffRatio" figmadeck/.claude/skills/figmadeck/
```

Expected: all empty.

- [ ] **Step 4: Push**

```bash
git push
```
