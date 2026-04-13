# Building the Paper

This project currently contains a LaTeX manuscript draft, but the local machine does not yet have a TeX toolchain installed.

## Current environment status

The following commands were checked and were not found:

- `latexmk`
- `pdflatex`
- `bibtex`
- `biber`

## Recommended build path later

Once a TeX toolchain is installed, the likely build command will be:

```bash
latexmk -pdf main.tex
```

If the manuscript later migrates to a different template or bibliography backend, this document should be updated accordingly.

## Draft-first workflow

Until a toolchain is installed, the safest workflow is:

1. Keep section content modular under `sections/`.
2. Maintain source tracking in `references/` and `notes/`.
3. Add BibTeX entries incrementally as literature review findings are confirmed.
4. Delay template-specific formatting cleanup until the venue is clearer.
