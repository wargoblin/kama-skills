# Figma Learning Procedure (FDL)

## Overview

Triggered by `/figmadeck <figma-url> --learn=N`. Maximum N: 10.

Uses the **Figma-native pipeline** (clone → fill via `figma-generation.md` → QA via `qa-cycle.md`). Does NOT use Slidev — no slides.md, no preset.md, no npx exports.

Fixes discovered during learning go to **skill files** (font fallback table, text-shortening heuristics, role detection logic in `figma-generation.md`), NOT to the Figma file itself.

---

## FDL-1: Initialization

**MANDATORY: Scan for existing `edu_*` directories before creating anything.**

```bash
ls -d edu_* 2>/dev/null
```

Read the output. If `edu_01/` and `edu_02/` exist → create `edu_03/`. If only `edu_01/` → create `edu_02/`. If nothing → create `edu_01/`.

**NEVER overwrite or reuse an existing `edu_*` directory.** Skipping this scan is a critical error — it destroys previous learning data.

Store `fileKey` and page info for the session. All artifacts go under `<edu_dir>/`.

---

## FDL-0: Calibration (learn_0)

**Purpose:** Verify the pipeline mechanics work before running diverse outlines.

Procedure:
1. Clone the template page via `figma-generation.md` — `use_figma`: `child.clone()` for all frames on Page 1 → new page
2. Do **NOT** change any text on the clone
3. Run `qa-cycle.md` comparing the clone against the original template
4. Expected result: **100% match** (clone is a perfect copy, all phases pass)

**If learn_0 fails:** the tooling is broken, not content adaptation. Debug page creation, screenshot comparison, or scoring before proceeding. Do NOT continue to learn_1..N with broken tooling.

This validates: `use_figma` page creation works, `get_screenshot` comparison works, Phase A structural check works, scoring is calibrated.

---

## FDL-2: Generate N Diverse Outlines

Outline language should **match the template's primary language**.
Detect from Step 3 analysis: examine the text content of the template's
TEXT nodes. What language are they written in? Generate outlines in
that same language.

If template language is ambiguous or mixed (e.g., half English, half
Russian) → ask the user: "What language should the generated
presentations use?"

If template is in English → generate English outlines.
If template is in Russian → generate Russian outlines.
Other languages → detect and match.

Must vary across:

- **Industries**: tech, education, healthcare, finance, creative, nonprofit, SaaS, retail, AI/ML, sustainability
- **Formats**: pitch, lecture, product launch, report, onboarding, keynote
- **Sizes**: 8–10 slides (short), 12–14 slides (medium), 15–16 slides (long)
- **Tone**: formal, casual, inspirational, data-heavy, storytelling, technical

Generate all N outlines upfront. Save each to `<edu_dir>/learn_<i>/outline.md`.

---

## FDL-3: Learning Cycle (i = 1..N)

### FDL-3a: Clone + Fill

Re-read `figma-generation.md` before each iteration. Run the full pipeline:
1. Clone template page → name `"Generated: learn_<i>"`
2. Analyze template slides (role detection, content type detection)
3. Match outline slides → template slides
4. Fill content via `use_figma` — one call per slide, font handling + overflow handling per spec

Use outline from `<edu_dir>/learn_<i>/outline.md`.

### FDL-3b: QA Cycle

Run `qa-cycle.md` until Fidelity Score ≥ 9/10.

All QA runs inside Figma: Phase A (structural, programmatic), Phase B (visual `get_screenshot` comparison), Phase C (design critique), Phase D (score + fix).

Safety stop: 5 QA iterations with score delta < 0.3.

### FDL-3c: Record Problem Patterns

After QA completes, analyze what caused issues during this iteration:

- **Role overflow**: which text roles overflowed most? (title / description / cards / label)
- **Font fallbacks**: which substitutions worked well, which degraded quality?
- **Content types**: which slide types (metric / cards / cta) needed most aggressive shortening?
- **Shortening thresholds**: how much text reduction was needed per role to fit?

Record to `<edu_dir>/learn_<i>/patterns.md`. If a pattern appeared in 2+ slides → it is systemic → update skill references:
- Font table in `figma-generation.md` (Step 5, font handling)
- Text shortening heuristics
- Role detection priority rules (Step 3)

### FDL-3d: Intermediate Report

Print after each completed iteration:

```
━━━ Figma Learning: Итерация <i>/<N> ━━━

Fidelity: X.X/10
QA iterations: <N>
Role overflows: <list: role → count>
Font fallbacks: <list: original → fallback → quality>
Patterns recorded: <count>
Skill file updates: <list or "none">
```

### FDL-3e: Git Commit + Tag

```bash
git add <edu_dir>/learn_<i>/
git commit -m "learn(figmadeck): iteration <i>/<N> — fidelity X/10"
git tag figmadeck-learn-<edu_dir>-iter-<i>
```

---

## FDL-4: Convergence

If Fidelity ≥ 9/10 for **two consecutive cycles** → early stop.

Print: `"Раннее завершение: fidelity стабилизировался на X/10 в циклах <c-1> и <c>."`

If score decreases from the previous cycle → warn but continue. Do not stop on a single regression.

---

## FDL-5: Final Report

Save to `<edu_dir>/figmadeck-learning-report.md`.

Contents:
- **Fidelity progression**: cycle-by-cycle scores (learn_0 through learn_N)
- **Adaptation patterns discovered**: which roles/content types are problematic and why
- **Font fallback effectiveness**: table of substitutions and their quality ratings
- **Shortening thresholds found**: per role, how many characters fit reliably
- **Skill file changes made**: what was updated in `figma-generation.md` and when
- **Recommendations**: what to improve in future learning runs or template design
