# repo-state: corl-2026-waypointgen-anon (generated 2026-10-30T00:00Z by wpg-kit cartographer)

## HEAD
- `c0f4f48` 2026-06-14 "v1.18 final submitted versions for CoRL: paper PDF, supp PDF+TeX, MDs, abstract. Matches what was
  submitted in supp. Anonymized." Branch `main`. Shallow clone (depth 1).

## Status
FROZEN public anonymous review artifact. 18 tracked files; no build files, no code, no `.gitignore`.

## Contents
- `v1.11/` (6): paper `.tex`/`.pdf` (13 pp, 2026-06-14)/`.md`; supplement `.tex`/`.pdf` (2 pp)/`.md`.
- `v1.18_submitted/` (6): paper `.pdf` (13 pp, 2026-05-30)/`.md`; supplement `.tex`/`.pdf` (7 pp)/`.md`; final abstract `.md`.
- Root (6): byte-identical duplicates of the `v1.11/` files.

## Live vs dead
- Nothing is built here. All files are frozen deliverables.

## Open markers
- No TODO/FIXME/`\iffalse` relevant to editing; the sources are not to be edited.

## Integrity-relevant facts (verified)
- v1.11 paper `.tex`: `\documentclass{article}` + `\usepackage{corl_2026}` (:12-13), `\bibliography{references}` (:495),
  figures under `figures/`; none of these inputs are in the artifact.
- v1.18 supplement `.tex`: `\input`s three table files (:22, :31, :42) that are not in the artifact.
- Version numbers are not chronological (v1.18 PDFs 2026-05-30; v1.11 PDFs 2026-06-14). The v1.11 `.tex` header (:3) says
  "v1.11 final CoRL 2026 submission (June 2026)"; the HEAD commit says v1.18 matches what was submitted.
- v1.18 supplement S.1 reports 422/713 = 59.2% compositional re-score with a different constraint enumeration (713) than the
  paper's Table 3 (579), as stated in that section.

## Anonymity
- Do not add identifying information to this repo (see root CLAUDE.md rules). Before quoting any frozen text elsewhere,
  re-check it for identifying strings.
