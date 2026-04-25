# Knowledge Wiki

A personal research repository for notes, findings, and summaries gathered from agent interactions.

## Structure

```
knowledge/
├── README.md           — this file
├── index.md            — master index of all entries
├── topics/             — organized research by topic
│   └── <topic>/        — one folder per subject area
│       └── <entry>.md  — individual research notes
└── inbox/              — unsorted/raw notes to be organized
```

## Adding Entries

Each entry lives in `topics/<topic>/<entry>.md` with the following frontmatter:

```markdown
---
title: <entry title>
topic: <topic>
date: YYYY-MM-DD
tags: [tag1, tag2]
source: <agent session or URL>
---

<content>
```

## Conventions

- Keep entries atomic — one concept, finding, or summary per file.
- Link related entries with relative markdown links.
- Use `inbox/` for quick captures; move to `topics/` once organized.
- Update `index.md` when adding entries.
- Convert arXiv IDs to clickable links: `arXiv:NNNN.NNNNN` → `[arXiv:NNNN.NNNNN](https://arxiv.org/abs/NNNN.NNNNN)`. Apply in both front matter and body text.
