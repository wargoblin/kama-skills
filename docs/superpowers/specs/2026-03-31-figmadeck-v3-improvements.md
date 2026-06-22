# Figmadeck v3: Improvements — Bug Fixes + Smart QA

## Overview

Four categories of changes:
1. **New QA features**: visual content-template fit assessment, content-template swap, distance preservation between elements
2. **New generation feature**: oversized typography budget — calculate character capacity before fill
3. **P1 bug fixes**: mixed-style text, component instances, clone/remove ordering, originalSlideNodeId handoff, auth timeout
4. **P2/P3 fixes**: subtitle detection, footer zone %, overflow absolute coords, design-rules cleanup, placeholder detection, prototype connections, page selection, learn language

---

## New Feature 1: Visual Content-Template Fit Assessment

### When: QA Phase E (NEW — after Phase D score, before declaring DONE)

After content is filled and structural/visual/critique checks pass, evaluate: **does the chosen template slide actually fit this content?**

### Step E1: Visual Assessment

Look at the filled slide screenshot and the original template screenshot side-by-side. Ask:

1. **Does the text content match the visual style of the slide?** Example: a slide with playful illustrations (computer, cart, box) filled with serious financial data = bad fit. A slide with clean two-column layout filled with a 3-step process = bad fit.
2. **Is the text-to-visual ratio preserved?** If the template had 30% text / 70% visual, but after fill it's 80% text / 20% visual because the text is much longer — bad fit.
3. **Are visual elements (illustrations, icons, charts) relevant to the content?** If the template has a shopping cart illustration but the content is about team structure — bad fit.

### Step E2: Retemplate Decision

If Step E1 concludes the fit is **poor**:

1. Review ALL available templates from the original Figma page (the slide map from Step 3)
2. Select a better-fitting template for this content:
   - Text-heavy content → text-only or two-column template
   - Metrics/numbers → stat or metric template
   - Process/steps → timeline or numbered steps template
   - If NO better template exists → keep current, log as "best available"
3. Delete the current filled slide
4. Clone the new template → fill with same content → run QA Phases A-D for this slide only
5. Run Step E1 again on the new version
6. If still poor fit → keep the better of the two, log for learning

### Step E3: Compare and Keep

If the retemplate was tried, compare original fill vs retemplate:
- Which has better structural score? (Phase A)
- Which has better visual fit? (Phase E1)
- Keep the winner, delete the loser

---

## New Feature 2: Distance Preservation Between Elements

### When: Step 5 (Fill Content) — before and after text replacement

### Before Fill: Capture Original Distances

For each slide, before replacing any text, run a `use_figma` call to measure distances between ALL pairs of adjacent elements:

```javascript
await figma.setCurrentPageAsync(generatedPage);
const frame = await figma.getNodeByIdAsync(slideId);

const children = frame.children.filter(n => n.visible);
const distances = [];

for (let i = 0; i < children.length; i++) {
  for (let j = i + 1; j < children.length; j++) {
    const a = children[i], b = children[j];
    const ax = a.absoluteTransform[0][2], ay = a.absoluteTransform[1][2];
    const bx = b.absoluteTransform[0][2], by = b.absoluteTransform[1][2];

    // Vertical gap (a above b)
    const vGap = by - (ay + a.height);
    // Horizontal gap (a left of b)
    const hGap = bx - (ax + a.width);

    // Only track adjacent pairs (gap > 0 and < 200px — reasonably close)
    if (vGap > 0 && vGap < 200) {
      distances.push({
        nodeA: a.id, nodeB: b.id,
        direction: "vertical", originalGap: vGap
      });
    }
    if (hGap > 0 && hGap < 200) {
      distances.push({
        nodeA: a.id, nodeB: b.id,
        direction: "horizontal", originalGap: hGap
      });
    }
  }
}
return { slideId, distances };
```

### After Fill: Check and Restore Distances

After text replacement, re-measure the same pairs:

```javascript
for (const d of distances) {
  const a = await figma.getNodeByIdAsync(d.nodeA);
  const b = await figma.getNodeByIdAsync(d.nodeB);
  // recalculate gap...
  const newGap = /* same formula */;

  if (newGap < 16) {
    // GAP VIOLATION — need fix
    issues.push({ ...d, newGap, deficit: 16 - newGap });
  } else if (newGap < d.originalGap * 0.5) {
    // GAP DEGRADED — more than 50% reduction
    issues.push({ ...d, newGap, deficit: d.originalGap * 0.5 - newGap });
  }
}
```

### Fix Strategies (in priority order)

When a gap violation is detected:

1. **Increase text container width** — make the text block wider so text wraps to fewer lines → height decreases → vertical gap increases. Only works for vertical gaps where nodeA is a text node.
   ```javascript
   // Try widening text by 10% increments up to 30%
   const textNode = await figma.getNodeByIdAsync(d.nodeA);
   if (textNode.type === "TEXT") {
     const origWidth = textNode.width;
     textNode.resize(origWidth * 1.1, textNode.height);
     textNode.textAutoResize = "HEIGHT";
     // Check if gap improved...
   }
   ```

2. **Shorten text content** — rephrase to be more concise (preserve core meaning, aim for ~70% of original length)

3. **Reduce fontSize** — minimum = original × 0.85

4. **Move the lower element down** — only if there's free space below it that doesn't violate footer zone or other element boundaries

5. **Accept reduced gap** — if gap is still ≥ 16px after strategies 1-3, accept it even if less than original. Log the reduction.

### Minimum Gap Rule

**Absolute minimum: 16px between any two elements that were originally separate.** If two elements had 0 gap in the original (touching or overlapping by design), preserve that — don't add artificial spacing.

---

## New Feature 3: Oversized Typography Character Budget

### Problem

Templates with oversized decorative titles (fontSize > 80px, like "PITCH DECK" at ~200px) are designed for **very few characters** — often just 1-3 short words. When replaced with longer text (especially Russian, which is 20-40% longer than English), the text overflows catastrophically: letters overlap, text bleeds outside the container, the slide becomes unreadable.

The agent currently treats this as "decorative element = template design, not an error" — **this is WRONG**. The oversized text IS content that MUST be adapted.

### Fix: Character Budget Calculation in Step 5

Before replacing ANY text node, calculate the **character budget** — how many characters can physically fit:

```javascript
function calculateCharBudget(node) {
  const fontSize = node.fontSize;
  const avgCharWidth = fontSize * 0.6; // approximate for sans-serif
  const containerWidth = node.width;
  const containerHeight = node.height;
  const lineHeight = node.lineHeight.value || fontSize * 1.2;

  const charsPerLine = Math.floor(containerWidth / avgCharWidth);
  const maxLines = Math.floor(containerHeight / lineHeight);
  const totalBudget = charsPerLine * maxLines;

  return { charsPerLine, maxLines, totalBudget, fontSize };
}
```

### Decision Logic

```
IF fontSize > 80px (oversized decorative):
  1. Calculate character budget
  2. IF newText.length > totalBudget:
     → AGGRESSIVELY shorten text to fit budget
     → Not "preserve meaning at 70%" — shorten to EXACT budget
     → Prefer: key phrase, 2-3 impactful words max
     → Example: "Презентация, которая убеждает" → "Убеждай" or "Pitch Day"
  3. IF newText.length > totalBudget * 2 (way too long):
     → This template slot is INCOMPATIBLE with this content
     → Trigger Phase E: find a template with smaller title font
  4. NEVER leave oversized text overflowing — this is CRITICAL severity

IF fontSize 40-80px (large but not decorative):
  Apply normal overflow handling (shorten → reduce font → speaker notes)

IF fontSize < 40px (normal):
  Apply normal overflow handling
```

### QA Integration

In QA Phase A, add a specific check:

```markdown
### Oversized Text Overflow

For each TEXT node with fontSize > 80px:
- Is the text visually contained within its bounding box? (not clipped, not overlapping neighbors)
- Can you READ the text? (letters not overlapping each other)
- Does it look intentional? (like the original template's dramatic typography)

If ANY oversized text is unreadable or overflowing → **CRITICAL**
This is NOT "decorative design" — it's a fill error that MUST be fixed.
```

---

## P1 Fix #1: Mixed-Style Text Nodes

### Problem

`node.fontName` returns `figma.mixed` when a text node contains multiple font styles. `node.characters = newText` after `loadFontAsync(figma.mixed)` crashes.

### Fix in `figma-generation.md` Step 5 (applyText function)

Before assigning text, check for mixed styles:

```javascript
async function applyText(node, newText, parentFrame) {
  // CHECK FOR MIXED STYLES
  if (node.fontName === figma.mixed) {
    // Get all style segments
    const segments = node.getStyledTextSegments([
      'fontName', 'fontSize', 'fontWeight', 'fills'
    ]);

    // Load ALL fonts used in the node
    const uniqueFonts = new Set();
    for (const seg of segments) {
      const key = `${seg.fontName.family}|${seg.fontName.style}`;
      if (!uniqueFonts.has(key)) {
        uniqueFonts.add(key);
        const resolved = await resolveFont(seg.fontName);
        await figma.loadFontAsync(resolved);
      }
    }

    // Set text (this resets all styling to the first segment's style)
    node.characters = newText;

    // Restore the FIRST segment's style to the entire text
    // (best we can do — original mixed styling was for different content)
    const primaryFont = segments[0].fontName;
    const resolved = await resolveFont(primaryFont);
    await figma.loadFontAsync(resolved);
    node.setRangeFontName(0, newText.length, resolved);

    return;
  }

  // NORMAL (single style) — existing logic
  const resolved = await resolveFont(node.fontName);
  await figma.loadFontAsync(resolved);
  node.characters = newText;
}
```

---

## P1 Fix #2: Component Instance Text Editing

### Problem

Text nodes inside component instances may throw on direct `node.characters` assignment or silently detach the instance.

### Fix in `figma-generation.md` Step 5

Before editing a text node, check if it's inside an instance:

```javascript
function findParentInstance(node) {
  let current = node.parent;
  while (current) {
    if (current.type === "INSTANCE") return current;
    current = current.parent;
  }
  return null;
}

// Before text edit:
const parentInstance = findParentInstance(textNode);
if (parentInstance) {
  // Try exposed component properties first
  const props = parentInstance.componentProperties;
  for (const [key, prop] of Object.entries(props)) {
    if (prop.type === "TEXT" && prop.value === textNode.characters) {
      // This text is an exposed property — use setProperties
      parentInstance.setProperties({ [key]: newText });
      return; // Done — no direct edit needed
    }
  }
  // No matching exposed property — direct edit creates local override
  // This is acceptable but log it
  // Proceed with normal applyText...
}
```

---

## P1 Fix #3: Clone/Remove Ordering

### Problem

If the donor slide (used for cloning) is also on the "unused" list, removing it before cloning destroys it.

### Fix in `figma-generation.md` Step 4

Add explicit ordering rule:

```markdown
**CRITICAL: Operation order in Step 4:**
1. FIRST: clone all needed donor slides (create extras)
2. THEN: remove all unused slides
3. LAST: reorder remaining slides by x position

Never remove a slide before all cloning is complete. The clone
operation must happen on the ORIGINAL node — a removed node
cannot be cloned.
```

---

## P1 Fix #4: originalSlideNodeId Handoff

### Problem

QA Phase B needs `originalSlideNodeId` but it's not explicitly passed from the clone step.

### Fix

In `figma-generation.md` Step 2, the return value already includes `origId`:
```javascript
return { pageId: newPage.id, slides: [{id, name, origId, x}] };
```

In `qa-cycle.md` Phase B, add explicit instruction:

```markdown
**IMPORTANT:** The `originalSlideNodeId` used for comparison comes from
the clone step (Step 2) return value — the `origId` field for each slide.
This is the node ID on the ORIGINAL template page (Page 1), NOT the
cloned page. Pass this mapping through the entire pipeline:

slideMapping = {
  adaptedSlideId: "99:125",      // on generated page
  originalSlideNodeId: "1:42",   // on template page
}
```

---

## P1 Fix #5: Auth Timeout and Retry

### Fix in `auth.md`

Add to both First Run and Subsequent Run flows:

```markdown
**Timeouts and error handling:**
- Navigation timeout: 30 seconds (`{ timeout: 30000 }`)
- If navigation times out → network error, NOT auth failure
- Network error handling: retry up to 3 times with 10s pause
- Auth failure handling: ask for new credentials, max 3 attempts
- If 3 auth attempts all fail → error: "Unable to authenticate
  after 3 attempts. Check credentials and network connection."

**Distinguish network errors from auth failures:**
- Network error: page didn't load at all (timeout, DNS, connection refused)
- Auth failure: page loaded, but login form shows error message
- 2FA failure: 2FA form shows error after code submission
```

---

## P2 Fix #6: Subtitle Detection

### Fix in `figma-generation.md` Step 3 role detection table

Add priority 3.5 between current priorities 3 and 4:

```markdown
| 3.5 | Second-largest fontSize ≥ 28px, within top 60% of frame, fontSize < largest × 0.85 | subtitle |
```

And add `subtitle` as a valid fill role in Step 5 — map it to `outlineSlide.subtitle`.

---

## P2 Fix #7: Footer Zone Percentage

### Fix in `figma-generation.md` and `qa-cycle.md`

Replace ALL instances of hardcoded `56px` / `44-56px` / `44–56px` footer zone with:

```markdown
Footer zone = bottom 8% of frame height
(e.g., 1080px frame → 86px zone; 540px → 43px; 1440px → 115px)

Formula: `frame.height * 0.08`
```

---

## P2 Fix #9: Overflow Check — Absolute vs Relative Coordinates

### Fix in `figma-generation.md` overflow handling

Replace:
```javascript
if (node.y + node.height > parentFrame.height)
```

With:
```javascript
const nodeBottom = node.absoluteTransform[1][2] + node.height;
const frameBottom = parentFrame.absoluteTransform[1][2] + parentFrame.height;
if (nodeBottom > frameBottom)
```

Same fix in `qa-cycle.md` Phase A boundary check.

---

## P2 Fix #11: Remove CSS/Slidev Content from design-rules.md

### Fix

In `design-rules.md`:
- **Section 6 (Element Spacing & Overflow):** Remove references to `position: absolute`, `flexbox`. Replace with Figma-native equivalents: "auto-layout constraints", "absolute positioning in Figma canvas".
- **Section 8 (Animation):** Remove entirely — `v-click`, CSS `transform`/`opacity`, `prefers-reduced-motion` are not Figma concepts. Replace with a short note: "Figma presentations use Smart Animate and prototype transitions. This is outside figmadeck's scope — the template's existing prototype connections are preserved by clone."
- **Quick-Reference Checklist:** Remove `v-click` and CSS animation items.

---

## P3 Fix #17: Template-Agnostic Placeholder Detection

### Fix in `qa-cycle.md` Phase A

Replace the hardcoded `placeholderPatterns` regex list with a comparison against the ORIGINAL template text:

```markdown
### Unchanged Placeholder Text Detection

Instead of matching hardcoded patterns, compare each text node's
content against what the ORIGINAL template had:

1. For each TEXT node on the adapted slide, find the corresponding
   node on the original template (using origId mapping from Step 2)
2. Get the original text: `get_metadata` or stored from Step 3 analysis
3. If `adaptedText === originalText` AND this node was supposed to be
   filled (role is title/description/body, not decorative) →
   **CRITICAL: text was never replaced**

This is template-agnostic — works for ANY template regardless
of language or placeholder conventions.
```

---

## P3 Fix #15: Prototype Connections After Reorder

### Fix in `figma-generation.md` Step 4 (after reorder)

```markdown
### Update Prototype Connections

After reordering slides by x position, update prototype navigation:

For each slide (in order):
- If slide has existing prototype connections (reactions/interactions):
  - Remove old "navigate to" actions that point to slides no longer in sequence
  - Add new "on click → navigate to next slide" if the template used sequential navigation

Note: This is best-effort. Complex prototype interactions (conditional
navigation, overlays) cannot be automatically remapped. Log a warning
if complex interactions are detected.
```

---

## P3 Fix #22: Template Page Selection

### Fix in `figma-generation.md` Step 2

Replace `figma.root.children[0]` with smart page detection:

```markdown
### Template Page Selection

Do NOT hardcode `figma.root.children[0]`. Detect the template page:

1. If URL contains `node-id` pointing to a specific page → use that page
2. Else scan all pages, select the one with the MOST top-level FRAME
   children that match slide aspect ratio (~16:9 ±10%)
3. If ambiguous (multiple pages with similar frame counts) → use the
   first page by order (index 0) but log: "Multiple candidate pages
   found, using first page. Specify page with node-id in URL if wrong."
```

---

## P3 Fix #12: Learn Language Not Hardcoded

### Fix in `figma-learning.md` FDL-2

Replace:
```markdown
All outlines in **Russian**.
```

With:
```markdown
Outline language should **match the template's primary language**.
Detect from Step 3 analysis — what language are the existing text
nodes written in? Generate outlines in that same language.

If template language is ambiguous or mixed → ask the user:
"What language should the generated presentations use?"
```

---

## Files Modified

| File | Changes |
|------|---------|
| `qa-cycle.md` | Phase E (content-template fit), placeholder detection rewrite (#17), originalSlideNodeId (#4), overflow absolute coords (#9), footer zone % (#7) |
| `figma-generation.md` | Distance preservation, mixed-style text (#1), component instances (#2), clone/remove ordering (#3), subtitle detection (#6), footer zone % (#7), overflow coords (#9), prototype connections (#15), page selection (#22) |
| `figma-learning.md` | Learn language detection (#12) |
| `design-rules.md` | Remove CSS/Slidev content (#11) |
| `auth.md` | Timeout + retry (#5) |
