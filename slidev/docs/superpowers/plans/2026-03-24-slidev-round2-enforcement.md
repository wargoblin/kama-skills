# Slidev Round 2 Enforcement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Enforce 5 rules that exist but are systematically ignored — double all font minimums, zero-tolerance CSS vars, expanded COI, enhanced Ghost Deck, tighter bg-distribution.

**Architecture:** All changes are markdown edits to 3 skill instruction files. No code. Verification by reading files.

**Tech Stack:** Markdown (Claude Code skill files)

**Spec:** `docs/superpowers/specs/2026-03-24-slidev-round2-enforcement-design.md`

---

## File Map

| File | Path | Changes |
|------|------|---------|
| SKILL.md | `.claude/skills/slidev/SKILL.md` | Step 5 (2 HARD RULES), Step 4 (accent-rgb), Step 4.5 (COI + Ghost Deck + bg), Hard Constraint, QA-0b, QA-0c |
| design-principles.md | `.claude/skills/slidev/references/design-principles.md` | Principle 3 type scale |
| scoring-subroutine.md | `.claude/skills/slidev/references/scoring-subroutine.md` | Axis 3 descriptors |

---

### Task 1: design-principles.md — Principle 3 type scale ×2

**Files:**
- Modify: `.claude/skills/slidev/references/design-principles.md`

- [ ] **Step 1: Read Principle 3 type scale section**

Find the three type scale levels (hero, heading, body) with their em values.

- [ ] **Step 2: Update to doubled rem values**

Replace:
- Hero scale: `4-8em` → `8-16rem`
- Heading scale: `1.8-2.5em` → `4.4-5rem`
- Body scale: `0.85-1.1em` → `2.5-3rem`
- Statement/fact minimum: `3em` → `6rem`
- Update the "CRITICAL" measure note from `3em minimum` to `6rem minimum`

- [ ] **Step 3: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/design-principles.md
git commit -m "feat(slidev): double type scale minimums in Principle 3 (×2 all levels)"
```

---

### Task 2: scoring-subroutine.md — Axis 3 append

**Files:**
- Modify: `.claude/skills/slidev/references/scoring-subroutine.md`

- [ ] **Step 1: Read Axis 3 (Font discipline) row**

- [ ] **Step 2: APPEND size criteria to existing descriptors**

Keep all existing text (font count, italic, blacklist). Append to end of each cell:
- Low (1-3): append `; body text below 2.5rem; headings below 4.4rem`
- Mid (4-6): append `; body at 2.5rem but no dramatic scale contrast`
- High (7-10): append `; body ≥2.5rem, headings ≥4.4rem, hero numbers ≥8rem`

- [ ] **Step 3: Commit**

```bash
git add -f slidev/.claude/skills/slidev/references/scoring-subroutine.md
git commit -m "feat(slidev): append doubled font thresholds to Axis 3 scoring"
```

---

### Task 3: SKILL.md — Step 5 HARD RULES (font floor + CSS vars)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Read Step 5 opening (around line 1420)**

Find the "### Step 5: Write slides.md" heading and the "Structure:" line after it.

- [ ] **Step 2: Insert FONT SIZE FLOOR before "Structure:"**

Insert the font floor rule from spec §1.1, including the high-density archetype exception.

- [ ] **Step 3: Insert CSS VARIABLE ENFORCEMENT after font floor**

Insert the CSS var rule from spec §2.1, including the color/font variable lists and rgba guidance.

- [ ] **Step 4: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add FONT SIZE FLOOR and CSS VAR ENFORCEMENT to Step 5"
```

---

### Task 4: SKILL.md — Hard Constraint exemption + Step 4 accent-rgb

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Find Hard Constraint section (~line 1083)**

Read the "Hard Constraint / Visual Rhythm Override" section.

- [ ] **Step 2: Add Font Size Exemption**

After the existing Hard Constraint text, add the exemption from spec §1.2 (shorten → notes → split priority).

- [ ] **Step 3: Find Step 4 styles/index.css generation (~line 1290)**

Add the --accent-rgb, --bg-base-rgb, --text-rgb requirement from spec §2.2.

- [ ] **Step 4: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): add font-size Hard Constraint exemption, RGB decomposed vars"
```

---

### Task 5: SKILL.md — Step 4.5 (COI + Ghost Deck + bg algorithm)

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

- [ ] **Step 1: Find Cost-of-Inaction Check in Step 4.5**

Read the current COI type list.

- [ ] **Step 2: Replace with expanded type list + keyword detection**

Replace per spec §3.1 — add report, quarterly, annual, workshop, training, educational + keyword detection.

- [ ] **Step 3: Add type-specific COI framing**

Add the framing patterns from spec §3.2 (with Russian + English examples).

- [ ] **Step 4: Find Ghost Deck test in Step 4.5**

Read the current test text.

- [ ] **Step 5: Replace with Enhanced Ghost Deck test**

Replace per spec §4.1 — concrete FAIL conditions, GOOD patterns, cover+CTA exemption.

- [ ] **Step 6: Find bg-level algorithm step 6**

Read the current `ceil(R * 0.2)` formula.

- [ ] **Step 7: Update to tighter formula**

Change `ceil(R * 0.2)` → `floor(R * 0.15)`, spacing `every 4th-5th` → `every 5th-6th`. Update validation from `≥50%` → `≥55%`.

- [ ] **Step 8: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): expand COI scope, enhance Ghost Deck test, tighten bg algorithm"
```

---

### Task 6: SKILL.md — QA pipeline threshold updates

**Files:**
- Modify: `.claude/skills/slidev/SKILL.md`

This task updates ALL old font/QA thresholds per spec §1.6 table.

- [ ] **Step 1: QA-0b Principle 3 check (~line 824)**

Change `hero-sized (4-8em)` → `hero-sized (8-16rem)` and `3em minimum` → `6rem minimum`.

- [ ] **Step 2: QA-0b Font size minimum (~line 835)**

Change `Body text below 1.25rem` → `Body text below 2.5rem` and `Headings below 2.2rem` → `Headings below 4.4rem`. Make this check BLOCKING (add "STOP. Do not proceed to visual export.").

- [ ] **Step 3: QA-0b overflow caveat (~line 850)**

Replace `flag MAJOR 'split slide'` with the shorten-first priority from spec §1.2.

- [ ] **Step 4: QA-0b CSS Variables check**

Replace the 30% threshold with zero-tolerance BLOCKING per spec §2.3.

- [ ] **Step 5: QA-0b bg-level distribution**

Update thresholds: `bg-base < 55%` (was 50%), `bg-alt > 35%` (was 40%).

- [ ] **Step 6: QA-0b Action titles check**

Add the new WARNING check from spec §4.2 (logged to QA-10 report).

- [ ] **Step 7: QA-0c Typography checklist (~line 867-868)**

Change `Body text ≥1.25rem` → `Body text ≥2.5rem` and `Headings ≥2.2rem` → `Headings ≥4.4rem`.

- [ ] **Step 8: Search for any remaining old thresholds**

Grep SKILL.md for `1.25rem`, `2.2rem`, `4-8em`, `0.65rem` and update any remaining references.

- [ ] **Step 9: Commit**

```bash
git add -f slidev/.claude/skills/slidev/SKILL.md
git commit -m "feat(slidev): upgrade QA thresholds — BLOCKING font/CSS, tighter bg/titles"
```

---

### Task 7: Verification

- [ ] **Step 1: Grep all files for old thresholds**

Search for `1.25rem`, `2.2rem`, `0.65rem`, `0.7rem`, `4-8em`, `3em minimum` across all skill files to ensure nothing was missed.

- [ ] **Step 2: Verify BLOCKING keywords**

Grep for `BLOCKING` in SKILL.md to confirm font size and CSS vars checks are marked as blocking.

- [ ] **Step 3: Verify new minimums**

Grep for `2.5rem`, `4.4rem`, `1.3rem`, `8-16rem` to confirm new values are present.
