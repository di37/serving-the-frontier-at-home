# book/ — LaTeX sources

Build and editing notes. For what the book is and who it is for, see the [root README](../README.md).

## Build

```bash
tectonic main.tex        # -> main.pdf (229 pages)
```

Or, with a TeX Live installation:

```bash
latexmk -xelatex main.tex
```

Requires `tikz`, `tcolorbox`, `cleveref`, `siunitx`, `listings`, `needspace`, `adjustbox`, `placeins`, `titlesec`, `graphicx`.

A clean build emits no errors and no undefined references. It does emit four overfull hboxes (all ≤ 17 pt) and one overfull vbox; those are known and tracked. A new overfull warning means something you changed reflowed badly.

## Layout

- `main.tex` — skeleton: cover, preface, both parts, appendix, closing CTA
- `preamble.tex` — packages, colors, listing styles, TikZ styles (`hbnode`, `hbbox`, `hbgpu`, …), callout boxes (`hbnote`, `hbwarning`, `hblesson`, `hbwarstory`), and the breakable code/path macros (`\code`, `\envvar`, `\repofile` — url-based, `obeyspaces`)
- `brand.sty` — visual identity: palette, typography, cover, `\brandcta`
- `chapters/` — preface, ch01–ch17, appendix; one file each
- `figures/` — 36 generated PNGs plus their caption sidecars. The other 48 figures are inline TikZ in the chapter sources.

`main-classic.tex` and `preamble-classic.tex` are the pre-brand skeleton, kept for reference. They are not part of the build.

## Preamble invariants

These are load-bearing. Undoing any of them reintroduces a defect that a page-by-page QA pass already caught once:

- `titlesec` `nobottomtitles*` with `\bottomtitlespace 0.1\textheight` — stops headings stranding at the foot of a page.
- `\chaptertransition` carries `\needspace` and `\nopagebreak` so its rule cannot split from its text.
- `placeins` with `\FloatBarrier` before every `\section*{Takeaways}` — stops figures floating into the takeaway lists and stranding a bullet.
- `>{\raggedright\arraybackslash}p{…}` on any table column holding code, paths, or environment variables. Justified `p{}` columns shred monospace identifiers: `docker-compose.stage-c.override.yml` renders as `docker- compose . stage- c . override.yml`.

## Editing gotchas

- `\code` / `\envvar` / `\repofile` are verbatim-like. Raw `_ # $` are fine inside them, but **no backslash commands** (no `\ldots`), and they cannot be used in math mode or in TikZ node text — escape `_` manually there.
- Never define a macro named `\toks`; it is a TeX primitive. Use `\tokspersec`.
- Never use `\enlargethispage`, and never `\resizebox` — use `\adjustbox{max width=\textwidth}{…}`.
- Anchor TikZ text with `anchor=base`, not the default. A `\tiny` node measures ~17.7 pt tall in the full-book build against ~4.2 pt in a chapter-only build, so centre-anchored text shifts down and can strike a rule that looked fine in isolation.
- On a `|-` path, `pos<0.5` lands on the *vertical* leg. Nudging a label along one can move it somewhere unexpected.
- Read the page count with `pdfinfo main.pdf`. macOS Spotlight metadata (`mdls -name kMDItemNumberOfPages`) goes stale after a rebuild and once reported 190 for a 194-page file.
- A successful compile proves nothing about figures. Render the page and look at it: the most common defect in hand-written TikZ is a line or arrow drawn through its own label.

## Versioning

The cover in `main.tex` is the single source of truth for the edition number — currently **v4.0**. The Pages workflow greps it out of `main.tex` and stamps it onto the landing page beside the live page count, so bumping the cover is the only edit needed; nothing else has to be kept in sync. The build fails if that version string goes missing.

## Prose conventions

- Sentence case for all headings.
- `\Cref{…}` for every cross-reference. Never a hardcoded "Chapter 9".
- `1{,}743` for thousands separators in prose, captions, and TikZ; plain commas only inside `lstlisting` and `verbatim`.
- One term per referent: **repository** (not "repo", except the Hugging Face term "repo id"), **patch train** (not "hotfix chain"), **PR~NN** (`PR-N` inside listings, where `~` renders literally).
- Issue references are attributive without a prefix: `the \ghissue{136} chain`.
- Every performance figure carries its measurement date and lane. The 62–83 tok/s decode band is *August-published*, not "measured" — Chapter 9 records the lower September numbers.
- Compare like with like: acceptance figures pair overall against overall, never an overall band against a position-0 rate.
