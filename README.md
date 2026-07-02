# Infrastructure Journal

This repository contains my personal notes from working on
Linux and Oracle-based systems.

Entries are written as postmortems:
what broke, what was checked, and what fixed it.

It also includes notes on setting up, installing,
and configuring software and infrastructure components.

---

## File Naming Convention

**Format**

```
YYYY-MM-DD__area__short-description.md
```

**Example**

```
2026-02-14__ords__service-fails-after-windows-patch.md
```

This format ensures:
- chronological ordering
- easy scanning by area
- long-term maintainability

---

## Folder Structure

```
infra-journal/
└── 2026/
    ├── 2026-02-14__ords__service-fails-after-windows-patch.md
    ├── 2026-02-20__linux__high-io-wait.md
    └── 2026-03-01__weblogic__ssl-cert-expiry.md
```

Each year contains its own entries and an `INDEX.md`
for quick navigation.


# 📘 Ultimate Markdown Cheat Sheet

A comprehensive, scannable reference guide covering basic syntax, GitHub Flavored Markdown (GFM), and advanced formatting tricks. Use this as a quick reference when writing your documentation.

---

## 📋 Table of Contents
1. [Structure & Headers](#1-structure--headers)
2. [Text Formatting](#2-text-formatting)
3. [Lists & Reminders](#3-lists--reminders)
4. [Links & Visual Media](#4-links--visual-media)
5. [Blockquotes](#5-blockquotes)
6. [Code Formatting](#6-code-formatting)
7. [Tables](#7-tables-github-flavored-markdown)
8. [Advanced Tricks & Utilities](#8-advanced-tricks--utilities)

---

## 1. Structure & Headers
Use hashes (`#`) followed by a space. The number of hashes sets the heading level.

```markdown
# Heading 1 — Main Title (HTML <h1>)
## Heading 2 — Section Title (HTML <h2>)
### Heading 3 — Sub-section (HTML <h3>)
#### Heading 4 — Minor Heading (HTML <h4>)
##### Heading 5 — Sub-minor Heading (HTML <h5>)
###### Heading 6 — Smallest Heading (HTML <h6>)

--- (Horizontal rule / divider line. Can also use *** or ___)
```

---

## 2. Text Formatting
Apply typography styling by wrapping text with specific symbols.

```markdown
*Italic text* or _Italic text_             — Slanted text for emphasis
**Bold text** or __Bold text__             — Strong, heavy text
***Bold & Italic*** or ___Bold & Italic___ — Combined styling
~~Strikethrough text~~                     — Crosses a line through text (GFM)
==Highlighted text==                       — Text highlighted like a marker
```

---

## 3. Lists & Reminders
Organize points using unordered, ordered, or nested syntax. Always put a space after the prefix symbol.

### Unordered Lists (Bullet Points)
```markdown
* Item A
- Item B
+ Item C
```

### Ordered Lists (Numbered)
```markdown
1. First item
2. Second item
1. Third item (Parsers auto-sequence numbers; this will render as "3.")
```

### Nested Lists (Indented)
Indent the child line by exactly 2 or 4 spaces.
```markdown
1. Parent List Item
   * Nested Bullet Point
   * Another Nested Bullet Point
```

### Checklists / Task Lists (GFM)
```markdown
- [ ] Incomplete task (Must include the space inside the brackets)
- [x] Completed task (Filled with a lowercase "x")
```

---

## 4. Links & Visual Media
Connect your document to external web resources or embed image media.

```markdown
[Clickable Text](https://example.com) — Standard hyperlink
[Clickable Text](https://example.com "Hover Title") — Link with hover text
![Image Alt Text](https://url-to-image.jpg) — Direct image embed (Note the !)
[![Alt Text](ImageURL)](LinkURL) — Clickable image linking to a webpage
```

---

## 5. Blockquotes
Format quotes, callout items, or emphasize specific note blocks.

```markdown
> This is a standard blockquote.
>> This is a nested blockquote inside a parent quote.
```

---

## 6. Code Formatting
Format programming snippets, terminal commands, or file paths cleanly.

### Inline Code
```markdown
Wrap words inside a sentence with single backticks: `git status`
```

### Fenced Code Blocks
Wrap multi-line code with three backticks above and below. Specify the language name after the top backticks for clean syntax highlighting.

````markdown
```javascript
function greet() {
  console.log("Hello, World!");
}
```
````

---

## 7. Tables (GitHub Flavored Markdown)
Create structured grids using pipes (`|`) and hyphens (`-`). Use colons (`:`) in the separator line to control text alignment.

```markdown

| Left-Aligned Header | Center-Aligned Header | Right-Aligned Header |
| :---                | :---:                 |                 ---: |
| Row 1 Data          | Centered Text         |               \$10.00 |
| Row 2 Data          | Centered Text         |              \$150.00 |
```

---

## 8. Advanced Tricks & Utilities

### Escaping Formatting Characters
If you want to show a literal character (like an asterisk or hashtag) without triggering Markdown formatting, place a backslash (`\`) right before it.
```markdown
\# This will show up as a real hashtag, not a heading!
\* This will display as an actual asterisk, not italic text. *
```

### Line Breaks
Pressing Enter once won't start a new line in standard Markdown. To force a clean line break without starting a new paragraph, add **two spaces** at the end of a line before hitting Enter, or use an HTML break tag:
```markdown
This line has two spaces at the end.  
This line will appear directly below it.

Alternatively, use the HTML break tag.<br>
This line will also appear directly below.
```

