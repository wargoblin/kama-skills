# Design Lessons — Learned From Real Generations

Read this file BEFORE reviewing any slide. These are real problems found in real generations. Each lesson = a specific failure that was missed by structural checks and visual checklist.

This file grows with each generation. Add new lessons when users report problems that QA missed.

---

## Cover / Title Slides

**Single word in oversized title = empty.** The original template "PITCH DECK" fills the space with 2 heavy words. "ПИТЧ" alone leaves 60% of the title area blank. If the content has only 1-2 short words for an oversized slot → either add a second line (subtitle) or reduce fontSize to fill the space more naturally.

**Subtitle wrapping to 3+ lines = cramped.** If subtitle container was designed for 1-2 lines of English and the Russian text takes 3+ lines, either shorten the text or expand the container width.

## Footer / Breadcrumb

**Breadcrumb "Презентация, которая убеждает" wrapping to 2 lines = FAIL.** This was consistently missed by structural checks because each word fits, but the full phrase doesn't fit in a narrow container. ALWAYS check if breadcrumb renders as ONE line. If not → expand container width to 460px+.

**Breadcrumb fix must be applied to ALL slides.** When one subagent fixes a breadcrumb on one slide, the same fix must be applied to all slides in the Global Consistency Pass. Inconsistent breadcrumbs across slides = obvious defect.

**Footer placeholder text ("Reviews / Mobile Strategy", "Product review — Month XX") = CRITICAL.** These are English placeholders from templates. In a Russian presentation they MUST be replaced with real content.

**Page numbers must be sequential 1..N.** After deleting/reordering template slides, original numbers (5, 7, 14...) remain. Must be renumbered.

## Text Overflow

**Russian text is 20-40% longer than English.** EVERY text replacement needs overflow checking. The most common failures:
- Labels inside narrow shapes ("СТРУКТУРА ПИТЧА" in a 240px blob → breaks to "СТРУКТ УРА")
- Descriptions below cards (grow down → overlap with cards/lines below)
- Footer text (wraps to 2 lines in narrow container)

**Text touching container edge = no padding.** Text inside a card/container must have ≥8px padding from all edges. Text flush against the bottom of a brown card = unprofessional.

**Text crossing dashed line separators = FAIL.** If the template has dashed lines between items and the Russian text grows beyond its zone → the text crosses the line. Fix: shorten text OR move the line down.

## Decorative Elements

**Widget/poll elements from FigJam = remove.** Templates from Figma Community often contain interactive elements (polls, votes, sliders). These must be removed (`node.remove()`) not hidden.

**Stray arrows / artifacts from previous fixes = remove.** If a previous QA iteration created a small white arrow on top of a pink arrow, or any other element that wasn't in the original template → it's an artifact. Remove it.

**"ЕЩЁ" or "→" as placeholder label on a decorative element = smells like a hack.** If the original had meaningful text and we replaced it with a single symbol because the real text didn't fit → this is NOT a fix. Either find text that fits or retemplate the slide.

## Template Fit

**Illustrations that don't match content.** E-commerce illustrations (cart, box, monitor) on a slide about P-S-I framework = wrong fit. Hide the illustrations → if slide becomes empty → retemplate.

**"Template-inherent" is NOT an excuse.** If the template doesn't work for the content, swap to a different template from the original page. "Cannot fix, template design" is only valid if you tried: expand → shorten → retemplate, and NONE worked.

**Inverted hierarchy (body fontSize > heading fontSize) on statement slides.** Some templates put body text larger than the heading. This is template design, but if it confuses the message → retemplate to a layout with clear hierarchy.

## Content Integrity

**NEVER invent content.** Fixer agents have been caught adding sentences not in the outline ("Обменяйтесь скелетами с соседней командой..."). Only text from the outline is allowed. Shortening/rephrasing is OK, inventing is NOT.

**One-word cover title looks like a placeholder.** "УБЕЖДАЕТ" alone on a huge cover → looks unfinished. The cover should clearly identify what this presentation is about.

## Consistency Across Slides

**Header positions must be consistent.** If slides 1-8 have headers at Y=40, and slide 3 has header at Y=200 after a QA fix moved it down → the fix broke deck consistency. Fixer must check all slides before applying position changes.

**Same elements should look the same across slides.** If one subagent fixed a breadcrumb on slide 3 but not slide 5, the deck looks inconsistent. Global Consistency Pass exists for this reason.

## Stale Issues That Keep Recurring

These are problems that QA consistently misses despite having rules for them:

1. **Breadcrumb wrapping** — missed because word-break check passes (each word fits, but the phrase doesn't)
2. **Text-on-line overlap** — missed when overlap check filters out "same top-level section" siblings
3. **Empty slides after hiding visuals** — missed because coverage exception says "template is sparse"
4. **Artifacts from previous fixes** — missed because structural check doesn't know what was in the original generation vs what was added by a fix
5. **Single-word oversized titles** — missed because there's no "word count too low" check for oversized slots

## Lessons From 2026-04-02 Generation (pitch template)

**2-word title slots need 2 words.** Template "PITCH DECK" = 2 words. If content has only 1 word ("ПИТЧ" or "УБЕЖДАЕТ"), the cover looks empty. Solution: use the 2-title-node structure — upper node + lower node = 2 words (e.g. "ПИТЧ" / "AI", "ИТОГИ" / "ШАГ").

**"ЕЩЁ" / "→" as labels = hack.** When real text doesn't fit a narrow container (66px), agents replace with meaningless symbols. Better: use short meaningful words ("ДА!", "ШАГ", "ЕЩЁ!") — max 3-4 chars that carry intent.

**Figma auto-renames text nodes by content.** After setting `node.characters = "ПИТЧ"`, the node name changes from "Title" to "ПИТЧ". Subsequent code that searches by `node.name.includes("Title")` will FAIL. Always search by node ID or by position/fontSize, never by name after fill.

**Group homogeneity check prevents pattern violations.** When filling cards [P, S, I, 5с], the 4th item "5с" overflows because the slot fits 1 character. Pre-fill check: if N-1 items follow a pattern (single char), the outlier must match. "5с" → "!" fixed the overflow.

**Header 4 containers are often only 180px wide at 65px font.** That's ~3.7 chars max. Words like "ОШИБКИ" (6 chars), "ЦЕЛИ" (4 chars) can break. Always expand Header 4 to 400px+ after filling with longer text.

**Breadcrumb "Презентация, которая убеждает" needs 460px+ width.** Default container is 290px — guaranteed to wrap. Apply to ALL slides in Global Consistency Pass, not just the one where it was noticed.

**Statement slides have inverted hierarchy by design.** Templates like 1:102 and 1:185 put body at 65px and heading at 45px. This is intentional — body IS the hero. Don't "fix" this unless retemplating.

## Lessons From 2026-04-08 Review

**Missing title slide = broken presentation.** Every presentation MUST start with a cover/title slide. If the outline doesn't have one explicitly, the generator must prepend it using the presentation topic. A presentation that starts with a content slide looks like a broken file.

**Text-on-text overlap inside containers.** The overlap check only compared text vs top-level sections, missing cases where two text nodes WITHIN the same card/container both grew after Russian text replacement. Example: title (grew from 1 line to 2) now overlaps the description below it inside the same card. Fix: pairwise overlap check between ALL text nodes, not just cross-section.

**Template placeholder text left unchanged.** When a text node's role wasn't recognized (no matching `newText`), the fill loop did `continue` — leaving the original English template text. "Reviews / Mobile Strategy" in a Russian presentation = obvious defect. Fix: every text node MUST be filled; if no role match, use slide title or empty string — never leave template text.

**English-only text in non-English deck = placeholder.** A text node containing only ASCII characters (length > 5, fontSize > 14) in a Russian presentation is almost certainly an unfilled placeholder. QA now flags this automatically.
