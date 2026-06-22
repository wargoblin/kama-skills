# Universal Design Rules for Presentations

*Synthesized from research by Garr Reynolds (Presentation Zen), Nancy Duarte (Resonate / Slide:ology), Edward Tufte (The Cognitive Style of PowerPoint), Richard Mayer (multimedia learning theory), Nielsen Norman Group (eyetracking studies), WCAG accessibility standards, and the IxDF visual hierarchy guidelines.*

These rules apply to **every slide regardless of template or style**. They govern readability, cognitive load, hierarchy, contrast, and structural integrity — not aesthetics. Style choices belong to the Figma preset. These rules hold across all presets.

---

## Section 1: Text Density & Content Limits

### The Core Limit: One Idea Per Slide

Every slide must communicate exactly one central idea. Nancy Duarte's "3-second test" is the operational standard: if a viewer cannot grasp what the slide is about within 3 seconds, the slide is too complex. This is not a soft guideline — it is a design constraint. A slide that requires 10 seconds of reading before the idea becomes apparent has already lost the audience.

This principle is supported by the research of Nielsen Norman Group (sample of 45,237 page views): users allocate only 4.4 additional seconds for every 100 additional words. Extra content is almost entirely ignored. Duarte states this directly: "Presentations are instant media... Can your message be absorbed in three seconds? If not — simplify."

### Maximum 75 Words Per Content Slide

**Threshold:** More than 75 words on a single content slide = wall of text. More than 150 words = a document disguised as a slide.

**Why:** Human working memory holds 3–5 chunks of new information (Nelson Cowan, 2001 — a refinement of Miller's classic 7±2, which applies to familiar categories, not novel content). A slide with 120 words presents far more than 5 chunks simultaneously. The brain cannot process spoken speech and read written text with equal quality at the same time — this is Mayer's Redundancy Principle: when the same information appears in two modalities simultaneously (spoken + written), recall is worse than either modality alone.

**When violated:** Count the words. Move everything above 75 to speaker notes. Rephrase every bullet as a short declarative phrase (≤ 6 words). If the slide still exceeds the limit, split it into two slides — one idea each.

### The 6×6 Rule (Maximum 6 Bullets, 6 Words Each)

**Threshold:** More than 6 bullet points per slide = cognitive overload begins. More than 6 words per bullet = the viewer enters reading mode rather than listening mode. Some practitioners use the 5×5 rule or even 4×4 for high-impact presentations.

**Why:** Each additional bullet forces the audience to track more items simultaneously. Long bullets become mini-paragraphs that require reading, which competes with listening. Dense bullet lists appear as undifferentiated noise — no item stands out.

**When violated:** Reduce to 3–5 most important points. Shorten each to a 3–6 word phrase. Remove sub-bullets entirely (if they exist, the parent bullet is too abstract). Consider replacing the list with a diagram that shows relationships between items.

### Maximum 2 Levels of Nesting

**Threshold:** Nesting deeper than 2 levels (sub-sub-bullets) = structural confusion guaranteed.

**Why:** Each level of indentation requires the viewer to track hierarchical context. Chains of sub-bullets hide the main point. Edward Tufte argued that the bullet-point format "fragments, serializes, and confuses" complex information — nesting makes this exponentially worse. The eye must travel left and right across indentation levels, breaking smooth reading.

**When violated:** Flatten the hierarchy. Move deep sub-bullets to their own slides. Replace nested lists with numbered steps (for sequences) or diagrams (for hierarchies).

### 3–5 Information Units Per Slide (Cowan 2001)

**Threshold:** More than 5 distinct new information units per slide = risk of cognitive overload for most of the audience.

**Why:** Cowan's 2001 research established that the real working memory limit for novel content is 3–5 chunks. Miller's "magical number seven" applied to familiar categories (months of the year count as one chunk even though there are 12). On a presentation slide, each independent data point, concept, or claim is a separate chunk.

**When violated:** Group information into labeled categories (each category = one chunk). Break the slide into multiple slides. Follow Mayer's Segmentation Principle: divide complex content into learner-manageable segments.

### 6+ Lines of Text = Overload Zone

**Threshold:** More than 6 lines of text on one slide = cognitive overload zone, regardless of word count.

**Why:** Even if each line is short, 6+ lines create a wall effect. NNGroup eyetracking research shows users read only 20–28% of words on a page viewed in real time. On slides viewed simultaneously with a speaking presenter, the fraction is lower still.

**When violated:** Reduce to 4–5 lines maximum. Convert excess lines to speaker notes. Replace text blocks with a single declarative headline.

---

## Section 2: Typography & Readability

### Font Size Minimums

Font size rules for slides displayed on screen or projected (using the rem scale where 1rem = 16px on a standard 1280px canvas):

| Role | Minimum | Notes |
|------|---------|-------|
| Body text / bullets | 1.25rem (≈20px) | 24pt minimum for projected slides per Practical Typography |
| Slide heading | 2.2rem (≈35px) | For regular content slide titles |
| Caption / label / footnote | 0.65rem (≈10px) | Absolute floor — below this, content should be removed |
| Hero / statement text | 3.5rem+ (≈56px+) | For statement slides, metrics, cover titles |

Guy Kawasaki's "30pt rule" encapsulates the spirit: if you need text smaller than 30pt on a projected slide, you have too much content. The practical translation for HTML/CSS slides: body text below 1.25rem is always a violation.

**When violated:** Increase font size until readability is not in question. Remove content rather than shrinking text. Never reduce below minimums to fit content — shorten the content instead (see Hard Constraint in SKILL.md).

### Maximum 2 Font Families

**Threshold:** More than 2 font families in a presentation = visual chaos. More than 3 font families = explicit anti-pattern.

**Why:** Every font change disrupts the reading rhythm. Multiple font "personalities" create visual noise — no unified design voice. It also undermines credibility: if font choices appear random, content choices may appear random too (IxDF guideline: "one or two font families" maximum for professional appearance).

**Rule:** Use one font family with weight variations (Light, Regular, Bold, Black) for hierarchy, OR two families chosen for intentional contrast (geometric + humanist, display + body). The contrast between them should be deliberate, not accidental.

**When violated:** Consolidate to one or two families. Use weight and size variation within one family to create hierarchy before adding a second family.

### Body Text: Left-Align Only

**Threshold:** More than 2 lines of center-aligned body text = readability failure. Any center-aligned paragraph = anti-pattern.

**Why:** Center-aligned text blocks are hard to read because both edges are ragged. The ragged left edge destroys the anchor point the eye uses to find the start of each new line. Professional publications almost never center body text — only short single-line headings and decorative elements. Practical Typography: "Centered blocks of text are hard to read because the left edge is uneven." Left-aligned text with a consistent left margin supports the F-pattern and Z-pattern reading behaviors documented by NNGroup eyetracking research.

**Correct usage:** Center-align only for single-line headings, section dividers, standalone quotes, and cover slide titles. Left-align everything else: bullets, body paragraphs, multi-line labels.

### Line Height: 1.2–1.45× for Body, 0.9–1.1× for Headings

**Thresholds (Practical Typography):**
- Body text below 1.2× = text feels cramped, hard to track from line to line
- Body text above 1.7× = text falls apart into disconnected fragments
- Headings: tighter is correct — 0.9–1.1× because large display text needs less air between lines
- The common "double spacing" (2.0×) significantly exceeds the acceptable range

**Why:** Line height governs whether the reader's eye can smoothly return from the end of one line to the start of the next. Too tight and lines merge visually; too loose and the text loses coherence.

### Maximum Line Width: 65ch

**Threshold:** Lines longer than 65–75 characters (65ch) = difficult to track from end of one line to start of next. Lines shorter than 45ch = forced line breaks disrupt reading rhythm.

**Why:** Smashing Magazine and Practical Typography converge on 45–75 characters as the optimal range for reading comfort. Slides with full-width text columns create very long lines that look like documents rather than slides. Narrow text columns improve readability and add visual breathing room.

**When violated:** Constrain text containers with `max-width: 65ch` or equivalent. Add left/right padding to text areas. Never let text run edge-to-edge on a wide slide.

### Bold ≤ 20% of Text

**Threshold:** More than 20–25% of text bolded = emphasis loses meaning. When everything is bold, nothing is bold.

**Why:** Bold is a comparative signal — it works only relative to unbolded text. Practical Typography: "When everything is emphasized, nothing is emphasized." If 50% of text is bold, bold no longer means "important."

**Correct usage:** Bold should mark only the truly critical 2–4 words in a block. Use bold for 10–15% of text at most — the words that must not be missed.

**When violated:** Reduce bold to only words of genuine urgency. If everything seems important, the solution is to cut content until only the important remains.

### One Emphasis Technique Per Element

Use one technique at a time: bold OR color change OR size increase — not bold + color + underline simultaneously. Multiple competing emphasis techniques create visual noise rather than hierarchy.

**Never use underline** on screen — it signals hyperlink and confuses the viewer.

**Italic:** Reserve exclusively for direct quotes, attributions, and captions. Never use italic in headings. Italic is the weakest emphasis technique for slides.

### Tabular-Nums on Metrics

Any numeric metric displayed as a hero number or in a table must use `font-feature-settings: "tnum"` (tabular numbers). This ensures digits align vertically and have equal width, preventing numbers from visually "jumping" when they change or are compared.

---

## Section 3: Spacing & Layout

### Minimum 30–40% Whitespace

**Threshold:** Less than 30% empty space = approaching clutter. Less than 20% = explicit overload problem. More than 7 separate visual elements on one slide = cognitive overload (IxDF visual hierarchy guideline: "Limit the number of large elements to a maximum of two").

**Why:** Whitespace is not wasted space — it is an active design element. It isolates important content and allows it to breathe, reduces cognitive load, creates a sense of elegance and professionalism, and directs the eye toward focal points. IxDF research: formatted content for scanning yields a 47% improvement in usability compared to dense, unformatted content. Garr Reynolds: amateur slides are densely packed; professional slides have generous whitespace.

**Whitespace as design:** More whitespace around an element = more visual weight for that element. The empty space itself is a design choice that makes content matter. Resist the instinct to fill every centimeter of the slide.

**When violated:** Remove every element that does not directly support the single central message. Increase margins. If the slide feels empty, do not fill it — emptiness creates emphasis.

### 8px Grid System

All element positioning, padding, and gaps must use multiples of 8px. This creates visual consistency without visible effort. Standard spacing values: 8, 16, 24, 32, 48, 64, 80px.

**Why:** An 8px grid aligns with most typography's natural rhythm and most icon sizing. It produces layouts that feel "right" without being able to articulate why. Misalignment by even 3–4px is detectable by the human eye and reads as carelessness.

### Consistent Margins: 5–8% of Slide Width

Slide margins must be consistent across all slides in the deck and must be at least 5–8% of the slide width on each side. On a 1280px canvas, this means minimum 64–100px left/right margins.

**Why:** Consistent margins create the invisible grid that organizes visual space. IxDF: misaligned elements signal negligence — "If the designer didn't notice this, what else did they miss?" Never let content run edge-to-edge unless it is a deliberate full-bleed design choice from the Figma template.

### No Floating Elements

Every element on the slide must belong to a visible or implied group. An element "floating" without proximity to related content violates Gestalt Proximity: elements placed close together are perceived as a group; elements clearly not adjacent to anything are perceived as unrelated.

**When violated:** For each isolated element, ask: what does it relate to? Position it visually adjacent to that thing. If it relates to nothing, remove it.

### Maximum 7 Visual Elements Per Slide

Count every distinct visual unit: text block, image, icon, chart, decorative shape, divider line. More than 7 items = cognitive overload. IxDF: "Limit the number of large elements to a maximum of two."

**When violated:** Merge related elements into groups (a group of 3 items counts as 1 visual unit). Remove decorative elements that add no information.

---

## Section 4: Visual Hierarchy

### Minimum 3:1 Heading-to-Body Size Ratio

**Threshold:** Heading must be at least 3× the size of body text. If the heading is 1.8rem, body must be no larger than 0.6rem — but since body has its own minimum (1.25rem), the heading must scale up to create the ratio.

**Why:** The brain performs "pre-attentive" scanning — it registers differences in size, contrast, and color before conscious attention begins. Without a clear size hierarchy, pre-attentive scanning finds nothing, and full attention is required to parse every element. This is extremely cognitively expensive. IxDF: "When all elements receive the same emphasis, nothing stands out — viewers experience visual noise." Users have 50 milliseconds to form a first impression — they need hierarchy to immediately understand what matters.

**Squint test:** Squint at the slide until it blurs. The most important element must be obvious even with blurred vision. If the heading and body text appear to have similar visual weight, the ratio is insufficient.

### Minimum 3 Hierarchy Levels

Every slide must have at least 3 distinct visual levels: primary (heading), secondary (body/content), and tertiary (labels, captions, footnotes). The three levels must be visually distinguishable by at least two attributes: size, weight, color, or contrast.

**When violated:** Introduce a third level even if it requires adding a small label or footnote. The absence of a tertiary level flattens the slide into a two-level visual space that is harder to scan.

### Maximum 3 Font Sizes Per Slide

**Threshold:** More than 3 distinct font sizes on one slide = visual complexity exceeds the benefit. IxDF: "Use no more than 3 sizes — small, medium, and large."

**When violated:** Collapse size variations to 3: heading, body, and caption. Use weight (bold/regular) to create sub-hierarchy within a size level rather than adding a fourth size.

### Maximum 2 Large Elements

Only 2 elements should dominate a slide. Everything else must be clearly subordinate. If 3 or more elements compete for visual dominance, the viewer cannot determine what to look at first.

**Why:** The IxDF rule: "Limit the number of large elements to a maximum of two." A dominant element must be noticeably larger — not marginally larger — than secondary elements. The difference must be perceptible at a glance.

### Title = Conclusion, Not Label

Slide titles must be active conclusions, not passive category labels.

**Wrong:** "Q3 Revenue Analysis" / "Market Findings" / "Recommendations"
**Right:** "Revenue grew 23% despite headwinds" / "APAC drove 68% of growth" / "Three changes will cut churn by 18%"

**Why (Storytelling with Data / Duarte):** Question titles ("Was our campaign effective?") split attention — the viewer simultaneously tries to answer the question, interpret the chart, and listen to the speaker. Label titles waste the most visible space on the slide. Active conclusion titles immediately establish the takeaway; the viewer can evaluate the supporting evidence rather than trying to extract the conclusion.

**When violated:** Rewrite every label title as a declarative statement: subject + verb + outcome. The title answers the question "so what?"

---

## Section 5: Contrast & Accessibility

### WCAG AA Minimums: 4.5:1 Body, 3:1 Large Text

**WCAG AA thresholds (minimum for professional presentations):**
- Normal text (below 18pt regular / 14pt bold): contrast ratio minimum **4.5:1**
- Large text (18pt+ regular or 14pt+ bold): contrast ratio minimum **3:1**
- Graphic elements and UI components: minimum **3:1** against adjacent colors

**WCAG AAA (enhanced accessibility):**
- Normal text: **7:1**
- Large text: **4.5:1**

**Critical precision:** #777777 gray on white produces a ratio of 4.47:1 — this does NOT pass the 4.5:1 requirement. Never round up to the threshold value.

**Typical violations:**
- Light gray on white: #999999 on white = ~2.8:1 (fails)
- Yellow on white: ~1.07:1 (catastrophic failure)
- White on yellow: ~1.07:1 (catastrophic failure)

### 7:1+ for Projection

For projected presentations in physical rooms, target 7:1 or higher for body text. Projectors wash out colors — particularly yellows and light blues. A ratio that passes on screen may fail in the room. Use near-black text on near-white backgrounds, or white text on very dark backgrounds.

**Dark-mode presentations:** White or near-white text (#f0f4ff or similar) on backgrounds dark enough to achieve 7:1. Never pure #000000 backgrounds (use tinted variants: #0c0e16, #0f1724).

**When violated:** Use a contrast checker (WebAIM Contrast Checker). Darken text or lighten the background until the ratio passes. Never use medium-toned combinations (mid-gray on mid-toned background).

### No Color-Only Differentiation

Color must never be the sole means of conveying information. Every color distinction must be redundant with shape, pattern, label, or position.

**Why:** Approximately 300 million people worldwide are colorblind. Red-green color blindness (deuteranopia + protanopia) affects approximately 8% of men and 0.5% of women of Northern European descent. In an audience of 50, 2–4 people likely have some form of color vision deficiency. The WCAG principle 1.4.1 explicitly prohibits color as the only visual differentiator.

**Common violation:** Using red for "bad/decline" and green for "good/growth" in data charts. These look identical to 8% of male viewers.

**Fix:** Add labels, patterns, or shapes alongside color. Replace red/green pairs with blue/orange for maximum colorblind safety.

### Grayscale Test

Convert any data visualization to grayscale (or mentally test). If the chart loses meaning — if categories become indistinguishable — the chart was relying on color as its sole differentiator. Redesign before using color.

### No Pure Black / No Pure White Backgrounds

Pure #000000 and pure #ffffff are visually harsh. Use tinted variants:
- Dark presentations: #0c0e16, #0f1724, #111828
- Light presentations: #faf9f6, #f8f9fa, #f1f5f9

The tinting reduces eye strain and makes the design feel intentional rather than default.

---

## Section 6: Element Spacing & Overflow Prevention

### No Overlap — Ever

No text, image, or content element may overlap another content element. Overlap of content elements is always a layout failure. The only permitted overlapping is deliberate decorative layering from the Figma template (background orbs, texture patterns, decorative shapes) that are behind content as locked background layers.

**Why:** Overlapping content elements force the viewer to mentally separate them, adding cognitive work with no benefit. Overlap also signals layout failure — it reads as "this didn't fit."

**When violated:** Reduce content (shorten/rephrase) or restructure the layout. Never scale elements down to overlap less — fix the underlying content or layout cause.

### Footer Width Check

Footer elements (slide number, logo, source citation) must never extend beyond the slide boundary. The footer zone (bottom 44–56px of a 540px canvas, or equivalent 8–10% of slide height) must be reserved and kept clear of main content.

**Why:** Footer overflow is a common failure point in templates. The footer zone must be reserved at layout design time — content placed without awareness of this zone will collide with footer elements.

**Rule:** Reserve the bottom 44–56px as a no-content zone for all non-footer elements. Footer elements themselves must fit within this zone without overlap.

### Minimum 8px Gap Between All Elements

No two content elements may have less than 8px of clear space between them. This is the minimum gap for elements to be visually distinct rather than merged.

**For related elements (grouped content):** 8–16px internal spacing.
**For unrelated elements (separate sections):** 24–48px spacing to signal separation.

### Text Shortening Priority Order

When text overflows or risks overflow, apply this priority order:

1. **Rephrase** the content to communicate the same meaning in fewer words (preserve core meaning)
2. **Move secondary details** to speaker notes
3. **Split the slide** into two slides (last resort)

**CRITICAL: Never reduce font size below minimums** (body ≥ 1.25rem, heading ≥ 2.2rem, label ≥ 0.65rem). Shrinking text to fit is always wrong — the content must be reduced instead.

### Absolute Positioning Only for Decorative Elements

Absolute positioning in Figma canvas (non-auto-layout frames) for main content elements is an anti-pattern: it breaks when text length changes. Use absolute positioning only for:
- Slide number badges
- Logo placement
- Decorative shapes (behind content, `z-index: 0` equivalent — locked layers)
- Background textures

All content elements must be positioned by Figma auto-layout (horizontal/vertical) so they reflow correctly when content changes.

---

## Section 7: Data Visualization

### No 3D Charts — Ever

3D charts are a critical failure. The 3D perspective applies foreshortening: segments at the front of the chart appear larger than segments at the back, even when they represent identical values. This is pure perceptual distortion — the chart actively lies about the data.

Edward Tufte's "Lie Factor" concept applies directly: when the graphical representation distorts the actual differences in the data, the chart is dishonest. Cole Nussbaumer Knaflic (Storytelling with Data): "We're helping rid the world of ineffective graphs, one 3D pie chart at a time."

**No exceptions:** Never use 3D effects on bars, pies, donuts, or any chart type.

### Maximum 5–6 Segments in Pie / Donut Charts

**Threshold:** More than 6 segments in a pie or donut = impossible to compare meaningfully.

**Why:** Human eyes are poor at judging angles and arc lengths. We cannot accurately assess relative proportions from circular shapes. When segments are close in size, it is impossible to determine which is larger. Even when sizes differ, viewers can determine rank but not magnitude of difference. A darker segment appears larger than an equal-sized lighter segment — color distorts size perception.

**Fix:** If you need more than 6 categories, use a horizontal bar chart. Bars aligned to a common baseline allow viewers to compare not just what is bigger but by how much.

**Rule:** Largest segment at 12 o'clock, decreasing clockwise. Label segments directly — never rely on a color-coded legend. Prefer donut over pie (empty center improves readability and allows a key number to appear in the center).

### Bar Charts: Axis Must Start at Zero

**Threshold:** Any bar chart with a Y-axis that does not start at zero is a misleading chart.

**Why:** Bar length encodes value. A truncated axis makes a 3% difference appear as a massive visual difference. This is a fundamental graphical dishonesty. The only legitimate exception: when zero has no meaningful interpretation in the context AND the truncation is explicitly labeled.

### Maximum 5–6 Lines on Line Charts

**Threshold:** More than 6 lines on a single line chart = visual overload. More than 6–7 categories with distinct colors in any chart = a legend no one can use.

**Fix:** Label lines directly rather than using a color-coded legend. If more than 6 lines are required, split into small multiples (several smaller charts of the same type for comparison).

### Remove Gridlines

Gridlines should be removed or made extremely light (10–20% opacity). Heavy gridlines are chartjunk (Tufte). If gridlines are needed for orientation, use only the minimum necessary.

Remove all of: decorative backgrounds on chart areas, borders and boxes around charts, drop shadows on bars or segments, gradient fills on chart elements. Keep only what encodes data.

### Hero Metric ≥ 2× Supporting Metrics

When displaying a primary metric alongside supporting metrics, the hero number must be at least 2× larger (visually) than supporting numbers. All metrics same size in identical cards = no hierarchy = no communication.

**Pattern:** Hero metric at 3.5rem+, supporting metrics at 2.2rem minimum.

### Cite Sources

Every data visualization must include a source citation: small text (caption/label size), positioned below or at the bottom-left of the chart. Format: "Source: [name], [year]" or similar. Removing source citations undermines credibility and prevents fact-checking.

---

## Section 8: Figma Prototyping

Figma presentations use Smart Animate and prototype transitions for slide navigation. These are set in the template and preserved by clone — figmadeck does not modify them.

**What figmadeck preserves:** existing prototype connections between slides (navigate-to actions, transition types, animation curves).

**What figmadeck updates after reorder:** sequential navigation connections are remapped to match the new slide order (see figma-generation.md Step 4).

**What is NOT supported:** creating new animations, modifying transition types, adding interactive overlays. These must be done manually in Figma after generation.

---

## Quick-Reference Checklist

Use this checklist before finalizing any slide:

**Content:**
- [ ] One idea per slide — passes 3-second test
- [ ] ≤ 75 words on content slides
- [ ] ≤ 6 bullets, ≤ 6 words per bullet
- [ ] Nesting depth ≤ 2 levels
- [ ] ≤ 5 distinct new information units

**Typography:**
- [ ] Body ≥ 1.25rem, heading ≥ 2.2rem, label ≥ 0.65rem
- [ ] ≤ 2 font families, ≤ 3 font sizes per slide
- [ ] Body text: left-aligned only
- [ ] Line height: 1.2–1.45× body, 0.9–1.1× headings
- [ ] Line width: ≤ 65ch
- [ ] Bold ≤ 20% of text
- [ ] One emphasis technique per element
- [ ] Italic only in quotes/attributions

**Spacing & Layout:**
- [ ] ≥ 30% whitespace
- [ ] 8px grid alignment
- [ ] Margins ≥ 5% slide width on each side
- [ ] No floating elements
- [ ] ≤ 7 visual elements
- [ ] Footer zone (44–56px bottom) reserved

**Visual Hierarchy:**
- [ ] Heading ≥ 3× body text size
- [ ] ≥ 3 hierarchy levels present
- [ ] ≤ 3 font sizes per slide
- [ ] ≤ 2 large elements
- [ ] Squint test passes
- [ ] Title is a conclusion, not a label

**Contrast:**
- [ ] Body text: ≥ 4.5:1 (WCAG AA)
- [ ] Large text: ≥ 3:1 (WCAG AA)
- [ ] Projection target: ≥ 7:1
- [ ] No color-only differentiation
- [ ] Grayscale test passed
- [ ] No pure #000/#fff backgrounds

**Overflow:**
- [ ] No content overlap (except decorative layers)
- [ ] Footer width within bounds
- [ ] ≥ 8px gap between all elements
- [ ] Overflow resolved by shortening, not font shrinking

**Data Visualization:**
- [ ] No 3D effects on any chart
- [ ] Pie/donut: ≤ 6 segments
- [ ] Bar chart Y-axis starts at zero
- [ ] Line chart: ≤ 6 lines
- [ ] Gridlines removed or very light
- [ ] Hero metric ≥ 2× supporting metrics
- [ ] Source citation included

**Figma Prototyping:**
- [ ] Prototype connections point to correct slide order
- [ ] Auto-layout constraints preserved after text fill
