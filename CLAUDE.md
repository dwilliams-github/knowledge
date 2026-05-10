# Claude Instructions for this Wiki

## arXiv Links
Always convert arXiv IDs to clickable markdown links. Applies to both front matter and body text:

`arXiv:NNNN.NNNNN` → `[arXiv:NNNN.NNNNN](https://arxiv.org/abs/NNNN.NNNNN)`

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
