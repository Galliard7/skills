---
name: slides
description: Create stunning HTML slide decks — supports company branding via URL extraction
---

# /slides — HTML Presentation Generator

Create polished, self-contained HTML slide decks. Supports brand-aware style generation by extracting identity from a company URL.

## Usage

- `/slides` — interactive workflow: ask topic, style, and content
- `/slides <topic>` — pre-fill with topic, ask remaining questions
- `/slides --brand <url>` — extract brand identity from URL and style slides accordingly
- `/slides --brand <url> <topic>` — both brand extraction and topic

## Workflow

### Phase 1: Content Discovery

Ask the user:
1. What's the presentation about?
2. Who's the audience?
3. How many slides (suggest a range based on topic complexity)?
4. Any specific points that must be covered?

### Phase 2: Style Discovery

**Default path:** Show 3-4 style options (minimal, bold, corporate, creative) with color palette previews. Let the user pick or describe their own.

**Brand path (when `--brand` is provided):**

1. Fetch the URL and extract: colors (CSS vars, dominant hues), typography (font families, weights), visual tone (minimal/bold/corporate/playful), spacing patterns.
2. Generate a brand style spec with extracted colors mapped to `--bg-primary`, `--bg-secondary`, `--text-primary`, `--accent`, etc.
3. Show a one-slide brand preview. Ask: looks good / adjust / ignore brand and show presets instead.
4. If approved, skip to Phase 3 using the brand spec.

### Phase 3: Outline

Generate a slide-by-slide outline with titles and bullet points. Present for approval before building.

### Phase 4: Build

Generate a single self-contained HTML file:
- All CSS inline (no external dependencies except Google Fonts)
- Keyboard navigation (arrow keys, escape for overview)
- Responsive layout
- Print-friendly `@media print` styles
- Slide counter
- Smooth transitions

### Phase 5: Deliver

Save to current directory as `{topic-slug}-deck.html`. Open in browser. Offer iteration.

## Design Principles

- **No generic AI aesthetics.** Every deck should feel intentionally designed.
- **Typography-first.** Pick a distinctive heading font. Size hierarchy matters more than color.
- **Restrained color.** 2-3 colors max. Use accent sparingly for emphasis.
- **Generous whitespace.** Don't fill every pixel. Let content breathe.
- **One idea per slide.** If a slide needs scrolling, it needs splitting.
