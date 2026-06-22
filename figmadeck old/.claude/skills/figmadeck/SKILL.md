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
- `references/design-rules.md` — **Universal design rules** (readability, spacing, hierarchy, contrast — applied always)

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
