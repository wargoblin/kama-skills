# Figmadeck Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a standalone `/figmadeck` skill that generates Slidev presentations from Figma templates with iterative QA comparison against the Figma source.

**Architecture:** All-new skill in `figmadeck/.claude/skills/figmadeck/`. Compact SKILL.md (~300 lines) dispatches to 5 reference files for procedures. No dependency on the slidev skill. Copies `slidev-syntax.md` as the only shared reference.

**Tech Stack:** Figma MCP tools (`use_figma`, `get_screenshot`, `get_metadata`, `get_design_context`), Slidev, markdown instruction files.

**Spec:** `docs/superpowers/specs/2026-03-30-figmadeck-design.md`

---

## File Structure

All files are **new** (create):

```
figmadeck/
  .gitignore
  README.md
  .claude/skills/figmadeck/
    SKILL.md
    references/
      slidev-syntax.md
      generation-pipeline.md
      qa-cycle.md
      figma-extraction.md
      figma-learning.md
    assets/
      demo-outline.md
```

---

### Task 1: Scaffolding — directory structure, .gitignore, README

**Files:**
- Create: `figmadeck/.gitignore`
- Create: `figmadeck/README.md`
- Create: directories for `.claude/skills/figmadeck/references/` and `assets/`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p "figmadeck/.claude/skills/figmadeck/references" "figmadeck/.claude/skills/figmadeck/assets"
```

- [ ] **Step 2: Write .gitignore**

Write `figmadeck/.gitignore`:

```gitignore
# Ignore all experiment files in this skill's test directory
*

# But keep these
!.gitignore
!.claude/
!.claude/**
!README.md
!.slidev-presets/
!.slidev-presets/**
```

- [ ] **Step 3: Write README.md**

Write `figmadeck/README.md`:

```markdown
# Figmadeck

Generate Slidev presentations from Figma design templates.

## Usage

```
/figmadeck <figma-url>                   Create preset from Figma template
/figmadeck <figma-url> --learn=N         Create preset + learning loop
/figmadeck <outline>                     Generate presentation (auto-match preset)
/figmadeck --preset <name> <outline>     Generate with specific preset
/figmadeck --export pdf [dir]            Export to PDF
/figmadeck --dev [dir]                   Launch dev server
/figmadeck --edit [dir] <comment>        Edit presentation
```

## How It Works

1. Extract a Figma presentation template into a preset with HTML archetypes
2. Generate presentations by filling archetype slots with outline content (verbatim copy)
3. Iterative QA compares each slide against the Figma original until fidelity >= 9/10
```

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add .gitignore README.md .claude/
git commit -m "feat(figmadeck): scaffold skill directory structure"
```

---

### Task 2: SKILL.md — entry point

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Write the full content of `figmadeck/.claude/skills/figmadeck/SKILL.md`:

```markdown
---
name: figmadeck
description: Generate Slidev presentations from Figma design templates. Extracts archetypes from Figma, generates slides by filling template slots, and iteratively compares against the Figma original until quality reaches 9/10. Also handles --learn, --export, --dev, --edit subcommands.
argument-hint: "[<figma-url> [--learn=N] | --preset <name> <outline> | <outline> | --export <format> [dir] | --dev [dir] | --edit [dir] <comment> | --help]"
---

# Figmadeck — Figma-to-Slidev Presentation Generator

Generate presentations by copying Figma design templates exactly. Every slide is a verbatim HTML archetype with slots filled from the outline. Quality is verified by comparing against the live Figma source.

## References

Before generating, read the relevant reference for the current mode:
- `references/slidev-syntax.md` — Slidev markdown syntax (slide separators, frontmatter, layouts)
- `references/generation-pipeline.md` — **Generation mode**: Steps 1-5 (outline → slides.md)
- `references/qa-cycle.md` — **QA**: Iterative Figma comparison until 9/10 fidelity
- `references/figma-extraction.md` — **Extraction mode**: FIG-1 through FIG-5 (Figma → preset)
- `references/figma-learning.md` — **Learn mode**: FDL iterative archetype improvement

## Input Parsing

Parse the user's input to determine the mode:

### Help

**`--help`**: Display usage and stop:

```
Figmadeck — Figma-to-Slidev Presentation Generator

Usage:
  /figmadeck <figma-url>                   Create preset from Figma template
  /figmadeck <figma-url> --learn=N         Create preset + learning loop (N cycles)
  /figmadeck <outline>                     Generate (auto-match Figma preset)
  /figmadeck --preset <name> <outline>     Generate with specific Figma preset
  /figmadeck --export <format> [dir]       Export (pdf|png)
  /figmadeck --dev [dir]                   Launch dev server
  /figmadeck --edit [dir] <comment>        Edit existing presentation
  /figmadeck --help                        Show this help
```

### Subcommands

**`--export <format> [dir]`**: Export existing presentation.
1. Resolve project directory (from `dir` or auto-detect `slides.md`)
2. Install deps if needed: `npm install && npx playwright install chromium`
3. Format `pdf`: `npx slidev export --format png --output slides-tmp` → combine PNGs to PDF via Pillow → clean up
4. Format `png`: `npx slidev export --format png --output slides`

**`--dev [dir]`**: Launch dev server.
1. Resolve project directory
2. `npm install` if needed
3. `npx slidev --open`

**`--edit [dir] <comment>`**: Edit existing presentation.
1. Resolve project directory, read `slides.md`
2. Apply the user's edit comment (modify slides.md as requested)
3. Run QA cycle (`references/qa-cycle.md`) to validate changes
4. Report what changed

### Mode Detection

After subcommand check, determine the main mode:

1. **Figma URL detected** (`figma.com/design/...` in input):
   - Parse `fileKey` and optional `nodeId` from URL (convert `-` to `:` in nodeId; for branch URLs use branchKey as fileKey; default nodeId `0:1`)
   - If `--learn=N` present → **Extraction + Learning mode**: read `references/figma-extraction.md`, then `references/figma-learning.md`. Max N: 10.
   - If no `--learn` → **Extraction mode**: read `references/figma-extraction.md`. After extraction, generate demo using `assets/demo-outline.md` with the new preset, run QA cycle, print summary. Stop.

2. **`--preset <name>` specified** → **Generation mode**: resolve preset (see Preset Resolution below), read `references/generation-pipeline.md`, then `references/qa-cycle.md`.

3. **`<outline>` without `--preset`** → **Auto-match + Generation**: run Auto-Match (below), then read `references/generation-pipeline.md`, then `references/qa-cycle.md`.

### Preset Resolution

Resolve preset name using this lookup (local takes priority):
1. **Direct path**: name contains `/`, `\`, or ends in `.md` → read as file path
2. **Local**: `./.slidev-presets/<name>.preset.md`
3. **Global**: `~/.claude/slidev-presets/<name>.preset.md`

If not found → error listing paths tried.

### Auto-Match (no --preset specified)

1. Scan `.slidev-presets/` and `~/.claude/slidev-presets/` — collect only files with `figma:` section in YAML frontmatter
2. If no Figma presets found → error: `No Figma presets found. Run /figmadeck <figma-url> first.`
3. LLM matching: evaluate outline topic against each Figma preset's name and aesthetic description
4. If match → use it
5. If no match → use default (first found Figma preset)

## Hard Constraint

**The generated presentation MUST have exactly the same number of slides, in exactly the same order, with exactly the same content as the outline.** Never add, remove, merge, or reorder slides. The outline is the contract.

### Font Size Exemption

When content overflows the slide at minimum font size:
1. **Shorten/rephrase** the content to fit elegantly (any content, not just titles — preserving core meaning)
2. **Move secondary details** to speaker notes
3. **Split slide** as last resort
Priority: shorten first → speaker notes second → split last. **NEVER reduce font below minimum** (body ≥ 1.25rem, heading ≥ 2.2rem, label ≥ 0.65rem).

### Visual Rhythm Override

When the outline has 4+ consecutive dense content slides without a natural section break → use `figma-section` archetype (if present in the preset) as a breathing slide for one of those slides. Choose the slide with the single most impactful number, quote, or insight — present it as a dramatic centerpiece. This is NOT adding slides — it's changing the archetype used for an existing outline slide.

## Autonomous Execution

**CRITICAL:** When executing, DO NOT stop to ask for confirmation between steps. Execute all steps autonomously. If you encounter a blocker, attempt to resolve it (2-3 attempts). Only stop if truly unresolvable.
```

- [ ] **Step 2: Verify SKILL.md structure**

Run: `grep -c "^#\|^##\|^###" figmadeck/.claude/skills/figmadeck/SKILL.md`
Expected: 15+ section headings

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/SKILL.md
git commit -m "feat(figmadeck): add SKILL.md entry point with dispatch logic"
```

---

### Task 3: slidev-syntax.md + demo-outline.md

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/slidev-syntax.md` (copy from slidev)
- Create: `figmadeck/.claude/skills/figmadeck/assets/demo-outline.md`

- [ ] **Step 1: Copy slidev-syntax.md**

```bash
cp slidev/.claude/skills/slidev/references/slidev-syntax.md figmadeck/.claude/skills/figmadeck/references/slidev-syntax.md
```

- [ ] **Step 2: Write demo-outline.md**

Write `figmadeck/.claude/skills/figmadeck/assets/demo-outline.md`:

```markdown
# Design Preview

## Slide 1: Cover
- Title: "Design Preview"
- Subtitle: "A visual showcase of this Figma template"

## Slide 2: Section — Overview
- "What this template can do"
- Brief description of capabilities

## Slide 3: Content — Key Features
- Feature 1: Consistent typography from Figma
- Feature 2: Exact color palette extraction
- Feature 3: Layout fidelity at pixel level

## Slide 4: Metrics
- Number: "9/10"
- Subtext: "Target Figma fidelity score"

## Slide 5: CTA — Get Started
- "Create your first presentation"
- "Run /figmadeck --preset <name> <outline>"
```

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/slidev-syntax.md .claude/skills/figmadeck/assets/demo-outline.md
git commit -m "feat(figmadeck): add slidev-syntax reference and demo outline"
```

---

### Task 4: generation-pipeline.md — Steps 1-5

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md`

- [ ] **Step 1: Write generation-pipeline.md**

Write `figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md` with the full generation procedure. The content MUST follow the spec section "generation-pipeline.md: Steps 1-5" exactly. Read the spec at `docs/superpowers/specs/2026-03-30-figmadeck-design.md` lines 99-148 for the authoritative content.

The file must include these sections with their full detail:

1. **Step 1: Parse Outline** — read outline (inline or file), count slides, extract content, determine content types
2. **Step 2: Ghost Deck Test** — rewrite label-titles to action-titles, shorten content that's clearly too long
3. **Step 3: Scaffolding** — copy theme from preset, write package.json/uno.config.ts, create Icon.vue ONLY if archetypes use `<Icon>`
4. **Step 4: Composition Planning** — match content types to Figma archetypes WITHIN current preset only, visual rhythm check, output composition table
5. **Step 5: Write slides.md** — VERBATIM COPY of archetype.html, replace only `{{SLOT}}` markers, check text fit via flexibility.yaml, adapt element count, add per-slide frontmatter
6. **PROHIBITED actions list** — the 7 explicit prohibitions from the spec (no rewriting HTML, no replacing inline styles, no adding elements, etc.)

Include the flexibility.yaml slot filling rules:
- Repeating elements: scaling strategies (grid-auto, wrap, stack)
- Text slots: overflow strategies (shorten/rephrase, speaker notes)
- Preserved properties: spacing-ratio, font-size-hierarchy, color-usage, border-radius

The file should be ~150-200 lines and self-contained (an agent reads ONLY this file + SKILL.md to generate a presentation).

- [ ] **Step 2: Verify structure**

Run: `grep -c "Step [1-5]" figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md`
Expected: 5

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/generation-pipeline.md
git commit -m "feat(figmadeck): add generation pipeline (Steps 1-5)"
```

---

### Task 5: qa-cycle.md — iterative QA with Figma comparison

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md`

- [ ] **Step 1: Write qa-cycle.md**

Write `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md` with the full QA cycle procedure. Read the spec section "qa-cycle.md" (lines 151-222) for authoritative content.

The file must include:

1. **Overview**: Runs after slides.md is generated. Loops until Fidelity Score >= 9/10.
2. **Phase A: CSS + Figma Structural/Style Comparison**
   - Basic checks: font-size floor (body >= 1.25rem, heading >= 2.2rem, label >= 0.65rem), layout:none enforcement, hardcoded hex → var(--*), content overflow check
   - Figma structural comparison using `mcp__figma__get_metadata(nodeId, fileKey)` — positions (+-5%), proportions (+-10%), hierarchy (exact)
   - Figma style comparison using `mcp__figma__use_figma` — the full tolerance table (font-size +-0.15rem, font-weight exact, color deltaE<5, spacing +-0.25rem, padding +-0.5rem, border-radius +-2px, position +-5%)
   - Fix all issues → modify slides.md
3. **Phase B: Visual Export + Figma Screenshot Comparison**
   - Export PNG: `npx slidev export --format png --output slides-qa`
   - For EACH slide: `mcp__figma__get_screenshot(nodeId, fileKey)` → compare side by side (visual impression, color mood, typography, whitespace, content cut-off, footer)
   - For adapted archetypes: verify it looks like it belongs in the same presentation
   - Fix all visual issues → modify slides.md
   - Clean up QA PNGs after iteration
4. **Phase C: Score**
   - Fidelity = Visual (40%) + Structural (30%) + Style (30%)
   - Score >= 9/10 → DONE
   - Score < 9/10 → next iteration
   - Safety stop: 5 iterations with delta < 0.3 → stop with report
5. **Iteration Report format** (the ━━━ QA Iteration block)
6. **Comparison scope**: EVERY slide is compared, including those with adapted archetypes — compare against closest Figma original from the same preset
7. **Figma API calls**: Live on each iteration (not cached), except fallback to cached blueprint.json + reference.png when API unavailable

The file should be ~120-150 lines.

- [ ] **Step 2: Verify structure**

Run: `grep -c "Phase [ABC]" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md`
Expected: 3

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/qa-cycle.md
git commit -m "feat(figmadeck): add iterative QA cycle with Figma comparison"
```

---

### Task 6: figma-extraction.md — FIG-1 through FIG-5

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md`

- [ ] **Step 1: Write figma-extraction.md**

Write `figmadeck/.claude/skills/figmadeck/references/figma-extraction.md` with the full Figma extraction procedure. This is based on the existing procedure in `slidev/.claude/skills/slidev/SKILL.md` (lines 449-700), adapted for the figmadeck namespace.

The file must include:

1. **FIG-1: File Structure Reconnaissance** — `mcp__figma__use_figma` JS to find 16:9 frames, sort by position. Error if 0 frames. Max 20 frames.
2. **FIG-2: Global Style Extraction** — `mcp__figma__use_figma` JS to collect: Colors (bg-base, bg-alt, accent, text, muted + AI blacklist check), Fonts (heading/body + serif check), Spacing (median itemSpacing/padding), Shapes (cornerRadius → card_radius, icon_container), Effects (shadows, blurs). Convert to .preset.md format.
3. **FIG-3: Per-Slide Extraction** — 4 levels: Level 1 screenshot via `get_design_context` + `get_screenshot` (save reference.png), Level 2 structure via `get_metadata`, Level 3 detailed properties via `use_figma` JS (save blueprint.json), Level 4 code hint from `get_design_context`.
4. **FIG-4: Archetype Conversion** — Step 4-pre deduplication (5 strict criteria: children count, layoutMode, grid structure, element type sequence, nesting depth). Image-only → visual-break archetype. Content type detection heuristics. HTML skeleton with `{{SLOT}}` markers. Viewport conversion (Figma coords → 960x540). Flexibility.yaml generation.
5. **FIG-5: Preset Assembly** — Directory structure (`figmadeck-<name>-*`), preset frontmatter with `figma:` section, archetypes block, theme directory, source.json. Print extraction summary.

Include the blueprint.json example format and flexibility.yaml example format from the existing spec.

**Namespace**: use `figmadeck-<name>-figma/`, `figmadeck-<name>-theme/`, `figmadeck-<name>.preset.md` everywhere.

Read the authoritative content from:
- Spec: `docs/superpowers/specs/2026-03-30-figmadeck-design.md` lines 255-263
- Detailed procedures: `slidev/.claude/skills/slidev/SKILL.md` lines 449-700 (adapt namespace from `figma-*` to `figmadeck-*`)

The file should be ~200-250 lines.

- [ ] **Step 2: Verify structure**

Run: `grep -c "FIG-[1-5]" figmadeck/.claude/skills/figmadeck/references/figma-extraction.md`
Expected: 5

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-extraction.md
git commit -m "feat(figmadeck): add Figma extraction procedure (FIG-1 through FIG-5)"
```

---

### Task 7: figma-learning.md — FDL learn mode

**Files:**
- Create: `figmadeck/.claude/skills/figmadeck/references/figma-learning.md`

- [ ] **Step 1: Write figma-learning.md**

Write `figmadeck/.claude/skills/figmadeck/references/figma-learning.md` with the learning procedure. Read spec lines 226-251 for authoritative content.

The file must include:

1. **Overview**: Triggered by `/figmadeck <URL> --learn=N`. Same QA cycle as generation, but fixes propagate to archetypes permanently. Max N: 10.
2. **Difference table**: Regular vs Learn (presentations count, where fixes go, result)
3. **FDL-1: Initialization** — Preset created by FIG-Extract. Create `edu_XX/` directory.
4. **FDL-2: Generate N diverse outlines** — Russian language, varied industries/formats/sizes/tone. Save to `<edu_dir>/learn_<i>/outline.md`.
5. **FDL-3: Learning cycle (i = 1..N)**:
   - FDL-3a: Generate presentation using `generation-pipeline.md` (re-read preset from disk each cycle)
   - FDL-3b: Run `qa-cycle.md` until 9/10
   - FDL-3c: Analyze systemic fixes — which qa-cycle fixes recur because of the archetype (not content-specific) → apply to archetype.html / preset.md / flexibility.yaml
   - FDL-3d: Intermediate report (fidelity score, fixes applied, archetype changes)
   - FDL-3e: Git commit + tag `figmadeck-learn-<edu_dir>-iter-<i>`
6. **FDL-4: Convergence** — fidelity >= 9/10 for two consecutive cycles → early stop
7. **FDL-5: Final report** — fidelity progression, archetype changes summary, flexibility updates, preset path

Include the intermediate report format and final report format.

The file should be ~100-130 lines.

- [ ] **Step 2: Verify structure**

Run: `grep -c "FDL-[1-5]" figmadeck/.claude/skills/figmadeck/references/figma-learning.md`
Expected: 5

- [ ] **Step 3: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-learning.md
git commit -m "feat(figmadeck): add Figma learning procedure (FDL-1 through FDL-5)"
```

---

### Task 8: Final consistency check + push

**Files:**
- All files from Tasks 1-7

- [ ] **Step 1: Verify all files exist**

```bash
ls -la figmadeck/.claude/skills/figmadeck/SKILL.md
ls -la figmadeck/.claude/skills/figmadeck/references/
ls -la figmadeck/.claude/skills/figmadeck/assets/
ls -la figmadeck/.gitignore figmadeck/README.md
```

Expected: SKILL.md + 5 reference files + 1 asset + .gitignore + README.md = 9 files total.

- [ ] **Step 2: Cross-reference check**

```bash
# SKILL.md references all 5 reference files
grep "references/" figmadeck/.claude/skills/figmadeck/SKILL.md

# generation-pipeline.md mentions qa-cycle.md
grep "qa-cycle" figmadeck/.claude/skills/figmadeck/references/generation-pipeline.md

# figma-learning.md references both generation-pipeline and qa-cycle
grep "generation-pipeline\|qa-cycle" figmadeck/.claude/skills/figmadeck/references/figma-learning.md

# All files use figmadeck-* namespace (not figma-*)
grep -r "figma-<name>" figmadeck/.claude/skills/figmadeck/ | grep -v "figmadeck-<name>"

# No references to slidev SKILL.md or design-principles.md
grep -r "design-principles\|composition-archetypes\|scoring-subroutine" figmadeck/.claude/skills/figmadeck/
```

Expected: first 3 greps return matches, last 2 return no matches.

- [ ] **Step 3: Verify SKILL.md is under ~300 lines**

```bash
wc -l figmadeck/.claude/skills/figmadeck/SKILL.md
```

Expected: < 350 lines.

- [ ] **Step 4: Final commit if needed**

```bash
cd figmadeck && git add .claude/skills/figmadeck/
git commit -m "feat(figmadeck): complete skill implementation — all references and assets"
```

- [ ] **Step 5: Push**

```bash
git push
```
