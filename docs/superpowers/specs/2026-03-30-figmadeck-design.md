# Figmadeck Skill Design

## Overview

A dedicated, lightweight skill (`/figmadeck`) for generating Slidev presentations from Figma templates. Built from scratch — no dependency on the slidev skill. Takes the best from the FDL learning loop (iterative Figma comparison) and applies it to every generation, not just learning.

**Why a separate skill:** The slidev SKILL.md (2500+ lines) has dozens of design rules that conflict with Figma templates (icons, bg-alt, standard archetypes, design-principles). Patching these with overrides didn't work — the agent still "drifts" during standalone generation. A clean skill with only Figma-relevant rules eliminates the conflict entirely.

**Trigger:** `/figmadeck`

## Non-Goals

- Replacing the slidev skill (it handles non-Figma generation)
- Generating presets from scratch (figmadeck only creates presets from Figma files)
- Standard archetypes, design-principles, scoring subroutines, AI-detection rubric

---

## File Structure

```
figmadeck/                                # Top-level: skill source + test playground
  .claude/skills/figmadeck/               # Actual skill (tracked by git)
    SKILL.md                              # Entry point (~300 lines)
                                          # Flags, dispatch, high-level rules
    references/
      figma-extraction.md                 # FIG-1 — FIG-5: extract from Figma
      generation-pipeline.md              # Steps 1-5: outline to slides.md
      qa-cycle.md                         # Iterative QA with Figma comparison
      figma-learning.md                   # FDL: learn mode (--learn=N)
      slidev-syntax.md                    # Slidev markdown syntax reference (copy from slidev skill)
    assets/
      demo-outline.md                     # Demo outline for testing
  .gitignore                              # Allowlist: *, !.gitignore, !.claude/, !.claude/**, !README.md, !.slidev-presets/, !.slidev-presets/**
  README.md                               # Skill docs
```

**Follows the repo convention** from CLAUDE.md: each skill = top-level directory with `.claude/skills/<name>/` inside. The `.gitignore` uses the allowlist pattern (ignore all `*`, then `!` keep `.claude/`, README, presets). Everything else in `figmadeck/` is test playground (gitignored).

**SKILL.md** (~300 lines): flag parsing, dispatch logic, Hard Constraint, Font Size Exemption, Visual Rhythm Override. Delegates to references for procedures.

**references/**: each file is self-contained. Agent reads SKILL.md → understands which reference to read → reads it → executes.

---

## SKILL.md: Commands & Dispatch

### Syntax

```
/figmadeck <URL>                         Create preset from Figma template
/figmadeck <URL> --learn=N               Create preset + FDL learning
/figmadeck <outline>                     Generate (auto-match or default preset)
/figmadeck --preset <name> <outline>     Generate with specific preset
/figmadeck --export <format> [dir]       Export (pdf|png)
/figmadeck --dev [dir]                   Dev server
/figmadeck --edit [dir] <comment>        Edit existing presentation
/figmadeck --help                        Show help
```

### Dispatch Logic

```
URL detected (figma.com/design/...)?
  ├─ yes + --learn=N → figma-extraction.md → figma-learning.md
  ├─ yes (no --learn) → figma-extraction.md → demo generation
  └─ no
       ├─ --preset <name> → resolve preset → generation-pipeline.md → qa-cycle.md
       ├─ --export/--dev/--edit → handle inline (short)
       └─ <outline> (no --preset) → auto-match → generation-pipeline.md → qa-cycle.md
```

### Auto-Match (no --preset specified)

1. Scan `.slidev-presets/` and `~/.claude/slidev-presets/` — only files with `figma:` in frontmatter
2. LLM matching: evaluate outline topic against each Figma preset's name and aesthetic description
3. If match found → use it
4. If no match → use default (first found Figma preset)
5. If no Figma presets exist → error: `No Figma presets found. Run /figmadeck <URL> first.`

### Hard Constraint

The generated presentation MUST have exactly the same number of slides, in exactly the same order, with exactly the same content as the outline. Never add, remove, merge, or reorder slides.

### Font Size Exemption

When text overflows the slide at minimum font size:
1. Shorten text (rephrase shorter — allowed for any content, not just titles)
2. Move secondary details to speaker notes
3. Split slide as last resort
Never reduce font below minimum.

### Visual Rhythm Override

When outline has 4+ consecutive dense content slides without a natural break → insert `figma-section` archetype as breathing slide. Safety net only — composition planning should handle this naturally.

---

## generation-pipeline.md: Steps 1-5

### Step 1: Parse Outline

Read outline (inline text or file path). Count slides, extract title and content for each. Determine content types.

### Step 2: Ghost Deck Test

Review all titles. Rewrite label-titles ("Our Team", "Market") into action-titles ("15 engineers scaling B2B products", "Market 24B RUB growing +22%"). Content can also be shortened/rephrased at this stage if it's clearly too long for a slide format.

### Step 3: Scaffolding

- Copy `figmadeck-<name>-theme/` from preset → `<project>/theme/`
- Write `package.json` with Slidev dependencies
- Write `uno.config.ts`
- Create `components/Icon.vue` ONLY if archetype.html files contain `<Icon>` tags. If no archetypes use icons → no Icon.vue.

### Step 4: Composition Planning

For each slide in the outline:
1. Determine content type (intro, scope, context, metric, team, process, vision, cta, visual-break)
2. Find Figma archetype with matching `contentType` from `figma.archetypes[]` **within the current preset only** — never mix presets, never use external archetypes
3. If no exact match → find the closest Figma archetype by structure within the same preset
4. Visual Rhythm check: if 4+ dense slides in a row → insert `figma-section` as breathing slide
5. Output: table `[slide_number, content_type, archetype_id, figma_nodeId]`

### Step 5: Write slides.md

For each slide:

1. **COPY** archetype.html **VERBATIM** — literal copy, not "inspired by"
2. **REPLACE** only `{{SLOT}}` markers with content from outline
3. **Check text fit** against `max_length` from flexibility.yaml:
   - If too long → shorten/rephrase the content to fit elegantly (preserving meaning)
   - If still too long → move overflow to speaker notes
   - Never reduce font below minimum
4. **Adapt element count** via flexibility.yaml:
   - More items than template → apply scaling strategy (grid-auto / wrap / stack)
   - Fewer items → merge or reduce slots
5. Add per-slide frontmatter (`layout: none`) and `<style>` with `.slidev-layout { padding: 0 !important; overflow: hidden; }`

**PROHIBITED in Step 5:**
- Rewriting HTML structure of the archetype
- Replacing inline `style=""` with CSS classes
- Adding elements without corresponding `{{SLOT}}`
- Adding `<Icon>` components if not in archetype
- Changing CSS values (font-size, padding, gap, colors, flex ratios)
- Using bg-alt where template uses bg-base
- Using any visual element (icons, decorations, shapes) not present in the Figma template's archetypes

---

## qa-cycle.md: Iterative QA with Figma Comparison

Runs after Step 5. Loops until Fidelity Score >= 9/10.

### QA Iteration Loop

```
repeat:
  Phase A: CSS + Figma Structure
  Phase B: Visual Export + Figma Screenshot
  Phase C: Score
  if score >= 9 → DONE
  if score < 9 → fix and repeat
  if 5 iterations with no improvement (delta < 0.3) → safety stop
```

### Phase A: CSS + Figma Structural/Style Comparison

**Basic checks:**
1. Font-size floor: body >= 1.25rem, heading >= 2.2rem, label >= 0.65rem
2. `layout: none` enforcement on every slide
3. Hardcoded hex → replace with `var(--*)`
4. Content overflow: text fits within slot boundaries, no element overlap

**Figma structural comparison** (for each slide, using its archetype's `nodeId`):
- Call `mcp__figma__get_metadata(nodeId, fileKey)` → positions/sizes
- Compare element positions: tolerance +-5% of viewport
- Compare element proportions (width/height): tolerance +-10%
- Compare nesting hierarchy: exact match

**Figma style comparison** (for each slide):
- Call `mcp__figma__use_figma` with JS → extract fonts, colors, spacing, radius
- Compare:

| Property | Tolerance |
|----------|-----------|
| font-size | +-0.15rem |
| font-weight | exact |
| color | deltaE < 5 (CIEDE2000) |
| spacing (gap) | +-0.25rem |
| padding | +-0.5rem |
| border-radius | +-2px |
| position | +-5% viewport |

Fix all issues found → modify slides.md.

### Phase B: Visual Export + Figma Screenshot Comparison

1. Export PNG: `npx slidev export --format png --output slides-qa`
2. For EACH slide:
   - Get fresh Figma screenshot: `mcp__figma__get_screenshot(nodeId, fileKey)`
   - Compare side by side: overall visual impression, color mood, typography proportions, whitespace balance, content not cut off, footer visible
   - For slides that used an adapted archetype (not exact content type match): verify it still looks like it belongs in the same presentation
3. Fix all visual issues found → modify slides.md
4. Clean up QA PNGs after iteration

### Phase C: Score

Fidelity Score = Visual (40%) + Structural (30%) + Style (30%)

- Score >= 9/10 → **DONE**. Clean up, print summary.
- Score < 9/10 → next iteration
- 5 iterations with score delta < 0.3 → **safety stop** with report of unresolved issues

### Iteration Report

```
━━━ QA Iteration <N> ━━━
Fidelity: X/10 (Visual: X | Structural: X | Style: X)
Fixed: <list of fixes applied>
Remaining: <list of unresolved issues>
```

---

## figma-learning.md: Learn Mode (--learn=N)

Triggered by `/figmadeck <URL> --learn=N`. Same QA cycle as regular generation, but fixes propagate to archetypes (permanent improvement).

### Difference from Regular Generation

| | Regular | Learn |
|---|---|---|
| Presentations generated | 1 | N (diverse outlines) |
| Where fixes go | slides.md (one-time) | archetype.html + preset.md + flexibility.yaml (permanent) |
| Figma comparison | qa-cycle.md (same) | qa-cycle.md (same) |
| Result | Ready presentation | Improved archetypes for all future generations |

### Process

1. **FIG-Extract** (from `figma-extraction.md`) — create preset + archetypes
2. **Generate N diverse outlines** — Russian, varied industries/formats/sizes/tone
3. **For each cycle i = 1..N:**
   a. Generate presentation (`generation-pipeline.md`)
   b. Run `qa-cycle.md` until 9/10
   c. Analyze which qa-cycle fixes are **systemic** (caused by archetype, not content) → apply to archetype.html / preset.md / flexibility.yaml
   d. Commit + tag `figmadeck-learn-<edu_dir>-iter-<i>`
4. **Convergence**: fidelity >= 9/10 for two consecutive cycles → early stop
5. **Final report**: fidelity progression, archetype changes summary, flexibility updates

Maximum N: 10.

---

## figma-extraction.md: FIG-1 through FIG-5

Identical to the procedure defined in the existing spec (`2026-03-30-slidev-figma-learning-design.md`), with one namespace change: `figmadeck-<name>-*` instead of `figma-<name>-*`.

- **FIG-1**: `use_figma` → find 16:9 frames, sort by position
- **FIG-2**: `use_figma` → collect global style (colors, fonts, spacing, shapes, effects)
- **FIG-3**: Per-slide extraction (screenshot + metadata + blueprint.json + code hint). Strict dedup (5 criteria). Image-only → visual-break archetype.
- **FIG-4**: Archetype conversion (content type detection, HTML skeleton with {{SLOT}}, flexibility.yaml, viewport conversion 960x540)
- **FIG-5**: Assemble preset + theme + figmadeck-ref directory

---

## Edge Cases

| Situation | Behavior |
|-----------|----------|
| URL not figma.com/design/ | Error, stop |
| No file access | Error, stop |
| No 16:9 frames | Error, stop |
| No Figma presets for `/figmadeck <outline>` | Error: `No Figma presets found. Run /figmadeck <URL> first.` |
| Figma API unavailable during QA | Retry 2x, then use cached blueprint.json + reference.png |
| Nesting > 3 levels | Flatten during extraction |
| Serif font | Replace with sans-serif |
| Accent in AI blacklist | Shift hue |
| > 20 slides in file | First 20, warning |
| QA score stuck (5 iterations, delta < 0.3) | Safety stop with report |

## Limitations (v1)

- Images not extracted from Figma (placeholder only, use separate image tool)
- Figma components not resolved (expanded instances only)
- Animations not extracted (no transitions from Figma)
- One Figma file = one preset
- No auto-creation of presets without Figma file (must run `/figmadeck <URL>` first)
