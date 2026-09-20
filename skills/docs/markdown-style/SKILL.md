---
name: markdown-style
description: 'Use when writing or editing Markdown documentation (*.md files). Covers GFM formatting, content structure, headings, lists, code blocks, links, images, tables, and front matter.'
author: hugobatista
---

## Content Rules
- **Headings**: H2/H3 only — no H1 (auto-generated). Hierarchical order; recommend restructuring at H4+
- **H1 emoji**: When writing a project README (where H1 is the title), append a descriptive emoji icon after the title text — e.g. `# Project Name 🚀`
- **Lists**: `-` for bullets, `1.` for numbered. Two-space indent for nesting
- **Code blocks**: Fenced with triple backticks; always specify language
- **Links**: `[text](URL)` — descriptive text, valid URL
- **Images**: `![alt text](URL)` — always include alt text
- **Tables**: `|`-delimited with aligned headers
- **Line length**: Max 400 chars; soft-break long paragraphs at ~80 chars
- **Whitespace**: Blank lines between sections; no excessive whitespace

## Front Matter
When front matter is required, include relevant metadata fields. Common fields:

```yaml
title:
description:
author: hugobatista
date:
tags:
```

## Validation
Run validation tools to verify front matter completeness, content rules, and formatting compliance.
