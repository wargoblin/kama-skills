# Scoring Subroutine

A reusable subroutine for scoring presentation slides on design quality. Input: `<dir>` (project directory with exported PNGs in `slides-qa/`). Called by `--polish`, `--learn` critic, and `--compare`.

## Important

Scores are Claude's subjective visual assessment, not deterministic metrics. They are used for relative comparison within a single polish/learn session. Treat thresholds as approximate guidelines, not hard cutoffs. Design memory numeric values are session-local estimates.

## Process

Read **EVERY** exported PNG from `<dir>/slides-qa/` with the Read tool. For each slide, evaluate and score 1-10 on these 9 axes:

| Axis | What to evaluate | Low (1-3) | Mid (4-6) | High (7-10) |
|---|---|---|---|---|
| **Composition variety** | Different archetype structures, icon-trio max 1, no >30% same layout | Same layout every slide; 3+ uniform icon containers on one slide; one layout-group dominates >40% of content slides; split layouts >30%; 3+ consecutive same-group | Some variation, 2-3 patterns; some variety but layout-groups unbalanced; occasional same-group streaks | Each slide structurally distinct; icon containers vary in size, shape, or style; no group exceeds 40%; at least 3 different groups; no same-group streaks of 3+ |
| **Shape diversity** | Non-rectangular elements (circles, pills, badges) | All rectangles with same border-radius | Some shape variation | Rich mix of circles, pills, asymmetric splits |
| **Font discipline** | Strictly 2 font identities, body ≥1.25rem, headings ≥2.2rem, line-height 1.3-1.45, bold ≤15% | 3+ fonts, numbers in different font; >3 type treatments on one slide; italic outside quotes; blacklisted font for number-heavy deck; body text below 1.25rem; headings below 2.2rem | Mostly 2 but some inconsistency; body at 1.25rem but no dramatic scale contrast | Exactly 2 fonts, numbers consistent; max 3 treatments/slide; italic only in quotes; font digit rendering verified; body ≥1.25rem, headings ≥2.2rem, hero numbers ≥4rem |
| **Visual impact** | First impression, memorability, whitespace ≥30%, max 2 large elements per slide | Forgettable, generic; deck has zero visual anchors — no SVG charts, ghost typography, or decorative data viz | Decent but not striking | Would remember after seeing 20 others; Rule of Thirds applied on 30%+ of content slides |
| **Layout uniqueness** | Structural difference from neighbors | Same layout as adjacent slides | Some variation | Clearly distinct composition |
| **Typography drama** | Scale contrast, hero numbers, weight pairing | Everything same size | Some hierarchy | Clear 3+ scale levels, hero elements pop |
| **Color conviction** | Bold intentional palette, WCAG 4.5:1 contrast, no AI-blacklisted primary colors, 60-30-10 distribution | Washed out, no accent; pure #000/#FFF; inconsistent bg temperature; bg-system not used; mixed decorative gradient types; muted text fails WCAG on any surface; bg-levels form strict alternating checkerboard; bg-base fewer than half slides | Accent exists but mild; bg-levels show variation but base and alt roughly equal | Bold, purposeful, memorable palette; 3-level bg-system; single color temperature; all text-on-surface pairs pass WCAG 4.5:1; single decorative gradient type; bg-base dominates ≥55%, bg-alt sparingly for rhythm, bg-accent for bookends |
| **Content clarity** | Main message in 3 seconds, ≤40 words/slide, action titles not labels, recommendation by slide 3 | Cluttered, competing elements; monotonous bullet length (all within +-3 words) | Message findable but slow | Instant comprehension; varied sentence structure; cost-of-inaction slide present (pitch decks) |
| **Decorative quality** | Visible decorative elements, chart titles as insights, source citations on statistics | Flat background, no personality; pill/badge mud overlap with gradient; gradient exclusion zone violated; decorative elements invisible (opacity < 0.12 on light backgrounds); no atmosphere layer | Some decoration but subtle | Clearly visible motifs adding character; pills crisp on all backgrounds; decorative elements in corners/edges only |

## Output

Write score report to `<dir>/score-report.md`:

```
# Score Report

## Per-Slide Scores

| Slide | Comp | Shape | Font | Impact | Layout | Typo | Color | Content | Decor | AVG |
|-------|--------|--------|------|-------|---------|-------|-----|
| 1     | ...    | ...    | ...  | ...   | ...     | ...   | ... |
...

## Overall: X.X/10

## Weakest Slides
- Slide N (avg X.X): [which axes scored low and why]
...

## Strongest Slides
- Slide N (avg X.X): [what makes it strong]
...
```

## Thresholds (approximate — use judgment)

- Slide avg **< 7** → "weak" — candidate for redesign in `--polish`
- Slide avg **>= 9** → "strong" — do not touch during polish
- Overall avg **>= 9** → consider early exit from polish cycle
