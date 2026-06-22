# Stitch Learned Patterns

Auto-populated by `--stitch=learn` cycles. Each pattern is extracted from Stitch-generated HTML and converted to a reusable CSS component.

---

## Pattern: Hero Number Scale 10rem+
- Source: Stitch cycle 1, slides 2 and 7
- CSS: `font-size: 10rem; font-weight: 800; line-height: 1; letter-spacing: -0.04em;` (stat-hero centered) and `font-size: 12rem; font-weight: 900;` (breathing hero)
- When: stat-hero and breathing slides where a single number is the focal point
- Added to: SKILL.md Step 5 hero number minimum, design-principles.md Principle 3

## Pattern: Card Border-Top Accent Strip
- Source: Stitch cycle 1, slide 5 (traction case studies)
- CSS: `border-top: 2px solid var(--color-accent); box-shadow: 0 25px 50px -12px rgba(0,0,0,0.25);`
- When: Case study cards, portfolio items, metric comparison cards — professional emphasis
- Added to: styles/index.css card variants (.card-stripe)

## Pattern: 12-Column Grid with Varied Ratios
- Source: Stitch cycle 1, all content slides
- CSS: `display: grid; grid-template-columns: repeat(12, 1fr);` then content spans: `grid-column: span 7 / span 7` and `grid-column: span 5 / span 5` — ratios 7/5, 4/8, 2/3
- When: Any asymmetric split layout — vary the ratio per slide instead of using a fixed 60/40
- Added to: composition-archetypes.md grid ratio variety note

## Pattern: Surface Depth System (5+ Levels)
- Source: Stitch cycle 1, Tailwind config
- CSS: 5+ surface levels progressively lighter from base: `--surface-0: #0e0e0f; --surface-1: #131314; --surface-2: #1a191b; --surface-3: #201f21; --surface-4: #262627;`
- When: Dark themes benefit from subtle depth between card nesting levels and slide backgrounds
- Added to: design-principles.md surface depth recommendation

## Pattern: 3-Level Metric Hierarchy
- Source: Stitch cycle 1, slides 2, 8
- CSS: Level 1 hero `font-size: 7-10rem`, Level 2 supporting `font-size: 3-4rem`, Level 3 body `font-size: 1.1-1.5rem` — ratio ~7:3:1
- When: Slides with multiple metrics — never make all numbers the same size
- Added to: SKILL.md focal point rule enhancement
