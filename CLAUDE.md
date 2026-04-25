# Claude Instructions for this Wiki

## arXiv Links
Always convert arXiv IDs to clickable markdown links. Applies to both front matter and body text:

`arXiv:NNNN.NNNNN` → `[arXiv:NNNN.NNNNN](https://arxiv.org/abs/NNNN.NNNNN)`

## Inline LaTeX with Underscores
GitHub's markdown parser can interpret underscores inside `$...$` as italic syntax, breaking math rendering. Use the `$`...`$` backtick form for any inline formula containing underscores:

`$\tilde{z}_t^{(i)}$` → `$`\tilde{z}_t^{(i)}`$`
