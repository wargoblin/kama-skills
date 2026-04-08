# QA Cycle — Per-Slide Sequential

Process slides ONE AT A TIME. Check → fix → confirm → next slide.

## References

Read BOTH before reviewing:
- `references/design-rules.md` — universal design rules (spacing, hierarchy, contrast, density)
- `references/design-lessons.md` — **real failures from past generations** (breadcrumb wrapping, oversized titles, stray arrows, text overflow patterns). These are specific problems that QA consistently misses.

## Critical Rules

1. **NEVER add content not in the outline.** Only use text from the outline. You MAY shorten/rephrase to fit, but NEVER invent new sentences, add new facts, or expand beyond what the outline provides.
2. **NEVER skip the structural check.** Every slide gets a `use_figma` call BEFORE screenshots.
3. **Every fix gets a screenshot.** Fix → screenshot → verify. If problem remains → fix again.
4. **Fix priority: expand first, shorten last.** Expand container width → move elements → shorten text → reduce font → retemplate.
5. **Overlap with ANY element is CRITICAL.** Text overlapping lines, shapes, separators, icons — all unacceptable. Move the decorative elements OR shorten the text.
6. **Expand width guardrail:** Never expand beyond `frame.width - node.x - 70px`.
7. **Coverage exception:** If original template coverage ≤ 0.30, do not penalize generated slide for low coverage (template is naturally sparse).

---

## Execution: One Subagent Per Slide (parallel)

**Launch a SEPARATE subagent for EACH slide.** Subagents run in parallel. Each subagent:
- Receives: slideId, origSlideId, fileKey, pageId, outline content for this slide
- Reads: this qa-cycle.md + design-rules.md
- Performs: STEP 1-5 for its ONE slide only
- Prints: the full YES/NO checklist for its slide
- Fixes issues until clean or max 3 iterations
- Returns: CLEAN or REMAINING_ISSUES
- Dies when done

```
Launch subagents as FOREGROUND CONCURRENT agents (NOT background):

  Batch 1: 4 Agent tool calls in ONE message → slides 1-4 (parallel foreground)
  Wait for all 4 to return
  Batch 2: 4 Agent tool calls in ONE message → slides 5-8 (parallel foreground)
  Wait for all 4 to return
  Batch 3: remaining slides in ONE message (parallel foreground)
  Wait for all to return

Collect all results → final score
```

**IMPORTANT: Do NOT use `run_in_background: true`.** Background agents may hang on MCP calls. Launch all agents in a batch as regular foreground agents in a single message — they execute concurrently without background mode.

**Timeout:** If a subagent doesn't return within 5 minutes, the main agent should note it as "timed out" and retry that slide individually after the batch completes.

**Subagent prompt template:**
```
You are checking AND FIXING one slide in a Figma presentation.
Slide: [slideId] on page [pageId] in file [fileKey]
Original template slide: [origSlideId] on page "0:1"

Read these files FIRST:
- references/qa-cycle.md
- references/design-rules.md

Then execute:
1. STEP 1: Run use_figma structural check on this slide
2. STEP 2: Take screenshot, print ALL 19 YES/NO answers
3. STEP 3: If ANY answer is NO — YOU MUST FIX IT via use_figma.
   Do NOT skip fixes. Do NOT mark as "template design" unless
   the ORIGINAL template has the exact same issue.
   Fix priority: expand width → move elements → shorten text.
   After each fix: take new screenshot to verify.
4. STEP 4: Re-run checklist after fixes. Max 3 fix iterations.

YOUR JOB IS TO FIX, NOT JUST REPORT.
If you find a problem and don't fix it, you have failed your task.
Only report REMAINING_ISSUES if you tried 3 times and couldn't fix.

Each slide is yours alone — no conflicts with other agents.
```

### Per-Slide Steps (each subagent follows these)

```
STEP 1: Structural check (use_figma)
STEP 2: Screenshot + visual checklist (MUST print all 22 YES/NO answers)
STEP 3: Fix issues (if any)
STEP 4: Re-check until clean (max 3 iterations)
STEP 5: Content-template fit (Phase E)
```

---

## STEP 1: Structural Check

**Split into 2 lightweight `use_figma` calls** to avoid Plugin API hangs. Do NOT combine into one massive call.

### Call 1: Linear checks (O(n) — fast)

Coverage, boundary, word-break, oversized, inner padding, group homogeneity.

**PERFORMANCE RULES (prevent Plugin API hangs):**
- Add `figma.skipInvisibleInstanceChildren = true` at the START of every use_figma call
- Use `frame.children.filter()` (immediate children only) instead of `frame.findAll()` where possible
- Do NOT return large data (textInfo, textSummary, origChars arrays). Return only checks + issues.
- Keep return payload under 15KB to avoid 20KB use_figma response limit.
- **SEQUENTIAL MCP CALLS ONLY:** Send ONE tool call at a time. Wait for response before sending the next. Do NOT send use_figma + use_figma + get_screenshot in parallel — Figma Plugin API is single-threaded and will deadlock. Order: Call 1 → wait → Call 2 → wait → screenshot → wait.

```js
figma.skipInvisibleInstanceChildren = true; // CRITICAL: hundreds of times faster
await figma.setCurrentPageAsync(generatedPage);
const frame = figma.getNodeById(slideId);
const fY = frame.absoluteTransform[1][2], fH = frame.height, fW = frame.width;
const issues = [];

// Coverage — use findAllWithCriteria for speed, or filter immediate children
let ca = 0;
for (const n of frame.findAll(n => n.visible)) {
  if (["TEXT","FRAME","RECTANGLE","GROUP"].includes(n.type)) ca += n.width * n.height;
}
const contentCoverage = Math.min(1.0, ca / (fW * fH));

// Text checks (linear)
const textNodes = frame.findAll(n => n.type === "TEXT" && n.visible);
let boundaryBreaches=0, wordBreaks=0, oversizedOverflows=0, innerPaddingFails=0;

for (const t of textNodes) {
  const ty = t.absoluteTransform[1][2], fs = typeof t.fontSize==='number' ? t.fontSize : 16;
  if (ty + t.height > fY + fH + 1) { boundaryBreaches++; issues.push({type:"CRITICAL",desc:`Boundary: "${t.characters.substring(0,20)}"`}); }
  const cpl = Math.floor(t.width / (fs * 0.6));
  if (cpl > 0) { for (const w of t.characters.split(/\s+/)) { if (w.length > cpl) { wordBreaks++; issues.push({type:"CRITICAL",desc:`Word-break: "${w}"`,nodeId:t.id}); break; } } }
  if (fs > 80 && ty + t.height > fY + fH) { oversizedOverflows++; }
  if (t.parent && t.parent.type === "FRAME" && t.parent.id !== frame.id) {
    const px=t.parent.absoluteTransform[0][2], py=t.parent.absoluteTransform[1][2], tx=t.absoluteTransform[0][2];
    const padL=tx-px, padR=(px+t.parent.width)-(tx+t.width), padB=(py+t.parent.height)-(ty+t.height);
    if (padL<8||padR<8||padB<4) { innerPaddingFails++; issues.push({type:"FAIL",desc:`Padding: "${t.characters.substring(0,15)}" ${Math.round(Math.min(padL,padR,padB))}px`}); }
  }
}

// Group homogeneity
const cfs = frame.children.filter(n => n.type==="FRAME" && n.visible);
const sim = cfs.filter(cf => cfs.filter(o => Math.abs(o.width-cf.width)<50 && Math.abs(o.height-cf.height)<100).length >= 3);
if (sim.length >= 3) {
  const heroes = sim.map(cf => cf.findAll(n=>n.type==="TEXT").sort((a,b)=>(b.fontSize||0)-(a.fontSize||0))[0]).filter(Boolean);
  if (heroes.length >= 3) {
    const lens = heroes.map(h=>h.characters.length).sort((a,b)=>a-b);
    const med = lens[Math.floor(lens.length/2)];
    for (const h of heroes) { if (h.characters.length > med*3) issues.push({type:"FAIL",desc:`Group: "${h.characters.substring(0,15)}" ${h.characters.length}ch, med=${med}`}); }
  }
}

// KEEP RETURN SMALL — no textInfo, no textSummary, no large arrays
return { contentCoverage: Math.round(contentCoverage*100)/100, boundaryBreaches, wordBreaks, oversizedOverflows, innerPaddingFails, issueCount: issues.length, issues: issues.slice(0,10) };
```

### Call 2: Text-vs-text overlap + text-vs-elements overlap + placeholders

**Two overlap scopes:** (A) text vs top-level sections, and (B) text vs ALL other text nodes on the same slide — including siblings within the same container.

```js
figma.skipInvisibleInstanceChildren = true; // CRITICAL
await figma.setCurrentPageAsync(generatedPage);
const frame = figma.getNodeById(slideId);
const issues = [];
let overlapCount=0, proximityViolations=0, unchangedPlaceholders=0;

const textNodes = frame.findAll(n => n.type === "TEXT" && n.visible);
const topChildren = frame.children.filter(n => n.visible);

// --- SCOPE A: text vs top-level sections (cross-section overlap) ---
for (const t of textNodes) {
  const tx=t.absoluteTransform[0][2], ty=t.absoluteTransform[1][2];
  let tTop = t; while(tTop.parent && tTop.parent.id !== frame.id) tTop = tTop.parent;

  for (const section of topChildren) {
    if (section.id === tTop.id) continue;
    const bx=section.absoluteTransform[0][2], by=section.absoluteTransform[1][2];

    if (tx < bx+section.width && tx+t.width > bx && ty < by+section.height && ty+t.height > by) {
      overlapCount++;
      issues.push({type:"CRITICAL",desc:`Overlap: "${t.characters.substring(0,15)}" ↔ ${section.type} "${section.name}"`});
    } else {
      const dx = Math.max(0, Math.max(bx-(tx+t.width), tx-(bx+section.width)));
      const dy = Math.max(0, Math.max(by-(ty+t.height), ty-(by+section.height)));
      const dist = Math.sqrt(dx*dx + dy*dy);
      if (dist > 0 && dist < 16) {
        proximityViolations++;
        issues.push({type:"FAIL",desc:`Proximity: "${t.characters.substring(0,15)}" ${Math.round(dist)}px from ${section.type}`});
      }
    }
  }
}

// --- SCOPE B: text vs text (ALL pairs, including same container) ---
// This catches the #1 defect: two text nodes in the same card/container
// that both grew after Russian text replacement and now overlap each other
for (let i = 0; i < textNodes.length; i++) {
  for (let j = i + 1; j < textNodes.length; j++) {
    const a = textNodes[i], b = textNodes[j];
    const ax=a.absoluteTransform[0][2], ay=a.absoluteTransform[1][2];
    const bx=b.absoluteTransform[0][2], by=b.absoluteTransform[1][2];

    if (ax < bx+b.width && ax+a.width > bx && ay < by+b.height && ay+a.height > by) {
      overlapCount++;
      issues.push({type:"CRITICAL",desc:`Text overlap: "${a.characters.substring(0,15)}" ↔ "${b.characters.substring(0,15)}"`});
    } else {
      const dx = Math.max(0, Math.max(bx-(ax+a.width), ax-(bx+b.width)));
      const dy = Math.max(0, Math.max(by-(ay+a.height), ay-(by+b.height)));
      const gap = Math.min(
        dx > 0 ? dx : Infinity,
        dy > 0 ? dy : Infinity
      );
      if (gap > 0 && gap < 8 && gap < Infinity) {
        proximityViolations++;
        issues.push({type:"FAIL",desc:`Text proximity: "${a.characters.substring(0,15)}" ↔ "${b.characters.substring(0,15)}" ${Math.round(gap)}px`});
      }
    }
  }
}

// --- Placeholders (compare with original template) ---
await figma.setCurrentPageAsync(templatePage);
const origFrame = figma.getNodeById(origSlideId);
if (origFrame) {
  const origTexts = origFrame.findAll(n => n.type==="TEXT");
  const origCharsSet = new Set(origTexts.map(n => n.characters));
  // Also collect original text by node position (for nodes renamed after fill)
  const origByPos = {};
  for (const ot of origTexts) {
    const key = `${Math.round(ot.absoluteTransform[0][2])},${Math.round(ot.absoluteTransform[1][2])}`;
    origByPos[key] = ot.characters;
  }

  await figma.setCurrentPageAsync(generatedPage);
  for (const t of textNodes) {
    const fs = typeof t.fontSize==='number' ? t.fontSize : 16;
    // Check 1: exact match with any original template text (fontSize > 10, length > 2)
    if (fs > 10 && t.characters.length > 2 && origCharsSet.has(t.characters)) {
      unchangedPlaceholders++;
      issues.push({type:"CRITICAL",desc:`Placeholder unchanged: "${t.characters.substring(0,30)}"`});
    }
    // Check 2: position-based match (same coords, same text = was never filled)
    const posKey = `${Math.round(t.absoluteTransform[0][2])},${Math.round(t.absoluteTransform[1][2])}`;
    if (origByPos[posKey] && origByPos[posKey] === t.characters && fs > 10 && t.characters.length > 2) {
      if (!origCharsSet.has(t.characters)) { // avoid double-counting
        unchangedPlaceholders++;
        issues.push({type:"CRITICAL",desc:`Placeholder (by position): "${t.characters.substring(0,30)}"`});
      }
    }
    // Check 3: language mismatch — English text in a non-English presentation
    // If outline is in Russian but text node contains only ASCII letters → likely placeholder
    if (fs > 14 && t.characters.length > 5 && /^[A-Za-z\s\d\-\/.,]+$/.test(t.characters)) {
      unchangedPlaceholders++;
      issues.push({type:"CRITICAL",desc:`Possible English placeholder in non-English deck: "${t.characters.substring(0,30)}"`});
    }
  }
}

return { overlapCount, proximityViolations, unchangedPlaceholders, issues };
```

**Merge results from both calls** into the `checks` object before proceeding to STEP 2.

**If either call is missing → STEP 1 was NOT complete → cannot proceed.**

---

## STEP 2: Screenshot + Visual Checklist

Take screenshot of THIS slide: `get_screenshot(adaptedSlideId, fileKey)`

Go through this checklist. Answer YES/NO for each. ANY "NO" = issue to fix in STEP 3.

### Text Fit (answer for EVERY text element on the slide)
- [ ] Does ALL text fit inside its container? No clipping, no overflow beyond edges?
- [ ] Is every word complete? No mid-word breaks, no syllable splits?
- [ ] Does footer/breadcrumb text fit on ONE line? No wrapping to 2+ lines?
- [ ] Is there NO single lonely word on the last line of any text block? (widows)
- [ ] Does every text inside a card/container have visible padding from ALL edges? (not touching any edge)

### Content Integrity
- [ ] Is ALL text content from the outline? Nothing invented or added beyond what outline provides?
- [ ] Are there NO unchanged placeholder texts from the template? (English placeholders in Russian presentation = problem)
- [ ] Are there NO text nodes still containing original template text? (check EVERY text node with fontSize > 10)
- [ ] Are there NO artifacts from previous fixes? (stray arrows, duplicate shapes, extra elements that weren't in template)
- [ ] (Slide 1 only) Is this a proper title/cover slide with the presentation topic clearly visible?

### Layout & Spacing
- [ ] Is there at least 16px gap between any two UNRELATED elements? (text not pressed against a card, icon, or shape it doesn't belong to)
- [ ] Does text NOT overlap with any lines, separators, shapes, or decorative elements?
- [ ] Is the slide NOT overly empty? (if template had illustrations that were removed, was the space filled or slide retemplated?)
- [ ] Are margins consistent with other slides in the deck?

### Typography & Hierarchy
- [ ] Is the heading clearly the LARGEST and most dominant element? (3:1 ratio vs body text)
- [ ] Are there at most 3 font sizes visible on this slide?
- [ ] Is body text left-aligned? (not center-aligned for multi-line blocks)
- [ ] Can you understand the main point within 3 seconds?

### Comparison with Original Template
- [ ] Does the slide look like it belongs to the SAME presentation as the template?
- [ ] Are colors, fonts, and visual style preserved from the template?
- [ ] If visual elements (illustrations, icons) are present — do they match the CONTENT of this slide?

**MANDATORY OUTPUT: You MUST print the checklist below for this slide with YES or NO for EACH line.** Do not skip any question. Do not summarize as "all clean". Print every single answer.

```
Slide [N] Visual Checklist:
Text Fit:
  1. All text inside containers?              [YES/NO]
  2. Every word complete, no mid-breaks?      [YES/NO]
  3. Footer/breadcrumb on one line?           [YES/NO]
  4. No widow (lonely last word)?             [YES/NO]
  5. Text padding from container edges?       [YES/NO]
Content:
  6. All text from outline only?              [YES/NO]
  7. No unchanged placeholders?               [YES/NO]
  8. No template text remaining (fontSize>10)?[YES/NO]
  9. No artifacts from previous fixes?        [YES/NO]
  10. (Slide 1) Proper title/cover slide?     [YES/NO/N/A]
Layout:
  11. 16px+ gap between unrelated items?      [YES/NO]
  12. No text overlap with decorations?       [YES/NO]
  13. No text-on-text overlap?                [YES/NO]
  14. Slide not overly empty?                 [YES/NO]
  15. Margins consistent?                     [YES/NO]
Typography:
  16. Heading clearly dominant (3:1)?         [YES/NO]
  17. Max 3 font sizes?                       [YES/NO]
  18. Body text left-aligned?                 [YES/NO]
  19. Main point clear in 3 seconds?          [YES/NO]
Template:
  20. Same style as original template?        [YES/NO]
  21. Colors/fonts preserved?                 [YES/NO]
  22. Visual elements match content?          [YES/NO]
Score: [X]/22
```

**If any answer is NO → that item becomes a fix target in STEP 3.**
**If the checklist is not printed → STEP 2 was NOT executed → slide is NOT checked.**

Scoring: 22/22 = CLEAN. 19-21 = minor fixes. <19 = major fixes or retemplate.

---

## STEP 3: Fix Issues

If STEP 1 or STEP 2 found problems, fix them. **One fix at a time, verify after each.**

### Fix Priority (MANDATORY ORDER)

1. **Expand container width** — make text block wider so text wraps less. Cap: `frame.width - node.x - 70px`
2. **Move decorative elements** — shift lines/shapes to make room for text
3. **Shorten text** — rephrase shorter (ONLY with outline content, NEVER invent new text)
4. **Reduce fontSize** — minimum = original × 0.85
5. **Retemplate** — swap to a different template (see Phase E below)

### Fix Rules

- **Overlap with lines/separators**: move the lines apart OR shorten text. Both are valid.
- **Inner padding < 8px**: shrink text node width slightly OR move text inward
- **Breadcrumb wrapping**: if footer/breadcrumb text wraps to 2+ lines, expand width or shorten label
- **Artifacts from previous fixes**: if you see elements you created in a previous iteration that look wrong (white arrows, extra shapes) → `node.remove()` them

After EACH fix: take new screenshot → verify the fix worked.

---

## STEP 4: Re-Check

After all fixes for this slide:
1. Run STEP 1 (structural check) again
2. Run STEP 2 (screenshot) again
3. If zero CRITICAL and zero FAIL → slide is **CLEAN** → move to next slide
4. If issues remain → STEP 3 again (max 3 fix iterations per slide)
5. After 3 iterations if not clean → log remaining issues, move to next slide

---

## STEP 5: Content-Template Fit (Phase E)

After the slide is structurally clean, evaluate fit:

### E0: Remove widgets
```js
frame.findAll(n => n.type === "WIDGET" || n.type === "SHAPE_WITH_TEXT")
  .forEach(n => n.remove());
```

### E1: Does this template fit the content?
- Do visual elements (illustrations, icons) match the content topic?
- Is the text-to-visual ratio reasonable?
- If **poor fit** → E2

### E2: Fix poor fit
1. **Hide irrelevant visuals** → check coverage → if < 40% empty → E2b
2. **Rebalance** if 30-50% coverage → expand text, center content
3. **Retemplate** (E2b) if still broken: clone different template → fill → re-run Steps 1-4

**NEVER restore irrelevant visuals. NEVER go back to original if visuals were identified as wrong.**

---

## Global Consistency Pass

After ALL per-slide subagents complete, run ONE global pass to fix inconsistencies across slides:

1. **Breadcrumbs/footers**: find the footer element that was fixed (e.g., expanded to 460px) on one slide. Apply the SAME fix to ALL slides. One `use_figma` call that iterates all slides and normalizes footer containers.
2. **Page numbers**: verify sequential 1..N. One `use_figma` call.
3. **Font consistency**: verify all slides use the same font families. If one subagent swapped a font on its slide, apply the same swap everywhere.

---

## QA Level 2: Designer Critic

After Level 1 (structural + visual checklist) is complete, run a **Designer Critic** review.

### How it works

Launch ONE subagent as a **professional presentation designer** who reviews the ENTIRE deck.

**Designer Critic prompt:**
```
You are a professional presentation designer with 10 years of experience.
You are reviewing a completed presentation in Figma.

Read references/design-lessons.md FIRST — it contains real failures
from past generations that you MUST check for.

File: [fileKey]
Generated page: [pageId] — slides: [list of slideIds]
Original template page: "0:1" — this is the BASELINE design

STEP 1: Take screenshots of the ORIGINAL template page (3-4 slides)
        to understand the design language, element positions, spacing.

STEP 2: Take screenshots of ALL generated slides.

STEP 3: For each generated slide, compare against the original template and answer:
1. Would you show this slide to a client? (YES/NO)
2. Is element positioning CONSISTENT with other slides in the deck?
   (headers at same Y, footers at same Y, margins same)
3. What is the #1 thing that looks unprofessional?
4. How specifically to fix it?

STEP 4: Review the DECK AS A WHOLE — look at all slides together:
- Are headers at consistent positions across slides?
- Are font sizes consistent for same-level elements?
- Does any slide "break" the visual rhythm of the deck?
- Would flipping through all slides feel cohesive?

List the TOP 5 most impactful fixes, prioritizing DECK-LEVEL
consistency issues over per-slide cosmetic issues.

You do NOT fix anything yourself. You only critique.
```

**Fixer prompt (receives Designer's feedback):**
```
You are applying design fixes to a Figma presentation.
You MUST use the Agent tool to run as a SUBAGENT with clean context.

File: [fileKey], Page: [pageId]
Original template page: "0:1"
Designer/Client feedback: [list of fixes/requests]

RULES:
1. "Template-inherent" is NOT an excuse to skip a fix.
   If a template element doesn't work for the content →
   RETEMPLATE that slide (clone a different template from Page 1).
   "Template design" means "this template is wrong for this content" = swap it.

2. For EVERY fix requested, you MUST try at least 3 strategies:
   a. Expand/resize elements to fit
   b. Shorten/rephrase content
   c. Retemplate (clone different slide from original page)
   Only after trying all 3 can you mark something as "cannot fix."

3. BEFORE applying ANY fix:
   - Take screenshots of ALL slides to understand current state
   - Take screenshots of the ORIGINAL template slides — BASELINE
   - For each fix: will it maintain consistency with other slides?

4. SEQUENTIAL MCP calls only. One at a time. Wait for response.

5. After EACH fix: take screenshot to verify it worked.

6. Do NOT invent new content. Shorten/rephrase only from the outline.
```

**MANDATORY CYCLE (do NOT skip any step):**

```
Iteration 1:
  1. Designer Critic subagent → reviews all slides + original template → returns top 5 fixes
  2. Fixer subagent → evaluates fixes against deck consistency + original → applies valid fixes
  3. Designer Critic subagent (AGAIN, fresh) → reviews SAME slides after fixes
     → returns: "APPROVED" or new list of fixes
  4. If NOT approved → Fixer applies new fixes

Loop until Designer says "APPROVED" or no progress detected.
**No progress** = same number of issues (or more) for 2 consecutive iterations → stop.
Do NOT skip the re-review after fixes. The whole point is the CYCLE.
```

### What Designer catches that Level 1 misses
- "This breadcrumb wraps — expand it" (visual, not structural)
- "Too much empty space after hiding illustrations" (design judgment)
- "White arrow artifact — remove it" (context awareness)
- "Text touches card edge — needs 12px padding" (design standard, not just min 8px)
- "Slide feels unbalanced — text is all in top-left corner" (composition)

---

## QA Level 3: Client Simulator

After Designer Critic is complete, run a **Client Simulator** review.

### How it works

Determine the "client persona" from the outline: look at the topic, audience, and tone.
- Startup pitch → "Tech-savvy investor, values clarity and numbers"
- Educational lecture → "Professor, values structure and depth"
- Sales presentation → "Marketing director, values visual impact"
- Corporate report → "CEO, values brevity and bottom line"

Launch ONE subagent as this persona.

**Client Simulator prompt:**
```
You are [persona description based on outline topic and audience].
You ordered a presentation on [topic]. You are seeing it for the first time.

File: [fileKey], Page: [pageId]
Take a screenshot of EACH slide.

For each slide, answer as the client:
1. Do I understand this slide immediately? (YES/NO)
2. Does it match what I asked for in my brief? (YES/NO)
3. What would I ask to change? (in the client's voice: "Make the title bigger", "I don't understand slide 3", "Where are the numbers I mentioned?")

After all slides, give your overall reaction:
- Would you approve this presentation as-is? (YES/NO)
- What are your top 3 change requests?

You do NOT fix anything. You give feedback as the client.
```

**MANDATORY CYCLE (do NOT skip any step):**

```
Iteration 1:
  1. Client Simulator subagent → reviews all slides → returns top 3 requests
  2. Fixer subagent → applies requests via use_figma
  3. Client Simulator subagent (AGAIN, fresh) → reviews SAME slides after fixes
     → returns: "APPROVED" or new requests
  4. If NOT approved → Fixer applies new requests

Iteration 2 (if needed):
  5. Client Simulator subagent (AGAIN) → final approval
  6. If still NOT approved → log remaining requests as "client preferences"

Loop until Client says "APPROVED" or no progress detected.
**No progress** = same number of requests (or more) for 2 consecutive iterations → stop.
Do NOT skip the re-review after fixes.
```

### What Client catches that Designer misses
- "I can't understand slide 3 — what's the point?" (clarity)
- "Where are the revenue numbers I mentioned in the brief?" (content completeness)
- "This doesn't feel like MY presentation — too generic" (brand fit)
- "The conclusion slide doesn't have a call to action" (business need)

---

## Full Pipeline Summary

**ALL reviewers and fixers MUST be launched via Agent tool (subagents with clean context).** Do NOT run QA in the main thread. Each agent gets a fresh context with only the relevant prompt and references.

**ALL MCP calls are SEQUENTIAL** — one at a time, wait for response, then next. No parallel use_figma/get_screenshot.

```
Generation (clone → fill)
      ↓
Level 1: Per-slide subagents (batches of 4, foreground, sequential MCP)
      ↓ Global consistency pass (main thread, one use_figma call)
      ↓
Level 2: Designer Critic cycle:
    Designer subagent → Fixer subagent → Designer subagent (re-review)
    → until approved or no progress
      ↓
Level 3: Client Simulator cycle:
    Client subagent → Fixer subagent → Client subagent (re-review)
    → until approved or no progress
      ↓
Level 4: Senior Reviewer (strictest, reads design-lessons.md)
    Senior subagent → Fixer subagent → Senior subagent (re-review)
    → until APPROVED or no progress
    → update design-lessons.md with new learnings
      ↓
DONE → Final Report
```

**"Template-inherent" is a TRIGGER for retemplating, not an excuse to skip.**

---

## QA Level 4: Senior Reviewer

After Levels 1-3 complete, launch the **strictest critic** with access to Design Memory.

Launch ONE subagent as a **Senior Design Director** who has the final sign-off.

**Senior Reviewer prompt:**
```
You are a Senior Design Director with 15 years of experience.
You give the FINAL sign-off on presentations. Your standards are
the highest — you reject work that junior designers would approve.

Read these files FIRST:
- references/design-lessons.md — REAL failures from past generations.
  Every single issue listed there MUST be checked.
- references/design-rules.md — universal design rules.

File: [fileKey], Page: [pageId]
Original template page: "0:1"

STEP 1: Take screenshots of EVERY slide (sequential MCP).

STEP 2: For EACH slide, check EVERY item in design-lessons.md:
  - Single word in oversized title?
  - Breadcrumb wrapping to 2+ lines?
  - Text crossing decorative lines?
  - Stray arrows or artifacts from previous fixes?
  - Text touching container edge without padding?
  - Placeholder text remaining?
  - "ЕЩЁ" or "→" as meaningless label?
  - Content invented (not from outline)?

STEP 3: Check the DECK AS A WHOLE:
  - Consistent header positions?
  - Consistent breadcrumbs on all slides?
  - Sequential page numbers?
  - Visual rhythm (cohesive flip-through)?

STEP 4: For each problem found, give SPECIFIC fix instruction:
  - Which node to change
  - What to change it to
  - If template doesn't work → "retemplate slide N to template X"

Return: "APPROVED" or list of SPECIFIC fixes with priority.
You do NOT fix anything. You only give the final verdict.
```

**MANDATORY CYCLE:**
```
Senior Reviewer → Fixer subagent → Senior Reviewer (re-review)
→ until APPROVED or no progress (2 consecutive iterations same issues)
```

**After Senior Reviewer approves: update `references/design-lessons.md`** if any new failure patterns were discovered during this generation.

### Final Report

```
━━━ QA Complete ━━━
Level 1: X/10 (structural + visual checklist)
Level 2: Designer — [approved / X fixes applied]
Level 3: Client — [approved / X changes applied]
Clean slides: X/N
Generated page: <pageId>
```

---

## use_figma Rules

- `await figma.setCurrentPageAsync(page)` at START of EVERY call
- `await figma.loadFontAsync(font)` BEFORE any text change
- `return` all results (not console.log, not figma.notify)
- Colors 0–1 range, not 0–255
- On error: STOP, read, fix, retry
