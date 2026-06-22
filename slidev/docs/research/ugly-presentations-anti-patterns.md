# Ugly Presentations Anti-Pattern Bible
## A Comprehensive Diagnostic Checklist of Everything That Makes Presentations Fail

> **How to use this document:** Each anti-pattern includes a severity rating, detection checklist, and fix. Use it as a diagnostic tool — work through each category to identify exactly what's wrong with any bad presentation.

**Severity Scale:**
- 🔴 **CRITICAL** — Kills the presentation outright; audience disengages or mistrusts information
- 🟠 **MAJOR** — Seriously damages effectiveness and credibility
- 🟡 **MINOR** — Noticeable unprofessionalism, weakens impact

---

## Table of Contents

1. [The Death by PowerPoint Phenomenon](#1-the-death-by-powerpoint-phenomenon)
2. [Text & Content Overload](#2-text--content-overload)
3. [Bullet Point Hell](#3-bullet-point-hell)
4. [Typography Crimes](#4-typography-crimes)
5. [Color & Contrast Failures](#5-color--contrast-failures)
6. [Layout & Composition Disasters](#6-layout--composition-disasters)
7. [Visual Hierarchy Failures](#7-visual-hierarchy-failures)
8. [Data Visualization Failures](#8-data-visualization-failures)
9. [Image & Media Abuse](#9-image--media-abuse)
10. [Structural & Narrative Failures](#10-structural--narrative-failures)
11. [Cognitive Overload Patterns](#11-cognitive-overload-patterns)
12. [Inconsistency & Template Abuse](#12-inconsistency--template-abuse)
13. [AI-Generated Presentation Failures](#13-ai-generated-presentation-failures)
14. [Accessibility Failures](#14-accessibility-failures)
15. [Delivery-Design Disconnect](#15-delivery-design-disconnect)
16. [The Master Diagnostic Checklist](#16-the-master-diagnostic-checklist)
17. [Severity Reference Table](#17-severity-reference-table)

---

## 1. The Death by PowerPoint Phenomenon

### What It Is

"Death by PowerPoint" is the umbrella term for the collective suffering audiences experience when a presentation is rendered ineffective through a combination of poor design, cognitive overload, and bad delivery. The term was popularized and disseminated through Alexei Kapterev's iconic SlideShare presentation that became one of the most viewed presentations ever shared online. The core insight: **presentations can literally kill their message by the very act of using slides incorrectly**.

The phenomenon has four root causes that compound each other:
1. **Lack of significance** — No clear purpose or meaningful central message
2. **Poor structure** — Disorganized content flow with no narrative arc
3. **Complexity** — Excessive information and visual clutter on every slide
4. **Insufficient rehearsal** — Unpolished delivery that makes design problems worse

### Why It Matters

Research from Nielsen Norman Group establishes that users read only **20–28% of the words on a page** when they are present. On slides viewed in real time, this percentage is lower still, because attention is divided between listening to the presenter and scanning the slide. Every word that gets ignored is wasted design effort. Every slide that overloads working memory reduces retention of subsequent slides.

### The Core Irony

The deeper anti-pattern of Death by PowerPoint is that **presenters use slides as speaker notes**, embedding everything they want to say directly on screen. The result is that audiences either:
- Read ahead and stop listening
- Try to listen and can't process the text simultaneously
- Give up entirely and mentally check out

### Detection Checklist
- [ ] Can you remove the presenter and have the audience just read the slides?
- [ ] Is every slide dense with text that mirrors what will be spoken?
- [ ] Does each slide try to be a standalone document rather than a visual aid?
- [ ] Are there more than 2–3 distinct ideas competing for attention on any slide?

---

## 2. Text & Content Overload

### 2.1 The Wall of Text

🔴 **CRITICAL**

**What it looks like:** A slide covered edge-to-edge with dense paragraphs or long bullet points, with little or no whitespace. The slide resembles a page from a Word document or an email.

**Why it fails:**
- Human working memory can hold approximately 7 items (±2) at a time — Miller's Law, 1956
- More practically, Nelson Cowan's 2001 research suggests the actual limit is closer to **3–5 chunks** of new information
- Eye-tracking research from Nielsen Norman Group shows users read only 20% of available text on average
- Dense text walls trigger the "too long; didn't read" response — users disengage within seconds
- The brain cannot simultaneously process spoken words and read written text at the same quality — Mayer's Redundancy Principle

**Measurable thresholds:**
- More than 6 lines of text on a single slide = cognitive overload territory
- More than 75 words on a single content slide = Wall of Text
- More than 150 words = document masquerading as a slide

**Research data:** Nielsen Norman Group's study of 45,237 page views found that users allocate only 4.4 additional seconds per 100 extra words added — meaning the extra content is almost entirely ignored.

**Detection checklist:**
- [ ] Count the words on each slide — anything above 75 is suspect
- [ ] Can you read all text comfortably in under 8 seconds?
- [ ] Does the slide have at least 30–40% empty (whitespace) area?
- [ ] Are paragraphs used where bullet points or visuals would suffice?

**How to fix:**
- Apply the "one idea per slide" rule ruthlessly
- Move supporting detail to speaker notes
- Replace paragraphs with single declarative headlines
- Cut until each slide communicates one clear message in under 10 words

---

### 2.2 The 6×6 Rule Violation

🟠 **MAJOR**

**What it looks like:** Slides with more bullet points and words per line than the 6×6 rule allows.

**The rule:** No more than **6 bullet points per slide**, no more than **6 words per bullet point**. Some practitioners advocate even stricter limits: the 5×5 rule (5 bullets, 5 words each) or the 4×4 rule for high-impact presentations.

**Why it fails:**
- Each additional bullet forces the audience to track more items in parallel
- Long bullets become mini-paragraphs that require reading — disrupting listening
- Dense bullet lists are processed as uniform, undifferentiated noise

**Measurable thresholds:**
- More than 6 bullets = cognitive overload begins
- More than 6 words per bullet = forces reading mode
- More than 40 total words on a content slide = violates 6×6 intent

**Detection checklist:**
- [ ] Count bullets per slide — more than 6 is a violation
- [ ] Count words per bullet — more than 6–8 is a violation
- [ ] Does every bullet end in a period (suggesting it's a full sentence/paragraph)?
- [ ] Are bullets nested more than 2 levels deep?

**How to fix:**
- Reduce to the 3–5 most important points
- Trim each bullet to a 3–6 word phrase
- Remove sub-bullets entirely — if sub-bullets exist, the parent bullet is too abstract
- Progressive disclosure via animation: reveal one bullet at a time

---

### 2.3 Jargon & Acronym Overload

🟠 **MAJOR**

**What it looks like:** Slides dense with industry acronyms, technical terms, and domain-specific abbreviations that are never defined. Cole Nussbaumer Knaflic (Storytelling with Data) documented a real example with **more than 50 acronyms on a single slide**.

**Why it fails:**
- Violates the Curse of Knowledge: experts forget what non-experts don't know
- Forces audiences to spend cognitive resources decoding before understanding
- Critical insights get buried beneath layers of shorthand
- Alienates mixed-expertise audiences

**Detection checklist:**
- [ ] Could a smart person outside your field understand every word?
- [ ] Are acronyms defined at first use?
- [ ] Count acronyms per slide — more than 3–4 undefined acronyms = failure
- [ ] Would reading the slide aloud sound like alphabet soup?

**How to fix:**
- Define every acronym on first use, in parentheses
- Spell out the most important terms every time
- Include a glossary slide if many acronyms are unavoidable
- Ask: "What does this actually mean in plain language?"

---

### 2.4 The Presenter Notes Slide

🔴 **CRITICAL**

**What it looks like:** Slides that contain the full text of what the presenter will say — functioning as a teleprompter or script. Every sentence the presenter will speak is also on the screen.

**Why it fails:**
- Mayer's Redundancy Principle: when the same information is presented both verbally and in written form simultaneously, retention is **worse** than either modality alone
- Violates the fundamental rule that "information is interesting only once" — hearing and reading the same content simultaneously creates confirmation of what you already just read, not learning
- Audiences read faster than presenters speak; they finish the slide and then wait impatiently
- The presenter loses authority — they are revealed to be merely narrating their own notes

**Detection checklist:**
- [ ] Read a slide out loud — are you saying exactly what's written?
- [ ] Could an audience member skip your talk and get everything from the slides?
- [ ] Do slides function better as a handout than as a visual backdrop?

**How to fix:**
- Slides should show what cannot be said; speech should add what cannot be shown
- Every word on a slide must earn its place by adding visual/visual-only value
- Move narration-equivalent text to speaker notes
- Create a separate handout document if a written record is needed

---

## 3. Bullet Point Hell

### 3.1 Bullet Point Addiction

🟠 **MAJOR**

**What it looks like:** Every slide consists entirely of nested bullet points. The entire deck is one long hierarchical outline. There are no images, no diagrams, no charts — just bullets, sub-bullets, and sub-sub-bullets.

**Why it fails:**
- Edward Tufte famously argued in "The Cognitive Style of PowerPoint" that bullet-point format "frag­ments, seri­al­izes, and con­fuses" complex information
- Bullets artificially strip context from ideas, hiding the relationships between concepts
- The format implies that all items are equal weight when they are not
- Audiences lose the forest for the trees — the big picture is invisible inside a list
- The fragment structure of bullets actively destroys the logical connectives that make arguments clear (therefore, because, however, as a result)

**Measurable thresholds:**
- More than 3 consecutive bullet-only slides = Bullet Point Hell detected
- Nesting deeper than 2 levels = structural confusion guaranteed
- More than 50% of slides consisting only of text bullets = systemic problem

**Detection checklist:**
- [ ] Do more than half the slides consist only of bullet lists?
- [ ] Are there sub-bullets of sub-bullets (3+ levels of nesting)?
- [ ] Could you restructure any slide as a diagram, chart, or image?
- [ ] Do bullets use fragments instead of complete ideas?

**How to fix:**
- Convert conceptual bullet lists to diagrams showing relationships
- Use visuals to show what bullets describe
- Where bullets are necessary, limit to 3–5 and use progressive disclosure
- Ask: "What relationship do these items have?" and show that relationship visually

---

### 3.2 The Pyramid of Vague Sub-Bullets

🟠 **MAJOR**

**What it looks like:**
```
- Key Point
  - Sub-point
    - Sub-sub-point
      - More detail here
        - Even more detail
```

**Why it fails:**
- Each level of nesting requires cognitive effort to track context
- The hierarchical indentation creates false precision — it implies a logical structure that may not actually exist
- Sub-bullet chains obscure the main point
- The eye must travel left and right across indentation levels, disrupting smooth reading

**Detection checklist:**
- [ ] Are there any bullet points nested 3 or more levels deep?
- [ ] Do sub-bullets directly support the parent bullet or just add loosely related detail?
- [ ] Is the nesting hierarchy visually distinguishable (size, color, indent)?

**How to fix:**
- Flatten hierarchies — move deep sub-bullets to their own slides
- Replace nested lists with numbered steps (for sequence) or diagrams (for hierarchy)
- Kill sub-sub-bullets entirely — if you need them, the parent bullet is too abstract

---

### 3.3 The Fragment Bullet

🟡 **MINOR**

**What it looks like:** Bullets that are incomplete phrases with no grammatical structure or context:
- *"Efficiency gains"*
- *"Customer focus"*
- *"Q3 results"*
- *"Action items"*

**Why it fails:**
- Fragments carry no inherent meaning without context
- Presenter must verbally complete the thought — making the bullet useless as reference
- Audiences looking at the slide while listening cannot match the fragment to the spoken sentence
- When people re-read the deck later, fragments are meaningless

**How to fix:**
- Use complete declarative statements: "Efficiency gains improved by 23% in Q3"
- Or use strong noun phrases: "Q3 efficiency: 23% improvement"
- Every bullet should communicate one complete, standalone idea

---

## 4. Typography Crimes

### 4.1 Font Size Too Small

🔴 **CRITICAL**

**What it looks like:** Body text on slides in 10pt, 12pt, or even 14pt. Charts with 8pt axis labels. Footnotes in 7pt. Text that requires squinting from the front row.

**Why it fails:**
- Standard screens viewed from a distance require dramatically larger fonts than print documents
- The generally accepted minimum for projected slides is **24pt body text**; some advocates insist on 30pt
- For print documents: 10–12pt is acceptable body text (Practical Typography)
- For web: 15–25px is recommended for body text
- Small fonts force audience members to lean forward, creating physical distraction and visual fatigue

**Measurable thresholds:**
- Below 24pt for projected slide body text = too small
- Below 18pt for any text intended to be read from more than 2 meters = too small
- Below 12pt for print body text = too small
- Below 16px for web body text = too small (browsers default to 16px for reason)

**Detection checklist:**
- [ ] Stand 3 meters (10 feet) from your monitor — can you read every text element?
- [ ] Is any body text below 24pt in a projected presentation?
- [ ] Are chart labels, footnotes, or captions legible at presentation distance?
- [ ] Does text in images/screenshots remain readable?

**How to fix:**
- Set minimum font size rules: 30pt titles, 24pt body, 20pt captions — then test by standing back
- Increase font sizes until legibility is uncomfortable to doubt
- Remove text rather than shrinking it to fit
- For charts: regenerate with larger labels, not scaled-down versions

---

### 4.2 Too Many Fonts (Font Salad)

🟠 **MAJOR**

**What it looks like:** A single slide or deck using 5+ different typefaces. Comic Sans for one bullet, Times New Roman for another, Impact for the title, Arial for captions, and a script font for emphasis.

**Why it fails:**
- Each typeface switch disrupts reading rhythm
- Multiple font personalities create visual chaos — no coherent design voice
- Signals amateur design immediately to trained eyes
- Undermines credibility — if font choices are random, content choices may be too

**Measurable thresholds:**
- More than 2 font families in a presentation = Font Salad territory
- More than 3 font families = definite anti-pattern
- Mixing serif and sans-serif *poorly* without intentional contrast = common mistake
- IxDF guidelines recommend: "one or two font families" maximum for professional appearance

**Detection checklist:**
- [ ] Count distinct typefaces across the deck — more than 2 families is suspect
- [ ] Are fonts chosen arbitrarily or because they represent something?
- [ ] Do fonts feel harmonious together or competing?
- [ ] Has anyone called a font "fun" without strategic intent?

**How to fix:**
- Establish a 2-font system: one font for headings, one for body (or single-font with weight/style variation)
- Pair fonts intentionally: high-contrast pairing (e.g., geometric sans + humanist serif) reads as intentional
- Use weight variations (bold, regular, light) within one family for hierarchy instead of new fonts

---

### 4.3 Comic Sans and Novelty Fonts

🟠 **MAJOR**

**What it looks like:** Comic Sans, Papyrus, Jokerman, Curlz, or other novelty/display fonts used for body text or professional content.

**Why it fails (specific fonts):**
- **Comic Sans**: Designed for cartoon speech bubbles, widely mocked as the emblematic marker of amateur design. Destroys credibility in professional contexts because its cultural associations are overwhelmingly juvenile and incompetent
- **Papyrus**: Presents as "ancient/exotic" but delivers neither — Practical Typography describes it as "an alphabet from the early '80s wearing a week's stubble"
- **Bodoni**: Beautiful but its high-contrast thin/thick strokes become "annoying to read after three words" at small sizes
- **Bookman**: "Skunked" by its associations with 1970s design — "If fonts were clothing, this would be the corduroy suit"
- **Copperplate**: Overused to signal sophistication; now signals the opposite
- **Office Script fonts**: "Clunky and cheap" compared to quality calligraphy

**Why novelty fonts fail generally:**
- Novelty fonts "lure inexperienced typographers with false virtues" — the decorative quality obscures rather than enhances content
- They become entrenched as hallmarks of amateurish typography
- Heavy display characteristics make them unsuitable for body text

**Detection checklist:**
- [ ] Is any font primarily known for being "fun," "cute," or "fancy"?
- [ ] Would you use this font in a legal document? If not, reconsider for professional content
- [ ] Does the font look like it came from a late-90s Word processor?

**How to fix:**
- Professional default recommendation: Inter, IBM Plex Sans, Source Sans Pro, Lato, or any system sans-serif for body
- For headings: Fraunces, DM Serif Display, Playfair Display, or high-quality geometric sans
- Test: does the font disappear behind the content or call attention to itself?

---

### 4.4 Centered Body Text

🟠 **MAJOR**

**What it looks like:** Multi-line body text, bullet points, or paragraphs centered rather than left-aligned.

**Why it fails:**
- Practical Typography: "Centered text blocks are difficult to read because both edges of the text block are uneven"
- The ragged left edge destroys the anchor point eyes use to find the start of each new line
- Professional publishing almost never uses centered body text — only short titles and decorative elements
- Left-aligned text with a consistent left margin supports the F-pattern and Z-pattern reading behaviors documented by eye-tracking research
- Centering is "the typographic equivalent of vanilla ice cream—safe but boring"

**Measurable thresholds:**
- More than 2 lines of centered body text = readability failure
- Any paragraph of centered text = anti-pattern
- Centered text in bullet lists = particularly problematic

**Detection checklist:**
- [ ] Is body text center-aligned?
- [ ] Are bullet points centered rather than left-aligned?
- [ ] Is centering used as a default everywhere instead of intentionally for specific elements?

**How to fix:**
- Left-align all body text, bullet points, and extended text blocks
- Reserve centering only for: single-line titles, section headers, isolated quotes
- Right-align only where there is a specific visual reason (e.g., numerical columns in tables)

---

### 4.5 Line Spacing Too Tight or Too Loose

🟡 **MINOR**

**What it looks like:** Text with no breathing room between lines (leading of 1.0 or less), or text with so much space between lines it fragments into disconnected pieces (leading above 1.8).

**Measurable thresholds (Practical Typography):**
- Too tight: below 120% of font size (below 1.2 line-height multiplier)
- Optimal: **120%–145%** of font size (1.2–1.45 line-height)
- Too loose: above 170% (above 1.7 line-height)
- The common "double spacing" (233%) is far outside acceptable range
- Single spacing in word processors (117%) is technically too tight

**Detection checklist:**
- [ ] Does text feel cramped and hard to follow line-to-line?
- [ ] Does text feel disconnected and hard to read as continuous prose?
- [ ] Has the line height been modified from the template default? If yes, was it intentional?

---

### 4.6 Line Length Violations

🟡 **MINOR**

**Measurable thresholds:**
- Optimal line length: **45–90 characters per line** (Practical Typography)
- Web content optimal: **45–75 characters per line** (Smashing Magazine)
- Too short: below 45 characters — forces excessive line breaks, disrupts reading rhythm
- Too long: above 90 characters — eye struggles to track from end of one line to start of next
- Average website is 88.74 characters — close to the upper limit

**Why it matters for presentations:**
- Slides with full-width text columns force extremely long lines
- Long lines in presentations look like documents, not slides
- Narrow text columns with short lines improve readability and add visual breathing room

---

### 4.7 Tracking & Kerning Abuse

🟡 **MINOR**

**What it looks like:** Text with excessive letter-spacing (tracking so wide that individual characters float isolated), or text with compressed tracking so tight letters touch.

**Why it fails:**
- Wide tracking (loose letter-spacing) destroys word shape recognition — reading relies on recognizing word shapes, not individual letters
- Tight tracking creates letter collisions that reduce legibility
- ALL CAPS with standard tracking is already harder to read than mixed case — adding wide tracking compounds the problem

**Detection checklist:**
- [ ] Does the text look "airy" with large gaps between letters?
- [ ] Do any letters touch or overlap?
- [ ] Is tracking being used to fill or shrink text to fit a space (rather than for design intent)?

---

### 4.8 ALL CAPS Abuse

🟡 **MINOR**

**What it looks like:** Large sections of body text, bullet points, or multiple consecutive slides in ALL CAPITAL LETTERS.

**Why it fails:**
- Mixed-case text is read by recognizing word shapes — the ascending and descending letters create distinctive silhouettes
- ALL CAPS text loses these shape cues — every word is a uniform rectangle
- Research shows ALL CAPS text is **10–25% slower to read** than mixed case
- Extended ALL CAPS reads as SHOUTING — inappropriate for most contexts
- Legitimate uses: short acronyms, section labels, design emphasis on isolated words

**Detection checklist:**
- [ ] Is any body text in ALL CAPS?
- [ ] Are bullet points in ALL CAPS?
- [ ] Does more than 10% of slide content use ALL CAPS?

---

## 5. Color & Contrast Failures

### 5.1 Low Contrast Text (Illegible Text)

🔴 **CRITICAL**

**What it looks like:** Light gray text on white background. White text on light yellow. Dark navy on black. Teal on green. Any combination where the text blends into the background.

**WCAG contrast ratio requirements:**
- **Normal text (below 18pt or 14pt bold): minimum 4.5:1 contrast ratio** (WCAG AA)
- **Large text (18pt+ or 14pt+ bold): minimum 3:1 contrast ratio** (WCAG AA)
- **Enhanced level: 7:1** for normal text (WCAG AAA)
- **Graphics/UI components: minimum 3:1** against adjacent colors

**Common failures:**
- Light gray text on white: #999999 on white = ~2.8:1 (FAILS)
- Medium gray on white: #777777 on white = ~4.47:1 (FAILS AA — cannot round up to 4.5)
- Yellow on white: approximately 1.07:1 (catastrophic failure)
- Red on green: visually jarring AND fails for colorblind users
- White on yellow: approximately 1.07:1 (catastrophic failure)

**Note:** You cannot round a contrast ratio up to the threshold. 4.47:1 fails the 4.5:1 minimum.

**Detection checklist:**
- [ ] Check contrast with a tool like WebAIM Contrast Checker
- [ ] Test each text element against its actual background color
- [ ] Test text over images/gradients at the lightest point of the background
- [ ] Does any text appear to "disappear" or require squinting?

**How to fix:**
- Use dark (#1a1a1a) text on light backgrounds or white (#ffffff) text on dark backgrounds
- Check contrast ratios with free tools before publishing
- Avoid mid-tone combinations (medium gray, medium blue on medium backgrounds)
- For text over images: add a semi-transparent overlay to guarantee contrast

---

### 5.2 Rainbow Vomit (Too Many Colors)

🟠 **MAJOR**

**What it looks like:** A single slide or deck using 8+ different colors with no system or logic. Every bullet is a different color. Text uses color for decoration. Background gradients cycle through the spectrum. Charts use every color available.

**Why it fails:**
- Color should convey meaning — when everything is colorful, color signals nothing
- Multiple unrelated colors create visual competition with no hierarchy
- "When everything is emphasized, nothing is emphasized" (Practical Typography)
- Destroys brand coherence and professional appearance
- Increases cognitive load — brain tries to assign meaning to each color

**Measurable thresholds:**
- More than 3–4 colors on a single slide = Rainbow Vomit territory
- More than 5–6 colors in an entire presentation = systemic problem
- IxDF guidelines recommend: "Limit your color use to 2 primary and 2 secondary colors" for simple designs
- Column Five: "Limit to 6 colors maximum per layout" for infographics

**Common specific failures:**
- Coloring each bullet point differently (color = decoration, not meaning)
- Using neon/bright accent colors on every element
- Gradient backgrounds that compete with content
- Multicolored headers where color has no semantic purpose

**Detection checklist:**
- [ ] Count distinct colors on the most colorful slide — more than 5 is suspect
- [ ] Does each color mean something specific, or are colors applied randomly?
- [ ] Does the deck have an established, consistent color palette?
- [ ] Are neon, fluorescent, or highly saturated colors used extensively?

**How to fix:**
- Establish a palette: 1 primary brand color, 1–2 secondary colors, 1 accent for emphasis
- Use color purposefully: one color = one concept (e.g., red = danger/bad, green = good/safe)
- Neutrals (dark gray, white, light gray) should carry most of the visual load
- Reserve bright accent colors for the single most important element per slide

---

### 5.3 Neon on White / Fluorescent Color Abuse

🟠 **MAJOR**

**What it looks like:** Hot pink, electric lime, neon orange, or fluorescent yellow text or backgrounds. Colors so saturated they vibrate visually against white backgrounds.

**Why it fails:**
- Highly saturated colors cause **chromostereopsis** — a visual phenomenon where certain color combinations create an uncomfortable, vibrating optical effect
- The eye continuously tries to focus on two planes simultaneously, causing physical fatigue
- Neon on white combinations often have surprisingly poor contrast ratios (yellow on white = near-invisible)
- Extremely saturated backgrounds cause **chromatic adaptation** — after looking at a neon slide, everything else appears washed out

**Detection checklist:**
- [ ] Do any color combinations cause visual "vibration" or discomfort?
- [ ] Are any backgrounds more intensely colored than the content they contain?
- [ ] Is text on a fluorescent background actually legible?

---

### 5.4 Colorblind-Unfriendly Combinations

🟠 **MAJOR**

**Why it matters:** Approximately **300 million people worldwide** are colorblind. Red-green colorblindness (deuteranopia and protanopia) affects approximately 8% of men and 0.5% of women of Northern European descent. In a room of 50 people, 2–4 will likely have some form of color vision deficiency.

**Specific combinations to avoid:**
- **Red & Green**: The most common colorblind failure — deuteranopes cannot distinguish these
- **Green & Brown**: Appear identical to many colorblind users
- **Green & Blue**: Difficult for tritanopia (rare but exists)
- **Blue & Gray**: Can appear similar to some colorblind users
- **Blue & Purple**: Difficult to distinguish
- **Green & Gray**: Can appear similar
- **Green & Black**: Particularly problematic for deuteranopes

**The Red/Green danger in data visualization:**
Using red for "bad/decrease" and green for "good/increase" is the single most common colorblind failure in business presentations. These colors look **identical** to the 8% of male audience members with red-green colorblindness.

**Detection checklist:**
- [ ] Does your data visualization rely on red vs. green to encode meaning?
- [ ] Run your slides through a colorblind simulation tool (Coblis, Color Oracle)
- [ ] Can all information be understood if the slide is printed in grayscale?
- [ ] Is color the *only* differentiator, or are shape/pattern/label also used?

**How to fix:**
- Use blue/orange instead of red/green for contrast pairs
- Add shape encoding alongside color encoding
- Ensure color is never the *only* means of conveying information (WCAG 1.4.1)
- Test with colorblind simulation tools

---

### 5.5 Simultaneous Contrast Failures

🟡 **MINOR**

**What it looks like:** Colors that look different on different backgrounds even though they're the same hex value. A medium gray that looks dark on a white slide and light on a dark slide, causing inconsistent hierarchy across a deck.

**Why it fails:**
- Adjacent colors influence each other's perceived lightness and saturation (simultaneous contrast effect)
- Colors chosen in isolation look different when surrounded by other colors
- Chromatic luminance: higher saturation colors appear lighter than equally-valued lower-saturation colors

**Detection checklist:**
- [ ] Does the same color appear visually inconsistent across slides with different backgrounds?
- [ ] Are accent colors chosen by looking at them in context, not in isolation?

---

## 6. Layout & Composition Disasters

### 6.1 Visual Clutter / No Whitespace

🔴 **CRITICAL**

**What it looks like:** Slides packed with elements — text boxes, icons, images, charts, dividers, logos — with no breathing room. Every pixel is occupied. The slide has no visual rest points.

**Why it fails:**
- Whitespace is not "wasted space" — it is an active design element that creates focus
- Cluttered slides cause decision paralysis: users facing "a webpage that was so busy with various design elements...have no idea where to even begin to look" (IxDF)
- Without whitespace, the eye has no rest point and cannot determine what matters
- Audience attention is distributed uniformly across the slide instead of directed to key elements
- Research: scannable formatting produces a **47% usability improvement** vs. dense, unformatted content

**Measurable thresholds:**
- Less than 30% whitespace on a slide = clutter approaching
- Less than 20% whitespace = definite clutter problem
- More than 7 distinct visual elements per slide = cognitive overload
- IxDF visual hierarchy rules: "Limit how many elements are big to a maximum of 2"

**Detection checklist:**
- [ ] Step back from the slide — does it feel "full" or "crowded"?
- [ ] Is there a clear focal point, or do all elements compete equally for attention?
- [ ] Has every element been individually justified for inclusion?
- [ ] Are there elements present only because there was space to fill?

**How to fix:**
- Delete everything that doesn't directly support the one central message
- Increase margins; slides should rarely use edge-to-edge content
- Embrace the "one idea, one visual" principle
- If a slide feels empty, resist filling it — emptiness creates emphasis

---

### 6.2 Misalignment / Random Placement

🟠 **MAJOR**

**What it looks like:** Text boxes that are slightly off-center. Images that don't align to any grid. Elements that are "close enough" to aligned but not actually aligned. Ragged, inconsistent margins.

**Why it fails:**
- Misalignment signals carelessness — if the designer didn't notice this, what else did they miss?
- The human eye is extremely sensitive to small misalignments — they create visual tension
- Random placement destroys the invisible grid that organizes visual space
- IxDF identifies poor alignment as a primary visual hierarchy failure: "Scattered, misaligned blocks appear unprofessional and reduce scanability"

**Detection checklist:**
- [ ] Enable grid/snap-to-grid and verify all elements align to it
- [ ] Do elements that should share a left edge actually share it?
- [ ] Are margins consistent across all slides?
- [ ] Do images align with text edges, or float freely?

**How to fix:**
- Use a consistent grid: 8pt or 16pt grid for element placement
- Enable "snap to grid" in presentation software
- Use alignment tools to precisely align related elements
- Establish and follow consistent margins (e.g., 40px all sides)

---

### 6.3 Inconsistent Element Sizes (No Scale System)

🟠 **MAJOR**

**What it looks like:** Images of wildly different sizes on different slides. Text boxes that are slightly different sizes performing the same function. Elements that are "about the same size" but not actually identical.

**Measurable thresholds:**
- IxDF: "Use no more than 3 sizes — small, medium, and large" for scale variation
- "Use no more than 3 contrast variations for complex designs"
- Headers should have a consistent 3:1 ratio with body text for clarity

**Detection checklist:**
- [ ] Do all slide titles use exactly the same font size and position?
- [ ] Do all body text elements use consistent sizing?
- [ ] Are similarly-functioning elements (e.g., all call-out boxes) the same size?

---

### 6.4 The Floating Element

🟠 **MAJOR**

**What it looks like:** Elements placed seemingly at random with no relationship to other content. An image floated somewhere on the slide with no visual connection to the text it supposedly accompanies. Icons that hover near but don't clearly belong to any particular element.

**Why it fails:**
- Violates Gestalt Proximity principle: items that are close together appear grouped; items that are not clearly close to anything appear unrelated
- When an image is far from the text it illustrates, users don't connect them
- The NNGroup Kayak example: "The Skip link sits isolated in the top corner, far from main sign-in options, tricking users into mandatory account creation"

**Detection checklist:**
- [ ] For each element: what does it relate to? Is it visually adjacent to that element?
- [ ] Are images obviously paired with the text they support?
- [ ] Are icons clearly connected to their labels?

---

### 6.5 Equal Visual Weight on Everything (No Hierarchy)

🔴 **CRITICAL**

**What it looks like:** All text elements are the same size. All elements have the same visual weight. Nothing stands out. The title is the same size as the body text. The most important point looks identical to the least important.

**Why it fails:**
- "When all elements receive the same emphasis, nothing stands out — viewers experience visual noise" (IxDF)
- Audiences cannot determine what to pay attention to first
- The brain scans for hierarchy and when none is found, attention diffuses
- First impressions: users have only **50 milliseconds** to form a first impression — they need hierarchy to immediately grasp what matters

**Measurable thresholds:**
- Minimum 3:1 size ratio between titles and body text
- Maximum 2 elements "big" per slide
- Minimum 3 distinct levels of visual hierarchy per slide (primary, secondary, supporting)

**Detection checklist:**
- [ ] Cover the title — can you immediately tell what the most important element is?
- [ ] Is the title clearly bigger/bolder than body text?
- [ ] Does every text element on the slide feel the same "weight"?
- [ ] Is there one clear focal point on the slide, or several competing ones?

---

### 6.6 Clipart & Amateur Stock Photos

🟠 **MAJOR**

**What it looks like:** Microsoft Office clipart. Cartoon illustrations from the 1990s. Obvious Getty Images watermark still visible. Photos of people in suits shaking hands in front of a whiteboard. "Business people looking at graph" stock photos.

**Why it fails:**
- Generic imagery signals generic thinking — if you chose the first result for "teamwork," what was your thought process?
- Clipart is universally recognized as the aesthetic of amateur content
- Obvious stock photography creates cognitive dissonance: the content claims authenticity while the imagery screams "staged"
- Irrelevant images add visual noise without adding meaning — they are clutter in image form
- Kapterev's Death by PowerPoint slideshow specifically identifies clipart as a primary credibility killer

**Detection checklist:**
- [ ] Does any image look like it could be on a dentist's waiting room poster?
- [ ] Are there cartoon characters, simple clipart icons used as illustrations?
- [ ] Do any photos show people who obviously don't know each other "collaborating"?
- [ ] Does any image require a caption to explain what it has to do with the slide?

**How to fix:**
- Use high-quality photography from Unsplash, Pexels, Flickr Creative Commons
- Choose images that are evocative, specific, and conceptually connected to the content
- Use custom illustrations, diagrams, or real photographs when possible
- If an image doesn't add meaning, remove it

---

## 7. Visual Hierarchy Failures

### 7.1 The Flat Hierarchy

🔴 **CRITICAL**

**What it looks like:** Every text element is rendered in the same size, weight, and color. There is no obvious title, subtitle, or body hierarchy. The slide looks like a monochrome wall of uniform text.

**Why it fails:**
- Human visual processing is pre-attentive: the brain scans for size, contrast, and color differences before conscious attention begins
- Without hierarchy, the pre-attentive scan finds nothing and full attention is required to parse every element — enormously expensive cognitively
- The F-pattern eye-tracking research shows eyes fixate on larger, higher-contrast elements first — when everything is equal, the F-pattern cannot operate

**Visual hierarchy building blocks (IxDF, NNGroup):**
1. **Size** — larger = more important
2. **Color** — higher contrast / brighter = more important
3. **Contrast** — greater contrast from background = more important
4. **Alignment** — left-aligned vs. centered vs. indented signals hierarchy
5. **Repetition** — consistent treatment of same-level elements
6. **Proximity** — related elements grouped together
7. **Whitespace** — isolated elements receive more attention

**Detection checklist:**
- [ ] Can you scan the slide in 3 seconds and identify the main message?
- [ ] Is there a clear visual "path" from title → main content → supporting detail?
- [ ] Do titles draw the eye first?
- [ ] Is the most important information the most visually prominent?

---

### 7.2 Gestalt Principle Violations

🟠 **MAJOR**

Gestalt principles describe how humans perceptually organize visual information. Violating them creates confusion.

**Proximity violation:** Related elements placed far apart, unrelated elements placed close together.
- Example: A bullet list where unrelated items are visually adjacent, and related items are separated by unrelated content
- Impact: Users incorrectly group elements, misunderstand relationships

**Similarity violation:** Elements that should appear related don't share visual characteristics; unrelated elements share visual treatment.
- Example: All text the same color, including titles, body, captions, and citations — users cannot distinguish them
- Impact: Users treat all elements as equal-weight, miss hierarchy

**Figure-ground violation:** Foreground elements don't stand out from the background.
- Example: Text color too close to background color
- Impact: Critical content disappears into the background

**Continuity violation:** Natural visual flow is interrupted by elements that don't align with the implied directional path.
- Example: Scattered elements with no alignment creating a chaotic visual path

**Detection checklist:**
- [ ] Do related elements share at least one visual characteristic (color, size, style)?
- [ ] Are related elements in visual proximity to each other?
- [ ] Does content clearly stand out from its background?
- [ ] Is there a natural visual flow across the slide?

---

### 7.3 Incorrect Use of Emphasis (Everything Emphasized)

🟠 **MAJOR**

**What it looks like:** Multiple colors used for emphasis, with 40% of text bolded, 30% in different colors, and underlining on top of that.

**The core principle (Practical Typography):** "When everything is emphasized, nothing is emphasized."

**Why it fails:**
- Emphasis is a comparative signal — it only works relative to non-emphasized content
- If 50% of text is bold, bold no longer means important
- Multiple emphasis techniques competing simultaneously create visual noise, not hierarchy
- Common failure: every other bullet is a different color "for variety"

**Detection checklist:**
- [ ] What percentage of text is bold? Above 20–25% defeats the purpose of bold
- [ ] Are more than 2 emphasis techniques used on a single slide?
- [ ] Is emphasis used for decoration ("variety") rather than semantic meaning?
- [ ] Does everything feel equally important (because nothing is differentiated)?

**How to fix:**
- Use bold for maximum 10–15% of text — the truly critical words
- Choose one emphasis technique per slide: bold OR color OR size, not all three
- If you feel the urge to emphasize everything, the solution is to cut content until only the important remains

---

## 8. Data Visualization Failures

### 8.1 The 3D Pie Chart Disaster

🔴 **CRITICAL**

**The 3D pie chart is so universally bad that Storytelling with Data's tagline is "Helping rid the world of ineffective graphs, one 3D pie at a time."**

**Why pie charts fail:**
- "Our eyes aren't good at attributing quantitative value to two-dimensional spaces" (Cole Nussbaumer Knaflic)
- Humans compare angles and arc lengths poorly — we cannot accurately judge relative proportions from a circular shape
- When slices are close in size, it is "difficult (if not impossible) to tell which is bigger"
- Even when sizes differ, viewers can only determine rank, not magnitude of difference
- A darker-colored slice appears larger than a same-size lighter slice — color distorts perception of size

**Why 3D makes it catastrophically worse:**
- The 3D perspective applies a foreshortening effect: slices at the front of the pie appear larger than slices at the back, even when they are identical in value
- This is pure perceptual distortion — the chart is actively lying about the data
- The 3D effect adds no information while actively adding misinformation

**Measurable threshold:** More than **6 slices** in a pie chart = impossible to compare

**The fix:** Use a horizontal bar chart. Bars aligned to a common baseline allow viewers to compare not only which is largest, but exactly how much larger it is. The "parts of a whole" story can be told with bar charts using stacking or 100% stacked bar formats.

**Detection checklist:**
- [ ] Are any pie charts 3D?
- [ ] Does any pie chart have more than 5–6 slices?
- [ ] Do any slices require labels to communicate their value (if so, the chart failed)?
- [ ] Are two slices very close in size?

---

### 8.2 Chartjunk (Tufte's Framework)

🔴 **CRITICAL**

Edward Tufte coined "chartjunk" to describe visual elements in charts that add complexity without adding information. Every element that does not serve the data increases cognitive load while reducing data-ink ratio.

**Types of chartjunk:**

**1. Unnecessary gridlines**
Dense grid lines that fill the entire chart area, creating visual noise behind the data.
- Fix: Use only the minimum gridlines needed for reference, or eliminate entirely

**2. Decorative backgrounds**
Colored or textured backgrounds applied to chart areas for "visual interest."
- Fix: White or transparent backgrounds only

**3. Unnecessary borders and boxes**
Boxes drawn around charts, legend boxes with thick borders, frames inside frames.
- Fix: Remove all borders that don't encode information

**4. Data-free illustrations**
3D effects on bars, gradient fills, shadow effects on pie slices — decorative treatments that add file size and complexity but no data value.
- Fix: Flat, solid colors only

**5. Excessive data labels**
Every data point labeled with its exact value, even on a line chart with 50 points.
- Fix: Label only significant points (start, end, peak, threshold)

**6. Chart titles that state the obvious**
"Bar chart of quarterly revenue by region" — the chart type is visible; the title should state the insight.
- Fix: Use active takeaway titles: "Q3 Revenue Grew 23% Despite Regional Headwinds"

**7. Drop shadows**
Shadow effects on chart elements (bars, labels, legend boxes).
- Fix: Remove entirely

**Data-ink ratio principle (Tufte):**
Every element of ink in a chart should encode data. Elements that encode no data should be removed. The ratio of "data ink" to "total ink" should be maximized.

**Detection checklist:**
- [ ] Cover each non-data element (gridlines, borders, shadows) — is anything lost?
- [ ] Does the background color of the chart area add information?
- [ ] Are there 3D effects on flat data?
- [ ] Does the chart have decorative elements not found in serious publications?

---

### 8.3 Misleading Axes

🔴 **CRITICAL**

**Truncated Y-axis:**
- A bar chart where the Y-axis starts at 94 instead of 0 makes a 3% difference look enormous
- This visually exaggerates the magnitude of differences
- Legitimate use: only when zero is meaningless to the context AND this is explicitly noted

**Logarithmic scale misuse:**
- Log scales stretch small values and compress large values
- Applied to percentage data (0–100%), the log scale fundamentally distorts the story
- Legitimate use: only when data spans multiple orders of magnitude or grows exponentially
- "If your data does not have a logarithmic structure, avoid using a log scale" (Storytelling with Data)

**Inconsistent axis intervals:**
- Axis increments like 0, 3, 5, 16, 50 rather than 0, 5, 10, 15, 20
- Makes the chart physically impossible to use for accurate reading

**Dual Y-axes:**
- Two Y-axes with different scales on the same chart can imply any correlation between two unrelated variables
- "You can make almost any two variables appear correlated using dual axes" — this is a manipulation tool that appears analytical

**Detection checklist:**
- [ ] Does the Y-axis start at zero? If not, is the truncation clearly labeled?
- [ ] Are axis increments consistent and evenly spaced?
- [ ] Is a log scale used? Is this appropriate for the data?
- [ ] Does the chart use dual Y-axes?

---

### 8.4 Wrong Chart Type for the Data

🟠 **MAJOR**

**Common wrong chart type choices:**

- **Line chart for categorical data** — Line charts imply continuous change between data points; using them for discrete categories implies a trend that doesn't exist
- **Pie chart for more than 5 items** — Impossible to compare; use a bar chart
- **Stacked bar for comparing changes within groups** — The upper sections don't share a common baseline, making comparison nearly impossible
- **Bubble chart without clear size legend** — Audiences cannot estimate bubble sizes without explicit scale
- **Radar chart for most things** — Radar charts are nearly impossible to read accurately; bar charts almost always work better

**The Tableau rule:** More than 3–4 visualizations in a single view means the big picture is lost.

**Detection checklist:**
- [ ] Does the chart type match the question being answered?
- [ ] For comparisons: is there a common baseline?
- [ ] For time series: is chronological order maintained on the X-axis?
- [ ] For proportions: is the total clearly defined?

---

### 8.5 Color Encoding Failures in Charts

🟠 **MAJOR**

**Rainbow color schemes on sequential data:**
- Using red-orange-yellow-green-blue to encode a gradient (like temperature or intensity) is problematic because the rainbow has no perceptual order — the human eye does not see these as a sequence
- Use perceptually ordered single-hue progressions (light to dark) for sequential data

**Inverted color conventions:**
- Using red for positive values and green for negative reverses all established conventions
- In financial contexts particularly, audiences are hardwired: red = bad, green = good
- Reversing this causes systematic misreading

**Too many categories → too many colors:**
- More than 6–7 categories with distinct colors creates a legend too complex to use
- When a legend requires more than a few seconds to decode, the chart has failed
- Column Five: "Limit to 6 colors maximum per layout"

**Detection checklist:**
- [ ] Does any chart use more than 6–7 distinct colors?
- [ ] Are color conventions consistent with audience expectations (red/green = bad/good)?
- [ ] Is color the only way to distinguish data series, or are patterns/shapes also used?

---

## 9. Image & Media Abuse

### 9.1 Low-Resolution Images

🔴 **CRITICAL**

**What it looks like:** Blurry, pixelated photographs. Images that look fine on screen but become obvious low-resolution artifacts when projected. Screenshots of web pages taken at 72dpi that look fine in Slack but terrible at 120 inches.

**Why it fails:**
- Low-resolution imagery instantly signals low professional standards
- Pixelated images undermine credibility for technical content
- Screen projection requires 96–150 DPI; print requires 300 DPI minimum

**Detection checklist:**
- [ ] View each image at 100% zoom — is it sharp?
- [ ] Project the presentation if possible and check from the back of the room
- [ ] Are any images sourced from thumbnail previews or small web copies?

**How to fix:**
- Minimum 1920×1080 pixels for full-slide images
- For inset images: minimum 800px on the longer edge
- Source from high-resolution stock libraries (Unsplash, Pexels provide 2000–6000px images free)

---

### 9.2 Irrelevant or Generic "Concept" Images

🟠 **MAJOR**

**What it looks like:** A slide about "innovation" with a photo of a light bulb. A slide about "collaboration" with three people around a laptop. A slide about "growth" with a graph arrow and a plant. A slide about "technology" with blue circuit board patterns.

**Why it fails:**
- These images communicate nothing specific — they are visual filler
- They create the impression of visual richness without adding information
- Audiences see through the visual shorthand: it signals that no real thought went into the visual language
- Adding images for the sake of having images is worse than no images (adds distraction without adding value)
- TED: "Photos that are compositionally complex or lack obvious relevance to spoken content distract rather than reinforce meaning"

**Detection checklist:**
- [ ] For each image: can you articulate specifically how it reinforces the slide's message?
- [ ] Could this image appear on a different slide equally appropriately?
- [ ] Is the image the first result for a generic keyword search?
- [ ] Would removing the image make the slide cleaner and clearer?

---

### 9.3 Stretched or Distorted Images

🟠 **MAJOR**

**What it looks like:** Photographs where people appear slightly too wide or too tall. Logos that look wrong because they've been stretched to fill a space. Icons that are elongated horizontally.

**Why it fails:**
- Even subtle distortion (5–10%) is immediately noticeable to human visual perception
- Distorted images signal carelessness
- Stretched logos create legal/brand compliance issues
- Distorted human proportions are particularly jarring (uncanny valley for images)

**Detection checklist:**
- [ ] Are all images proportionally correct?
- [ ] Hold a ruler to the screen — do circles look circular?
- [ ] Are logos at their natural aspect ratio?

**How to fix:**
- Always scale images with the Shift key held (constrain proportions)
- Use "fit within" rather than "stretch to fill" when placing images in frames
- Crop images rather than distorting them to fit a space

---

### 9.4 Autoplay Video & Uncontrolled Media

🟡 **MINOR**

**What it looks like:** Videos that start playing automatically when the slide appears. Videos that run longer than 60 seconds mid-presentation without a clear purpose.

**Why it fails:**
- TED: "Using autoplay causes timing unpredictability and risks presenters accidentally skipping slides during startup delays"
- Videos that autoplay with audio can startle the audience and break the presenter's rhythm
- Long embedded videos shift control from presenter to playback — the audience watches a screen, not a presenter

**Measurable threshold:** Videos should be under 60 seconds unless they are the entire point of the slide.

---

## 10. Structural & Narrative Failures

### 10.1 No Central Message (No North Star)

🔴 **CRITICAL**

**What it looks like:** A presentation where no single clear message can be extracted. After watching the entire presentation, an audience member cannot answer "What was the main point?"

**Why it fails:**
- Without a central message, every slide is equally important — and therefore equally forgettable
- The brain retains information better when it can be anchored to a central thesis
- TED: "Reversed Development Order — building your slides before structuring your content is backwards"
- Kapterev: "Lack of significance — no clear purpose or meaningful message" is the first root cause of Death by PowerPoint

**Detection checklist:**
- [ ] Write the presentation's main message in one sentence. Can you?
- [ ] Does every slide connect directly to that central message?
- [ ] Could a 30-second summary capture the entire point?

**How to fix:**
- Start with the "So what?" — the single thing you want the audience to do or believe differently
- Build backward from that message: what evidence and context does the audience need to reach that conclusion?
- Kill any slide that doesn't directly serve the central message

---

### 10.2 No Narrative Arc

🟠 **MAJOR**

**What it looks like:** A flat sequence of information delivery. Topic 1, Topic 2, Topic 3... with no connective tissue, no escalating tension, no resolution. The presentation could be shuffled into a different order with no loss of meaning.

**Why it fails:**
- The human brain is wired for narrative — we remember stories far more effectively than lists of facts
- Serial position effect: audiences remember the **beginning (primacy)** and **end (recency)** of presentations much better than the middle
- Content in the middle of a presentation with no narrative drive is particularly vulnerable to forgetting
- Without a story arc (problem → tension → resolution), audiences feel no urgency to pay attention

**Classic structures that work:**
- Problem → Impact → Solution → Call to Action
- Before State → After State → Bridge
- The Hero's Journey (applied to product narratives, customer stories)
- SCQA: Situation → Complication → Question → Answer

**Detection checklist:**
- [ ] What is the opening hook — why should the audience care immediately?
- [ ] Is there an escalating tension or problem that builds through the middle?
- [ ] Is there a satisfying resolution or clear call to action at the end?
- [ ] If you reordered three slides randomly, would anyone notice?

---

### 10.3 Weak or Missing Opening

🔴 **CRITICAL**

**What it looks like:** Beginning a presentation with "Hi, my name is... and today I'll be talking about..." followed by an agenda slide. Or starting with the company's founding story and organizational structure before the audience has any reason to care.

**Why it fails:**
- Audiences decide within the first 30–60 seconds whether to mentally invest in a presentation
- An uninspiring opening is a missed opportunity to establish credibility and create engagement
- TED lists "lengthy explanations of talk topic" as one of the Top 10 Ways to Ruin a Presentation
- Agenda slides before the audience has context or investment communicates: "I'm going to tell you what I'll tell you, then tell you, then tell you what I told you" — treating the audience as passive recipients

**Detection checklist:**
- [ ] Does the opening immediately establish why the audience should care?
- [ ] Is the first slide something other than a title/agenda/introduction?
- [ ] Does the opening make a provocative statement, pose a question, or show a surprising image?

---

### 10.4 No Clear Call to Action

🟠 **MAJOR**

**What it looks like:** A presentation that ends with a "Thank You" slide and no direction for the audience. Or a conclusion slide that summarizes what was just said without specifying what the audience should do next.

**Why it fails:**
- Information without direction creates no change in behavior or belief
- The serial position effect means the final moments are among the most remembered — wasting this on "Thanks for listening" is a strategic failure
- TED identifies ending slides as a critical element that must close with impact

**Detection checklist:**
- [ ] Is the last slide a "Thank You" with no other content?
- [ ] Does the conclusion tell the audience specifically what to do, think, or decide?
- [ ] Is there a clear next step for interested audience members?

---

### 10.5 Too Many Slides / Wrong Pacing

🟠 **MAJOR**

**What it looks like:** A 20-minute presentation with 80 slides. Racing through slides at 15 seconds each. Or the opposite: 5 slides for a 30-minute presentation with walls of text on each.

**Why it fails:**
- Rapid slide changes prevent the audience from forming any mental model of each slide's content
- Too few content-dense slides force the audience into "reading mode" throughout
- Pacing that doesn't match the complexity of content signals poor planning

**General guidelines:**
- Conference talks: 1–2 slides per minute is a common guideline
- Complex technical content: 1 slide per 3–5 minutes
- High-level executive presentations: 1 slide per minute maximum
- TED Talks: average 1 image/slide per minute for 18-minute talks
- Video clips: under 60 seconds each; trigger them manually, not on autoplay

**Detection checklist:**
- [ ] How many seconds per slide on average? Under 30 seconds = too fast
- [ ] Are complex slides given enough time for processing?
- [ ] Does the slide count match the complexity of the content?

---

### 10.6 Slide Titles as Labels, Not Messages

🟠 **MAJOR**

**What it looks like:** Slide titles that say "Q3 Results" or "Market Analysis" or "Recommendations" — functional labels that name a category without communicating a finding.

**Why it fails (Storytelling with Data):**
- Question-format titles ("Are our campaigns effective?") create divided attention: audiences simultaneously try to answer the question, interpret the chart, and listen to the presenter
- Label titles waste the most prominent visual real estate on the slide
- Active takeaway titles state the finding explicitly: audiences immediately understand the conclusion and can evaluate the supporting evidence

**The principle:** Every slide title should be the **answer**, not the question. It should be the **finding**, not the category.

**Bad → Good examples:**
- "Revenue Analysis" → "Revenue Grew 23% Despite Market Headwinds"
- "Customer Feedback" → "NPS Score Reached All-Time High of 72 in Q3"
- "Recommendations" → "Three Changes Will Reduce Churn by 18%"
- "Q3 Results" → "Q3 Exceeded All Targets — Best Quarter in 5 Years"

**Detection checklist:**
- [ ] Does each title state the point of the slide, or just name a topic?
- [ ] Could any title appear on a different slide in the deck?
- [ ] Are any titles phrased as questions?

---

### 10.7 No Structure / Random Information Sequence

🟠 **MAJOR**

**What it looks like:** Information presented in the order it was gathered, not in the order most useful for the audience. Or information presented in the structure of the internal organization rather than the structure of the audience's thinking.

**Why it fails:**
- The organization of information is itself communication — the sequence implies causal or hierarchical relationships
- When audiences can't find a pattern, they assume randomness, not just non-linearity
- Kapterev: "Poor structure — disorganized content flow" is the second root cause of Death by PowerPoint

**The SCQA framework (Minto Pyramid Principle):**
- **Situation:** Establish common ground — what do we all know?
- **Complication:** What has changed or what problem exists?
- **Question:** What does this make the audience wonder?
- **Answer:** The answer IS the presentation's central message

---

## 11. Cognitive Overload Patterns

### 11.1 Working Memory Overflow

🔴 **CRITICAL**

**The research foundation:**
- George Miller's 1956 research: "The Magical Number Seven, Plus or Minus Two" established that most people can hold approximately 7 items (±2) in working memory at once
- Nelson Cowan's 2001 research refined this: the realistic limit for chunks of *new* information is approximately **3–5 items**
- The key word: "chunks" — a pre-existing category ("months of the year") counts as one chunk even though it contains 12 items

**How presentations violate this:**
- 12-item bullet lists: requires holding 12 items in working memory simultaneously
- Complex diagrams with 20 labeled elements
- Charts with 15 data series
- Multiple competing animations
- Text that must be read while listening to a different spoken message

**Nielsen Norman Group definition:** Cognitive overload occurs when "information exceeds our capacity to process it," leading to "longer time to understand information, missing important details, or getting overwhelmed and abandoning the task."

**Measurable thresholds:**
- More than 7 items in a list (without chunking) = likely overflow for many audience members
- More than 3–5 distinct pieces of new information per slide = cognitive overload risk
- Any combination of reading + listening + watching (triple-channel input) = guaranteed reduced retention

**Detection checklist:**
- [ ] Count distinct new pieces of information on each slide — more than 5 is risky
- [ ] Is the audience expected to read text while listening to different spoken content?
- [ ] Are any lists or diagrams longer than 7 items without grouping/chunking?

**How to fix:**
- Chunk information: group related items under labeled headers
- Split overcrowded slides across multiple slides
- Follow Mayer's segmenting principle: break content into learner-paced segments
- One idea per slide

---

### 11.2 The Redundancy Paradox

🟠 **MAJOR**

**The principle (Mayer's Redundancy Principle from Multimedia Learning Theory):**
When the same information is presented simultaneously in two modes (text on screen + identical spoken words), retention is *worse* than if it were presented in only one mode.

**Why this counterintuitive finding matters:**
- Presenters believe that repeating their message in multiple channels reinforces learning
- In reality, the dual-mode presentation forces the brain to process the same information twice through different channels — this uses more cognitive resources without adding new information
- The resources spent processing the redundant second channel come at the expense of deeper processing

**What works instead (Mayer's Coherence Principle):**
- Spoken words explain what images/charts show
- Screen text is *different from and complementary to* spoken words
- Images illustrate what cannot be said in words

**Detection checklist:**
- [ ] Are you planning to read your slides to the audience?
- [ ] Does the text on each slide contain what you're planning to say?
- [ ] If yes, either the slides need to change or the script needs to change

---

### 11.3 Split Attention Effect

🟠 **MAJOR**

**What it is (Sweller's Cognitive Load Theory):**
The split-attention effect occurs when learners must mentally integrate two or more sources of information that are physically separated — the effort of searching and integrating creates extraneous cognitive load.

**How it appears in presentations:**
- A legend far from the chart elements it describes
- Caption text physically separated from the image it describes
- Numbers in a table that must be mentally matched to a key shown on the other side of the slide
- Footnotes that modify content shown at the top of the slide

**Detection checklist:**
- [ ] Does any element require the audience to visually search for its label/key?
- [ ] Are legends adjacent to the data they describe, or separated?
- [ ] Do captions appear directly under or beside their images?

---

### 11.4 Extraneous Load from Decoration

🟠 **MAJOR**

**Intrinsic vs. extraneous cognitive load (Sweller):**
- **Intrinsic load**: the inherent complexity of the subject matter — unavoidable
- **Extraneous load**: complexity created by poor design choices — entirely avoidable and wasteful

**Sources of extraneous load in presentations:**
- Decorative animations that play before content appears
- Complex background patterns that compete with foreground content
- Irrelevant images that distract attention from the message
- Multiple simultaneous animation effects
- Background music or ambient sound during content slides
- Busy template borders and frames that add visual noise

**The Nielsen Norman Group principle:** Extraneous load is "mental processing that consumes resources without aiding comprehension." Every decoration that creates cognitive work without yielding information is parasitic — it consumes resources the audience needs for the actual content.

---

## 12. Inconsistency & Template Abuse

### 12.1 Inconsistent Design Language

🔴 **CRITICAL**

**What it looks like:** Slide 1 uses blue headings, slides 2–5 use green headings, slide 6 uses a different font entirely, slide 7 has a different background color. Each slide feels like it was made by a different person at a different time.

**Why it fails:**
- TED: "Failing to establish unified typography, color schemes, and imagery across slides makes decks feel disjointed rather than cohesive"
- Inconsistency signals carelessness and damages credibility
- IxDF: "Inconsistency: unpredictable font, color, or layout changes erode credibility and confuse viewers about what deserves attention"
- Each visual inconsistency forces audiences to check their understanding — "is this a different type of information, or just a formatting mistake?"

**Measurable indicators:**
- More than 2 headline font sizes used across slides (where no hierarchy logic exists)
- Multiple different accent colors with no system
- Inconsistent logo placement (left corner on some slides, right corner on others)
- Slide numbers in different positions
- Inconsistent bullet styles across slides

**Detection checklist:**
- [ ] Do all slide titles use exactly the same typography?
- [ ] Does the same type of content (body text, captions, quotes) look the same on every slide?
- [ ] Is the color palette consistent throughout?
- [ ] Do the margins and grid alignment match across slides?

---

### 12.2 Default Template Abuse

🟠 **MAJOR**

**What it looks like:** Slides clearly using the default Microsoft Office "Office Theme" or Google Slides "Simple Light" template with no customization. The coral/tan/navy color scheme that signals "no design thought went into this."

**Why it fails:**
- The default template is used by hundreds of millions of people — it immediately signals that no thought was given to visual identity
- Default templates are designed to be neutral/inoffensive, which means they are also optimized to be forgettable
- The default slide layout (title left, bullet list right; coral/tan/blue color palette) is so universally associated with low-effort presentations that it creates negative first impressions
- TED: "Using pre-built master slides if they feel restrictive and generic" undermines deck quality

**Detection checklist:**
- [ ] Would a design professional recognize your template immediately as a stock default?
- [ ] Does the deck have any visual identity specific to the presenter or topic?
- [ ] Are the colors and fonts the exact defaults of PowerPoint, Keynote, or Google Slides?

**How to fix:**
- Define a custom color palette with 3–4 colors
- Replace default fonts with a chosen 2-font system
- Create consistent custom slide layouts for each content type
- Even minimal customization (changing the primary color + font) dramatically differentiates from defaults

---

### 12.3 Over-Animated Presentations

🟠 **MAJOR**

**What it looks like:** Every element flies in from a different direction. Slide transitions use 3D cube flips, page peels, or blinds. Text spins onto the screen. Objects bounce and pulse.

**Why it fails:**
- TED: "Most of these effects don't do much to enhance the audience experience. At worst, they subtly suggest that the content of your slides is so uninteresting that a page flip or droplet transition will snap the audience out of their lethargy"
- Animations create extraneous cognitive load — the brain processes motion involuntarily, consuming attention resources needed for content
- Complex animations make file sizes large, increase the chance of technical failures, and look unprofessional in many contexts
- The Attention Awareness Trap: sudden motion in peripheral vision is processed pre-attentively and draws all attention away from the presenter

**Acceptable animation use:**
- Progressive disclosure: reveal bullet points one at a time to focus discussion
- Simple fade-in transitions between slides
- Animated diagrams that show a process building step by step
- Data visualization builds that reveal data series progressively

**Unacceptable animation use:**
- Flying text, spinning logos, bouncing elements
- Complex entrance/exit animations on decorative elements
- Cube, page peel, or zoom slide transitions
- Sound effects on animations

**Detection checklist:**
- [ ] Do any animations serve a pedagogical purpose, or are they purely decorative?
- [ ] Do any elements fly in from off-screen?
- [ ] Are slide transitions different from a simple fade or cut?
- [ ] Do any slides have sound effects?

---

### 12.4 Missing or Inconsistent Branding

🟡 **MINOR**

**What it looks like:** No logo on the slides. Or the logo appearing on some slides but not others. Or the wrong version of the logo (outdated, stretched, wrong color).

**Why it matters:**
- Professional presentations, especially for organizations, establish ownership and identity
- Inconsistent branding signals that the presentation was assembled from multiple sources without editorial control
- Missing branding means the presentation cannot be attributed when it circulates independently

---

## 13. AI-Generated Presentation Failures

### 13.1 The Generic AI Aesthetic

🟠 **MAJOR**

**What it looks like:** A deck that could have been about any topic for any company. Abstract "swoosh" backgrounds in teal and purple. Stock images of diverse teams collaborating. Gradient headers. Rounded cards with thin drop shadows. Icon rows with three equal-weight items. No visual personality or specificity.

**Why it happens:**
- AI presentation tools (Gamma, Tome, Beautiful.ai, Canva AI, etc.) are trained on millions of existing presentations
- They learn the statistical average of "professional-looking presentations" — which produces the least distinctive, most forgettable aesthetic
- These tools optimize for "not offensive" rather than "memorable" or "specific to this content"
- The result is **visual mean reversion**: everything regresses toward the same neutral, derivative aesthetic

**Named anti-patterns of AI-generated slides:**

**The Three-Icon Row:** Three icons in a horizontal row with equal spacing, each with a short label. Every AI presentation tool generates this layout for almost any topic. It has become the visual shorthand for "generated without thought."

**The Gradient Header with Abstract Wave:** A colored gradient banner across the top with an SVG wave or blob shape. Found on approximately 60–70% of AI-generated presentations regardless of content.

**The Quote Box:** A large pull-quote in a colored box with quotation marks rendered as large decorative characters. Used whether the quote warrants emphasis or not.

**The "Diverse Team" Stock Photo:** A photo of 3–6 people of different ethnicities looking at a laptop or whiteboard, smiling. Found on virtually all AI-generated slides about teamwork, collaboration, or culture.

**The Same Three Colors:** Teal (#2ec4b6 or similar), purple (#7b2d8b or similar), and orange (#ff6b35 or similar) — the statistical average of "modern and colorful" from training data.

**Detection checklist:**
- [ ] Could this deck have been made by an AI tool in under 5 minutes?
- [ ] Does the visual style feel specific to this content, or interchangeable with any other topic?
- [ ] Are there any design decisions that could only have been made by someone who knows this specific content?
- [ ] Is the layout a direct application of a template rather than a composition designed for this data?

---

### 13.2 The Bullet Point AI Default

🔴 **CRITICAL**

**What it looks like:** Every slide contains 4–6 bullet points of 8–12 words each. The content was summarized by an AI that was instructed to "create a presentation," and it did what language models do: it outputted text in the format it learned was called a presentation — which is bullet lists.

**Why it fails:**
- AI language models excel at generating hierarchical text outlines; they generate visual design only as a byproduct
- The result is that AI presentations are essentially structured text documents dressed up as slides
- The bullet points accurately summarize the input material but contain none of the visual encoding, emphasis, and composition that makes slides effective
- This is the opposite of good slide design — it is the Presenter Notes Slide anti-pattern at scale

**Detection checklist:**
- [ ] Were the slides generated primarily as text by an AI and then formatted?
- [ ] Does every slide consist primarily of bullet points?
- [ ] Is the content distribution uniform across slides (same word count, same bullet count)?

---

### 13.3 Statistical Visual Predictability

🟠 **MAJOR**

**What it looks like:** The same layout templates repeated throughout the deck. "Two columns with title" alternates with "Full image with text overlay" alternates with "Icon row with three items" — and this pattern repeats. There is no visual variation that responds to content requirements; the templates drive the content.

**Why it fails:**
- In good design, content determines layout; in template-driven design, layout constrains content
- When audiences recognize the template pattern, they stop seeing the content and start seeing the container
- Predictable visual rhythm is soporific — the visual pattern becomes invisible background noise

**Detection checklist:**
- [ ] Are the same 3–4 layout templates cycling through the deck?
- [ ] Does any slide have a layout that was specifically designed for the data it contains?
- [ ] If you replaced all the text with "Lorem ipsum" and all images with gray boxes, would the visual logic still make sense?

---

### 13.4 Fabricated Specificity (AI Content Hallucination Risk)

🔴 **CRITICAL**

**What it looks like:** Statistics that appear credible but have no attribution. Percentages stated with false precision ("73% of customers report..."). Named competitors or products described with incorrect details. Quotes attributed to real people who didn't say them.

**Why it fails:**
- AI language models can hallucinate specific-sounding facts that appear in the format of real statistics
- These fabricated specifics can be technically plausible and even briefly convincing
- When an audience member (or a journalist, or a regulator) checks the statistic and finds it doesn't exist, credibility collapses completely
- Professional presentations demand sourced, verifiable claims

**Detection checklist:**
- [ ] Is every statistic cited with a specific source (author, organization, year)?
- [ ] Were statistics checked against primary sources, not just summarized by AI?
- [ ] Are any quotes verified as authentically attributed?
- [ ] Does any claim feel "too neat" or suspiciously precise?

---

## 14. Accessibility Failures

### 14.1 Insufficient Color Contrast (Full WCAG Reference)

🔴 **CRITICAL**

See also section 5.1. This section provides the complete technical reference.

**WCAG 2.0/2.1 Contrast Requirements:**

| Context | WCAG Level | Contrast Ratio |
|---------|------------|----------------|
| Normal text, body copy | AA | 4.5:1 minimum |
| Large text (18pt+ or 14pt+ bold) | AA | 3:1 minimum |
| Normal text | AAA | 7:1 |
| Large text | AAA | 4.5:1 |
| UI components, graphics | AA | 3:1 |

**Specific failure examples:**
- #777777 gray on white: 4.47:1 — FAILS AA (cannot round up)
- #999999 on white: ~2.8:1 — FAILS
- Light yellow text on white: ~1.07:1 — catastrophic failure
- White on light gray: ~1.5:1 — fails all levels
- Slide themes using medium-gray text on white (very common default): typically fails

**Important:** Contrast ratios apply to ALL states — including:
- Disabled elements (commonly 38% opacity, which often drops below 3:1)
- Text in interactive states (hover, focus)
- Text within images (must meet ratio against actual image background color)
- Placeholder text in forms on slides

---

### 14.2 Color as the Only Differentiator

🔴 **CRITICAL**

**WCAG 1.4.1 principle:** Color must not be used as the only visual means of conveying information.

**Common violations:**
- A line chart where the only difference between data series is color (no labels, no patterns)
- A table where rows alternate between white and a very light color and this alternation is the only structure
- Status indicators using only red/green dots without labels

**Why it matters beyond WCAG:**
- 8% of men and 0.5% of women have red-green colorblindness
- Color alone fails when printed in grayscale
- Color alone fails when displayed on poor-quality projectors

---

### 14.3 No Alt Text for Images

🟡 **MINOR** (for in-person presentations) / 🟠 **MAJOR** (for distributed PDFs/decks)

Images without alt text cannot be interpreted by screen readers. For presentations that will be shared as PDF documents or hosted online, this is a significant accessibility barrier.

---

## 15. Delivery-Design Disconnect

### 15.1 Reading the Slides to the Audience

🔴 **CRITICAL**

**What it looks like:** The presenter stands with back partially to the audience, reading text off the screen. Or the presenter looks down at notes to recite what is also on the screen.

**Why it fails:**
- TED: "Reciting slides...violates the fundamental rule that 'information is interesting only once.'"
- Hearing and reading the same content simultaneously creates the redundancy effect — worse retention than either mode alone
- The moment audiences sense the presenter is reading, "your intimate connection evaporates, and everything feels a lot more formal"
- Reading signals either lack of preparation or lack of confidence in the material

**Detection checklist:**
- [ ] Does the presenter need to look at the screen to know what to say next?
- [ ] Is the text on screen a script or a visual aid?
- [ ] Would the presentation work if the projector failed?

---

### 15.2 Mismatched Spoken and Visual Content

🟠 **MAJOR**

**What it looks like:** The presenter is talking about Q3 but the slide shows an annual comparison chart. The presenter says "as you can see" while the audience is still trying to figure out what the slide means. The spoken content has moved on while the audience is still processing the previous point.

**Why it fails:**
- Audiences devote attention to either the slide or the presenter — when these compete, both suffer
- The split-attention effect is compounded when spoken and visual content are on different subjects

**Detection checklist:**
- [ ] Does each slide exactly match what will be spoken on it?
- [ ] Does the presenter wait for audiences to read a new complex slide before speaking?

---

### 15.3 No Rehearsal (Timing Failures)

🟠 **MAJOR**

**TED's specific guidance:**
- "Talks should reach final form at least one month before presentation"
- Inadequate rehearsal and timing checks are one of TED's Top 10 Ways to Ruin a Presentation
- "Avoid trying to summarize entire careers in single presentations"

**Consequences of unrehearsed presentations:**
- Timing runs over → presenter races through final slides → audience sees rushed, incomplete content
- Technical transitions that haven't been rehearsed fail in delivery
- Unclear pacing — lingering too long on early slides, rushing at the end

---

## 16. The Master Diagnostic Checklist

Use this checklist to audit any presentation systematically.

### Content & Message
- [ ] **One clear central message** — Can you state it in a single sentence?
- [ ] **Narrative arc** — Opening hook → rising information → climax/insight → resolution/CTA?
- [ ] **Strong opening** — Does slide 1 establish why the audience should care?
- [ ] **Clear CTA** — Does the closing tell the audience what to do next?
- [ ] **Jargon check** — All acronyms defined? Plain language used?
- [ ] **Source check** — All statistics cited and verified?

### Slides: Text & Layout
- [ ] **Word count per slide** — Under 75 words per content slide?
- [ ] **Bullets per slide** — Maximum 5–6 bullets?
- [ ] **Words per bullet** — Maximum 6–8 words?
- [ ] **No nesting beyond 2 levels** of bullets?
- [ ] **Slide titles are findings** — Declarative takeaway, not label?
- [ ] **Whitespace** — At least 30% of each slide is empty/breathing room?
- [ ] **Alignment** — All elements align to a consistent grid?
- [ ] **Centering** — Body text is left-aligned, not centered?

### Typography
- [ ] **Font count** — Maximum 2 font families?
- [ ] **No novelty fonts** — No Comic Sans, Papyrus, Curlz, etc.?
- [ ] **Font size** — Minimum 24pt body text for projected slides?
- [ ] **Line height** — Between 120%–145% of font size?
- [ ] **Emphasis** — Maximum 10–15% of text bolded? Bold means something?

### Color
- [ ] **Contrast ratio** — All text-background pairs meet 4.5:1 minimum? (Check with tool)
- [ ] **Color count** — Maximum 4–5 colors in the palette?
- [ ] **Colorblind check** — No red/green as the only differentiator?
- [ ] **No rainbow vomit** — Colors have systematic meaning, not decorative use?
- [ ] **Colorblind test** — Viewed with colorblind simulation tool?

### Images & Media
- [ ] **Image resolution** — All images sharp at presentation size?
- [ ] **No clipart or generic stock** — All images are purposeful and specific?
- [ ] **No distorted images** — All images at correct aspect ratio?
- [ ] **Video under 60 seconds** — Long videos have a clear purpose?
- [ ] **No autoplay** — Videos triggered manually?

### Data Visualization
- [ ] **No 3D charts of any type** — Especially no 3D pie charts?
- [ ] **Pie charts under 6 slices** — Or replaced with bar charts?
- [ ] **Axes start at zero** — Or truncation is explicitly labeled?
- [ ] **No chartjunk** — Decorative backgrounds, unnecessary gridlines, drop shadows removed?
- [ ] **No dual Y-axes** — Or justified with extreme care?
- [ ] **Color encoding correct** — Red = bad/negative, green = good/positive (or explained otherwise)?
- [ ] **Chart titles are findings** — Not "Bar Chart of Revenue" but "Revenue Grew 23%"?
- [ ] **Maximum 6 data series per chart** — Or aggregated?

### Cognitive Load
- [ ] **Max 5–7 new pieces of information per slide**?
- [ ] **No split attention** — Legends and labels are adjacent to what they describe?
- [ ] **No redundancy** — Slides do not repeat verbatim what will be spoken?
- [ ] **No excessive animation** — All animations serve a pedagogical purpose?
- [ ] **Single-channel input** — Audience is not expected to read and listen to different content simultaneously?

### Consistency
- [ ] **Consistent typography** — Same treatment for same element types throughout?
- [ ] **Consistent color use** — Same color means the same thing throughout?
- [ ] **Consistent layout grid** — Margins and alignment consistent?
- [ ] **Not a default template** — Visual identity customized beyond software defaults?
- [ ] **Not AI generic** — Design reflects specific content, not statistical average?

### Accessibility
- [ ] **Contrast ratios verified** with a tool (not just by eye)?
- [ ] **Color is not the only differentiator** — Shape/pattern/label also used?
- [ ] **Text is never below 18pt** for anything important?

---

## 17. Severity Reference Table

| Anti-Pattern | Category | Severity | Primary Harm |
|---|---|---|---|
| Wall of Text | Content Overload | 🔴 CRITICAL | Audience disengages; nothing retained |
| Presenter Notes Slides | Content Overload | 🔴 CRITICAL | Redundancy kills retention |
| No Central Message | Structure | 🔴 CRITICAL | Nothing memorable |
| Low Contrast Text | Color/Contrast | 🔴 CRITICAL | Content unreadable |
| Flat Hierarchy (no visual hierarchy) | Layout | 🔴 CRITICAL | Audience cannot find the point |
| 3D Pie Charts | Data Viz | 🔴 CRITICAL | Data actively misrepresented |
| Misleading Axes | Data Viz | 🔴 CRITICAL | Data actively misrepresented |
| Reading Slides to Audience | Delivery | 🔴 CRITICAL | Credibility destroyed |
| AI Content Hallucination | AI Failure | 🔴 CRITICAL | Factual errors undermine trust |
| Working Memory Overflow | Cognitive Load | 🔴 CRITICAL | Retention collapses |
| Insufficient contrast (WCAG) | Accessibility | 🔴 CRITICAL | Information inaccessible |
| Bullet Point Addiction | Content | 🟠 MAJOR | Audience loses structure/story |
| 6×6 Rule Violation | Content | 🟠 MAJOR | Cognitive overload |
| Jargon Overload | Content | 🟠 MAJOR | Audience excluded |
| Too Many Fonts (Font Salad) | Typography | 🟠 MAJOR | Amateur appearance |
| Comic Sans / Novelty Fonts | Typography | 🟠 MAJOR | Credibility destroyed |
| Centered Body Text | Typography | 🟠 MAJOR | Readability failure |
| Rainbow Vomit | Color | 🟠 MAJOR | Visual chaos, no hierarchy |
| Neon on White | Color | 🟠 MAJOR | Visual fatigue, vibration |
| Colorblind Failures | Color | 🟠 MAJOR | Information inaccessible to 8% of men |
| Visual Clutter / No Whitespace | Layout | 🔴 CRITICAL | Decision paralysis |
| Misalignment | Layout | 🟠 MAJOR | Unprofessional, trust eroded |
| Floating Elements | Layout | 🟠 MAJOR | Relationships unclear |
| Everything Emphasized | Layout | 🟠 MAJOR | Hierarchy destroyed |
| Chartjunk | Data Viz | 🔴 CRITICAL | Data obscured by decoration |
| Wrong Chart Type | Data Viz | 🟠 MAJOR | Data misrepresented or unreadable |
| Color Encoding Failures (charts) | Data Viz | 🟠 MAJOR | Data misread systematically |
| Clipart / Generic Stock | Images | 🟠 MAJOR | Credibility and engagement lost |
| Irrelevant Images | Images | 🟠 MAJOR | Distraction without value |
| Stretched Images | Images | 🟠 MAJOR | Professional appearance destroyed |
| No Narrative Arc | Structure | 🟠 MAJOR | Nothing remembered from the middle |
| No Call to Action | Structure | 🟠 MAJOR | Presentation produces no change |
| Slide Titles as Labels | Structure | 🟠 MAJOR | Main insight never stated |
| Inconsistent Design Language | Consistency | 🔴 CRITICAL | Trust eroded across deck |
| Default Template Abuse | Consistency | 🟠 MAJOR | Forgettable, generic appearance |
| Over-Animated Presentations | Consistency | 🟠 MAJOR | Distracting, unprofessional |
| AI Generic Aesthetic | AI Failure | 🟠 MAJOR | Forgettable, indistinguishable |
| AI Bullet Point Default | AI Failure | 🔴 CRITICAL | Document disguised as slides |
| Redundancy Effect (read/spoken) | Cognitive Load | 🟠 MAJOR | Retention reduced |
| Split Attention | Cognitive Load | 🟠 MAJOR | Mental effort wasted on search |
| Excessive Decoration/Animation | Cognitive Load | 🟠 MAJOR | Resources wasted on irrelevant input |
| Font Size Too Small | Typography | 🔴 CRITICAL | Content unreadable |
| 6×6 Rule Violation | Typography | 🟠 MAJOR | Reading load overwhelms |
| Line Spacing Failures | Typography | 🟡 MINOR | Readability slightly reduced |
| ALL CAPS Abuse | Typography | 🟡 MINOR | 10–25% slower reading |
| Tracking/Kerning Abuse | Typography | 🟡 MINOR | Word shape recognition disrupted |
| Low Resolution Images | Images | 🔴 CRITICAL | Unprofessional; content unreadable |
| Autoplay Video | Media | 🟡 MINOR | Timing/control problems |
| Weak Opening | Structure | 🔴 CRITICAL | Audience never engages |
| Too Many/Too Few Slides | Pacing | 🟠 MAJOR | Pacing destroys comprehension |
| No Rehearsal | Delivery | 🟠 MAJOR | Timing failures compound all other issues |
| Simultaneous Contrast Failures | Color | 🟡 MINOR | Color appears inconsistent across slides |
| Missing Brand/Identity | Consistency | 🟡 MINOR | Attribution and professionalism reduced |

---

## Key Research References

**Cognitive Science Foundations:**
- Miller, G.A. (1956). "The Magical Number Seven, Plus or Minus Two" — working memory capacity 7±2 items
- Cowan, N. (2001). Realistic working memory capacity: approximately 3–5 chunks for new information
- Sweller, J. — Cognitive Load Theory: intrinsic vs. extraneous cognitive load, split-attention effect
- Mayer, R.E. — Multimedia Learning Theory: coherence principle, redundancy principle, segmenting principle

**Reading & Attention Research:**
- Nielsen Norman Group: users read only 20–28% of text on a page
- Nielsen Norman Group: scannable content produces 47–124% usability improvements over unformatted content
- Nielsen Norman Group: F-pattern and Z-pattern scanning behaviors from eye-tracking research
- Pernice, K. (NNGroup): eye-tracking shows that first lines receive more attention than subsequent lines

**Typography Standards:**
- Butterick, M. — Practical Typography: 120–145% line height optimal; 45–90 chars/line optimal; 10–12pt print minimum; 15–25px web minimum
- Smashing Magazine: 16px minimum web body text; 45–75 chars/line optimal
- 3:1 header-to-body ratio for clarity (IxDF)

**Color & Contrast:**
- WCAG 2.0/2.1: 4.5:1 normal text; 3:1 large text (AA minimum)
- Venngage: 300 million colorblind people worldwide; avoid red/green, green/brown, blue/gray combinations
- Adobe Spectrum: Chromostereopsis, simultaneous contrast, chromatic luminance effects

**Data Visualization:**
- Tufte, E. — "The Cognitive Style of PowerPoint," chartjunk, data-ink ratio
- Knaflic, C.N. — Storytelling with Data: pie chart problems, axis scale manipulation, takeaway titles
- Storytelling with Data: "Helping rid the world of ineffective graphs, one 3D pie at a time"
- Tableau: maximum 3–4 visualizations per view; 6 colors maximum

**Presentation Structure:**
- TED: Top 10 Ways to Ruin a Presentation (text overload, reciting slides, excessive animation)
- Kapterev, A. — Death by PowerPoint: 4 root causes (lack of significance, poor structure, complexity, insufficient rehearsal)
- Duarte, N. — Slide:ology; Resonate
- Reynolds, G. — Presentation Zen

**Visual Hierarchy:**
- IxDF: 50 milliseconds to form first impressions; 2 primary + 2 secondary colors maximum; 3 sizes (S/M/L) maximum
- NNGroup Gestalt: proximity violations cause users to miss available options and misunderstand relationships
- NNGroup: visual clutter creates decision paralysis; users "have no idea where to even begin to look"

---

*This document is intended as a living reference. Anti-patterns should be evaluated in context — some "rules" have legitimate exceptions. The critical distinction is always: does this design choice serve the audience's understanding, or does it serve some other goal (decoration, habit, default behavior, tool limitations)?*

*Document version: 1.0 — March 2026*
