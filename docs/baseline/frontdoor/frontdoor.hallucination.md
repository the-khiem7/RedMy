---
baseline_schema: "2.0"
pack: "frontdoor"
document: "hallucination"
status: "active"
updated: "2026-10-13"
code_ref: "uncommitted"
---

# RedMy README and illustrations - decisions and open questions

## Entry index

| ID | About | Status | Related |
|---|---|---|---|
| FD-D1 | Choice of the picture signature of the repo | closed | FD-D2, FD-D4 |
| FD-D2 | Typefaces used in the images | closed | FD-D1 |
| FD-D3 | README facts that came from the user | closed | - |
| FD-D4 | Which pictures the README carries | closed | FD-D1, FD-D5 |
| FD-D5 | How the README presents the data contract | closed | RM-D1, FD-D4 |
| FD-D6 | Voice and claim policy of the README | closed | FD-D7 |
| FD-D7 | First rewrite rejected as pictures on old text | closed - lesson | FD-D6 |

## Closed decisions

### FD-D1: Visual direction is the sorting office

- Decision: The repo's pictures use the sorting-office signature (commits as letters sorted into the pigeonhole of their issue, nothing franked until approval); its six-line note sits at the top of `assets/source/redmy.html`.
- Why: chosen by the user on 2026-10-13 from round 2 of the local direction board, with no reason stated - the board noted it as the only direction that draws the commit-to-issue sorting that redmy-generate does.
- What breaks if ignored: a figure drawn in another look makes the pictures read as unrelated, and a figure that would still fit another project fails the swap test.
- Rejected: A press proof (jargon unfamiliar to some readers), B split tally stick (silent on the approval gate, and its notches read as a count), C signal-box track (an arbitrary train metaphor, and sync does not sit on one line), E kitchen ticket rail (reads as shop receipts), F off-registration proof (the misregistered title costs legibility); each reason is the risk written on the board, not a reason the user gave.

### FD-D2: Bowlby One for titles, Courier Prime for all other text

- Decision: Bowlby One sets the title and the brass nameplates and Courier Prime sets all other text, in regular weight except the first two lines of the hero tagline in bold.
- Why: Bowlby One won a three-title specimen, and on 2026-10-13 the user asked to change the body text from Special Elite to Courier Prime.
- What breaks if ignored: fonts load from a font host at render time, so rendering offline puts a fallback face into every image; render.py prints a WARN for a font that did not load.
- Rejected: Special Elite (the first body face, replaced at the user's request), Bevan and Ultra (title specimen faces, not chosen in the specimen), Cutive Mono (specimen body face, not chosen in the specimen).

### FD-D3: Install owner, README language and agent wording

- Decision: Install commands use `the-khiem7/RedMy`, the README is in English, and it names no agent product, only "an agent that supports skills" with a Redmine MCP server that provides the tools the skills call.
- Why: the git remote is `the-khiem7/RedMy` and the user chose it over the `<owner>` placeholder; the existing docs are English; the skills name Redmine tools but no agent.
- What breaks if ignored: a placeholder owner leaves the commands uncopyable, and naming an agent promises an environment the repo does not state.
- Rejected: keeping `<owner>` (a reader cannot copy it), naming Claude Code and the Redmine plugin (the repo does not name them), a Vietnamese README (the existing docs are English).

### FD-D4: Figure set: hero, flow, one scene per skill, contract matrix

- Decision: The README carries the hero, the flow diagram (`redmy-flow`), one scene for each skill and the contract matrix (`redmy-contract`), each as a light and a dark WebP with an alt text that narrates it.
- Why: the user asked for illustrations in place of the first plan's flow diagram, then asked for the flow diagram back beside the scenes, then asked for the data-contract section as an illustration; the flow and the matrix hold exact relations a scene would blur.
- What breaks if ignored: text inside an image cannot be searched, so each alt text, and for the matrix the table kept in a details block, are the only text forms.
- Rejected: a before-and-after row figure (`redmy-row`, planned and never built, because the apply scene already shows `Not logged` replaced by a worklog link), and scenes without the flow diagram (the first plan, reversed when the user asked for the flow diagram).

### FD-D5: Data-contract section: wording, matrix and table

- Decision: The README says only what the skills do (the one file a skill creates is the `history/` file), shows who touches which file as a matrix image, and keeps the full table with its "what it holds" column inside a details block.
- Why: RM-D1 in the pack `redmy` disproves the old claim that the skills create the data files on first run, and the table carries column names the image only summarizes.
- What breaks if ignored: a README claim the skills do not back sends a reader into an empty project expecting a bootstrap, and removing the table leaves no copyable text form of the column names.
- Rejected: keeping the sentence "or let the skills create it on first run" (the skill steps never did that), and replacing the table with the image alone (loses the text form).

### FD-D6: README copy follows a pitch and claims only what the skill text states

- Decision: The README opens with a one-sentence promise and a short pitch, then the one rule (nothing is logged until you approve), the loop, the skills, install and requirements, with reference material last; every claim traces to a SKILL.md or a reference file and none cites a metric or a comparison.
- Why: the user asked for a brand-pitching feel and on 2026-10-13 confirmed both the audience sentence (for anyone who logs hours against assigned Redmine issues and works in git repositories, inferred from the skills) and the voice.
- What breaks if ignored: a claim the repo cannot back is read by a visitor as fact.
- Rejected: the first rewrite that kept the original structure and added pictures (see FD-D7).

### FD-D7: Lesson: pictures added to the old README are not a redesign

- Mistake: the first rewrite kept the original README layout (skill table, data contract, install block) and attached a hero and scenes to it.
- Looked reasonable because: the old text was accurate, and the frontdoor-readme workflow says to move reference material below the visual sections instead of deleting it.
- Disproved by: the user on 2026-10-13, who said it read as pictures added to the existing README and that the original text did not suit a banner or carry a brand pitch.
- Fix: the README was rewritten around a promise, one rule and the loop, with the old reference material kept below as Reference.

## Open questions

None open.
