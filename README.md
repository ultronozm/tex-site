# tex-site

Reusable machinery for publishing a repository of TeX notes as a GitHub
Pages site.

The intended downstream shape is a content repository with this repository
as an upstream parent.  The parent owns the build/conversion machinery; the
child owns notes, images, bibliographies, local preambles, and site-specific
index organization.

## What This Provides

- GitHub Actions snapshot deployment to a `deploy` branch.
- Cached TeX Live + `latexmk` PDF builds.
- Pandoc HTML conversion with Emacs postprocessing.
- `listing.json` generation from note titles, abstracts, and git dates.
- A DataTables-based `index.org` template.
- Cross-document `xr` support by carrying `.aux` files on the deploy branch.

## Downstream Setup

1. Fork or otherwise copy this repository into a notes repository.
2. Keep this repository as an `upstream` remote so machinery changes can be
   merged later.
3. Add notes as top-level `*.tex` files.
4. Put non-standalone TeX inputs such as `common.tex`, body files, and test
   scaffolds in `config.json` under `exclude`.
5. Set `site.githubRepository` in `config.json` to `OWNER/REPO` if generated
   HTML pages should link to source history.
6. Enable GitHub Pages from the `deploy` branch, root directory.

The child repository may freely customize `index.org`, `tex.css`, and
`config.json`; those are defaults, not sacred interfaces.  If you want the
lowest-friction upstream merge path, keep local customization in content files
and avoid unnecessary edits to `.github/workflows/*.yml`, `compile.sh`,
`convert.sh`, `make-index.sh`, and `tex2html.el`.

## Required Note Shape

Each standalone note should compile with:

```tex
\documentclass[reqno]{amsart}
\input{common.tex}

\begin{document}

\title{A note title}

\begin{abstract}
  A short description used by the index.
\end{abstract}

...

\end{document}
```

`common.tex` is intentionally not supplied by the parent.  Each downstream site
has its own packages, macros, bibliography conventions, and local style.

## Deployment Model

The `build` workflow reconstructs the deploy branch as a snapshot:

1. Check out `main`.
2. Carry forward existing `*.pdf`, `*.html`, `*.aux`, and `listing.json` from
   `deploy`, if the branch already exists.
3. Rebuild changed notes and their dependents.
4. Force-push `deploy` as a fresh snapshot.
5. Run `make-index` on the deploy branch to refresh `listing.json` and
   `index.html`.

The force-push is deliberate: the deploy branch is generated state, not
history.  Permanent URLs are preserved because paths are stable.

## Known Tradeoffs

- TeX Live is pinned to 2025 in the default workflow.  Downstream sites can
  unpin after testing their corpus.
- The workflow strips `tocindent` labels from carried `.aux` files before
  compiling.  This avoids an `amsart`/`xr` interaction where imported
  `tocindent` labels can break another document's sectioning commands.
- Missing external aux files are currently treated as conversion errors by
  `tex2html.el`.  A future hardening pass should degrade dangling links to
  visible `??` markers and report them in CI.
