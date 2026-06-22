# Figmadeck v3 Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add smart QA features (content-template fit, distance preservation, oversized typography budget) and fix 13 bugs (P1-P3) in the figmadeck skill.

**Architecture:** 7 tasks grouped by file — each task modifies ONE file with all its related changes. Tasks are independent (different files) so can be parallelized.

**Tech Stack:** Markdown instruction files for figmadeck skill. Figma Plugin API concepts (mixed fonts, component instances, absoluteTransform).

**Spec:** `docs/superpowers/specs/2026-03-31-figmadeck-v3-improvements.md`

---

## File Structure

All files **already exist** — modifications only:

```
figmadeck/.claude/skills/figmadeck/references/
  figma-generation.md   (358 lines) — 10 changes: distance preservation, oversized typography,
                                      mixed-style text, component instances, clone/remove ordering,
                                      subtitle detection, footer zone %, overflow coords,
                                      prototype connections, page selection
  qa-cycle.md           (229 lines) — 4 changes: Phase E (content-template fit),
                                      placeholder detection rewrite, originalSlideNodeId handoff,
                                      oversized text QA check
  figma-learning.md     (137 lines) — 1 change: learn language detection
  design-rules.md       (473 lines) — 1 change: remove CSS/Slidev content
  auth.md               (71 lines)  — 1 change: timeout + retry
```

---

### Task 1: Update `figma-generation.md` — Step 5 improvements (mixed text, instances, oversized, distance)

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-generation.md`

This task adds 6 changes to the fill content step (Step 5) and its surrounding logic.

- [ ] **Step 1: Read the current file**

Read `figmadeck/.claude/skills/figmadeck/references/figma-generation.md` in full. Identify:
- The `applyText` function (around line 194)
- The font handling section (around line 189)
- The text overflow section (around line 241)
- The return shape (around line 260)

- [ ] **Step 2: Add oversized typography budget**

Find the `applyText` function. BEFORE the font handling logic, add the character budget calculation. Read spec "New Feature 3" (lines 146-210) for the exact code.

Insert before the font check:
- `calculateCharBudget(node)` function
- Decision logic: fontSize > 80px → calculate budget → aggressively shorten if over budget → trigger Phase E if 2x over budget
- fontSize 40-80px → normal overflow handling
- fontSize < 40px → normal overflow handling

- [ ] **Step 3: Add mixed-style text handling**

In the `applyText` function, add the `figma.mixed` check from spec P1 Fix #1 (lines 224-261). This must be the FIRST check in `applyText`:
- Check `node.fontName === figma.mixed`
- If mixed: `getStyledTextSegments` → load all fonts → set characters → restore primary font with `setRangeFontName`
- If not mixed: proceed with existing logic

- [ ] **Step 4: Add component instance handling**

BEFORE `applyText` is called in the fill loop, add the instance check from spec P1 Fix #2 (lines 276-301):
- `findParentInstance(node)` function: walk parent chain for `INSTANCE` type
- If inside instance: try `setProperties()` for exposed TEXT props
- If no matching prop: proceed with direct edit (creates local override), log it

- [ ] **Step 5: Add distance preservation**

From spec "New Feature 2" (lines 51-143). Add two new subsections:

**Before Fill: Capture Original Distances** — add a section that describes measuring gaps between all pairs of adjacent visible children before ANY text replacement. Include the full JS code from spec.

**After Fill: Check and Restore Distances** — after text replacement, re-measure same pairs. If gap < 16px or gap < originalGap * 0.5 → apply fix strategies in order:
1. Increase text container width (10% increments up to 30%)
2. Shorten text (~70% of original)
3. Reduce fontSize (min 85%)
4. Move lower element down
5. Accept reduced gap if ≥ 16px

Include the minimum gap rule: 16px minimum between originally-separate elements. Elements that were touching (0 gap) in original → preserve that.

- [ ] **Step 6: Fix overflow coords (P2 #9)**

Find the overflow check that uses `node.y + node.height > parentFrame.height`. Replace with absoluteTransform version:
```javascript
const nodeBottom = node.absoluteTransform[1][2] + node.height;
const frameBottom = parentFrame.absoluteTransform[1][2] + parentFrame.height;
if (nodeBottom > frameBottom)
```

- [ ] **Step 7: Verify all changes**

```bash
grep -c "figma.mixed\|findParentInstance\|calculateCharBudget\|originalGap\|absoluteTransform.*height\|setRangeFontName" figmadeck/.claude/skills/figmadeck/references/figma-generation.md
```
Expected: 6+ matches (one per major change)

- [ ] **Step 8: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-generation.md
git commit -m "feat(figmadeck): Step 5 improvements — mixed text, instances, oversized budget, distance"
```

---

### Task 2: Update `figma-generation.md` — Steps 2-4 fixes (page selection, clone ordering, subtitle, footer, prototype)

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-generation.md`

This task modifies Steps 2, 3, and 4.

- [ ] **Step 1: Fix page selection (P3 #22)**

Find Step 2 where it uses `figma.root.children[0]`. Replace with smart page detection from spec (lines 489-504):
- If URL contains node-id pointing to a page → use that
- Else scan all pages → select one with MOST frames matching 16:9 aspect ratio
- If ambiguous → use index 0, log warning

- [ ] **Step 2: Fix subtitle detection (P2 #6)**

Find Step 3 role detection table. Add priority 3.5 from spec (lines 385-391):
```
| 3.5 | Second-largest fontSize ≥ 28px, within top 60% of frame, fontSize < largest × 0.85 | subtitle |
```

- [ ] **Step 3: Fix footer zone percentage (P2 #7)**

Find ALL instances of hardcoded `56px`, `44-56px`, `44–56px` footer zone references. Replace with:
```
Footer zone = bottom 8% of frame height
Formula: frame.height * 0.08
```

- [ ] **Step 4: Fix clone/remove ordering (P1 #3)**

Find Step 4 (Match Outline → Template). Add the explicit ordering rule from spec (lines 316-325):
```
CRITICAL: Operation order in Step 4:
1. FIRST: clone all needed donor slides
2. THEN: remove all unused slides
3. LAST: reorder remaining slides by x position
```

- [ ] **Step 5: Add prototype connections fix (P3 #15)**

Find Step 4 after the reorder step. Add the prototype connections update from spec (lines 470-483):
- After reordering by x: remove old navigate-to actions pointing to removed slides
- Add new sequential navigation if template used it
- Log warning for complex interactions

- [ ] **Step 6: Verify**

```bash
grep -c "16:9.*aspect\|3\.5.*subtitle\|frame.height.*0\.08\|clone.*FIRST\|prototype.*connection" figmadeck/.claude/skills/figmadeck/references/figma-generation.md
```
Expected: 5 matches

- [ ] **Step 7: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-generation.md
git commit -m "feat(figmadeck): Steps 2-4 fixes — page selection, subtitle, footer %, clone ordering, prototype"
```

---

### Task 3: Update `qa-cycle.md` — Phase E + placeholder + origId + oversized check

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md`

- [ ] **Step 1: Read current file**

Read `figmadeck/.claude/skills/figmadeck/references/qa-cycle.md` in full. Identify:
- Phase A (structural check) — where to add oversized text check
- Phase B (visual comparison) — where to add origId handoff
- Phase D (score) — after which to add Phase E
- Unchanged Placeholder Text Detection section — to rewrite

- [ ] **Step 2: Add oversized text check to Phase A**

In Phase A, after the "Text Truncation" check and before "Unchanged Placeholder Text Detection", add:

```markdown
### Oversized Text Overflow

For each TEXT node with fontSize > 80px:
- Check: is the text visually contained within its bounding box?
- Check: does `node.width` contain all characters without overlap? (`node.characters.length * fontSize * 0.6 > node.width * maxLines` → overflow)
- Check: `node.absoluteTransform[1][2] + node.height > parentFrame.absoluteTransform[1][2] + parentFrame.height`?

If ANY oversized text is unreadable or overflowing → severity **CRITICAL**

**This is NOT "decorative design" — it is a fill error. Oversized decorative titles (like "PITCH DECK") are designed for 1-3 short words. Longer text MUST be shortened to fit the character budget, or the slide must be retemplated (Phase E).**
```

- [ ] **Step 3: Rewrite placeholder detection (P3 #17)**

Find the "Unchanged Placeholder Text Detection" section. Replace the hardcoded `placeholderPatterns` regex approach with template-agnostic comparison from spec (lines 441-462):

```markdown
### Unchanged Placeholder Text Detection

Compare each text node's content against the ORIGINAL template text:

1. For each TEXT node on the adapted slide, find the corresponding
   original node using the `origId` mapping from Step 2 (clone step)
2. Call `use_figma` to read the original node's `.characters` on the template page
3. If `adaptedText === originalText` AND this node has role
   title/description/subtitle/body (not footer, not decorative) →
   **CRITICAL: text was never replaced**

This is template-agnostic — works for any template, any language.
```

- [ ] **Step 4: Add origId handoff to Phase B (P1 #4)**

Find Phase B visual comparison section. Add at the top:

```markdown
**IMPORTANT:** The `originalSlideNodeId` used for comparison comes from
the clone step (Step 2) return value — the `origId` field for each slide.
This is the node ID on the ORIGINAL template page, NOT the cloned page.

slideMapping = {
  adaptedSlideId: "99:125",      // on generated page
  originalSlideNodeId: "1:42",   // on template page (Page 1)
}

Pass this mapping through the entire pipeline. Do NOT guess or
reconstruct node IDs — use exactly what the clone step returned.
```

- [ ] **Step 5: Fix footer zone % in Phase A**

Find hardcoded `44–56px` / `56px` footer zone references in Phase A. Replace with `frame.height * 0.08`.

- [ ] **Step 6: Fix overflow coords in Phase A**

Find boundary check that uses `node.y + node.height > parent.height`. Replace with absoluteTransform version.

- [ ] **Step 7: Add Phase E (content-template fit)**

After Phase D (Score + Fix) section, add the complete Phase E from spec (lines 13-48):

```markdown
## Phase E: Content-Template Fit Assessment

Runs AFTER Phase D score is computed, BEFORE declaring DONE.

### Step E1: Visual Assessment

For each slide, look at the filled screenshot and the original template screenshot. Evaluate:

1. **Does the text content match the visual style?** Playful illustrations + serious financial data = bad fit. Clean layout + wrong content type = bad fit.
2. **Is the text-to-visual ratio preserved?** Template 30% text / 70% visual → after fill 80% text / 20% visual = bad fit.
3. **Are visual elements relevant?** Shopping cart illustration + team structure content = bad fit.

If fit is **poor** → proceed to Step E2.
If fit is **good** → skip to DONE.

### Step E2: Retemplate

1. Review ALL available templates from the slide map (Step 3 analysis)
2. Select better-fitting template (text-heavy → text-only; metrics → stat; process → timeline)
3. If NO better template → keep current, log "best available"
4. Delete current filled slide → clone new template → fill → run Phases A-D on this slide only
5. Run Step E1 on the new version

### Step E3: Compare and Keep

If retemplated: compare original fill vs retemplate on structural score + visual fit. Keep the winner, delete the loser.

**Phase E adds at most 1 retemplate attempt per slide per QA iteration.** If Phase E keeps triggering across iterations, the template set is inadequate for this outline.
```

Update the DONE condition: "Score ≥ 9/10 AND Phase E passed (no poor fits remaining)"

- [ ] **Step 8: Verify**

```bash
grep -c "Phase E\|originalSlideNodeId.*origId\|template-agnostic\|fontSize.*80px.*CRITICAL\|frame.height.*0\.08\|absoluteTransform.*height" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md
```
Expected: 6+

- [ ] **Step 9: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/qa-cycle.md
git commit -m "feat(figmadeck): QA improvements — Phase E, placeholder rewrite, oversized check, origId"
```

---

### Task 4: Update `design-rules.md` — remove CSS/Slidev content (P2 #11)

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/design-rules.md`

- [ ] **Step 1: Read Sections 6 and 8**

Read `figmadeck/.claude/skills/figmadeck/references/design-rules.md`. Find:
- Section 6: Element Spacing & Overflow Prevention
- Section 8: Animation & Transitions
- Quick-Reference Checklist at the bottom

- [ ] **Step 2: Clean Section 6**

Remove CSS-specific references: `position: absolute`, `flexbox`, `display: flex`. Replace with Figma-native equivalents:
- "position: absolute" → "absolute positioning in Figma canvas (non-auto-layout frames)"
- "flexbox" → "Figma auto-layout (horizontal/vertical)"
- Keep the universal rules (min 16px gap, no overlap, footer zone)

- [ ] **Step 3: Replace Section 8**

Remove the entire current Section 8 content (v-click, CSS transform, opacity, prefers-reduced-motion, timing values). Replace with:

```markdown
## 8. Figma Prototyping

Figma presentations use Smart Animate and prototype transitions for slide navigation. These are set in the template and preserved by clone — figmadeck does not modify them.

**What figmadeck preserves:** existing prototype connections between slides (navigate-to actions, transition types, animation curves).

**What figmadeck updates after reorder:** sequential navigation connections are remapped to match the new slide order (see figma-generation.md Step 4).

**What is NOT supported:** creating new animations, modifying transition types, adding interactive overlays. These must be done manually in Figma after generation.
```

- [ ] **Step 4: Clean Quick-Reference Checklist**

Find the checklist at the bottom. Remove items mentioning: `v-click`, CSS `transform`, `opacity`, CSS animations, media queries. Replace with Figma-native items:
- "Prototype connections point to correct slide order"
- "Auto-layout constraints preserved after text fill"

- [ ] **Step 5: Verify**

```bash
grep -c "v-click\|transform.*opacity\|prefers-reduced-motion\|CSS.*animation" figmadeck/.claude/skills/figmadeck/references/design-rules.md
```
Expected: 0 (all CSS/Slidev references removed)

- [ ] **Step 6: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/design-rules.md
git commit -m "fix(figmadeck): remove CSS/Slidev content from design-rules, add Figma-native equivalents"
```

---

### Task 5: Update `auth.md` — timeout + retry (P1 #5)

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/auth.md`

- [ ] **Step 1: Read current auth.md**

Read `figmadeck/.claude/skills/figmadeck/references/auth.md`.

- [ ] **Step 2: Add timeout and retry section**

Find the "First Run Flow" section. After the flow description, add a new section from spec (lines 360-377):

```markdown
## Timeouts and Error Handling

**Navigation timeout:** All Playwright navigation calls use `{ timeout: 30000 }` (30 seconds).

**Distinguish error types:**
- **Network error**: page didn't load (timeout, DNS failure, connection refused) → NOT an auth problem
- **Auth failure**: page loaded, login form shows error message → wrong credentials
- **2FA failure**: 2FA form shows error after code submission → wrong code

**Retry policy:**
- Network errors: retry up to 3 times with 10-second pause between attempts
- Auth failures: ask for new credentials, max 3 attempts total
- 2FA failures: ask for new code, max 3 attempts

**Hard stop:** If 3 attempts all fail → error: `"Unable to authenticate after 3 attempts. Check your credentials and network connection."` Do NOT loop forever.
```

- [ ] **Step 3: Verify**

```bash
grep -c "timeout.*30000\|retry.*3\|Network error\|Auth failure\|2FA failure" figmadeck/.claude/skills/figmadeck/references/auth.md
```
Expected: 5

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/auth.md
git commit -m "fix(figmadeck): add auth timeout (30s), retry caps (3x), error type distinction"
```

---

### Task 6: Update `figma-learning.md` — language detection (P3 #12)

**Files:**
- Modify: `figmadeck/.claude/skills/figmadeck/references/figma-learning.md`

- [ ] **Step 1: Find hardcoded Russian**

Read `figmadeck/.claude/skills/figmadeck/references/figma-learning.md`. Find the FDL-2 section that says "All outlines in **Russian**" or similar.

- [ ] **Step 2: Replace with language detection**

Replace the hardcoded Russian language instruction with from spec (lines 510-525):

```markdown
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
```

- [ ] **Step 3: Verify**

```bash
grep "match the template" figmadeck/.claude/skills/figmadeck/references/figma-learning.md
```
Expected: 1 match

- [ ] **Step 4: Commit**

```bash
cd figmadeck && git add .claude/skills/figmadeck/references/figma-learning.md
git commit -m "fix(figmadeck): learn outlines match template language instead of hardcoded Russian"
```

---

### Task 7: Final verification + push

**Files:**
- All files from Tasks 1-6

- [ ] **Step 1: Verify all changes present**

```bash
# figma-generation.md: all 10 changes
grep -c "figma.mixed\|findParentInstance\|calculateCharBudget\|originalGap\|prototype.*connection\|16:9.*aspect\|3\.5.*subtitle\|frame.height.*0\.08\|clone.*FIRST\|absoluteTransform.*height" figmadeck/.claude/skills/figmadeck/references/figma-generation.md

# qa-cycle.md: Phase E + 3 fixes
grep -c "Phase E\|template-agnostic\|originalSlideNodeId\|fontSize.*80px" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md

# design-rules.md: no CSS/Slidev
grep -c "v-click\|CSS.*animation\|prefers-reduced-motion" figmadeck/.claude/skills/figmadeck/references/design-rules.md

# auth.md: timeout + retry
grep -c "timeout.*30000\|retry.*3" figmadeck/.claude/skills/figmadeck/references/auth.md

# figma-learning.md: language detection
grep -c "match the template" figmadeck/.claude/skills/figmadeck/references/figma-learning.md
```

Expected: figma-generation 10+, qa-cycle 4+, design-rules 0, auth 2+, figma-learning 1+

- [ ] **Step 2: No stale content**

```bash
# No hardcoded 56px footer zone remaining
grep -n "56px\|44-56px\|44–56px" figmadeck/.claude/skills/figmadeck/references/figma-generation.md figmadeck/.claude/skills/figmadeck/references/qa-cycle.md

# No "All outlines in Russian" remaining
grep "All outlines in.*Russian" figmadeck/.claude/skills/figmadeck/references/figma-learning.md

# No placeholderPatterns hardcoded regex list remaining
grep "placeholderPatterns\|Reviews.*Mobile Strategy\|Product review" figmadeck/.claude/skills/figmadeck/references/qa-cycle.md
```

Expected: all empty

- [ ] **Step 3: Push**

```bash
git push
```
