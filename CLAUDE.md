<!-- generated 2026-10-30 by wpg-kit cartographer from c0f4f48 -->
# corl-2026-waypointgen-anon - FROZEN anonymous CoRL 2026 review artifact (WaypointGen)

## What this is
- A public, anonymized snapshot of the WaypointGen CoRL 2026 paper and supplementary material: LaTeX sources, PDFs and
  Markdown renderings for v1.11 and the v1.18 submitted version.
- HEAD `c0f4f48` 2026-06-14 "v1.18 final submitted versions for CoRL: paper PDF, supp PDF+TeX, MDs, abstract. Matches what
  was submitted in supp. Anonymized." Shallow clone; only HEAD is visible.
- Status: FROZEN (artifact-frozen). It is not a working manuscript repo.

## Rules for any session in this repo
1. Do not edit, rename or delete any submitted file (everything under `v1.11/`, `v1.18_submitted/`, and the root copies).
2. Do not add identifying information of any kind: no names, usernames, emails, affiliations, labs, machine names, IPs,
   absolute paths, account or repository names, links to non-anonymous resources.
3. This repo is public. If text is ever copied out of these files, re-check it for identifying strings first.
4. Do not add build tooling; the sources are not meant to be rebuilt here.

## Layout
- `v1.11/` - v1.11 paper (`.tex`, `.pdf` 13 pp, `.md`) and supplement (`.tex`, `.pdf` 2 pp, `.md`).
- `v1.18_submitted/` - v1.18 as submitted: paper (`.pdf` 13 pp, `.md`; no `.tex`), supplement (`.tex`, `.pdf` 7 pp, `.md`),
  final abstract (`.md`).
- Root: 6 files byte-identical to the 6 files in `v1.11/`.

## Gotchas (verified)
- The `.tex` files are not self-contained: the paper uses `\usepackage{corl_2026}`, `\bibliography{references}` and
  `figures/...`, none of which are included; the v1.18 supplement `\input`s three table files that are not included.
- Version numbering is not chronological: v1.18 PDFs were created 2026-05-30, v1.11 PDFs 2026-06-14.
- See `.claude/memory/repo-state.md`.
