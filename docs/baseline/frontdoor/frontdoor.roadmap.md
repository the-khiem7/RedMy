---
baseline_schema: "2.0"
pack: "frontdoor"
document: "roadmap"
status: "active"
updated: "2026-10-13"
code_ref: "uncommitted"
---

# RedMy README and illustrations - roadmap

| Phase | Status | Depends on |
|---|---|---|
| Phase 1 - Direction and signature | complete | - |
| Phase 2 - Build the figures | complete | Phase 1 |
| Phase 3 - Rewrite the README | complete | Phase 2 |
| Phase 4 - Land the changes | planned | Phase 3 |
| Phase 5 - Check the hosting site | planned | Phase 4 |

## Phase 1 - Direction and signature

- Acceptance: directions shown on the local board and one chosen by the user.
- Final evidence: round 1 (A, B, C) and round 2 (D, E, F) were shown and the user chose D; the board records the choice. The reasoning is FD-D1 and FD-D2 in hallucination.

## Phase 2 - Build the figures

- Acceptance: each figure renders in both themes without a WARN, is looked at, and has its smallest text pairs checked for contrast.
- Final evidence: render.py rendered the hero, flow, sync, generate, apply and contract items with no WARN; each was looked at in light and dark; the lowest contrast is 4.68:1. Which figures exist and why is FD-D4 in hallucination.

## Phase 3 - Rewrite the README

- Acceptance: the README follows FD-D6, every image path resolves, every anchor has its heading, and no en dash, em dash or count appears in prose.
- Final evidence: checked with a file listing and text searches; each claim was traced to a SKILL.md or a reference file by reading them. The data-contract wording follows FD-D5.

## Phase 4 - Land the changes

- Acceptance: the changes below are committed by the user; the agent does not commit.
- Uncommitted when this pack was saved: README.md and `assets/source/redmy.html` modified, `assets/redmy-contract.webp` and `assets/redmy-contract-dark.webp` new, `docs/baseline/redmy` modified and `docs/baseline/frontdoor` new.
- Verification gate: `git status` is clean after the commit.

## Phase 5 - Check the hosting site

- Acceptance: README.md opened on github.com in light and dark and at phone width, with the `<picture>` inside the `<h1>`, the dark-mode switch and the details blocks rendering, and the alt text readable with images off.
- Verification gate: observed on the site, not inferred from the markup.

## Risks

| Risk | Effect | Status |
|---|---|---|
| Fonts load from a font host at render time | an offline render bakes a fallback face into every image | open |
| The figures restate the skill steps | when a SKILL.md changes the figures go stale and nothing compares them | open |
| Each re-render of the image set adds its size again to git history | about 0.7 MB per full render; settle the design before the next one | open |
| Lint failure, a failed log and a skipped row are not drawn in the flow diagram | they are described only in the README details blocks | accepted |

## Next action

Phase 4, then Phase 5: commit the changes listed under Phase 4 when ready and push, then open README.md on github.com in light and dark and at phone width and record what was observed.
