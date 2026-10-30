# v1.11/ - v1.11 paper and supplement (SNAPSHOT, do not edit)

Scope: 6 tracked files; last commit touching it: c0f4f48 2026-06-14 (shallow clone).
Status: SNAPSHOT (frozen). The 6 root-level files of this repo are byte-identical copies of these.

## Key files
- `CoRL_2026_WayPoint_Gen_v1.11.tex` (568 lines) - `\documentclass{article}` + `\usepackage{corl_2026}`; `\bibliography{references}`.
- `CoRL_2026_WayPoint_Gen_v1.11.pdf` (13 pp, created 2026-06-14) and `.md` rendering.
- `waypointgen_supplementary_v1.11.{tex,pdf,md}` - 2-page supplement (`article` + `corl_2026`): benchmark details.

## Gotchas (verified)
- Not buildable here: the style file, bibliography and `figures/` it references are not in this artifact.
- The `.tex` header calls v1.11 the "final" version, but the HEAD commit says v1.18 is what was submitted.
