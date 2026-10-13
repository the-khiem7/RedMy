---
baseline_schema: "2.0"
pack: "frontdoor"
document: "introduction"
status: "active"
updated: "2026-10-13"
code_ref: "uncommitted"
---

# RedMy README and illustrations - introduction

## Scope

The README of the RedMy repository and the pictures it carries: the sorting-office signature, the figures, their HTML sources and the way they are rendered. Out of scope: what the skills do, and the data-contract gap, which the pack `redmy` tracks (RM-D1, RM-Q1 to RM-Q3 there). From that pack only the README's data-contract wording is in scope here (FD-D5).

## Current truth

| Claim | Basis |
|---|---|
| README.md runs in this order: hero and promise, pitch, How a cycle runs, One rule, Meet the skills, Get started, Requirements, Reference | observed: README.md |
| `assets/` holds 12 WebP files, a light and a dark file for each figure, 706,928 bytes in total, the largest `redmy-hero.webp` at 96,850 bytes | observed: file listing |
| Sources are `assets/source/redmy.html` (the items and the drawing script), `shared.css` (tokens) and `shared.js` (theme switch); the layout is in sourcecode | observed |
| The signature is the six-line note at the top of `redmy.html` | decided: FD-D1 |
| Every figure was looked at in both themes and the smallest text pairs pass the contrast check, the lowest at 4.68:1 | observed: contrast.py output |
| The README rendered on github.com in light and dark and at phone width | unverified: Phase 5 in roadmap |
| Working tree versus HEAD 37d69c3: README.md and `assets/source/redmy.html` modified, `assets/redmy-contract.webp` and its dark file untracked | observed: git status |

## Target

A visitor reads the README as the front door of the repo: a promise, one rule, the loop, the skills, then install. Each figure carries a relation or a scene that text would say slowly, every claim traces to the skills, and the figures regenerate from one command.

## Constraints

- Text uses the ASCII hyphen, never an en or em dash, and a paragraph is one line.
- No skill or file counts in README prose or in the images; a count in the old prose was removed so the figures cannot go stale.
- Install and run commands stay as text and never sit inside a details block.
- Sample values in images are fictional: `#101` to `#106`, `a1b2c3d`, `e4f5a6b`.
- The sources sit in `assets/source/`; the repo has no linter, docs site or Docker context that covers it.
- The data-contract wording follows FD-D5 and RM-D1.
