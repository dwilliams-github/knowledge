# Claude Instructions for this Wiki

## arXiv Links
Always convert arXiv IDs to clickable markdown links. Applies to both front matter and body text:

`arXiv:NNNN.NNNNN` → `[arXiv:NNNN.NNNNN](https://arxiv.org/abs/NNNN.NNNNN)`

## Index Organization

`index.md` has three kinds of sections:

- **`## Foundations`** — ML/generative-model concepts with no single publication date (surveys, foundational math).
- **`## Algorithms`** — algorithm entries (graph theory, data structures, etc.). These are concept-based, not paper-based.
- **`## YYYY` year sections** — paper-centric entries, sorted chronologically within each year by publication date.

When adding an ML/paper entry, place it in the correct year section based on the primary paper's publication date, not the date the wiki entry was created.

## Algorithm Entries

Algorithm entries live in `topics/algorithms/` (flat — no subtopic subfolders). They are concept-based rather than paper-based, so they appear in `## Algorithms` in `index.md`.

Frontmatter for algorithm entries omits `date` (no meaningful publication date):

```markdown
---
title: <entry title>
topic: algorithms
tags: [tag1, tag2]
---
```

## Images

Images live in the top-level `images/` directory. VS Code is configured to paste clipboard images there automatically (`.vscode/settings.json`, gitignored).

**Workflow when a user pastes an image:**
1. VS Code saves it as `images/image.png` (or `image-1.png`, etc.)
2. VS Code inserts `![alt text](image.png)` or similar into the markdown — the path will be wrong
3. Rename the file to something descriptive: `images/<entry-name>-<description>.png`
4. Update the markdown link to the correct relative path from the entry file (e.g., `../../images/<name>.png` for files in `topics/<topic>/`)
5. Commit both the image and the updated markdown together

If the user asks to be reminded of this procedure, describe the steps above.

## Inline LaTeX with Underscores
GitHub's markdown parser can interpret underscores inside `$...$` as italic syntax, breaking math rendering. Use the `$`...`$` backtick form for any inline formula containing underscores:

`$\tilde{z}_t^{(i)}$` → `$`\tilde{z}_t^{(i)}`$`
