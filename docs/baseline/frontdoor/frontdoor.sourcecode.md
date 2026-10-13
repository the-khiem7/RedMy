---
baseline_schema: "2.0"
pack: "frontdoor"
document: "sourcecode"
status: "active"
updated: "2026-10-13"
code_ref: "uncommitted"
---

# RedMy README and illustrations - source layout

## Files

| Path | Role |
|---|---|
| `assets/source/redmy.html` | the items and the inline drawing script |
| `assets/source/shared.css` | tokens for both themes and the text classes |
| `assets/source/shared.js` | sets `html[data-theme]` from `?theme=` |
| `assets/<id>.webp`, `assets/<id>-dark.webp` | rendered output, light and dark |

## Items

| Item id | Height px | README section |
|---|---|---|
| `redmy-hero` | 440 | hero |
| `redmy-flow` | 560 | How a cycle runs |
| `redmy-sync` | 440 | Meet the skills |
| `redmy-generate` | 440 | Meet the skills |
| `redmy-apply` | 440 | Meet the skills |
| `redmy-contract` | 640 | Reference, data contract |

## Conventions

- Each item is 1200 CSS px wide at zoom 2, so 2400 px in the image; theming only through `html[data-theme="dark"]`; `--cardline` is the border colour of the finished image.
- Cream objects (envelopes, slips, cards) keep a cream fill and a dark outline in both themes through `--paper`, `--env` and `--line`; things standing on the ground use `--ink`.
- A dashed outline means a draft (`Not logged`) and a solid outline means real.
- A group that runs off the edge carries `data-bleed` so render.py does not warn; the hero wall is one.
- The drawing helpers in the inline script are `cubby`, `env`, `slip`, `chip`, `nameplate` and `arrow`; the hero wall uses a fixed seed so every render is identical.

## Render

- The command is in the header comment of `redmy.html`: `uvx --from playwright --with pillow python -I <frontdoor-illustrate skill>/scripts/render.py assets/source/redmy.html --root assets --out assets`; `--items <id>` renders one item.
- It needs the network for the fonts and an installed Edge or Chrome, and no admin rights.
