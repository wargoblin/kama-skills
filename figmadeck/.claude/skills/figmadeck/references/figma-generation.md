# Figma-Native Generation Pipeline

Steps 2-5 of the figmadeck pipeline. Runs after auth check (Step 1). Followed by `qa-cycle.md` (Step 6).

---

## Step 2: Clone Template Page

**One `use_figma` call.**

### Template Page Selection

Do NOT hardcode `figma.root.children[0]`. Detect the template page:

1. If the Figma URL contains `node-id` pointing to a specific page → use that page
2. Else scan all pages, select the one with the MOST top-level FRAME children that match slide aspect ratio (~16:9 ±10%)
3. If ambiguous (multiple pages with similar frame counts) → use the first page by order (index 0) but log: `"Multiple candidate pages found, using first page. Specify page with node-id in URL if wrong."`

```js
// Smart page detection
function detectTemplatePage(urlNodeId) {
  if (urlNodeId) {
    // URL contains node-id — find the page containing that node
    for (const page of figma.root.children) {
      const node = page.findOne(n => n.id === urlNodeId);
      if (node) return page;
    }
  }
  // Scan all pages — pick the one with most 16:9 frames (±10%)
  let bestPage = figma.root.children[0];
  let bestCount = 0;
  for (const page of figma.root.children) {
    const count = page.children.filter(n => {
      if (n.type !== "FRAME") return false;
      const ratio = n.width / n.height;
      return ratio >= (16/9) * 0.9 && ratio <= (16/9) * 1.1;
    }).length;
    if (count > bestCount) { bestCount = count; bestPage = page; }
  }
  if (figma.root.children.filter(p => {
    const c = p.children.filter(n => {
      if (n.type !== "FRAME") return false;
      const ratio = n.width / n.height;
      return ratio >= (16/9) * 0.9 && ratio <= (16/9) * 1.1;
    }).length;
    return c === bestCount;
  }).length > 1) {
    console.log("Multiple candidate pages found, using first page. Specify page with node-id in URL if wrong.");
  }
  return bestPage;
}

// 1. Set context to the detected template page
const page1 = detectTemplatePage(/* urlNodeId from URL params, or null */);
await figma.setCurrentPageAsync(page1);

// 2. Create the new page
const newPage = figma.createPage();
newPage.name = "Generated: <outline-name>";  // e.g. "Generated: Q1 Sales Review"

// 3. Clone all children from the template, preserving exact positions
const slides = [];
for (const child of page1.children) {
  const clone = child.clone();
  newPage.appendChild(clone);
  clone.x = child.x;   // MUST set after appendChild
  clone.y = child.y;
  slides.push({ id: clone.id, name: clone.name, origId: child.id, x: clone.x });
}

// 4. Return identifiers for subsequent steps
return { pageId: newPage.id, slides };
```

**What clone preserves:** gradients, images, shadows, auto-layout, variable bindings, effects, fonts, colors — pixel-perfect by definition. No conversion artifacts.

**Return shape:** `{ pageId: string, slides: [{ id, name, origId, x }] }`

---

## Step 3: Analyze Template Slides

**IMPORTANT: Keep `use_figma` calls lightweight.** The Plugin API can hang on heavy scripts that traverse entire node trees. Split analysis into two calls:

**Call 1 — Frame inventory (fast):** List all top-level frames with basic info (id, name, x, width, height, childCount). No `findAll`, no deep traversal.

**Call 2 — Text analysis per batch of 3-5 slides:** For each batch, find TEXT nodes and detect roles. Return the slide map incrementally. If the template has 18 slides, use 4 calls (5+5+5+3) instead of one massive call.

**Never put `findAll` + role detection + content type detection in a single `use_figma` call for ALL slides.** This is the #1 cause of Plugin API hangs.

### Call 1: Frame Inventory

```js
const generatedPage = figma.root.children.find(p => p.id === pageId);
await figma.setCurrentPageAsync(generatedPage);

const slideMap = [];

for (const frame of generatedPage.children) {
  const textNodes = frame.findAll(n => n.type === "TEXT");

  const texts = textNodes.map(node => ({
    id: node.id,
    name: node.name,
    characters: node.characters,
    fontSize: node.fontSize,
    x: node.absoluteTransform[0][2],
    y: node.absoluteTransform[1][2],
    width: node.width,
    height: node.height,
    role: detectRole(node, frame, textNodes),
  }));

  slideMap.push({
    id: frame.id,
    name: frame.name,
    index: generatedPage.children.indexOf(frame),
    contentType: detectContentType(frame, texts, generatedPage),
    textNodes: texts,
  });
}

return { slideMap };
```

### Text Node Role Detection (hybrid: name priority + heuristics)

Apply rules in order — first match wins:

| Priority | Condition | Role |
|---|---|---|
| 1 | `node.name` includes "Title" or "Heading" (case-insensitive) | `title` |
| 2 | `node.name` includes "description", "body", or "desc" (case-insensitive) | `description` |
| 3 | Largest `fontSize` on slide AND `fontSize >= 36` | `title` |
| 3.5 | Second-largest `fontSize` ≥ 28px, within top 60% of frame (`node.y < frame.height * 0.6`), `fontSize < largest × 0.85` | `subtitle` |
| 4 | Second largest `fontSize`, in range 16-24px, `characters.length > 50` | `description` |
| 5 | `node.y > frame.height * (1 - 0.08)` AND `fontSize < 14` (bottom 8% of frame — `frame.height * 0.08`) | `footer` |
| 6 | `characters.length < 20` AND `characters === characters.toUpperCase()` | `label` |
| 7 | Everything else | `body` |

**Use `includes()` not `===` for name checks** — Unicode normalization differs after clone.

### Content Type Detection

Evaluate per frame after collecting text roles:

| Condition | contentType |
|---|---|
| First frame on page AND has a `title` role node with `fontSize >= 48` | `intro` |
| Last frame on page AND text includes CTA patterns ("contact", "next steps", "start", "connect") | `cta` |
| Single text node, numeric content, `fontSize >= 64` | `metric` |
| Frame has 3+ direct children that are frames (similar width/height) | `cards` |
| `< 30` total text characters AND 1+ image fills on child nodes | `visual-break` |
| Default | `content` |

---

## Step 4: Match Outline → Template Slides

Parse the outline to determine count and content type for each slide, then reconcile against the template `slideMap`.

### Mandatory Title Slide (FIRST)

**Before running the matching loop**, ensure the first outline slide is a cover/title:

1. If outline slide 0 has `contentType: "intro"` → OK, proceed
2. If outline slide 0 is NOT `contentType: "intro"` → **prepend a synthetic cover slide** using the presentation topic as title and the first slide's subtitle/description as subtitle
3. Find the template slide with `contentType: "intro"` (first frame, largest title fontSize ≥ 48). If none → use the first template frame
4. The cover slide is ALWAYS slide 0 in the final order — it is never skipped, never removed, never matched to a non-intro template

### Matching Algorithm

```
for each outlineSlide:
  1. Determine outlineSlide.contentType (same heuristics as Step 3)
  2. Find templateSlides where templateSlide.contentType === outlineSlide.contentType
  3. If multiple matches → context-based variant selection:
     - Warm color palette in template + positive/growth theme in outline → prefer
     - Cool/analytical palette + data/metrics theme → prefer
     - When ambiguous → pick first unassigned match
  4. Mark matched template slide as assigned
```

**CRITICAL: Operation order in Step 4:**
1. **FIRST**: clone all needed donor slides (create extras for outline slides that need them)
2. **THEN**: remove all unused slides
3. **LAST**: reorder remaining slides by x position

Never remove a slide before all cloning is complete. The clone operation must happen on the ORIGINAL node — a removed node cannot be cloned. If a donor slide also appears on the "unused" list, it must be cloned BEFORE it is removed.

### When Outline Has MORE Slides Than Template Slots

If no unassigned template slide matches the required content type, clone an existing one:

```js
// Additional use_figma call
await figma.setCurrentPageAsync(generatedPage);
const donor = generatedPage.children.find(f => f.id === bestMatchId);
const cloned = donor.clone();
generatedPage.appendChild(cloned);
cloned.x = donor.x + donor.width + 100;  // temporary position
cloned.y = donor.y;
return { newSlideId: cloned.id };
```

### Remove Unused Slides

```js
// In the same or a separate use_figma call
await figma.setCurrentPageAsync(generatedPage);
for (const unusedId of unusedTemplateIds) {
  const node = figma.getNodeById(unusedId);
  if (node) node.remove();
}
```

### Reorder by Outline Position

Set `node.x` so slides flow left-to-right in outline order:

```js
await figma.setCurrentPageAsync(generatedPage);
let xOffset = 0;
for (const slideId of orderedSlideIds) {
  const node = figma.getNodeById(slideId);
  node.x = xOffset;
  xOffset += node.width + 100;  // 100px gap between slides
}
```

### Update Prototype Connections

After reordering slides by x position, update prototype navigation connections:

```js
// For each slide in the new order:
for (let i = 0; i < orderedSlideIds.length; i++) {
  const slide = figma.getNodeById(orderedSlideIds[i]);
  if (!slide || !slide.reactions) continue;

  for (const reaction of slide.reactions) {
    if (!reaction.action) continue;
    const action = reaction.action;

    // Remove old "navigate to" actions pointing to slides no longer in sequence
    if (action.type === "NODE" && action.destinationId) {
      if (!orderedSlideIds.includes(action.destinationId)) {
        // Destination was removed — point to next slide in sequence instead
        const nextSlideId = orderedSlideIds[i + 1] || null;
        if (nextSlideId) {
          action.destinationId = nextSlideId;
        }
        console.log(`Prototype connection updated: slide ${i} → ${nextSlideId}`);
      }
    }
  }
}
```

**Note:** This is best-effort. Complex prototype interactions (conditional navigation, overlays, component-level interactions) cannot be automatically remapped. Log a warning if complex interactions are detected so the user can fix them manually in Figma after generation.

---

## Step 5: Fill Content

**One `use_figma` call PER SLIDE** (incremental, not batched). This keeps errors isolated — if one slide fails, others are unaffected.

### Pre-fill: Group Pattern Check (MANDATORY for repeated elements)

Before filling a slide that contains a **group of similar elements** (cards, list items, agenda items, process steps — identified by: multiple child frames with identical structure), run this check:

1. **Identify the group**: find all child frames that share the same internal structure (same number of text nodes, same roles)
2. **For each element in the group**, calculate the character budget of the "hero" text node (the one with the largest fontSize):
   ```
   budget = floor(containerWidth / (fontSize * 0.6)) * floor(containerHeight / (lineHeight || fontSize * 1.2))
   ```
3. **Compare each outline content item against its budget**:
   - If content fits budget → OK
   - If content exceeds budget → **generate 3 alternative texts** that fit:
     1. Single symbol/emoji that represents the concept (e.g., "!" instead of "5с")
     2. Shortened to 1-2 words max
     3. Abbreviation or initialism
   - **Pick the best alternative** that preserves meaning AND matches the pattern of sibling elements
4. **Homogeneity rule**: if N-1 elements in the group follow a pattern (e.g., single character), the outlier MUST be adapted to match. A group of `["P", "S", "I", "5с"]` → the outlier "5с" must become a single character like `"!"` or `"✓"`.

**This check prevents the most common fill failure**: content that fits semantically but not visually in a template slot designed for a different text pattern.

```js
// Called once per slide
await figma.setCurrentPageAsync(generatedPage);  // MANDATORY every call

const frame = figma.getNodeById(slideId);
const outlineSlide = /* matching outline slide data */;

for (const textNode of frame.findAll(n => n.type === "TEXT")) {
  const role = detectRole(textNode, frame, frame.findAll(n => n.type === "TEXT"));
  // Valid fill roles: title, subtitle, description, body, label
  // subtitle maps to outlineSlide.subtitle (from role detection priority 3.5)
  let newText = role === "subtitle"
    ? (outlineSlide.subtitle || outlineSlide.content?.subtitle)
    : outlineSlide.content[role];
  // ZERO PLACEHOLDER RULE: never leave template text unchanged
  if (!newText) {
    // Fallback: use slide title for unknown roles, section name for footers, empty for labels
    if (role === "footer") newText = outlineSlide.sectionName || outlineSlide.title;
    else if (role === "label") newText = ""; // clear decorative labels
    else newText = outlineSlide.title; // fallback to slide title
    // If still empty, clear the node to avoid template placeholder
    if (!newText) { newText = ""; }
  }

  // Component instance check — BEFORE calling applyText
  const parentInstance = findParentInstance(textNode);
  if (parentInstance) {
    // Try exposed component properties first
    const props = parentInstance.componentProperties;
    let handled = false;
    for (const [key, prop] of Object.entries(props)) {
      if (prop.type === "TEXT" && prop.value === textNode.characters) {
        // This text is an exposed property — use setProperties
        parentInstance.setProperties({ [key]: newText });
        handled = true;
        break; // Done — no direct edit needed
      }
    }
    if (handled) continue;
    // No matching exposed property — direct edit creates local override
    // This is acceptable but log it
    console.log(`Direct text edit inside instance ${parentInstance.id} — creates local override`);
  }

  await applyText(textNode, newText, frame);
}

return { slideId, filled: true };
```

### Before Fill: Capture Original Distances

**Before replacing ANY text**, measure gaps between ALL pairs of adjacent visible children in the slide frame. Store these as `originalGap` values for later comparison.

```js
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
// Store distances — used after fill to check and restore spacing
```

**Minimum Gap Rule:** 16px minimum between any two elements that were originally separate. If two elements had 0 gap in the original (touching or overlapping by design), preserve that — do not add artificial spacing.

### After Fill: Check and Restore Distances

After text replacement, re-measure the same pairs and apply fix strategies if gaps have degraded:

```js
const issues = [];
for (const d of distances) {
  const a = await figma.getNodeByIdAsync(d.nodeA);
  const b = await figma.getNodeByIdAsync(d.nodeB);
  // recalculate gap using same formula as above
  const ay = a.absoluteTransform[1][2], ax = a.absoluteTransform[0][2];
  const by = b.absoluteTransform[1][2], bx = b.absoluteTransform[0][2];
  const newGap = d.direction === "vertical"
    ? by - (ay + a.height)
    : bx - (ax + a.width);

  if (newGap < 16) {
    // GAP VIOLATION — need fix
    issues.push({ ...d, newGap, deficit: 16 - newGap });
  } else if (newGap < d.originalGap * 0.5) {
    // GAP DEGRADED — more than 50% reduction
    issues.push({ ...d, newGap, deficit: d.originalGap * 0.5 - newGap });
  }
}
```

**Fix Strategies (in priority order)** — apply to each gap violation, stop at first resolution:

1. **Increase text container width** — widen the text block so text wraps to fewer lines → height decreases → vertical gap increases. Only for vertical gaps where nodeA is a TEXT node.
   ```js
   const textNode = await figma.getNodeByIdAsync(d.nodeA);
   if (textNode.type === "TEXT") {
     const origWidth = textNode.width;
     textNode.resize(origWidth * 1.1, textNode.height);
     textNode.textAutoResize = "HEIGHT";
     // Check if gap improved — retry up to 3x (10%, 20%, 30%)
   }
   ```
2. **Shorten text content** — rephrase to be more concise (preserve core meaning, aim for ~70% of original length)
3. **Reduce fontSize** — minimum = original × 0.85
4. **Move the lower element down** — only if there is free space below it that doesn't violate the footer zone or other element boundaries
5. **Accept reduced gap** — if gap is still ≥ 16px after strategies 1-3, accept it even if less than the original. Log the reduction.

### Post-Fill: Text-vs-Text Overlap Check (MANDATORY)

After filling ALL text on a slide, check every pair of visible text nodes for overlap. This catches cases where two text nodes within the SAME container both grew after Russian text replacement.

```js
// Run AFTER text fill, BEFORE returning
const allTexts = frame.findAll(n => n.type === "TEXT" && n.visible);
const textOverlaps = [];

for (let i = 0; i < allTexts.length; i++) {
  for (let j = i + 1; j < allTexts.length; j++) {
    const a = allTexts[i], b = allTexts[j];
    const ax = a.absoluteTransform[0][2], ay = a.absoluteTransform[1][2];
    const bx = b.absoluteTransform[0][2], by = b.absoluteTransform[1][2];

    // AABB overlap test
    if (ax < bx + b.width && ax + a.width > bx &&
        ay < by + b.height && ay + a.height > by) {
      textOverlaps.push({
        nodeA: a.id, textA: a.characters.substring(0, 20),
        nodeB: b.id, textB: b.characters.substring(0, 20),
      });
    }
    // Minimum gap check (8px between text nodes)
    else {
      const dx = Math.max(0, Math.max(bx - (ax + a.width), ax - (bx + b.width)));
      const dy = Math.max(0, Math.max(by - (ay + a.height), ay - (by + b.height)));
      const gap = Math.min(dx || Infinity, dy || Infinity);
      if (gap < 8 && gap < Infinity) {
        textOverlaps.push({
          nodeA: a.id, textA: a.characters.substring(0, 20),
          nodeB: b.id, textB: b.characters.substring(0, 20),
          gap: Math.round(gap), type: "proximity"
        });
      }
    }
  }
}

// Fix overlaps: shorten the LONGER text node first
for (const ov of textOverlaps) {
  // Strategy 1: expand width of the upper/left node
  // Strategy 2: shorten the node with more characters
  // Strategy 3: reduce fontSize of the overlapping node (min × 0.85)
  // Apply same fix priority as gap violations above
}
```

**This check is NOT optional.** Text-on-text overlap is the most common visual defect in Russian presentations because Russian text is 20-40% longer than English.

### Helper Functions

```js
// Walk parent chain to find enclosing INSTANCE node (if any)
function findParentInstance(node) {
  let current = node.parent;
  while (current) {
    if (current.type === "INSTANCE") return current;
    current = current.parent;
  }
  return null;
}

// Calculate character budget for a text node (used for oversized typography)
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

### Font Handling

**MUST `loadFontAsync` before any text change.** Never skip this step.

```js
async function applyText(node, newText, parentFrame) {
  // STEP 0: Mixed-style text check — MUST be FIRST
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

  // STEP 0b: Oversized typography budget check
  // IF fontSize > 80px (oversized decorative):
  if (node.fontSize > 80) {
    const budget = calculateCharBudget(node);
    if (newText.length > budget.totalBudget * 2) {
      // Way too long — template slot INCOMPATIBLE with this content
      // Trigger Phase E: find a template with smaller title font
      console.log(`PHASE_E_TRIGGER: fontSize=${node.fontSize}, budget=${budget.totalBudget}, text=${newText.length} chars — slot incompatible`);
      // Still attempt aggressive shortening as interim
      newText = newText.split(/\s+/).slice(0, 2).join(" ");
    } else if (newText.length > budget.totalBudget) {
      // Aggressively shorten to fit budget — NOT "preserve meaning at 70%"
      // Prefer: key phrase, 2-3 impactful words max
      const words = newText.split(/\s+/);
      let shortened = "";
      for (const word of words) {
        if ((shortened + " " + word).trim().length <= budget.totalBudget) {
          shortened = (shortened + " " + word).trim();
        } else break;
      }
      newText = shortened || words[0]; // at minimum, use first word
      console.log(`Oversized text shortened for fontSize=${node.fontSize}: "${newText}"`);
    }
    // NEVER leave oversized text overflowing — this is CRITICAL severity
  }

  // 1. Check availability
  const available = await figma.listAvailableFontsAsync();
  const fontFamily = node.fontName.family;
  const fontStyle = node.fontName.style;

  const isAvailable = available.some(
    f => f.fontName.family === fontFamily && f.fontName.style === fontStyle
  );

  // 2. Resolve font (original or fallback)
  let resolvedFont = node.fontName;
  if (!isAvailable) {
    resolvedFont = resolveFallback(fontFamily, fontStyle);
    console.log(`Font fallback: ${fontFamily} → ${resolvedFont.family}`);
    node.fontName = resolvedFont;  // set before characters
  }

  // 3. Load font — MANDATORY before characters assignment
  await figma.loadFontAsync(resolvedFont);

  // 4. Assign text
  node.characters = newText;

  // 5. Overflow handling
  node.textAutoResize = "HEIGHT";
  const nodeBottom = node.absoluteTransform[1][2] + node.height;
  const frameBottom = parentFrame.absoluteTransform[1][2] + parentFrame.height;
  if (nodeBottom > frameBottom) {
    handleOverflow(node, newText, parentFrame);
  }
}
```

### Font Fallback Table

| Original Family | Fallback Family |
|---|---|
| Whyte | Inter |
| Whyte Inktrap | Inter |
| Darker Grotesque | Manrope |
| Neue Haas Grotesk | Inter |
| GT Walsheim | Inter |
| Playfair Display | DM Serif Display |
| Custom serif (unknown) | DM Serif Display |
| Any other unavailable | Inter |

Always match the original `style` (Regular, Bold, Italic, etc.) in the fallback family. If the exact style is also unavailable, use Regular.

### Text Overflow Handling

Applied in order — stop at first resolution:

1. **textAutoResize = "HEIGHT"** — let text grow downward (always set this first)
2. **Check overflow:** `node.absoluteTransform[1][2] + node.height > parentFrame.absoluteTransform[1][2] + parentFrame.height`
3. If still overflowing: **shorten/rephrase** the text (preserve core meaning, aim for ~70% length)
4. If still overflowing: **reduce fontSize** — minimum = `current × 0.85` (never go below 85% of original)
5. If still overflowing: **expand container** — increase `node.height` if space exists below
6. Last resort: add a note text node positioned below the slide frame (visible in Figma but outside slide bounds)

### Footer Update (MANDATORY)

After filling main content, **ALWAYS** update footer text nodes. These are template placeholders that MUST be replaced:

```js
// Find footer nodes: bottom 8% of frame height, small text
// Formula: frame.height * 0.08 (e.g. 1080px → 86px; 540px → 43px; 1440px → 115px)
const footerZone = frame.height * 0.08;
const footerNodes = frame.findAll(n =>
  n.type === "TEXT" &&
  n.absoluteTransform[1][2] + n.height > frame.height - footerZone &&
  n.fontSize <= 14
);

for (const fNode of footerNodes) {
  await figma.loadFontAsync(fNode.fontName);

  // Determine what to fill based on position
  const relativeX = fNode.absoluteTransform[0][2] - frame.absoluteTransform[0][2];

  if (relativeX < frame.width * 0.25) {
    // Left area: breadcrumb (company/project name)
    fNode.characters = outlineSlide.projectName || "Presentation";
  } else if (relativeX < frame.width * 0.5) {
    // Center-left: section name
    fNode.characters = outlineSlide.sectionName || outlineSlide.title;
  } else if (relativeX > frame.width * 0.9) {
    // Far right: page number (updated in Step 5b)
    // Will be set in the final numbering pass
  } else {
    // Center-right: subsection or topic
    fNode.characters = outlineSlide.subtitle || outlineSlide.title;
  }
}
```

**Never leave template footer text** like "Reviews / Mobile Strategy" or "Product review — Month XX". These are **CRITICAL** failures in QA.

### Page Numbering (Step 5b — after all slides filled)

After ALL slides have been filled with content, run one final `use_figma` call to update page numbers:

```js
await figma.setCurrentPageAsync(generatedPage);

// Get slides in x-order (left to right = presentation order)
const slides = generatedPage.children
  .filter(n => n.type === "FRAME")
  .sort((a, b) => a.x - b.x);

for (let i = 0; i < slides.length; i++) {
  const frame = slides[i];
  // Find the page number node: rightmost text in footer zone, contains a number
  const footerTexts = frame.findAll(n =>
    n.type === "TEXT" &&
    n.absoluteTransform[1][2] + n.height > frame.height - (frame.height * 0.08)
  );
  const pageNumNode = footerTexts
    .sort((a, b) => b.absoluteTransform[0][2] - a.absoluteTransform[0][2])[0]; // rightmost

  if (pageNumNode) {
    await figma.loadFontAsync(pageNumNode.fontName);
    pageNumNode.characters = String(i + 1);
  }
}
```

### Empty Visual Areas

During Step 3 (Analyze), identify **non-text areas**: frames with no TEXT children, large gray rectangles, image placeholders. Mark them in the slide map as `visualArea: true`.

During Step 5, for each visual area:
- If outline provides data for it (chart data, images, metrics) → keep and adapt
- If outline has NO data for it → **hide** the placeholder: `node.visible = false`
- After hiding, check if remaining content needs to be repositioned to fill the freed space

### Special Characters

After clone, Unicode normalization may differ. Rules:

- Use `node.characters.includes(searchTerm)` not `===` for text matching
- For footer/breadcrumb nodes: find by **position** (`node.y > frame.height * (1 - 0.08)` AND `node.fontSize < 14` — bottom 8% of frame) rather than by exact text content
- Do not hardcode special characters (em-dash, bullet, arrow) as string literals — use the node's existing character content as reference

### Return Shape (per slide call)

```js
return {
  slideId,
  filled: true,
  fontSubstitutions: [{ original, fallback }],  // empty if none
  overflowsResolved: number,
  footerUpdated: true,
};
```

---

## use_figma Rules

These apply to ALL `use_figma` calls in the pipeline:

- **MUST** `await figma.setCurrentPageAsync(page)` at the START of every `use_figma` call — page context resets between calls
- **MUST** `await figma.loadFontAsync(font)` BEFORE any `node.characters` assignment — even if the font was loaded in a previous call
- **`layoutSizingHorizontal = "FILL"`** MUST be set AFTER `parent.appendChild(child)`, never before
- **MUST** `return` all created or mutated node IDs — do not use `console.log` or `figma.notify` for output
- Colors are in 0–1 range, not 0–255
- Use `includes()` not `===` for text/name matching (Unicode differences after clone)
- On any error: STOP — read the full error message, fix the script, retry. Do not ignore errors.
