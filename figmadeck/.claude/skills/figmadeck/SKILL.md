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

## Hard Constraints

**The generated presentation MUST have exactly the same number of slides, in exactly the same order, with exactly the same content as the outline.** Never add, remove, merge, or reorder slides beyond what the outline specifies.

### Mandatory Title Slide

**CRITICAL:** The first slide MUST always be a cover/title slide (`contentType: "intro"`). Rules:
1. If the outline's first slide is explicitly a title/cover → use it as-is
2. If the outline does NOT start with a title slide → **prepend one automatically**: use the presentation topic as the title and the first outline slide's subtitle or description as the subtitle
3. The title slide MUST be matched to an `intro`-type template (largest title fontSize, cover layout). If no `intro` template exists → use the first template frame and adapt it as a cover
4. A title slide with only 1 short word in an oversized slot looks empty — use 2+ words or a title+subtitle pair (see `design-lessons.md`)

### Zero Placeholder Text

**CRITICAL:** After generation, **NO template placeholder text may remain** in ANY text node. Rules:
1. Every visible TEXT node with `fontSize > 10` MUST be filled with content from the outline or contextually appropriate metadata (page number, project name, section name)
2. If a text node has no matching role in the outline → fill with contextual content (section name, slide title, empty string for decorative labels) — but NEVER leave the original English/template text
3. During Step 5 (Fill), if `newText` is undefined for a role → do NOT `continue` silently. Instead, set the node to the slide's title, section name, or `""` (empty) as a last resort
4. QA MUST flag any text node where `characters` matches the original template text AND `fontSize > 10`

### No Text Overlap

**CRITICAL:** Text elements MUST NOT overlap each other or any other visible elements. Rules:
1. After filling each slide, run a pairwise overlap check between ALL visible text nodes on that slide (not just text vs top-level sections)
2. If two text nodes overlap → apply fix priority: expand container width → shorten text → reduce fontSize → move element
3. Russian text is 20-40% longer than English — every text replacement MUST include an overlap check against sibling text nodes within the same parent container
4. The minimum gap between any two text elements is 8px. Between unrelated elements: 16px

### Content Adaptation

When outline content is longer than the template placeholder text:
1. **Shorten/rephrase** to fit (preserve core meaning)
2. **textAutoResize = "HEIGHT"** to let text grow
3. **Reduce fontSize** (minimum = current × 0.85)
4. Last resort: move overflow detail to a note

## Mandatory QA — All 4 Levels

**CRITICAL — NON-NEGOTIABLE:** After generation, ALL 4 QA levels MUST execute in strict order. **NONE may be skipped.**

| Level | What | Can be skipped? |
|-------|------|-----------------|
| Level 1 | Per-slide structural + visual checklist | **NO** |
| Level 2 | Designer Critic cycle | **NO** |
| Level 3 | Client Simulator cycle | **NO** |
| Level 4 | Senior Reviewer cycle | **NO** |

**This applies regardless of:** deck size, how clean L1 reports, time spent, context pressure, or number of presentations in a session. Each presentation gets its own full 4-level QA. If any level is skipped, the presentation is **INCOMPLETE** and must not be delivered.

Print `✅ Level N COMPLETE — proceeding to Level N+1` between each level as a verification gate.

## Autonomous Execution

**CRITICAL:** Execute all steps autonomously without stopping for confirmation. If blocked, attempt to resolve (2-3 attempts). Only stop if truly unresolvable.
