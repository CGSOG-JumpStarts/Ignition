---
name: ignition
description: "Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions \"deck,\" \"slides,\" \"presentation,\" or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill."
license: Proprietary. LICENSE.txt has complete terms
---

# Ignition Skill

## Quick Reference

| Task | Recommended | Alternative |
|------|------------|-------------|
| **Full deck pipeline** (content → outline → slides → QA) | `@Deck Builder` agent | — |
| Brainstorm & refine ideas | `@Content Coach` agent | — |
| Extract slide outline | `@Planner` agent | `/extract-content` prompt |
| Generate slides from outline | `@Generator` agent | `/create-presentation` prompt |
| QA a presentation | `@QA Reviewer` agent | — |
| Edit existing slides | `@Editor` agent | — |
| Read/analyze content | `python -m markitdown presentation.pptx` | — |
| Edit or create from template | Read [editing.md](editing.md) | — |
| Create from scratch | Read [pptxgenjs.md](pptxgenjs.md) | — |

> **Agents vs Prompts:** Agents (in `.github/agents/`) are the recommended workflow — they chain together automatically, persist context via `outline.md`, and eliminate redundant questions. The `/extract-content` and `/create-presentation` prompts still work as lightweight standalone alternatives.

---

## Reading Content

```bash
# Text extraction
python -m markitdown presentation.pptx

# Visual overview
python scripts/thumbnail.py presentation.pptx

# Raw XML
python scripts/office/unpack.py presentation.pptx unpacked/
```

---

## Editing Workflow

**Read [editing.md](editing.md) for full details.**

1. Analyze template with `thumbnail.py`
2. Unpack → manipulate slides → edit content → clean → pack

---

## Creating from Scratch

**Read [pptxgenjs.md](pptxgenjs.md) for full details.**

Use when no template or reference presentation is available.

---

## Generating from Content

### Recommended: Agent Pipeline

Use `@Deck Builder` in Copilot Chat for the full end-to-end pipeline. One invocation takes you from content to finished `.pptx`:

1. **Invoke `@Deck Builder`** — paste content, provide a file path, or describe your topic
2. The orchestrator routes through specialized subagents:
   - `@Content Coach` (if raw ideas need structuring)
   - `@Planner` (extracts content → selects narrative framework → produces slide outline → writes `outline.md`)
   - **→ User approval checkpoint** (review and approve the outline)
   - `@Generator` (reads `outline.md` → generates PptxGenJS script → runs it → produces `.pptx`)
   - `@QA Reviewer` (content + narrative + visual + enterprise QA)
3. Output is saved to `output/YYYY-MM-DD-slug/` with `outline.md`, `generate-deck.js`, and `presentation.pptx`

Each subagent is also independently invokable for ad-hoc use:
- `@Planner` — just extract an outline from a file
- `@Generator` — generate slides from an existing outline
- `@QA Reviewer` — QA any existing `.pptx`
- `@Editor` — make surgical edits to individual slides
- `@Content Coach` — brainstorm and refine ideas before building a deck

### Alternative: Standalone Prompts

The `/create-presentation` and `/extract-content` prompts still work as lightweight alternatives. They are standalone and do not use the agent pipeline or `outline.md` handoff.

**Supported input formats:** Markdown (`.md`), plain text (`.txt`), Word (`.docx`), PDF (`.pdf`), or existing PowerPoint (`.pptx`).

**Rerun behavior:** If the output folder already exists, a time suffix is appended (e.g., `-1430`) to avoid overwriting previous work. To overwrite instead, explicitly ask to "replace" or use the "same folder".

### Outline Handoff Format

The `outline.md` file is the contract between the Planner and Generator agents. It persists the narrative framework choice and slide plan so the Generator never re-asks:

```markdown
---
narrative-framework: SCQA           # Chosen framework (or "standard")
source-file: content.md             # Original input file
date: 2026-02-13                    # Generation date
slug: quarterly-review              # Kebab-case slug for folder naming
output-dir: output/2026-02-13-quarterly-review  # Relative path
---

## Slide Outline: [Presentation Title]

**Narrative Style**: SCQA — Content follows a clear current-state → problem → solution structure.

### Slide 1 — Title Slide
- **Layout**: Title slide (full-bleed accent background)
- **Title**: [title]
- **Subtitle**: [subtitle]

### Slide 2 — [Section Name]
- **Narrative Phase**: Situation
- **Layout**: [layout from catalog]
- **Section Label**: SITUATION
- **Action Title**: [the "So What" headline]
- **Key Conclusion**: [≤ 25-word takeaway]
- **Content**: [key points]
- **Visual**: [suggested visual element]

... (all slides)
```

The Generator reads this file and executes it without re-extracting content or re-asking about narrative style.

---

## Storytelling Frameworks

Enterprise presentations are structured narratives, not data dumps. The storytelling framework determines how slides are sequenced to guide the audience toward a decision.

### Choosing a Framework

When creating a presentation, the agent will ask the user to choose one of three approaches:
1. **Auto-detect** (recommended) — the agent analyzes content signals and picks the best framework
2. **Choose a style** — the user selects from the frameworks below
3. **Standard** — no narrative framework; slides follow document order

### Auto-Detection Rules

If auto-detecting, apply the **first matching** rule:

| Content Signal | Framework | Rationale |
|---------------|-----------|----------|
| Executive summary, recommendation, or decision request | **Pyramid Principle** | Executives want the answer first |
| Clear "current state" + "problem" + proposed solution | **SCQA** | Natural consulting narrative |
| Strong before/after contrast, transformation language | **Sparkline** | Gap between "what is" and "what could be" drives urgency |
| Risk language, cost-of-inaction data, urgency framing | **PAS** | Agitation creates momentum for approval |
| Market trends, competitive landscape, opportunity | **Pitch Deck** | FOMO and momentum structure |
| Hero/user journey, adoption story, change management | **Hero's Journey** | Emotional arc builds buy-in |
| No clear match | **SCQA** | Default — works for most enterprise content |

### Framework Reference

#### SCQA — The Consulting Standard

Developed by Barbara Minto (McKinsey). Gets to the point quickly. **Default for enterprise content.**

| Phase | Slides | Purpose |
|-------|--------|--------|
| **Situation** | 1–2 | The context everyone agrees on ("Our system handles 10k users") |
| **Complication** | 1–2 | The "but" — what changed, what's broken, what's at risk |
| **Question** | 1 (section divider) | Frame the problem to solve |
| **Answer** | 3–6 | Proposed solution, approach, evidence, and plan |

Best for: Executive summaries, technical due diligence, strategy alignment.

#### Pyramid Principle — Answer First

Start with the conclusion. Group evidence into pillars. Ideal when the audience's time is limited.

| Phase | Slides | Purpose |
|-------|--------|--------|
| **The Lead** | 1 | State the conclusion or recommendation upfront |
| **Pillar 1** | 1–2 | First supporting argument with evidence |
| **Pillar 2** | 1–2 | Second supporting argument with evidence |
| **Pillar 3** | 1–2 | Third supporting argument with evidence |
| **Synthesis** | 1 | Tie pillars together, reinforce the lead |

Best for: Executive briefings, board-level updates, short decision meetings.

#### Hero's Journey — Narrative Arc

The "Hero" (client or user) faces a challenge and finds a guide (your solution) to achieve a new reality.

| Phase | Slides | Purpose |
|-------|--------|--------|
| **Status Quo** | 1–2 | Life before the problem |
| **Call to Adventure** | 1–2 | A disruption or market shift occurs |
| **The Mentor** | 1–2 | Your solution enters as the guide |
| **The Crossing** | 2–3 | Implementing the change (approach, phases, evidence) |
| **The Resolution** | 1–2 | The new, improved reality |

Best for: Visionary keynotes, product launches, innovation proposals.

#### Sparkline — Contrast Style

Popularized by Nancy Duarte. Alternates between "What Is" (current pain) and "What Could Be" (future potential) to create urgency.

| Phase | Slides | Purpose |
|-------|--------|--------|
| **What Is** | 1–2 | Current pain point or limitation |
| **What Could Be** | 1–2 | The better future |
| **What Is** | 1–2 | Deeper into the problem (cost, risk, friction) |
| **What Could Be** | 1–2 | Solution in action (demo, proof, case study) |
| **The Gap** | 1 | What's required to bridge from here to there |
| **The New Bliss** | 1 | The ultimate benefit |

Best for: Pitch decks, internal transformation proposals, change management.

#### PAS — Problem, Agitation, Solution

Creates urgency by highlighting the cost of inaction before introducing relief.

| Phase | Slides | Purpose |
|-------|--------|--------|
| **Problem** | 1–2 | Identify a specific, painful problem |
| **Agitation** | 2–3 | Show the cost of inaction (lost revenue, security risks, technical debt) |
| **Solution** | 3–5 | Introduce your solution, how it works, evidence, and plan |

Best for: Sales presentations, business cases for budget approval, urgent proposals.

#### Pitch Deck — Opportunity & FOMO

Builds momentum by making inaction feel like falling behind.

| Phase | Slides | Purpose |
|-------|--------|--------|
| **The Market Shift** | 1–2 | A massive, undeniable change in the world |
| **Winners vs. Losers** | 1–2 | Who's adapting and who's falling behind |
| **The Promised Land** | 1–2 | The successful future state |
| **The Magic Gift** | 2–4 | Your product/service is the superpower that gets them there |
| **Proof** | 1–2 | Evidence, metrics, case studies |

Best for: Funding rounds, vendor selection, partnership proposals.

### Quick Selection Guide

| Context | Framework | Goal |
|---------|-----------|------|
| Executive wants a decision | Pyramid Principle | Save time; start with the answer |
| Strategy change or new initiative | SCQA | Align everyone on context before the solve |
| Innovation pitch or vision | Hero's Journey | Build emotional buy-in |
| Urgent budget or resource ask | PAS | Focus on the risk of doing nothing |
| Internal transformation | Sparkline | Highlight the gap between now and the future |
| Competitive deal or funding | Pitch Deck | Create FOMO and momentum |

### Narrative & Visual Design Integration

The storytelling framework drives **slide sequencing**, while the enterprise design system drives **slide rendering**. They work together:

- **Section dividers** mark framework phase transitions (e.g., Situation → Complication)
- **Eyebrow labels** use framework phase names (e.g., "SITUATION", "PILLAR 1", "THE CROSSING")
- **Sidebar conclusions** can emphasize the narrative's key tension or resolution at each phase
- **Speaker notes** include narrative position, emotional beat, and transition cues between phases
- **Layout variety** still applies — never repeat the same layout across consecutive slides, even within the same narrative phase

---

## Design Ideas

**Don't create boring slides.** Plain bullets on a white background won't impress anyone. Use the Sogeti Template system as your default approach for professional, information-dense presentations.

### Sogeti Slidedoc: PowerPoint Theme Specification

The "Slidedoc" theme is a highly structured, editorial-style presentation system designed for readability and information density. It bridges the gap between a document and a presentation, utilizing a modular grid, clear typographic hierarchy, and a vibrant multi-accent palette to organize complex content into digestible "chapters."

#### 1. Global Theme Colors

The palette is built on a foundation of professional navies and greys, contrasted with a high-energy primary accent (Deep Sky Blue) and a secondary utility palette for data visualization and categorization.

| Slot | Usage | Hex | RGB |
| :--- | :--- | :--- | :--- |
| **Background 1** | Primary Canvas / Text Highlight | `#FFFFFF` | (255, 255, 255) |
| **Text 1** | Primary Body / Headlines | `#191919` | (25, 25, 25) |
| **Accent 1** | Primary Highlight / Navigation | `#0070C0` | (0, 112, 192) |
| **Accent 2** | Secondary Highlight / Active Hyperlink | `#00B0F0` | (0, 176, 240) |
| **Accent 3** | Dark Semantic / Background Fills | `#002060` | (0, 32, 96) |
| **Accent 4** | Secondary Blue / Phase 5 | `#0E86E0` | (14, 134, 224) |
| **Light Grey** | Sidebar fills / Muted backgrounds | `#F2F2F2` | (242, 242, 242) |
| **Card Background** | Card/panel surface fills | `#F8F9FA` | (248, 249, 250) |
| **Footer Grey** | Footer text | `#999999` | (153, 153, 153) |

#### Semantic Color Usage

Every color in the palette has a specific purpose. Using a color outside its intended role creates visual noise and weakens the design system.

| Color | Semantic Role | When to Use | When NOT to Use |
| :--- | :--- | :--- | :--- |
| **Accent 1** (`0070C0`) | Primary highlight | Insight text, hyperlinks, active navigation, chart emphasis | Background fills, body text |
| **Accent 2** (`00B0F0`) | Eyes-on emphasis | The ONE thing the audience should notice first — footer line, key metric, active TOC item | Multiple elements per slide; never use for large fills |
| **Accent 3** (`002060`) | Dark semantic | Full-bleed backgrounds for title/section/closing slides only | Cards, sidebar fills, inline highlights |
| **Accent 4** (`0E86E0`) | Secondary categorization | Eyebrow labels, phase indicators, secondary chart series | Primary headlines, body text |
| **Light Grey** (`F2F2F2`) | Muted background | Sidebar fills, alternating table rows, de-emphasized panels | Primary content areas; never for text color |
| **Card Bg** (`F8F9FA`) | Card surface | Card and panel fill color with subtle shadow | Full-slide backgrounds |

#### 2. Typography Hierarchy

The theme exclusively utilizes the **Montserrat** typeface family, emphasizing a modern, geometric aesthetic.

* **Slide Titles (H1):** Montserrat Bold | 32–40pt | Color: Text 1 (`#191919`) or White (`#FFFFFF`).
* **Subheaders / Section Headers (H2):** Montserrat Bold | 18–24pt | Color: Accent 2 (`#00B0F0`).
* **Body Text:** Montserrat Regular or Medium | 10–12pt | Line spacing: 1.2–1.3x for high-density reading.
* **Hero Metrics / Data Highlights:** Montserrat Bold | 60pt+ | Color: Primary Accent or Section-specific Color.
* **Captions / Fine Print:** Montserrat Regular | 8pt | Color: Text 1 at 60% opacity.

#### 3. Master Slide Layouts

The layout system is governed by a multi-column modular grid (up to 4 columns) optimized for "reading-instead-of-projecting."

* **Title Slides:** Full-bleed background color (Accent 1 or 3) featuring the "Edge-to-edge Texture" (waves). Text is left-aligned or centered with a clear vertical hierarchy for Author and Role.
* **Section Dividers:** High-contrast solid color fills using the Accent palette. Large H1 text positioned in the top-left or center.
* **Standard Layouts:**
  * **Weighted Left/Right:** 1/3 vs 2/3 column split. The smaller column typically holds a summary or "insight" box in Accent 2.
  * **Multi-Column Content:** 2, 3, or 4 equal columns used for comparative text or feature lists.
* **Impact Layouts:**
  * **Large Statistic:** Features a massive hero number (XX%) on the left with descriptive prose on the right.
  * **Panel Layouts:** Uses color-blocked "panels" on the left or right to separate high-level concepts from details.

#### 4. Graphic & Shape Styles

* **The "Wave" Texture:** Organic, layered wave shapes used as edge-to-edge backgrounds or corner accents to soften the rigid grid.
* **Bullet System:** Custom characters instead of standard discs. Level 1 uses a `•` (bullet, U+2022) in Accent 2; Level 2 uses an en-dash `–` (U+2013).
* **Connectors & Arrows:**
  * **Flow Arrows:** Solid block chevrons for process steps.
  * **Connectors:** 1pt lines with circular "node" terminals for network and org charts.
* **Corner Treatment:** Primarily sharp, 90-degree corners for containers and panels, maintaining a clean, professional "Swiss Design" feel.
* **Callouts:** Rectangular speech bubbles with small triangular tails, typically filled with Accent 2 or white with Accent 2 text.

#### 5. Compositional Rules

* **Whitespace:** Aggressive use of margins and gutters between columns (minimum 0.5" gutter) to prevent "wall of text" fatigue.
* **Alignment:** Strict left-alignment for all primary text blocks. Centered alignment is reserved only for hero metrics or specific Title Only slides.
* **Visual Flow:** "Z-Pattern" reading flow is encouraged. Headlines are always prominent at the top-left, followed by a secondary summary in a bright accent color, then detailed body prose.
* **Data Density:** Tables and charts are minimalist. They remove all non-essential gridlines and use color only to highlight the most important "Insight" or "Total" rows.

### For Each Slide

**Every slide needs a visual element** — image, chart, icon, or shape. Text-only slides are forgettable.

All dimensions below target **LAYOUT_16x9** (10" × 5.625") with 0.5" safe margins.

#### Enterprise Header Stack

All content slides (not title, section divider, or closing slides) must use a standardized 3-tier header that establishes narrative hierarchy before the body content:

| Layer | Position | Specification | Purpose |
| :--- | :--- | :--- | :--- |
| **Eyebrow** | y: 0.25 | 9pt, Montserrat Bold, Accent 5, UPPERCASE | Section label — tells the audience which chapter they're in |
| **Headline** | y: 0.50 | 24pt, Montserrat Bold, Text 1 | Action Title — the slide's "So What" (not just a topic label) |
| **Insight** | y: 0.90 | 12pt, Montserrat Italic, Accent 1 | Key Conclusion — the takeaway the audience should remember |

**Rules:**
- Main canvas area starts at **y: 1.5** — no body content above this line.
- All header text uses `margin: 0` to prevent PptxGenJS default padding.
- If no section label exists, skip the eyebrow. If no insight exists, skip the insight line. The headline is always present.
- Title slides, Section Dividers, and Closing slides are **exempt** from the header stack.

#### Briefing Slide (Hidden — Slide 0)

Every deck begins with a **hidden** presenter-briefing slide inserted **before** the title slide. This slide never appears in a slideshow but is visible in edit mode, giving any presenter instant context.

| Section | Content |
| :--- | :--- |
| **Target Audience** | Who this deck is for — role, seniority, org, knowledge level |
| **Overall Messaging** | The single strategic message the audience should take away |
| **Win Theme / Concept** | The differentiating idea or value proposition that ties the narrative together |

**Implementation:**

```javascript
{
  const slide = pres.addSlide();
  slide.hidden = true;                       // Invisible in slideshow mode

  slide.addText("PRESENTER BRIEFING", {
    x: 0.5, y: 0.3, w: 9, h: 0.5,
    fontSize: 24, bold: true, fontFace: "Montserrat",
    color: THEME.accent3
  });

  slide.addText([
    { text: "Target Audience\n", options: { bold: true, fontSize: 12, color: THEME.accent2, breakLine: true } },
    { text: "[audience description]\n\n", options: { fontSize: 10, color: THEME.text1, breakLine: true } },
    { text: "Overall Messaging\n", options: { bold: true, fontSize: 12, color: THEME.accent2, breakLine: true } },
    { text: "[strategic message]\n\n", options: { fontSize: 10, color: THEME.text1, breakLine: true } },
    { text: "Win Theme / Concept\n", options: { bold: true, fontSize: 12, color: THEME.accent2, breakLine: true } },
    { text: "[differentiating idea]", options: { fontSize: 10, color: THEME.text1 } }
  ], { x: 0.5, y: 1.0, w: 9, h: 4, valign: "top", fontFace: "Montserrat" });
}
```

**Rules:**
- Must be the **first** `pres.addSlide()` call (Slide 0 in the deck).
- `slide.hidden = true` is mandatory.
- No Enterprise Header Stack, no Auto-Footer, no visual elements.
- Content is derived from the source material or the outline's frontmatter.

#### Auto-Footer

Every slide except the title slide **and the hidden briefing slide** must include a footer:
- **Accent line**: thin rectangle at y: 5.30, x: 0.5, w: 9.0, h: 0.008, fill Accent 2
- **Footer text**: "© 2026 SOGETI - CONFIDENTIAL" at y: 5.32, 8pt Montserrat, color `999999`, left-aligned
- Footer must be added as the **last element** on each slide to ensure it renders on top.

#### Content Layouts

- **Two-column 50/50** — text left (x:0.5, w:4.25), illustration right (x:5.25, w:4.25). Best for text + image or text + chart pairings.
- **Weighted 1/3 + 2/3** — narrow left panel (x:0.5, w:2.8) with accent fill (Accent 2 or 3) for insight callout or summary; wide right panel (x:3.6, w:5.9) for body content. Matches the Slidedoc "Weighted Left/Right" pattern.
- **Three-column equal** — three columns (w:2.7 each, gutter:0.45) for feature comparisons, triple content blocks, or pros/cons/neutral. Start at x:0.5 with columns at x:0.5, x:3.65, x:6.8.
- **Four-column card grid** — four narrow columns (w:2.0, gutter:0.4) for phase breakdowns, feature lists, or metric categories. Cards at x:0.5, x:2.9, x:5.3, x:7.7.
- **Icon + text rows** — stacked rows with icon in a colored circle (0.45" oval, accent fill) at left, bold header + description text to its right. 3–4 rows spaced 1.0–1.2" apart vertically.

#### Impact & Data Layouts

- **Hero stat + prose** — massive number (60–72pt, Montserrat Bold, accent color) on the left third with a small label below; descriptive paragraph fills the right two-thirds. Use for single KPI or headline metric.
- **KPI dashboard row** — 3–4 stat cards in a horizontal row (y:0.8, h:1.5) each showing a large number + small label, followed by a detail area below (y:2.8) with chart or table. Cards use white fill with subtle shadow.
- **Comparison columns** — two side-by-side panels (before/after, current/proposed, option A/B). Use contrasting accent fills (Accent 1 vs Accent 4) for visual differentiation.

#### Visual & Narrative Layouts

- **Half-bleed image** — full-height image covering left or right half (w:5.0, h:5.625, x:0 or x:5.0) with content overlay on the opposite side. Add a semi-transparent rectangle (transparency: 30–50%) over the image edge if text overlaps.
- **Left color panel + right content** — solid accent rectangle (x:0, y:0, w:3.5, h:5.625) with white text for section title or key takeaway; remaining right area (x:4.0, w:5.5) for body content on white background.
- **Quote / callout slide** — large italic text (24–28pt) centered on slide with accent-colored left border bar (x:1.5, y:1.5, w:0.08, h:2.5, Accent 2 fill); attribution in caption style below. Background: white or light neutral.
- **Process / timeline flow** — 4–5 numbered circles (0.5" ovals, Accent 1 fill, white number) connected by horizontal lines (1pt, Accent 2), with step labels below each. Position at y:2.0 spanning x:0.8 to x:9.2 for full-width flow.
- **2×2 content grid** — four equal quadrants (w:4.25, h:2.2 each) arranged in a 2×2 matrix below a slide title. Each quadrant contains an icon or small image + header + short text. Add 0.3" gutter between quadrants.

#### Structural Slides

- **Section divider** — full-bleed accent background (Accent 1 or 3) with large H1 title (36–40pt, white, Montserrat Bold) positioned top-left (x:0.8, y:1.5). Optional subtitle in Accent 2 below. Include Wave texture shapes for brand identity.
- **Agenda / TOC** — left column lists numbered items (Montserrat Medium, 14pt) with the active item highlighted in Accent 2; right side shows a brief description or preview of the current section. Useful for decks with 5+ sections.

#### Data Display Patterns

- **Large stat callouts** — big numbers 60–72pt with small labels (8–10pt) below
- **Side-by-side comparison** — before/after, pros/cons, or option A/B columns
- **Timeline or process flow** — numbered steps connected by arrows or lines
- **Minimalist table** — no outer border, header row in Accent 3 with white text, alternating row fills (white / light gray `F2F2F2`), highlight "Total" or "Insight" row in Accent 2

#### Visual Polish

- Icons in small colored circles (0.4–0.5" ovals) next to section headers
- Italic accent text for key stats or taglines
- Subtle card shadows: `{ type: "outer", blur: 6, offset: 2, angle: 135, color: "000000", opacity: 0.12 }`


### Typography

**Default: Use Sogeti Template (Montserrat family).**

**Sogeti Typography (Recommended):**

| Element | Specification |
|---------|---------------|
| Slide title | Montserrat Bold/Medium, 32–44pt |
| Subheaders | Montserrat Bold, 14–18pt |
| Body text | Montserrat Regular/Light, 10–12pt |
| Hero metrics | Montserrat Bold, 48pt+ |
| Captions | Montserrat Medium, 8–9pt |



### Spacing

- 0.5" minimum margins
- 0.3-0.5" between content blocks
- Leave breathing room—don't fill every inch

### Avoid (Common Mistakes)

- **Don't repeat the same layout** — vary columns, cards, and callouts across slides
- **Don't center body text** — left-align paragraphs and lists; center only titles
- **Don't skimp on size contrast** — titles need 36pt+ to stand out from 14-16pt body
- **Don't default to blue** — pick colors that reflect the specific topic
- **Don't mix spacing randomly** — choose 0.3" or 0.5" gaps and use consistently
- **Don't style one slide and leave the rest plain** — commit fully or keep it simple throughout
- **Don't create text-only slides** — add images, icons, charts, or visual elements; avoid plain title + bullets
- **Don't forget text box padding** — when aligning lines or shapes with text edges, set `margin: 0` on the text box or offset the shape to account for padding
- **Don't use low-contrast elements** — icons AND text need strong contrast against the background; avoid light text on light backgrounds or dark text on dark backgrounds
- **NEVER use accent lines under titles** — these are a hallmark of AI-generated slides; use whitespace or background color instead
- **NEVER use colored header bars** — full-width accent rectangles at the top of content slides create a dated "two-toned" look; use navy/accent title text on white background with visual interest from cards, icons, or layout variety instead
- **Don't add accent bars on top of cards** — thin colored rectangles at the top edge of card panels add visual clutter; cards already have shadows and fill colors for differentiation
- **Don't use `rectRadius` > 0.08** — subtle rounding for cards is OK, but large rounded corners feel casual, not enterprise
- **Don't place content above y: 1.5 on body slides** — reserve the header zone for the eyebrow/headline/insight stack
- **Don't mix icon families** — pick one family per deck (`react-icons/hi2` or `react-icons/bi`) and use it consistently

---

## QA (Required)

**Assume there are problems. Your job is to find them.**

Your first render is almost never correct. Approach QA as a bug hunt, not a confirmation step. If you found zero issues on first inspection, you weren't looking hard enough.

### Content QA

```bash
python -m markitdown output.pptx
```

Check for missing content, typos, wrong order.

**Verify speaker notes are present and presenter-ready:**

The `markitdown` output includes speaker notes. Check that:
- Every content slide has notes (title/closing slides may be brief)
- Notes include talking points, not just a repeat of slide text  
- Transitions between slides are noted
- Timing cues exist for key sections
- Anticipated Q&A is flagged where relevant

**When using templates, check for leftover placeholder text:**

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|this.*(page|slide).*layout"
```

If grep returns results, fix them before declaring success.

### Visual QA

**⚠️ USE SUBAGENTS** — even for 2-3 slides. You've been staring at the code and will see what you expect, not what's there. Subagents have fresh eyes.

Convert slides to images (see [Converting to Images](#converting-to-images)), then use this prompt:

```
Visually inspect these slides. Assume there are issues — find them.

Look for:
- Overlapping elements (text through shapes, lines through words, stacked elements)
- Text overflow or cut off at edges/box boundaries
- Decorative lines positioned for single-line text but title wrapped to two lines
- Source citations or footers colliding with content above
- Elements too close (< 0.3" gaps) or cards/sections nearly touching
- Uneven gaps (large empty area in one place, cramped in another)
- Insufficient margin from slide edges (< 0.5")
- Columns or similar elements not aligned consistently
- Low-contrast text (e.g., light gray text on cream-colored background)
- Low-contrast icons (e.g., dark icons on dark backgrounds without a contrasting circle)
- Text boxes too narrow causing excessive wrapping
- Leftover placeholder content

Enterprise-specific checks:
- Header Alignment — verify eyebrow at y ≈ 0.25, headline at y ≈ 0.5, insight at y ≈ 0.9 on all content slides
- Vertical Grid Axis — verify no body content starts above y: 1.5
- Footer Presence — verify every non-title slide has the accent line and "© 2026 SOGETI - CONFIDENTIAL" text at y ≈ 5.3
- Sidebar Consistency — if a sidebar panel appears, verify main content is shifted right and no elements overlap the sidebar
- Icon Family — verify all icons are from the same family (no mixing Heroicons with BoxIcons)

For each slide, list issues or areas of concern, even if minor.

Read and analyze these images:
1. /path/to/slide-01.jpg (Expected: [brief description])
2. /path/to/slide-02.jpg (Expected: [brief description])

Report ALL issues found, including minor ones.
```

### Verification Loop

1. Generate slides → Convert to images → Inspect
2. **List issues found** (if none found, look again more critically)
3. Fix issues
4. **Re-verify affected slides** — one fix often creates another problem
5. Repeat until a full pass reveals no new issues

**Do not declare success until you've completed at least one fix-and-verify cycle.**

---

## Converting to Images

Convert presentations to individual slide images for visual inspection:

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

This creates `slide-01.jpg`, `slide-02.jpg`, etc.

To re-render specific slides after fixes:

```bash
pdftoppm -jpeg -r 150 -f N -l N output.pdf slide-fixed
```

---

## Dependencies

- `pip install "markitdown[pptx]"` - text extraction
- `pip install Pillow` - thumbnail grids
- `npm install -g pptxgenjs` - creating from scratch
- LibreOffice (`soffice`) - PDF conversion (auto-configured for sandboxed environments via `scripts/office/soffice.py`)
- Poppler (`pdftoppm`) - PDF to images
