# v1.18_submitted/ - v1.18 paper and supplement as submitted (SNAPSHOT, do not edit)

Scope: 6 tracked files; last commit touching it: c0f4f48 2026-06-14 (shallow clone).
Status: SNAPSHOT (frozen; the HEAD commit message says these match what was submitted).

## Key files
- `CoRL_2026_WayPoint_Gen_SUBMITTED_v1.18.pdf` (13 pp, created 2026-05-30) and `.md` rendering. No paper `.tex` is included.
- `waypointgen_supplementary_SUBMITTED_v1.18.{tex,pdf,md}` - 7-page supplement (`\documentclass{article}`, no bibliography).
- `FINAL_ABSTRACT_v1.18_20260530T120017Z.md` - final abstract text.

## Gotchas (verified)
- The supplement `.tex` `\input`s three table files (lines 22, 31, 42) that are not part of this artifact, so it cannot be rebuilt.
- S.1 of the supplement reports a different constraint enumeration (713) than the main paper's Table 3 (579); both are stated there.
