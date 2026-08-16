# coursebook

A Claude Code skill for writing **bilingual course and workshop manuals as a single self-contained HTML file**, audited for consistency and served from GitHub Pages.

One file. Two languages. No build step, no dependencies, no framework. A student gets a link; a teacher gets a file that still opens in five years, from a USB key, with the wifi down.

**Built from two working manuals:**

| | |
|---|---|
| [Récits génératifs](https://urbandronedesign.github.io/AI_GEN_NARRATIVES/) | producing a short film with generative AI — 18 chapters, ~68 000 words FR+EN, 595 KB |
| [3DVIZ](https://urbandronedesign.github.io/3DVIZ/) | SketchUp & Rhino → Twinmotion |

---

## Why a single file

Every requirement of a course document points the same way. Students read on phones at 1 a.m. and on lab machines with no admin rights. A workshop happens in a room with broken wifi. A colleague wants to fork it. A manual has to outlive the toolchain that produced it.

A static-site generator satisfies none of these: it needs installing, breaks on version drift, and dies when its theme is abandoned. So — one `.html` file, inline CSS, inline JS, no external requests.

The cost is that the file gets large and you edit it with scripts rather than by hand. That trade is correct.

## What you get

- **Bilingual FR/EN** — one toggle, remembered between visits, two CSS rules doing the whole job
- **Client-side search** — built from the DOM, indexing only what is actually on screen
- **Cumulative level bands** — one manual serving first to fifth year, where nothing has to be unlearned between levels
- **Figure slots** — placeholders for screenshots you cannot take yet, hidden from readers, revealed with `?figures` as your to-do list
- **Print stylesheet** — sensible page breaks, FAQ answers forced open, never a black PDF from dark mode
- **A 7-pass audit** — the pass that matters catches a table row added in French and forgotten in English, which no reader of either edition can see
- **Deterministic starter archives** — so CI can tell "the template changed and nobody rebuilt the zip" from "the zip was rebuilt on a different day"

## Install

```bash
git clone https://github.com/urbandronedesign/coursebook.git ~/.claude/skills/coursebook
```

That is it — Claude Code discovers skills in `~/.claude/skills/`. Confirm with `/coursebook`, or just ask:

> Use the coursebook skill to start a manual for my motion-design workshop.

For one project only, clone into `.claude/skills/` inside the project instead. See the [skills documentation](https://docs.claude.com/en/docs/claude-code/skills).

## Quick start

```bash
mkdir my-course && cd my-course && mkdir tools .github/workflows -p
cp ~/.claude/skills/coursebook/assets/skeleton.html manual.html
cp ~/.claude/skills/coursebook/assets/audit.py      tools/audit.py
cp ~/.claude/skills/coursebook/assets/audit.yml     .github/workflows/audit.yml
python tools/audit.py          # passes on the empty skeleton — a green baseline
```

The skeleton arrives working: language toggle, search, bands, theme, print, figure slots. Fill it in; do not restructure it.

## What is in the repo

```
SKILL.md                      the workflow: 6 phases, the 3 mechanisms, the conventions
assets/skeleton.html          the manual — working language/search/bands/theme/print/figures
assets/companion.html         a companion page, with a live row filter
assets/audit.py               the 7-pass audit, driven by a CONFIG block at the top
assets/build-zip.py           deterministic ZIP of the downloadable starter folder
assets/audit.yml              the CI workflow
references/architecture.md    the class contract — what you must not break
references/components.md      copy-paste HTML for every block, and when each is wrong
references/diagrams.md        inline SVG: the dg-* classes, bilingual labels, what to draw
references/content-model.md   chapter order, deliverables, bands, exercises, symptom index
references/voice.md           how these manuals are written
references/auditing.md        what each pass catches, how to read a failure
references/publishing.md      repo, Pages, archive determinism, mistakes that cost time
```

Read `SKILL.md` first. The reference files are meant to be opened when the task calls for them, not up front.

## The audit

```bash
python tools/audit.py            # full report
python tools/audit.py --quiet    # failures only
```

Exit code is the number of failing passes, so CI needs no wrapper. Standard library only, a couple of seconds.

| | Pass | Catches |
|---|---|---|
| 1 | HTML validity | one unclosed `</div>` swallowing three sections into a language wrapper |
| 2 | Links | a dangling anchor, or a page link to a file that no longer exists |
| 3 | Fact parity | a section renumbered, a path renamed or a cross-reference added in one edition only |
| 4 | Structural parity | a table row, callout or list item present in one language and not the other |
| 5 | Identifiers | a renamed identifier creeping back — scoped so ordinary prose never false-positives |
| 6 | Cross-file | the manual quoting a value that no longer matches the file a student downloads |
| 7 | Level bands | a band that hides different content in each edition |

A bilingual 600 KB file cannot be proofread by reading. This is what makes the format viable: it turns "did I remember to update the English version" from a hope into an exit code.

## Design principles it enforces

- **Separate the perishable from the durable.** Versions, model names and menu paths get concentrated in one dated chapter plus a companion page. Methods go everywhere else. *Where the manual and your installation disagree, believe the software.*
- **Organise reference material by symptom, not by feature.** Students arrive with "the character's coat keeps changing colour", not "consistency".
- **Every exercise ends with an observable criterion.** "A 3 s clip renders in under 10 minutes" — not "when it looks good".
- **Name the trap.** The mistake everyone makes, and what it costs in hours.
- **Verify every fact and every link.** One wrong VRAM figure costs the reader's trust in every other number.
- **Credit the tools and their authors, by name and link.**

## Licence

MIT — see [LICENSE](LICENSE). Fork it, adapt it, use it for your own courses.

If you build a manual with it, a link back is welcome but not required.

---

Zaki Jawhari · [L'École de Design Nantes Atlantique](https://www.lecolededesign.com/)
