---
name: coursebook
description: "Write, audit and publish bilingual (FR/EN) course and workshop manuals as a single self-contained HTML file served from GitHub Pages — the format used by 3DVIZ and AI_GEN_NARRATIVES. Use when authoring a new course manual, workshop handbook, technical documentation for students, or a companion reference page; when adding chapters, figures, diagrams, exercises, FAQ entries or a glossary to an existing one; when checking FR/EN consistency; when building a downloadable starter project folder; or when setting up the repo, the audit script and Pages deployment. Covers the language-switching contract, client-side search, cumulative year bands, figure slots for screenshots taken later, print stylesheet, deterministic ZIP archives, and the writing conventions that keep a technical manual true as tools change."
---

# Coursebook

Produce a course manual that is **one HTML file**: bilingual, searchable, printable, and readable from a USB key with no network. No build step, no dependencies, no framework. A student gets a link; a teacher gets a file that still opens in five years.

This skill is the generalised form of two working manuals — `3DVIZ` (SketchUp/Rhino → Twinmotion) and `AI_GEN_NARRATIVES` (generative film production). Follow it and a third course will match them without re-deciding anything.

## Why one file

Every requirement of a course document points the same way. Students read on phones at 1 a.m. and on lab machines with no admin rights. A workshop happens in a room with broken wifi. A colleague wants to fork it. A manual has to outlive the toolchain that produced it. A static-site generator satisfies none of these: it needs installing, breaks on version drift, and dies when its theme is abandoned.

So: **one `.html` file, inline CSS, inline JS, no external requests.** The cost is that the file gets large (600 KB is normal) and you edit it with scripts rather than by hand. That trade is correct — take it and stop revisiting it.

## Workflow

Work in this order. Each phase assumes the previous one is settled; going backwards is what produces incoherent manuals.

### 1 · Settle the frame before writing a word

Answer these five, in writing, before touching HTML. They are cheap now and expensive later.

| Question | Why it decides the structure |
|---|---|
| **What can the reader *do* at the end?** | A capability, not a topic list. "Produce a 60-second film" beats "understand AI video". Everything that does not serve it gets cut. |
| **What is the one artefact they produce?** | A manual organised around a single artefact carried through every chapter teaches; one organised by topic informs. Name the artefact and its scale (20 shots × 3 s; one interior at 3 quality levels). |
| **Who reads it — one audience or several?** | Several means cumulative bands (see below). Decide now: retrofitting bands means re-tagging every chapter. |
| **What do they hand in?** | Deliverables drive chapter order. If the method chapters produce nothing assessable, students skip them. |
| **What here will be false in a year?** | Model names, versions, prices, VRAM figures, UI paths. These get **concentrated** in one chapter plus a companion page — never sprinkled. |

Then write the chapter list as a flat numbered spine following **the real order of production**, not the order of a syllabus. Group into parts afterwards.

### 2 · Stand up the repo

```
course-repo/
├── index.html                     meta-refresh redirect to the manual
├── manual-name.html               the manual — the deliverable
├── companion.html                 optional: fast-moving reference
├── README.md                      the repo front page: links, chapter table
├── CAPTURE-GUIDE.md               maintainer-only: figures still to take
├── images/                        screenshots, tracked
├── templates/                     optional: the downloadable starter folder
│   ├── PROJECT_2026_project-name/
│   ├── project-folder.zip
│   └── build-zip.py
├── tools/audit.py                 the consistency audit
└── .github/workflows/audit.yml    CI
```

Copy `assets/skeleton.html` to the manual name, `assets/audit.py` to `tools/`, `assets/audit.yml` to `.github/workflows/`. Set `CONFIG` at the top of `audit.py`. Run it immediately — it should pass on the empty skeleton, and it does. That gives you a green baseline, so the first failure is a real one.

Name the manual so it is recognisable in a downloads folder years later: `recits_generatifs_manuel-v1.html`, not `index.html`.

GitHub Pages: `main` / root. No Jekyll needed; no `.nojekyll` needed unless you add `_`-prefixed paths.

### 3 · Write chapter by chapter, both languages together

**Never write the whole manual in one language and translate later.** It produces two documents that drift, and the audit will make you pay for it in one go. Write a chapter in FR, then its EN twin, then move on. The two blocks must carry the same section numbers, the same identifiers, the same table rows.

Because the file is large, **edit it with a Python script, not by hand**:

```python
import io
p = "manual.html"
t = io.open(p, encoding="utf-8").read()
old = "…unique anchor text…"
assert t.count(old) == 1, "count=%d" % t.count(old)      # never skip this
t = t.replace(old, old + "\n" + new_block)
io.open(p, "w", encoding="utf-8", newline="").write(t)    # newline="" preserves line endings
```

The `assert t.count(old) == 1` is not optional. A silent multi-replace in a 600 KB file is very hard to find later.

For a long chapter, write the FR and EN blocks as separate scratch files and splice them in — do not try to compose 400 lines inside a shell heredoc.

See **`references/components.md`** for the copy-paste HTML of every component (callouts, tables, exercises, copyable blocks, figure slots, FAQ entries, glossary rows) and when each is the right one.

### 4 · Draw the diagrams as inline SVG

A course manual needs 5–10 real diagrams. Inline hand-written SVG, bilingual via `.fr-only` / `.en-only` on `<text>` elements, styled with the `dg-*` classes so they follow the theme. Never an image of a diagram: it cannot be searched, cannot be themed, and blurs when printed.

See **`references/diagrams.md`**.

### 5 · Leave figure slots for screenshots you cannot take yet

Screenshots of your own tools come later, often much later. Do not wait for them and do not leave gaps: use the figure-slot component. Unfilled slots are **hidden from readers** and revealed by appending `?figures` to the URL — that is the maintainer's to-do list, rendered. Each slot says precisely what the capture must show, so someone else can take it. Keep the list in `CAPTURE-GUIDE.md` too.

### 6 · Audit, then publish

```bash
python tools/audit.py           # 7 passes, exit code = failing passes
```

Run it after every editing session, not just before publishing. See **`references/auditing.md`** for what each pass catches and how to read a failure.

Then commit, push, and check the live URL. See **`references/publishing.md`** for Pages setup, the downloadable-archive determinism traps, and the mistakes that have actually cost time on these repos.

## The three mechanisms you must not break

Everything in the skeleton works because of a small class contract. Adding content is free; changing these is not.

**1 · Language.** Two CSS rules do the whole job:

```css
body.lang-fr .en-only, body.lang-en .fr-only{display:none !important}
```

Wrap each chapter's content in `<div class="fr-only">` and `<div class="en-only">`. For a short inline pair, use two `<span>`s. A glossary is the one place both languages appear in the same row — that is the point of a glossary.

**2 · Search.** The index is built from the DOM at load, and rebuilt whenever language or level changes. It only indexes what is **actually on screen** — a hit that scrolls to hidden content is worse than no hit. That is `indexable() = inActiveLang() && !bandHidden()`. If you add a new kind of indexable block, add it to `buildIndex()` and keep the filter.

**3 · Cumulative bands.** For a manual serving several years or levels, tag content with the **lowest** band that needs it:

- untagged — everyone
- `data-level="p"` — practitioner and up
- `data-level="r"` — research only

A first-year sees the core pipeline; a fifth-year sees everything; **nothing is unlearned between levels.** Two bands is the maximum — a third is a second manual wearing a disguise. Every tagged block needs a twin in the other language, and the audit checks the count.

Delete the level buttons and the band CSS if the course has one audience. Do not leave a filter that filters nothing: it is the single most common bug in this format, and it looks like it works.

See **`references/architecture.md`** for the full contract, including the figure system, the print stylesheet, and the `localStorage` keys.

## Writing conventions

The format is half of it. These conventions are what make a technical manual still true next year. Full version in **`references/voice.md`**; the load-bearing ones:

- **Separate the perishable from the durable.** Model names, versions, UI paths and prices go in one clearly dated chapter and a companion page. Methods, principles and failure modes go everywhere else. Say so explicitly in the preamble: *"where the manual and your installation disagree, believe the software."*
- **Organise reference material by symptom, not by feature.** Students arrive with "the character's coat keeps changing colour", not "consistency". A Symptom → Cause → Fix → Chapter index is the most-used page in these manuals.
- **Every exercise ends with an observable criterion.** "Ready to move on when: a 3 s clip renders in under 10 minutes" — not "when it looks good".
- **Name the trap.** For each stage, the mistake everyone makes and what it costs. This is the highest-value paragraph in most chapters.
- **Show the return path.** In any pipeline diagram, mark that a fault found downstream was *made* upstream. It changes how students read the whole document.
- **Verify every fact and every link before shipping.** Version numbers, licence terms, hardware requirements, and each external URL with an actual HTTP check. A manual that confidently states a wrong VRAM figure loses the reader's trust for everything else. If you cannot verify it, write what you do know and mark it dated.
- **Credit the tools and their authors by name and link.** Non-negotiable when you ship someone's software to students.
- **Keep the student's view and the maintainer's view separate.** If teachers run server components students never touch, the manual documents only what students can reach; the rest goes in the maintainer guide. Mixing them makes students try to install a server.
- Write for someone competent and rushed. No throat-clearing, no "in today's fast-moving world", no exclamation marks. Say the thing.

## Companion pages

Split a page out of the manual when material either **turns over faster** than the manual or **wants to be open in a second tab** while working. Prompt libraries, tool references, model tables, machine lists, checklists. Same visual language, no sidebar, no bands, and a live row filter instead of a search index — a companion page is short enough to filter.

Copy `assets/companion.html`. Cross-link in **both** directions: the manual's sidebar and preamble point at it, its topbar and footer point back. Register it in `CONFIG["pages"]` so the audit checks it too.

## Downloadable starter folder

If the course has students producing a project, ship the folder pre-started rather than describing it. A tree in a manual gets mistyped; a ZIP does not.

Track empty directories with `.gitkeep`, exclude them from the archive, and build the ZIP with `assets/build-zip.py` — which is deterministic, so CI can tell "the template changed and nobody rebuilt the archive" from "the archive was rebuilt on a different day". Getting that determinism right is fiddly and the traps are documented in **`references/publishing.md`**. Read it before debugging a mismatch.

Name the archive after the folder it contains. A `PROJECT_2026_x/` tree inside `starter.zip` is a small thing that confuses every student once.

Name identifiers in **English** — folders, files, CSV columns, status values — even for a French-language course. Students read English fine, and English identifiers survive being pasted into tools, scripts and forums. Prose stays in the reader's language. The audit's sweep pass enforces this once you list the old names.

## Reference files

Load these as needed rather than up front.

| File | Read it when |
|---|---|
| `references/architecture.md` | Touching the CSS/JS contract, adding an indexable block, debugging the filter or the figure system |
| `references/components.md` | Writing content — the copy-paste HTML for every block, and which to use |
| `references/diagrams.md` | Drawing an SVG diagram |
| `references/content-model.md` | Deciding chapter order, deliverables, bands, exercises, the symptom index |
| `references/voice.md` | Writing prose, or reviewing someone's draft |
| `references/auditing.md` | A pass fails, or you want to add a check |
| `references/publishing.md` | Setting up the repo, Pages, CI, the ZIP, or hitting a determinism mismatch |

## Assets

| File | Use |
|---|---|
| `assets/skeleton.html` | The manual. Working: language, search, bands, theme, print, figures. Fill it in; do not restructure it. |
| `assets/companion.html` | A companion page, with the live row filter. |
| `assets/audit.py` | The 7-pass audit. Set `CONFIG` at the top. Passes on the empty skeleton, and catches injected faults. |
| `assets/build-zip.py` | Deterministic ZIP of the starter folder. |
| `assets/audit.yml` | The CI workflow. Copy to `.github/workflows/`. |

## Before you call it done

- `python tools/audit.py` → 7/7
- Both editions read end to end, with no leftover text from the other language
- Every external link HTTP-checked
- Level filter set to the lowest band: the manual still makes sense, nothing dangles
- Print preview: sensible page breaks, no cut tables, FAQ answers visible
- Opened from `file://` with the network off
- Read on a phone width
- `?figures` reviewed, and every slot listed in `CAPTURE-GUIDE.md`
- README numbers (word count, file size, chapter count) match reality — these go stale silently after every reframe
