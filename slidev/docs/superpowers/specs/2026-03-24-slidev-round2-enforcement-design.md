# Slidev Skill: Round 2 — Enforcement Improvements Design Spec

**Date:** 2026-03-24
**Status:** Draft
**Scope:** 5 systemic issues found in analysis of 5 post-round-1 presentations. Focus on enforcement — rules exist but are not followed during generation.

## Background

After round 1 improvements (serif ban, theme generation, bg algorithm, icon variation, QA hardening), analysis of 5 new presentations revealed that several rules are documented but systematically ignored during generation. The root cause is twofold: (1) font size minimums are too low for presentation context, (2) QA checks are non-blocking warnings that don't prevent bad output.

## Improvement Summary

| # | Area | Severity | Description |
|---|------|----------|-------------|
| 1 | Font sizes ×2 | Critical | Double all font-size minimums to match presentation standards |
| 2 | CSS vars enforcement | Critical | Zero hardcoded hex tolerance, BLOCKING QA |
| 3 | Cost-of-inaction scope | Major | Expand auto-detection to more deck types |
| 4 | Action titles | Major | Enhanced Ghost Deck test with concrete FAIL conditions |
| 5 | bg-base distribution | Major | Tighten algorithm: ≥55% bg-base, less bg-alt |

## Files Modified

| File | Sections Changed |
|------|-----------------|
| SKILL.md | Step 5 (font floor + CSS var rule), Step 4.5 (COI scope + bg algorithm + Ghost Deck), Step 4 (accent-rgb in styles/index.css), QA-0a, QA-0b (3 checks upgraded), QA-0c, Hard Constraint section, --dev/--edit/--learn anti-pattern lists, QA-10 report |
| design-principles.md | Principle 3 (type scale ×2, unified to rem units) |
| scoring-subroutine.md | Axis 3 (font discipline — APPEND size thresholds, keep existing criteria) |

---

## Section 1: Font Size ×2 — All Minimums Doubled

### Problem

Body text generated at 0.85-1.0rem across all 5 presentations. The 1.25rem minimum is both too low for presentations and not enforced. Research supports 30pt (2.5rem) as industry minimum (Kawasaki rule, TED guidelines).

### Design

#### 1.1 New Font Size Floor (SKILL.md Step 5)

Add as **HARD RULE** at the top of Step 5 section (before the "Structure:" line, approximately line 1420 in SKILL.md):

**High-density archetype exception:** For archetypes classified as `high` density (comparison-table, data-spotlight, bento-grid cells), table cells and compact data elements may use the **label/eyebrow minimum (1.3rem)** instead of the body minimum (2.5rem). This prevents overflow in data-dense layouts. The archetype's primary description text still uses 2.5rem.

```
FONT SIZE FLOOR — NEVER set any inline font-size below these minimums:
- Hero/stat numbers: ≥8em (was 4em)
- Headings (h1, h2): ≥4.4rem (was 2.2rem)
- Body text (descriptions, paragraphs, card content): ≥2.5rem (was 1.25rem)
- Labels/eyebrows (uppercase, pill text): ≥1.3rem (was 0.65rem)
- Pill badge text: ≥1.4rem (was 0.7rem)

If content doesn't fit at these sizes — SHORTEN THE TEXT, never shrink the font.
Maximum words per card: 15. Maximum words per bullet: 10.
```

#### 1.2 Hard Constraint Exemption for Font Size

Add exemption to the existing Hard Constraint section in SKILL.md:

```
FONT SIZE EXEMPTION from Hard Constraint:
When text at the required minimum font size overflows the slide:
1. Shorten text to the core message (rephrasing is allowed)
2. Move secondary details to speaker notes
3. Split slide into two if core message cannot be shortened
Priority: shorten first → speaker notes second → split last.
NEVER reduce font below the minimum.
```

#### 1.3 QA-0b — BLOCKING Font Size Check

Upgrade from non-blocking warning to BLOCKING:

```
Font size minimum (BLOCKING — stops pipeline):
- Scan all font-size: declarations in inline styles
- Labels/eyebrows (text-transform:uppercase or letter-spacing:0.1em+ or inside pill) → min 1.3rem
- Pill badge text → min 1.4rem
- Body text (everything else except headings) → min 2.5rem
- Headings (h1, h2) → min 4.4rem
- If ANY violation → STOP. Do not proceed to visual export.
  Auto-fix: raise to minimum. If content overflows → follow §1.2 exemption priority
  (shorten first → speaker notes → split). This replaces the old line 850 overflow caveat
  ("flag MAJOR 'split slide'") which skipped the shorten-first step.
```

#### 1.4 design-principles.md — Principle 3 (Typographic Drama)

Update type scale descriptions. **Unify to `rem` units** (the file currently uses `em`; convert all type scale values to `rem` for consistency with SKILL.md):

| Level | Old (em) | New (rem, ×2) |
|-------|----------|---------------|
| Hero | 4-8em | 8-16rem |
| Heading | 1.8-2.5em | 4.4-5rem |
| Body | 0.85-1.1em | 2.5-3rem |
| Statement/fact main text | 3em minimum | 6rem minimum |

Update the "CRITICAL: Measure statement/fact/section slide main text against 3em minimum" to 6rem.

#### 1.5 scoring-subroutine.md — Axis 3 (Font Discipline)

**APPEND** new size criteria to existing Low/Mid/High descriptors (do NOT replace — keep existing font count, italic, blacklist conditions):
- **Low (1-3):** append "; body text below 2.5rem; headings below 4.4rem"
- **Mid (4-6):** append "; body at 2.5rem but no dramatic scale contrast"
- **High (7-10):** append "; body ≥2.5rem, headings ≥4.4rem, hero numbers ≥8em"

#### 1.6 Exhaustive SKILL.md Locations with Old Font Thresholds

ALL locations in SKILL.md that reference old font-size values and must be updated:

| Location | Old value | New value |
|----------|-----------|-----------|
| Step 5 — new HARD RULE (§1.1) | (new insertion) | All new minimums |
| QA-0b Principle 3 check (~line 824) | "hero-sized (4-8em)" | "hero-sized (8-16rem)" |
| QA-0b Principle 3 check (~line 824) | "3em minimum" for statement/fact | "6rem minimum" |
| QA-0b Font size minimum (~line 835) | "Body text below 1.25rem = CRITICAL" | "Body text below 2.5rem = CRITICAL" |
| QA-0b Font size minimum (~line 835) | "Headings below 2.2rem = CRITICAL" | "Headings below 4.4rem = CRITICAL" |
| QA-0c Typography checklist (~line 867) | "Body text ≥1.25rem" | "Body text ≥2.5rem" |
| QA-0c Typography checklist (~line 868) | "Headings ≥2.2rem" | "Headings ≥4.4rem" |
| QA-0b overflow caveat (~line 850) | "flag MAJOR 'split slide'" | Follow §1.2 shorten-first priority |
| Any --dev/--edit/--learn anti-pattern references | "font size ≥1.25rem" | "font size ≥2.5rem" |

---

## Section 2: CSS Variables — Zero Hardcoded Hex Tolerance

### Problem

4 of 5 presentations hardcode hex values instead of using CSS variables. The 30% threshold from round 1 is too lenient — generators default to hex.

### Design

#### 2.1 SKILL.md Step 5 — HARD RULE

```
CSS VARIABLE ENFORCEMENT — When writing inline styles in slides.md:
- NEVER hardcode hex color values (#XXXXXX or #XXX). Always use var(--name).
- Required COLOR variables (defined in styles/index.css or theme):
  var(--color-accent), var(--color-text), var(--color-muted),
  var(--bg-base), var(--bg-alt), var(--bg-accent),
  var(--color-surface), var(--color-surface-border),
  var(--color-accent-dim), var(--color-accent-bg)
- Required FONT variables (for font-family properties):
  var(--font-heading), var(--font-body)
- For rgba variants: use var(--accent-rgb) → rgba(var(--accent-rgb), 0.15)
  Also define --bg-base-rgb, --text-rgb for white/dark text rgba overlays.
- rgba() with numeric literals like rgba(255,255,255,0.1) is ALSO a violation
  if it corresponds to a defined color. Use rgba(var(--bg-base-rgb), 0.1) instead.
  Exception: rgba(255,255,255,...) and rgba(0,0,0,...) for generic transparency
  overlays that are NOT tied to the palette are acceptable.
- The ONLY acceptable hardcoded values: pixel sizes, rem values, percentages.
- Hardcoded hex in color/background/border-color/font-family = VIOLATION.
```

#### 2.2 SKILL.md Step 4 (Write styles/index.css — approximately line 1290)

Ensure decomposed RGB variables are always generated for rgba() composability:

```
When generating styles/index.css, ALWAYS include alongside the hex variables:
  --accent-rgb: R, G, B;      /* from --color-accent */
  --bg-base-rgb: R, G, B;     /* from --bg-base */
  --text-rgb: R, G, B;        /* from --color-text */
```

#### 2.3 QA-0b — BLOCKING CSS Vars Check

Upgrade from 30% threshold warning to zero-tolerance BLOCKING:

```
CSS Variables enforcement (BLOCKING — stops pipeline):
- Scan all inline style="..." for hardcoded hex (#XXXXXX, #XXX).
- Exclude: inside url(), inside CSS comments.
- If ANY hardcoded hex color found in color/background/border-color → STOP.
  Auto-fix: replace with matching var(--name) from styles/index.css.
  If no matching variable exists → create one in styles/index.css, then use it.
```

---

## Section 3: Cost-of-Inaction — Expanded Scope

### Problem

Auto-insert triggered for 2 of 5 business decks. Missed: quarterly report, investor report, workshop. Type list too narrow.

### Design

#### 3.1 SKILL.md Step 4.5 — Expanded Type List + Keyword Detection

Replace the current type list:

```
Cost-of-Inaction Check applies to:
- Explicit types: pitch, investor, sales, proposal, fundraising, business case,
  report, quarterly, annual, workshop, training, educational
- Keyword detection: if the full outline text (titles + bullet content) contains 3+ of these words → treat as business:
  "рост", "выручка", "инвестор", "revenue", "ROI", "клиенты", "рынок",
  "прибыль", "метрики", "KPI", "конкуренты", "growth", "market", "profit",
  "traction", "отчёт", "результаты"
```

#### 3.2 Type-Specific COI Framing

Different deck types require different COI angle:

```
COI title patterns by type (language matches slide content — Russian examples below,
generate English equivalents for English-language outlines):
- pitch/investor/fundraising: "Что потеряет инвестор, не присоединившись сейчас" / "What investors lose by waiting"
- report/quarterly/annual: "Что произойдёт, если не масштабировать текущие результаты" / "The cost of not scaling these results"
- sales/proposal: "Сколько стоит каждый месяц без этого решения" / "Every month without this costs [X]"
- workshop/training/educational: "Что произойдёт, если участники не применят знания" / "What happens if you don't apply this"
```

---

## Section 4: Action Titles — Enhanced Ghost Deck Test

### Problem

3 of 5 decks use topic labels ("Продукт", "Команда", "Финансовый прогноз") instead of declarative titles. Current Ghost Deck test is too vague.

### Design

#### 4.1 SKILL.md Step 4.5 — Concrete FAIL Conditions

Replace the current Ghost Deck test prose with:

```
Ghost Deck Test — ENHANCED:
After creating the Composition Plan, read all planned action titles in sequence.

FAIL conditions (any one = rewrite the title immediately):
- Title is 1-2 words ("Продукт", "Команда", "Рынок") → FAIL
- Title is a noun phrase without verb or claim
  ("Финансовый прогноз", "Конкурентный анализ") → FAIL
- Title could apply to any company in the sector without change → FAIL
- Title does not contain a number, comparison, or outcome → WEAK (rewrite recommended)

GOOD title patterns:
- Contains a specific number: "Выручка 72 млн ₽ к 2027 году"
- Makes a claim: "Enterprise растёт в 2,5 раза быстрее рынка"
- States an outcome: "Каждый рубль инвестиций с конкретным возвратом"
- Asks a provocative question: "Готовы к партнёрству?"
- Uses a verb: "Подключите первый объект бесплатно"

Cover slide (slide 1) and CTA slide (last slide) are exempt — brand name and
call-to-action phrasing are acceptable as titles. (Aligns with existing Rule 30.)
```

#### 4.2 QA-0b — Action Title Check

Add new check (WARNING, not blocking — requires content understanding for fix):

```
Action titles check (WARNING → logged in QA-10 final cleanup report):
- For each <h1> or <h2> that serves as a slide title:
  If title is 1-2 words → WARNING "topic label, not action title"
  If title is a noun phrase with no verb, number, or comparison → WARNING "weak title"
- Not auto-fixable. Logged in QA-10 final cleanup report for manual review.
- Note: QA-10 is the existing cleanup phase in the Visual QA Loop that produces the final report.
```

---

## Section 5: bg-base Distribution — Tighter Algorithm

### Problem

Only 1 of 5 decks achieved bg-base ≥50% (startup-pitch at 64%). Others at 44-50%. The `ceil(R * 0.2)` formula assigns too many bg-alt slides.

### Design

#### 5.1 SKILL.md Step 4.5 — Tighter bg-level Formula

Update step 6 of the bg-level assignment algorithm:

```
Old: 6. Assign bg-alt to ceil(R * 0.2) additional slides, spaced evenly (every 4th-5th)
New: 6. Assign bg-alt to floor(R * 0.15) additional slides, spaced every 5th-6th
```

Update validation:

```
Old: Validation: bg-base must be ≥50% of total.
New: Validation: bg-base must be ≥55% of total. If not, convert bg-alt to bg-base
     starting from the last assigned bg-alt slide.
```

#### 5.2 QA-0b — Updated Thresholds

```
bg-level distribution (non-blocking):
- Warn if bg-base < 55% (was 50%)
- Warn if bg-alt > 35% (was 40%) — note: breathing slides (step 3-4 of algorithm)
  auto-assign bg-alt, so 25% was unreachable on standard decks. 35% accounts for
  2-3 automatic bg-alt assignments while still catching excessive manual ones.
- Warn if strict alternating pattern across 4+ slides
```

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| ×2 all font sizes, not just body | User requirement: all levels doubled uniformly |
| BLOCKING QA for fonts and CSS vars | Non-blocking warnings proved ineffective across 5 presentations |
| Hard Constraint exemption for font size | Text must be shortened, not font shrunk — exemption enables this |
| Zero hardcoded hex tolerance | 30% threshold from round 1 was too lenient — 4/5 decks still hardcoded |
| COI keyword detection | Type-only matching missed reports and workshops with business content |
| Action titles as WARNING not BLOCKING | Requires content understanding; can't auto-fix without hallucination risk |
| bg-base ≥55% (not 60%) | Practical threshold for small decks (9-12 slides) with 2 accent bookends |

## Breaking Changes

None. All changes tighten existing rules — no field renames or structural changes.

## Feedback Traceability

| Finding | Source | Section |
|---------|--------|---------|
| Body font 0.85-1.0rem | All 5 presentations FAIL | 1 |
| Hardcoded hex colors | 4/5 presentations FAIL | 2 |
| COI missing in 3 business decks | kvartalnyj, investor-report, ubezhdaet | 3 |
| Topic label titles | invest-zapros, investor-report, startup-pitch partial | 4 |
| bg-base 44-50% | 4/5 presentations borderline | 5 |
