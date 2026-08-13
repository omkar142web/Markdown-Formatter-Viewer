# Markdown Formatter & Viewer

A single-file, zero-build Markdown → HTML documentation viewer with live rendering, a smart auto-centering table of contents, syntax highlighting, and a rich set of GFM and extended Markdown features.

**Open `mdFormatter.html` in any modern browser — no server, build step, or install required.**

## Features

### Core workflow
- **Upload** `.md` / `.markdown` files via the file picker or drag & drop
- **Live preview** rendered with [Marked](https://marked.js.org/) and sanitized with [DOMPurify](https://github.com/cure53/DOMPurify)
- **Embedded sample document** for a quick tour
- **Dedicated document scroll container** with reading progress bar and back-to-top button

### Table of contents
- Auto-generated from `h1`–`h6` headings with a nested list matching heading levels
- **Scroll-spy** tracks the heading currently in view
- **Auto-centering**: the active TOC item is scrolled toward the vertical center of the sidebar (clamped to its scroll boundaries, with tolerance to avoid jitter)
- Two independent scroll operations: clicking a TOC entry scrolls only the document, while active-item changes scroll only the TOC
- Mobile drawer with a floating toggle button; opening it reveals the current section
- Duplicate headings get unique slugs; attribute lists (`{#custom-id}`) are honored

### Markdown rendering
- Standard Markdown + GFM: tables, task lists, strikethrough, autolinks, footnotes
- **GitHub-style alerts** (`[!NOTE]`, `[!TIP]`, `[!IMPORTANT]`, `[!WARNING]`, `[!CAUTION]`) as styled callouts
- **YAML / TOML / JSON front matter** shown as a labeled, mono-spaced block
- **Math / LaTeX** (`$...$` inline, `$$...$$` display) rendered on demand with [KaTeX](https://katex.org/)
- **Mermaid diagrams** rendered on demand from CDN
- **GitHub-style emoji shortcodes** (`:rocket:` → 🚀)
- **`==highlight==`** inline mark extension
- Table cell alignment support and horizontally scrollable table wrappers
- External links open in a new tab with `rel="noopener noreferrer"`

### Code & UX
- Syntax highlighting via [Highlight.js](https://highlightjs.org/) with theme-aware light/dark styles
- Copy button on every code block
- Interactive task-list checkboxes (viewer-only; the original file is never modified)
- Word count, heading count, code-block count, and estimated reading time
- Dark / light theme (persisted, keyboard shortcut `Ctrl+Shift+D`)
- Fullscreen mode
- Toast notifications with graceful error handling (broken images, failed optional CDNs, etc.)

### Export
- Download the rendered document as a **standalone HTML file**
- Includes syntax highlighting, KaTeX and Mermaid support when the document uses them — needs an internet connection for those CDN assets
- Excludes toolbar, TOC, and other app-only UI

## Keyboard shortcuts

| Shortcut | Action |
| --- | --- |
| `Ctrl+O` | Open a Markdown file |
| `Ctrl+Shift+D` | Toggle dark / light theme |
| `Esc` | Close the mobile TOC drawer |

## Getting started

1. Clone the repository.
2. Open `mdFormatter.html` in a browser — or serve the directory with any static server:
   ```bash
   python -m http.server 8000
   ```
3. Drag a Markdown file onto the drop zone or click **Choose file**.

## Test documents

- **`Markdown Formatter Stress Test.md`** — a comprehensive document exercising headings, tables, code (incl. Bash), blockquotes, links, images, task lists, footnotes, raw HTML, collapsibles, math, Mermaid, emoji, escaping, and front matter.
- **`git-guide.md`** — a realistic long document for scroll-spy / TOC testing.