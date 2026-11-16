# Architecture — the class contract

Everything in `assets/skeleton.html` works because of a small set of class names and one JS file that reads them. Content is free to add. These are not.

---

## 1 · Language

```css
body.lang-fr .en-only, body.lang-en .fr-only{display:none !important}
```

`<body class="lang-fr">` at load; `setLang()` swaps the class, persists to `localStorage`, sets `document.documentElement.lang`, and rebuilds the search index.

**The language is in the URL.** `?lang=fr|en` wins over the saved choice at load
(`urlLang()`), and `setLang()` rewrites it with `history.replaceState` (`langUrl()`),
keeping any other parameter and the `#anchor`. A link copied from the address bar
therefore opens in the language it was read in — the one thing `localStorage` alone
cannot do, since it lives in the reader's browser. Order at load: URL → saved → default.
A meta-refresh `index.html` must forward it too:
`<script>location.replace('manual.html'+location.search+location.hash)</script>`
before the `<meta http-equiv="refresh">` (kept as the no-JS fallback).

**Wrapping rules**

| Situation | Use |
|---|---|
| A whole chapter | Two sibling `<div class="fr-only">` / `<div class="en-only">` inside one `<section class="chap">` |
| A heading, a label, a table cell | Two `<span>`s: `<span class="fr-only">…</span><span class="en-only">…</span>` |
| An SVG label | Two `<text>` elements with the same coordinates |
| A glossary row | Neither — both languages belong in the same row |
| A `<pre>` block whose content is identical in both | Neither. Duplicating it is how the two copies drift. |

**Two failure modes to know:**

*Content trapped in one wrapper.* Put a `<figure>` inside the FR block only and the EN edition simply does not have it. It is invisible until someone reads the English version. When a figure belongs to both, duplicate it into both blocks and suffix the second `id` with `-en` — ids must stay unique.

*Language pollution.* One French sentence left inside an `.en-only` block is silent. Check with the tag-stripped text of each edition:

```python
import io, re, sys
sys.path.insert(0, "tools")
from audit import view
t = io.open("manual.html", encoding="utf-8").read()
body = re.sub(r"<(script|style)[^>]*>.*?</\1>", " ", view(t, "en"), flags=re.S|re.I)
txt = re.sub(r"<[^>]+>", " ", body)
FR = r"\b(vous|votre|des|une|qui|dans|pour|est|avec)\b"
for line in txt.splitlines():
    s = re.sub(r"\s+", " ", line).strip()
    if len(s) > 25 and len(re.findall(FR, s, re.I)) >= 3:
        print(s[:120])
```

Three tells on one line is the threshold that avoids flagging loanwords.

---

## 2 · Search

Client-side, built from the DOM at load and rebuilt on every language or level change. No index file, nothing to keep in sync.

```
buildIndex()  walks every  section.chap  and pushes:
                the whole section        (title + first 4000 chars)
                every h3 / h4            (title only — headings are how people search)
                every details.faq        (question + answer, with a handle to open it)
                every tbody tr           (rows are content: a symptom index is a table)
                every .pr block          (copyable blocks)
```

**The invariant: the index contains only what is on screen.**

```js
function indexable(el){ return inActiveLang(el) && !bandHidden(el); }
```

A search result that scrolls to hidden content is worse than no result — the reader concludes the manual is broken. If you add a new kind of block worth finding, add it to `buildIndex()` *and* filter it with `indexable()`.

`visibleText()` clones a node and strips the inactive language before reading `textContent`, so a French query never matches English text sitting invisibly in the same section.

Scoring: title hit = 6, each body occurrence = 1, top 25. Deliberately crude and entirely adequate. Do not add fuzzy matching; do add missing content.

Clicking a FAQ result opens the `<details>` before scrolling — otherwise the reader lands on a collapsed summary and sees nothing.

---

## 3 · Cumulative level bands

For one manual serving several years or levels. Tag content with the **lowest** band that needs it.

| Tag | Visible to | Meaning |
|---|---|---|
| *(untagged)* | everyone | the core spine |
| `data-level="p"` | A3, A4·5 | practitioner depth |
| `data-level="r"` | A4·5 | research / transmission |

```css
body.lvl-a12 [data-level="p"],
body.lvl-a12 [data-level="r"],
body.lvl-a3  [data-level="r"]{display:none !important}
```

Cumulative matters pedagogically: what a first-year learns stays true in fifth year, and **nothing has to be unlearned**. A first-year does not do "part of the course" — they do the whole course, smaller.

Rules:

- **Two bands maximum.** A third means you are writing two manuals in one file.
- Wrap a band in a `<div data-level="…">` **inside** a language block, never around one. The audit's pass 7 counts FR and EN occurrences and fails if they differ, because an unpaired band hides different content in each edition.
- Tag a visible marker too: `<span class="lv-tag">A3+</span>` on the heading. A section that silently vanishes reads as a bug.
- Bands leave the search index automatically via `bandHidden()`.
- **If there is one audience, delete the band CSS, the buttons, and `setLevel`.** A filter that filters nothing is the classic bug of this format: the buttons highlight, so it looks like it works, and nobody notices for months. Verify by setting the lowest band and confirming content actually disappears.

---

## 4 · Figure slots

A figure you cannot capture yet still gets its place, its description, and its filename.

```html
<figure class="shot" id="fig-thing">
  <div class="shot-frame">
    <img src="images/thing.png" alt=""
         onload="this.closest('.shot').classList.add('has-image')">
  </div>
  <div class="shot-todo">
    <span class="id">Figure 3</span>
    <p class="what">What the capture must show, precisely enough for someone else to take it.</p>
    <span class="file">images/thing.png</span>
  </div>
  <figcaption>What to look at, not what it is.</figcaption>
</figure>
```

The `onload` handler is the whole mechanism: a file that exists adds `.has-image`, which shows the image and hides the placeholder. A file that does not exist leaves the slot in place — and:

```css
.shot:not(.has-image){display:none}                    /* readers see nothing */
body.show-figures .shot:not(.has-image){display:block} /* ?figures reveals it */
```

So readers never see "image missing", and `manual.html?figures` is your to-do list rendered in place. No build step, no manifest.

Duplicate figures into both language blocks, suffixing the second id (`-en`).

---

## 5 · Print

The print block is not decoration — students print these before crits, and PDF export is how the manual reaches people who will not open a browser tab.

- sidebar, topbar, search results and copy buttons hidden
- palette forced to black on white (dark mode must not produce a black PDF)
- `section.chap { page-break-before: always }` — one chapter per page
- `page-break-inside: avoid` on callouts, tables, exercises, copyable blocks
- **FAQ answers forced visible** — `details` collapsed in print means blank pages
- `h3, h4 { page-break-after: avoid }` — no heading orphaned at a page foot

Check print preview whenever you add a component type.

---

## 6 · State

```js
localStorage: cb-lang    'fr' | 'en'   (overridden by ?lang= in the URL)
              cb-theme   'light' | 'dark'   (absent = follow the system)
              cb-level   'a12' | 'a3' | 'all'
```

**All courses on one GitHub Pages origin share `localStorage`.** `user.github.io/courseA` and `/courseB` are the same origin. Keep `lang` and `theme` keys shared on purpose — a reader who chose English wants English in every manual. **Namespace the level key per course** (`cb-level-genai`), because band semantics differ between courses and a shared key silently hides content in the wrong one.

Theme: three states. Explicit choice stamps `data-theme` on `<html>`; absent means follow `prefers-color-scheme`. Every colour is a variable defined on bare `:root`, then redefined in *both* the media query (guarded `:root:not([data-theme="light"])`) and `:root[data-theme="dark"]`, so the toggle wins in both directions. Never give a colour its only definition inside a media query.

---

## 7 · Editing a 600 KB file

- Always `assert t.count(old) == 1` before replacing.
- Always `newline=""` when writing, or Python rewrites every line ending and the diff becomes the whole file.
- Anchor on distinctive prose, not on tag soup.
- Both editions in the same script, so you cannot forget one.
- Read the fragment back before writing a large insertion — if the anchor sits inside an SVG or a `<pre>`, indentation matters.
- On Windows, set `PYTHONIOENCODING=utf-8` before printing anything containing `│`, `·` or accents, or the script dies in `cp1252` at the print statement and you debug the wrong thing.
- The single file is canonical. If you assembled it from parts, the parts are stale the moment you edit the file directly — delete them or mark them dead.
