# Slidev Round 3: Professional Quality Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform presentation output from "basic minimum" to professional quality — fix font rendering, add multi-layer backgrounds, visual anchors, decoration library, enforce card diversity/focal points/pattern breaks, add --stitch=learn mode.

**Architecture:** Markdown edits to skill files + 1 new reference file (decoration-library.md). No code compilation. All changes are instruction/rule additions.

**Tech Stack:** Markdown (Claude Code skill files), CSS patterns

**Spec:** `docs/superpowers/specs/2026-03-26-slidev-round3-professional-quality-design.md`

---

## File Map

| File | Path | Action |
|------|------|--------|
| decoration-library.md | `.claude/skills/slidev/references/decoration-library.md` | **CREATE** |
| stitch-learned-patterns.md | `docs/research/stitch-learned-patterns.md` | **CREATE** |
| SKILL.md | `.claude/skills/slidev/SKILL.md` | Modify |
| design-principles.md | `.claude/skills/slidev/references/design-principles.md` | Modify |
| preset-format.md | `.claude/skills/slidev/references/preset-format.md` | Modify |
| scoring-subroutine.md | `.claude/skills/slidev/references/scoring-subroutine.md` | Modify |
| composition-archetypes.md | `.claude/skills/slidev/references/composition-archetypes.md` | Modify |

---

### Task 1: Create decoration-library.md

**Files:**
- Create: `.claude/skills/slidev/references/decoration-library.md`

- [ ] **Step 1: Create the file with full content from spec §5.1**

Write the entire decoration library with all sections: Shapes (circle, arc, blob, dot-grid, line-horizontal, ring, diamond), Behaviors (gradient-fill, solid-fill, glow, border-only), Positions (8 positions), Scales (subtle/medium/bold/hero), Opacity Presets (light/dark theme values).

Each component is a ready-to-use HTML snippet with `{{PLACEHOLDER}}` slots. The full content is specified verbatim in spec §5.1.

- [ ] **Step 2: Create stitch-learned-patterns.md placeholder**

Write `docs/research/stitch-learned-patterns.md` with header:

```markdown
# Stitch Learned Patterns

Auto-populated by `--stitch=learn` cycles. Each pattern is extracted from Stitch-generated HTML and converted to a reusable CSS component.

---

(No patterns yet — run `/slidev --stitch=learn=N` to populate)
```

- [ ] **Step 3: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/decoration-library.md
git add slidev/docs/research/stitch-learned-patterns.md
git commit -m "feat(slidev): create decoration component library + stitch-learned-patterns placeholder"
```

---

### Task 2: SKILL.md — Font fixes (§1.1, §1.2, §1.3)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Add FONT LOADING RULE to Step 4 (~line 1767)**

Find the "Include:" list in Step 4 (Write styles/index.css). At the TOP of the list, before the first bullet, insert the FONT LOADING RULE from spec §1.1:

```
FONT LOADING RULE — styles/index.css MUST begin with an explicit @import:

@import url('https://fonts.googleapis.com/css2?
  family=<HEADING_FONT>:wght@400;500;600;700;800
  &family=<BODY_FONT>:wght@400;500;600;700
  &display=swap');

Do NOT rely on Slidev's automatic font loading from frontmatter — it loads
only weights 400+600 by default. The @import ensures ALL weights used in
inline styles are available, preventing synthetic bold rendering artifacts.
```

- [ ] **Step 2: Add Cyrillic Whitelist + Blacklist to Font Selection**

Find the Serif Blacklist section in Font Selection. AFTER it, insert the CYRILLIC FONT RULES from spec §1.2 (whitelist of 8 heading + 8 body fonts, blacklist of 10 Latin-only fonts).

- [ ] **Step 3: Replace Good sans+sans pairs table**

Find the current Good pairs table. Replace with the 5 Cyrillic-verified pairs from spec §1.3:
- Manrope + DM Sans
- Plus Jakarta Sans + Nunito
- Montserrat + Source Sans Pro
- Rubik + IBM Plex Sans
- Geist + Noto Sans

- [ ] **Step 4: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): mandatory @import for font weights, Cyrillic whitelist/blacklist"
```

---

### Task 3: SKILL.md — Step 5 HARD RULES (§2.1, §3.1, §3.2, §4.2, §4.3, §4.4)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Add BACKGROUND LAYER SYSTEM after Font Size Floor (~line 1920)**

Find "**FONT SIZE FLOOR**" in Step 5. AFTER the CSS VARIABLE ENFORCEMENT block (both already exist), insert the BACKGROUND LAYER SYSTEM from spec §2.1 (Layer 1 base + Layer 2 atmosphere + Layer 3 texture, opacity calibration for light/dark).

- [ ] **Step 2: Add GHOST TYPOGRAPHY rule**

After the BACKGROUND LAYER SYSTEM, insert the GHOST TYPOGRAPHY rule from spec §3.1 (12-20rem, opacity 0.04-0.08, 2-3 slides per deck max, example HTML).

- [ ] **Step 3: Add DATA VIZ RULE**

After GHOST TYPOGRAPHY, insert the DATA VIZ RULE from spec §3.2 (if 3+ metrics → at least one SVG: donut/bar/line/stacked).

- [ ] **Step 4: Add CARD DIVERSITY rule**

After DATA VIZ RULE, insert CARD DIVERSITY from spec §4.2 (BANNED: 3+ identical cards, must use bento/mixed/hierarchy).

- [ ] **Step 5: Add FOCAL POINT rule**

After CARD DIVERSITY, insert FOCAL POINT from spec §4.3 (ONE dominant element 2x+ larger).

- [ ] **Step 6: Update existing Structural Break Rule with Pattern Break**

Find the existing STRUCTURAL BREAK RULE in Step 5. Replace/extend with the PATTERN BREAK from spec §4.4 (>40% same pattern → must replace with full-bleed/visual-dominant/minimal/inverted/asymmetric).

- [ ] **Step 7: Update point 8 (Apply Decorative Layer)**

Find point "8. **Apply Decorative Layer (Principle 6)**". Replace with the DECORATION FROM LIBRARY instructions from spec §5.2 (pick shape + behavior + position + scale from decoration-library.md, diversity rules, never invent CSS).

- [ ] **Step 8: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): background layers, ghost type, data viz, card diversity, focal point, pattern break, decoration library usage"
```

---

### Task 4: SKILL.md — QA-0b updates (§4.1, §4.3)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Add Decoration visibility BLOCKING check to QA-0b**

Find the Feedback-Driven QA-0b Checks section. Add the decoration visibility check from spec §4.1:
- Light bg + decorative opacity < 0.12 → STOP, auto-fix to 0.15
- Decorative element < 200px → STOP, auto-fix to 250px

- [ ] **Step 2: Add Focal point WARNING check to QA-0b**

Add after decoration check:
- If largest and second-largest elements differ by < 1.5x in font-size → WARNING "no clear focal point"

- [ ] **Step 3: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): BLOCKING decoration visibility + focal point WARNING in QA-0b"
```

---

### Task 5: SKILL.md — --stitch=learn mode (§6)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Add help text entry**

Find the help text section. Add:
```
  /slidev --stitch=learn=N              Learn from Stitch as design teacher (N cycles)
```

- [ ] **Step 2: Add --stitch=learn procedure**

Find the end of the --stitch modes section (after "Stop here after Stitch procedure completes"). Insert the full STL-1 through STL-3 procedure from spec §6.2:

- STL-1: Generate N diverse outlines
- STL-2: For each cycle: Stitch generates → our generator makes same → comparative analysis across 6 dimensions → extract patterns → apply improvements → interim report → git commit+tag
- STL-3: Final report with progress.md

Include the complete comparative analysis template (BACKGROUNDS/LAYOUT/TYPOGRAPHY/DECORATION/CARDS/HIERARCHY) and the interim report format in Russian.

- [ ] **Step 3: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add --stitch=learn mode — learn CSS patterns from Stitch comparison"
```

---

### Task 6: Reference files updates (design-principles, preset-format, scoring, archetypes)

**Files:**
- Modify: `.claude/skills/slidev/references/design-principles.md`
- Modify: `.claude/skills/slidev/references/preset-format.md`
- Modify: `.claude/skills/slidev/references/scoring-subroutine.md`
- Modify: `.claude/skills/slidev/references/composition-archetypes.md`

- [ ] **Step 1: design-principles.md — Principle 6 decoration calibration**

Find Principle 6 (Decorative Layer). Add opacity calibration table:
```
Opacity calibration by background luminance:
  Light (>70%): atmosphere 0.15-0.30, texture 0.06-0.12, borders 0.15-0.25
  Dark (<30%):  atmosphere 0.08-0.15, texture 0.03-0.08, borders 0.10-0.18

Minimum decorative element size: 200px. Elements < 200px are invisible at
presentation scale.
```

Also add reference to decoration-library.md: "See references/decoration-library.md for the full library of pre-tested decoration components."

- [ ] **Step 2: design-principles.md — Background layer principle**

Add to Principle 1 (Visual Rhythm) or as new sub-principle:
```
BACKGROUND LAYER SYSTEM: Every slide must have minimum 2 background layers.
A single solid color is not professional-grade. Layer atmosphere gradients and
texture patterns on top of the base color. See Step 5 BACKGROUND LAYER SYSTEM
for implementation details.
```

- [ ] **Step 3: preset-format.md — decoration-library reference**

Find the shapes section. Add note:
```
For decoration elements, the generator uses the combinatorial library in
references/decoration-library.md — pre-tested CSS components assembled from
shape + behavior + position + scale. The library grows via --stitch=learn.
```

- [ ] **Step 4: scoring-subroutine.md — Axis 4 + Axis 9**

Find Axis 4 (Visual impact) row. Append to Low (1-3) descriptor:
```
; deck has zero visual anchors — no SVG charts, ghost typography, or decorative data viz
```

Find Axis 9 (Decorative quality) row. Append to Low (1-3) descriptor:
```
; decorative elements invisible (opacity < 0.12 on light backgrounds); no atmosphere layer
```

- [ ] **Step 5: composition-archetypes.md — atmosphere layer placeholders**

For each archetype HTML skeleton that has a background div (cover-hero, section-divider, stat-hero, cta-warm, and all archetypes with background layers), add a comment inside the background div:
```html
<!-- ATMOSPHERE LAYER: insert radial-gradient or texture div here (see decoration-library.md) -->
```

- [ ] **Step 6: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/design-principles.md \
         slidev/.claude/skills/slidev/references/preset-format.md \
         slidev/.claude/skills/slidev/references/scoring-subroutine.md \
         slidev/.claude/skills/slidev/references/composition-archetypes.md
git commit -m "feat(slidev): update design-principles, preset-format, scoring, archetypes for round 3"
```

---

### Task 7: Verification

- [ ] **Step 1: Grep for consistency**

Verify new rules are in place:
```bash
grep -c "BACKGROUND LAYER SYSTEM" SKILL.md     # should be 1
grep -c "GHOST TYPOGRAPHY" SKILL.md              # should be 1
grep -c "DATA VIZ RULE" SKILL.md                 # should be 1
grep -c "CARD DIVERSITY" SKILL.md                # should be 1
grep -c "FOCAL POINT" SKILL.md                   # should be 1
grep -c "DECORATION FROM LIBRARY" SKILL.md       # should be 1
grep -c "FONT LOADING RULE" SKILL.md             # should be 1
grep -c "CYRILLIC WHITELIST" SKILL.md            # should be 1
grep -c "stitch=learn" SKILL.md                  # should be 2+
grep -c "ATMOSPHERE LAYER" composition-archetypes.md  # should be 5+
```

- [ ] **Step 2: Verify decoration-library.md structure**

Check that decoration-library.md has all sections: Shapes (7), Behaviors (4), Positions (8), Scales (4), Opacity Presets (6).

- [ ] **Step 3: Verify no old font pairs remain**

Grep SKILL.md for Outfit, Sora, Barlow, Urbanist — should appear only in Cyrillic Blacklist, not in Good pairs.
