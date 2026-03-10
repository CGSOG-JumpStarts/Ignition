# Ignition — AI-Powered Presentation Builder

Ignition is a GitHub Copilot skill that turns your agentic coding tool into a professional presentation builder. Drop your content (Markdown, Word doc, PDF, plain text, or an existing PowerPoint) into GitHub Copilot Chat and get a polished, enterprise-quality `.pptx` in minutes — complete with speaker notes, a consistent design system, and a storytelling framework tailored to your audience.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| [GitHub Copilot](https://github.com/features/copilot) subscription | Individual, Business, or Enterprise |
| [Visual Studio Code](https://code.visualstudio.com/) | Latest stable version recommended |
| [GitHub Copilot Chat extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) | Installed and signed in |
| [Node.js](https://nodejs.org/) 18+ | Required to run the PptxGenJS generation scripts |
| [Python](https://www.python.org/) 3.10+ | Required for content extraction and QA tooling |
| [LibreOffice](https://www.libreoffice.org/) | Required for PDF conversion during visual QA |

> **Agent Mode required.** Ignition uses `.github/agents/` — you need GitHub Copilot agent mode enabled. In VS Code, open Copilot Chat and switch the mode selector from **Ask** or **Edit** to **Agent**.

---

## Quickstart

### 1. Clone the Repository

```bash
git clone https://github.com/CGSOG-JumpStarts/Ignition.git
cd Ignition
```

### 2. Install Dependencies

```bash
# Python tools (content extraction, QA, thumbnails)
pip install "markitdown[pptx]" Pillow

# Node.js tools (slide generation)
npm install -g pptxgenjs
```

> LibreOffice is required for PDF conversion (visual QA). It is listed in Prerequisites above. Install it via your OS package manager (`brew install --cask libreoffice` on macOS, `apt install libreoffice` on Ubuntu/Debian).

### 3. Open the Folder in VS Code

```bash
code .
```

VS Code will automatically detect the `.github/agents/` and `.github/prompts/` directories and make them available in Copilot Chat.

### 4. Switch Copilot Chat to Agent Mode

1. Open the **Copilot Chat** panel (`Ctrl+Alt+I` / `Cmd+Alt+I`)
2. Click the mode selector at the bottom of the chat input
3. Select **Agent**

You are now ready to use Ignition.

---

## Building a Deck with `@Deck Builder`

The recommended way to build a presentation is the `@Deck Builder` agent — a single invocation takes you from raw content to a finished `.pptx`.

### Basic Usage

In Copilot Chat (Agent mode), type:

```
@Deck Builder create a deck from @my-content.md
```

Or describe your topic directly:

```
@Deck Builder build a 10-slide executive briefing on our Q3 cloud migration results
```

Or hand it an existing file of any supported type:

```
@Deck Builder turn @proposal.docx into a sales presentation
```

**Supported input formats:** `.md`, `.txt`, `.docx`, `.pdf`, `.pptx`

### What Happens Next

`@Deck Builder` orchestrates a pipeline of specialized sub-agents automatically:

```
@Deck Builder
    └─▶ @Content Coach   (optional — if ideas need structuring first)
    └─▶ @Planner         extracts content → picks narrative framework → writes outline.md
            ⬇ ── USER APPROVAL CHECKPOINT ──
    └─▶ @Generator       reads outline.md → generates PptxGenJS script → runs it → produces .pptx
    └─▶ @QA Reviewer     content + narrative + visual quality check
```

1. **Review the outline** — the Planner will present a structured slide-by-slide outline and pause for your approval. Edit or approve it.
2. **Slides are generated** — the Generator produces a `generate-deck.js` script and runs it.
3. **QA is performed** — the QA Reviewer checks content accuracy, visual layout, and slide design.

Output is saved to:

```
output/
└── YYYY-MM-DD-your-topic/
    ├── outline.md          ← structured slide plan
    ├── generate-deck.js    ← the script that produced the deck
    └── presentation.pptx   ← your finished slides
```

---

## Available Agents

Each agent is also independently invokable for targeted tasks:

| Agent | Invoke | What It Does |
|---|---|---|
| **Deck Builder** | `@Deck Builder` | Full end-to-end pipeline: content → outline → slides → QA |
| **Content Coach** | `@Content Coach` | Brainstorm, structure, and refine ideas before building |
| **Planner** | `@Planner` | Extract a slide outline from any content file |
| **Generator** | `@Generator` | Generate slides from an existing `outline.md` |
| **QA Reviewer** | `@QA Reviewer` | Quality-check any existing `.pptx` |
| **Editor** | `@Editor` | Make surgical edits to specific slides in an existing deck |

### Example: Generate Slides from an Existing Outline

If you already have an `outline.md` ready:

```
@Generator build the deck from outline.md
```

### Example: QA an Existing Presentation

```
@QA Reviewer review @presentation.pptx
```

### Example: Edit a Single Slide

```
@Editor on slide 4 of @presentation.pptx, replace the bullet list with a 3-column card layout
```

---

## Standalone Prompts (Lightweight Alternative)

If you prefer not to use the full agent pipeline, two standalone prompts are available:

| Prompt | What It Does |
|---|---|
| `/create-presentation` | Generate a presentation from content in a single step |
| `/extract-content` | Extract a structured slide outline from a file |

These work in **Ask** or **Edit** mode as well as Agent mode, but they do not chain sub-agents or produce an `outline.md` handoff file.

**Example:**

```
/create-presentation @quarterly-report.pdf create a 12-slide executive summary
```

---

## Choosing a Storytelling Framework

When planning your deck, the Planner will choose (or ask you to choose) a narrative framework that shapes how slides are sequenced:

| Framework | Best For |
|---|---|
| **SCQA** *(default)* | Strategy alignment, technical reviews, most enterprise content |
| **Pyramid Principle** | Executive briefings where the answer comes first |
| **Hero's Journey** | Visionary pitches, product launches, innovation proposals |
| **Sparkline** | Transformation proposals, internal change management |
| **PAS** | Budget requests, urgent business cases |
| **Pitch Deck** | Funding rounds, vendor selection, competitive deals |

You can specify a framework explicitly:

```
@Deck Builder build a PAS-style deck from @security-audit.md
```

---

## Tips

- **Iterate on the outline first.** The quality of your deck is determined by the outline. Spend time reviewing and refining it before approving — the Generator follows it exactly.
- **Paste content directly.** You don't need a file. Just paste text into the chat and ask `@Deck Builder` to build from it.
- **Rerun safely.** If an output folder already exists, a time suffix is appended (e.g., `-1430`) so previous work is never overwritten. To overwrite, say "replace the existing folder."
- **Use sub-agents independently.** Once you're familiar with the pipeline, use `@Planner`, `@Generator`, and `@QA Reviewer` individually for faster iteration.
- **Check speaker notes.** Every content slide includes presenter-ready speaker notes with talking points and transition cues. Review them with `python -m markitdown output/*/presentation.pptx`.

---

## Repository Layout

```
.github/
├── agents/
│   ├── deck-builder.agent.md       ← orchestrator
│   ├── content-coach.agent.md
│   ├── planner.agent.md
│   ├── generator.agent.md
│   ├── qa-reviewer.agent.md
│   └── editor.agent.md
├── prompts/
│   ├── create-presentation.prompt.md
│   └── extract-content.prompt.md
└── skills/
    └── ignition/                   ← design system, PptxGenJS docs, editing guides
        ├── SKILL.md
        ├── pptxgenjs.md
        ├── editing.md
        └── scripts/                ← Python helper scripts
output/                             ← generated decks land here (git-ignored)
```

---

## License

The Ignition skill and agents are proprietary. All rights reserved.
