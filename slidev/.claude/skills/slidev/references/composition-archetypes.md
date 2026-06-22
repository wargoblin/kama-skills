# Composition Archetypes

Named slide composition patterns for content-aware layout selection. Each archetype defines a complete visual structure with HTML skeleton. Used by Step 4.5 (Composition Planning) during generation.

## Content Type → Archetype Mapping

| Content Type | Candidate Archetypes (in preference order) |
|---|---|
| `intro` | cover-hero |
| `credentials` | bento-grid, icon-trio, card-mosaic |
| `context` | two-col-text, asymmetric-split |
| `vision` | section-divider, asymmetric-split, quote-pull |
| `scope` | bento-grid, icon-trio, two-col-text |
| `process` | timeline-horizontal, timeline-zigzag |
| `team` | profile-grid, card-mosaic |
| `metric` | stat-hero, data-spotlight |
| `portfolio` | card-mosaic, bento-grid |
| `trust` | icon-trio, two-col-text, bento-grid |
| `terms` | two-col-text, comparison-table |
| `cta` | cta-warm |

## Archetypes

### cover-hero
**Group:** hero | **Density:** low
**Content types:** intro
**Focal point:** center (hero slide — center alignment justified)
**Use when:** First slide of the presentation — title, company, key metadata.
**Visual:** Full-bleed decorative background. Title centered and oversized. Subtitle in body font. Metadata (date, region) as small dots row.
**Shape elements:** Chip/pill badge for label (opaque bg, line-height:1), dot separators for metadata.
**Gradient slots:** background = STRUCTURAL (exempt from gradient type limit)

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:60px 80px;">
  <!-- ATMOSPHERE LAYER: insert radial-gradient or texture div here (see decoration-library.md) -->
  <div style="display:inline-flex;align-items:center;justify-content:center;line-height:1;background:var(--bg-accent);border:1.5px solid rgba(var(--accent-rgb),0.55);border-radius:20px;padding:6px 18px;margin-bottom:28px;">
    <span style="font-size:0.7rem;text-transform:uppercase;letter-spacing:0.18em;color:var(--color-accent);font-weight:600;">{{LABEL}}</span>
  </div>
  <h1 style="font-size:3.4rem;font-weight:800;color:var(--color-text);margin:0 0 12px;font-family:var(--font-heading);line-height:1.08;">{{TITLE}}</h1>
  <p style="font-size:1.25rem;color:var(--color-muted);margin:0 0 28px;font-family:var(--font-body);">{{SUBTITLE}}</p>
  <div style="display:flex;align-items:center;gap:16px;color:var(--color-muted);font-size:1.25rem;">
    {{METADATA_DOTS_ROW}}
  </div>
</div>
```

### section-divider
**Group:** hero | **Density:** low
**Content types:** vision
**Focal point:** center (breathing slide — center alignment justified)
**Use when:** Transition between major sections. One big idea or section title.
**Visual:** Centered large heading (3.5rem+). Lifted background with centered glow. Minimal content — just heading + optional subtitle.
**Shape elements:** None — pure typography.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:60px 80px;">
  <!-- ATMOSPHERE LAYER: insert radial-gradient or texture div here (see decoration-library.md) -->
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.18em;color:var(--color-accent);font-weight:600;margin-bottom:16px;">{{LABEL}}</span>
  <h1 style="font-size:3.5rem;font-weight:800;color:var(--color-text);margin:0 0 24px;font-family:var(--font-heading);line-height:1.1;">{{HEADING}}</h1>
  <p style="font-size:1.25rem;color:var(--color-muted);max-width:700px;line-height:1.6;font-family:var(--font-body);">{{DESCRIPTION}}</p>
</div>
```

### stat-hero
**Group:** hero | **Density:** low
**Content types:** metric
**Focal point:** center (breathing slide — center alignment justified)
**Use when:** Slide has 1-2 key metrics to emphasize (budget, KPI, headline number).
**Visual:** Giant centered number (5em+). Brief label above. Supporting context below. Optional small stat pills at bottom.
**Shape elements:** Number as typographic hero (no card wrapper), pill badges for supporting stats.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:48px 80px;">
  <!-- ATMOSPHERE LAYER: insert radial-gradient or texture div here (see decoration-library.md) -->
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.18em;color:var(--color-accent);font-weight:600;margin-bottom:16px;">{{LABEL}}</span>
  <h1 style="font-size:5.5rem;font-weight:800;color:var(--color-accent);margin:0;line-height:1;font-family:var(--font-heading);">{{HERO_NUMBER}}</h1>
  <p style="font-size:1.25rem;color:var(--color-text);margin:8px 0 32px;font-family:var(--font-body);">{{SUPPORTING_CONTEXT}}</p>
  <div style="display:flex;gap:12px;">
    {{SUPPORTING_STATS_AS_PILLS}}
  </div>
</div>
```

### quote-pull
**Group:** hero | **Density:** low
**Content types:** vision
**Focal point:** center (breathing slide)
**Use when:** Slide centers on a key quote, mission statement, or visionary text.
**Visual:** Oversized quote text in serif/body font. Attribution below. No cards or blocks.
**Shape elements:** Decorative quotation marks as CSS pseudo-element.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:60px 100px;">
  <!-- ATMOSPHERE LAYER: insert radial-gradient or texture div here (see decoration-library.md) -->
  <p style="font-size:2rem;font-weight:400;color:var(--color-text);line-height:1.5;font-family:var(--font-body);font-style:italic;max-width:800px;">{{QUOTE_TEXT}}</p>
  <span style="display:block;margin-top:20px;font-size:0.85rem;color:var(--color-muted);font-family:var(--font-heading);text-transform:uppercase;letter-spacing:0.1em;">{{ATTRIBUTION}}</span>
</div>
```

### cta-warm
**Group:** cta | **Density:** low
**Content types:** cta
**Focal point:** center (CTA slide — center alignment justified)
**Gradient slots:** background = STRUCTURAL (exempt from gradient type limit)
**Use when:** Final CTA slide — call to action, contact info, next steps.
**Visual:** Warm background (accent with base overlay). Centered heading. Numbered steps in semi-transparent pills. Contact info row at bottom.
**Shape elements:** Numbered pills for steps, dot separators for contact.

```html
<div style="position:absolute;inset:0;z-index:0;background:linear-gradient(145deg, var(--color-accent) 0%, color-mix(in srgb, var(--color-accent) 70%, black) 100%);">
  <!-- ATMOSPHERE LAYER: insert radial-gradient or texture div here (see decoration-library.md) -->
</div>
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:60px 80px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.18em;color:var(--color-bg);font-weight:600;margin-bottom:16px;">{{LABEL}}</span>
  <h1 style="font-size:2.8rem;font-weight:800;color:var(--color-bg);margin:0 0 28px;font-family:var(--font-heading);line-height:1.15;">{{HEADING}}</h1>
  <div style="display:flex;flex-direction:column;gap:10px;max-width:600px;width:100%;margin-bottom:32px;">
    {{NUMBERED_STEPS_AS_PILLS}}
  </div>
  <div style="display:flex;align-items:center;gap:24px;color:var(--color-bg);font-size:1.25rem;">
    {{CONTACT_INFO_ROW}}
  </div>
</div>
```

### icon-trio
**Group:** grid | **Density:** medium or `--bg-alt`
**Content types:** credentials, scope, trust
**Focal point:** left-third intersection (heading + primary content area)
**Use when:** 3-5 features, benefits, or capabilities to present with icons. **For 4+ items**: prefer a 2×2 grid layout (2 rows × 2 columns using `display:grid;grid-template-columns:1fr 1fr`) instead of a single horizontal row — single-row 4+ items causes cramped spacing and uneven text wrapping across descriptions of different lengths. When 4+ items must use a single row, limit each item description to max 8 words.
**Visual:** 3 circle icon containers in a horizontal row with labels below each. Clean, spaced. **Title alignment rule**: Center-align ITEM_TITLE only when titles are ≤15 characters (1–2 words). For longer titles, switch the column to left-aligned: remove `align-items:center` from the column div, set `text-align:left` on title and description. This prevents the hourglass centering pattern (wide icon → narrow wrapped centered title → wide description).
**Shape elements:** Circle containers for icons (not rectangles), short text labels.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 20px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:flex;justify-content:center;align-items:center;gap:48px;">
    {{REPEAT_3_5:
    <div style="display:flex;flex-direction:column;align-items:center;text-align:center;max-width:200px;">
      <div style="width:72px;height:72px;border-radius:50%;background:var(--color-accent-bg);border:1.5px solid var(--color-accent-dim);display:flex;align-items:center;justify-content:center;margin-bottom:16px;">
        <Icon name="{{ICON}}" :size="28" color="var(--color-accent)" />
      </div>
      <span style="font-size:1.25rem;font-weight:700;color:var(--color-text);margin-bottom:4px;">{{ITEM_TITLE}}</span>
      <span style="font-size:1.25rem;color:var(--color-muted);line-height:1.4;">{{ITEM_DESC}}</span>
    </div>
    }}
  </div>
</div>
```

### bento-grid
**Group:** grid | **Density:** medium-high or `--bg-alt`
**Content types:** credentials, scope, portfolio, trust
**Focal point:** upper-left third (featured card area)
**Use when:** 3-5 items of varying importance — one featured, others supporting. **CRITICAL**: Featured card primary metric MUST be ≥3.5rem — place icon ABOVE the metric in a separate row, NOT beside it. Side-by-side icon+number shrinks the number and defeats featured-card hierarchy.
**Visual:** One large zone (60% width) + 2-3 smaller zones stacked beside it. Different card sizes create visual interest.
**Shape elements:** Cards of different sizes (large featured + small supporting), accent card for the most important item.

**FEATURED CELL LAYOUT RULE**: Icon MUST be in its own row ABOVE the metric number — never in a horizontal flex row beside it. Use `display:flex;flex-direction:column;align-items:flex-start` inside the featured cell. Side-by-side icon+number shrinks the perceived size of the metric and collapses hierarchy.

**SIDE CARD DEDUP RULE**: When the featured cell contains a sub-grid of 3+ numbers (e.g., team breakdown), side cards MUST show DIFFERENT data — growth percentages, period comparisons, or qualitatively distinct metrics. NEVER repeat a number already shown in the featured sub-grid.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:grid;grid-template-columns:1.2fr 1fr;grid-template-rows:1fr 1fr;gap:14px;">
    <div style="grid-row:1/3;background:linear-gradient(135deg,rgba(var(--accent-rgb),0.12),var(--color-surface));border:1.5px solid var(--color-accent-dim);border-radius:14px;padding:24px;display:flex;flex-direction:column;justify-content:center;">
      <!-- CORRECT: icon in its own row ABOVE the number -->
      <div style="width:52px;height:52px;border-radius:50%;background:rgba(var(--accent-rgb),0.12);border:1.5px solid var(--color-accent-dim);display:flex;align-items:center;justify-content:center;margin-bottom:12px;">
        <Icon name="{{ICON}}" :size="22" color="var(--color-accent)" />
      </div>
      <div style="font-size:4rem;font-weight:800;color:var(--color-accent);line-height:1;font-family:var(--font-heading);">{{FEATURED_NUMBER}}</div>
      <div style="font-size:1.1rem;color:var(--color-muted);font-family:var(--font-body);margin-top:4px;">{{FEATURED_LABEL}}</div>
      {{FEATURED_SUPPORTING_CONTENT}}
    </div>
    <div style="background:var(--color-surface);border:1px solid var(--color-surface-border);border-radius:14px;padding:18px 22px;display:flex;align-items:center;gap:14px;">
      {{ITEM_2}}
    </div>
    <div style="background:var(--color-surface);border:1px solid var(--color-surface-border);border-radius:14px;padding:18px 22px;display:flex;align-items:center;gap:14px;">
      {{ITEM_3}}
    </div>
  </div>
</div>
```

### two-col-text
**Group:** split | **Density:** medium or `--bg-alt`
**Content types:** context, scope, terms, trust
**Focal point:** left-third intersection (heading above left column)
**Use when:** Content splits naturally into 2 groups or a before/after comparison.
**Visual:** Equal-width columns separated by a subtle vertical divider or gap. Heading spans full width.
**Shape elements:** Vertical divider line (optional), pill labels for column headers.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 20px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:grid;grid-template-columns:1fr 1fr;gap:32px;align-content:center;">
    <div>
      {{LEFT_COLUMN_CONTENT}}
    </div>
    <div>
      {{RIGHT_COLUMN_CONTENT}}
    </div>
  </div>
</div>
```

### asymmetric-split
**Group:** split | **Density:** medium
**Content types:** context, vision
**Focal point:** right-third intersection (text column offset from visual)
**Use when:** One visual element (icon, illustration, metric) paired with explanatory text.
**Visual:** 40/60 split — left side has a large visual element or hero metric, right side has text content.
**Shape elements:** Large circle or rounded container for the visual element on the narrow side.

```html
<div style="position:absolute;inset:0;z-index:1;display:grid;grid-template-columns:2fr 3fr;padding:44px 64px;gap:40px;align-items:center;">
  <div style="display:flex;justify-content:center;align-items:center;">
    {{VISUAL_ELEMENT}}
  </div>
  <div style="display:flex;flex-direction:column;justify-content:center;">
    <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
    <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
    {{TEXT_CONTENT}}
  </div>
</div>
```

### card-mosaic
**Group:** grid | **Density:** medium-high or `--bg-alt`
**Content types:** credentials, team, portfolio
**Focal point:** upper-left third (heading + first card)
**Use when:** 4 items of equal importance — portfolio pieces, team cards, project examples.
**Visual:** 2x2 grid of cards. One card uses accent styling for emphasis. Cards have different internal layouts (some with titles, some with icons).
**Shape elements:** Cards with varying borders (solid + accent highlight for one), rounded corners.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:grid;grid-template-columns:1fr 1fr;grid-template-rows:1fr 1fr;gap:14px;align-items:stretch;">
    {{CARD_1_SOLID}}
    {{CARD_2_SOLID}}
    {{CARD_3_SOLID}}
    {{CARD_4_ACCENT}}
  </div>
</div>
```

### timeline-horizontal
**Group:** timeline | **Density:** medium
**Content types:** process
**Focal point:** left-third (first step as entry point)
**Use when:** 4-6 sequential steps or stages.
**Visual:** 3×2 grid of stage cards with stage number/period labels. Footer with milestones.
**Shape elements:** Stage number labels in accent color, milestone dots at bottom.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:grid;grid-template-columns:1fr 1fr 1fr;grid-template-rows:1fr 1fr;gap:12px;align-items:stretch;">
    {{REPEAT_STAGES:
    <div style="background:var(--color-surface);border:1px solid var(--color-surface-border);border-radius:12px;padding:14px 18px;display:flex;flex-direction:column;justify-content:center;">
      <span style="display:block;font-size:0.65rem;color:var(--color-accent);font-weight:700;text-transform:uppercase;letter-spacing:0.1em;margin-bottom:4px;">{{STAGE_LABEL}}</span>
      <span style="font-size:1.25rem;color:var(--color-text);line-height:1.35;">{{STAGE_CONTENT}}</span>
    </div>
    }}
  </div>
  <div style="display:flex;justify-content:center;gap:24px;margin-top:12px;padding:10px 0;border-top:1px solid var(--color-surface-border);">
    {{MILESTONES_ROW}}
  </div>
</div>
```

### timeline-zigzag
**Group:** timeline | **Density:** medium
**Content types:** process
**Focal point:** upper-left third (heading + first zigzag item)
**Use when:** Long process (6+ steps) that needs compact vertical presentation.
**Visual:** Alternating left-right entries along a vertical center line. Each entry has a number badge and description.
**Shape elements:** Circle number badges, vertical connector line (CSS pseudo-element).

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:flex;flex-direction:column;justify-content:center;gap:8px;">
    {{REPEAT_STEPS:
    <div style="display:flex;align-items:center;gap:16px;">
      <div style="width:36px;height:36px;border-radius:50%;background:var(--color-accent-bg);border:1.5px solid var(--color-accent-dim);display:flex;align-items:center;justify-content:center;flex-shrink:0;">
        <span style="font-size:0.8rem;font-weight:700;color:var(--color-accent);">{{STEP_NUM}}</span>
      </div>
      <span style="font-size:1.25rem;color:var(--color-text);line-height:1.35;">{{STEP_TEXT}}</span>
    </div>
    }}
  </div>
</div>
```

### profile-grid
**Group:** grid | **Density:** medium
**Content types:** team
**Focal point:** upper-left third (heading + first profile card)
**Use when:** Showing team members, advisors, or key personnel.
**Visual:** 2×3 grid of profile cards. Each has a circle container for photo/icon + name + role. Ghost card style.
**Shape elements:** Circle containers for profile icons/photos, ghost border cards.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:grid;grid-template-columns:1fr 1fr;grid-template-rows:1fr 1fr 1fr;gap:10px;align-items:stretch;">
    {{REPEAT_MEMBERS:
    <div style="display:flex;align-items:center;gap:14px;background:transparent;border:1.5px solid var(--color-accent-dim);border-radius:12px;padding:0 20px;">
      <div style="width:40px;height:40px;border-radius:50%;background:var(--color-accent-bg);border:1px solid var(--color-accent-dim);display:flex;align-items:center;justify-content:center;flex-shrink:0;">
        <Icon name="{{ICON}}" :size="20" color="var(--color-accent)" />
      </div>
      <span style="font-size:1.25rem;color:var(--color-text);line-height:1.35;">{{MEMBER_INFO}}</span>
    </div>
    }}
  </div>
</div>
```

### data-spotlight
**Group:** table | **Density:** high
**Content types:** metric, trust
**Focal point:** upper-left third (heading + primary metric)
**Use when:** Detailed breakdown — cost table, statistics grid, KPI dashboard.
**Visual:** 3×2 grid of metric cards. Top row has larger numbers, bottom row smaller. One row may use accent styling.
**Shape elements:** Centered numbers as typographic elements (no wrapping cards for the number itself), compact info cards.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:48px 80px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.18em;color:var(--color-accent);font-weight:600;margin-bottom:16px;">{{LABEL}}</span>
  <h1 style="font-size:2.2rem;font-weight:700;color:var(--color-text);margin:0 0 24px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:12px;max-width:720px;width:100%;">
    {{METRIC_CARDS_AS_CENTERED_NUMBERS}}
  </div>
  <p style="font-size:1.25rem;color:var(--color-accent);margin-top:20px;font-weight:600;">{{BOTTOM_CALLOUT}}</p>
</div>
```

### comparison-table
**Group:** table | **Density:** high or `--bg-alt`
**Content types:** terms, scope
**Focal point:** upper-left third (heading + first comparison row)
**Use when:** Side-by-side comparison or structured list of conditions/features.
**Visual:** Vertical rows with icons. Ghost card style. Last row may have accent highlight.
**Shape elements:** Row-based layout with icons, ghost borders, optional accent on last row.

```html
<div style="position:absolute;inset:0;z-index:1;display:flex;flex-direction:column;padding:44px 64px;">
  <span style="display:block;font-size:0.7rem;text-transform:uppercase;letter-spacing:0.15em;color:var(--color-accent);font-weight:600;margin-bottom:6px;">{{LABEL}}</span>
  <h1 style="font-size:2.1rem;font-weight:700;color:var(--color-text);margin:0 0 16px;font-family:var(--font-heading);">{{HEADING}}</h1>
  <div style="flex:1;display:grid;grid-template-rows:repeat({{N}},1fr);gap:8px;align-items:stretch;">
    {{REPEAT_ROWS:
    <div style="display:flex;align-items:center;gap:14px;background:transparent;border:1px solid var(--color-surface-border);border-radius:10px;padding:0 20px;">
      <Icon name="{{ICON}}" :size="20" color="var(--color-accent)" />
      <span style="font-size:1.25rem;color:var(--color-text);">{{ROW_CONTENT}}</span>
    </div>
    }}
  </div>
</div>
```
