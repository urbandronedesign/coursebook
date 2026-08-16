# Auditing — the seven passes

A bilingual 600 KB single file cannot be proofread by reading. The audit is what makes the format viable: it converts "did I remember to update the English version" from a hope into an exit code.

```bash
python tools/audit.py            # full report
python tools/audit.py --quiet    # failures only
```

Exit code = number of failing passes, so CI needs no wrapper. Standard library only, a couple of seconds.

Configure it once in `CONFIG` at the top of `assets/audit.py`.

---

## The passes

Ordered by how expensive the mistake is to find late.

### 1 · HTML validity

`HTMLParser` subclass tracking a tag stack. Reports stray closes, never-opened closes, and anything left unclosed at EOF, with line numbers and what was still open.

In a file this size, one unclosed `</div>` silently swallows the rest of a chapter into a language wrapper — the FR edition looks fine and the EN edition is missing three sections. This pass is why that never ships.

Note the `VOID` set includes SVG self-closing elements (`path`, `rect`, `use`, `stop`…). Add to it if you use an SVG element the parser then thinks is unclosed.

### 2 · Links

Two checks per page:

- every `href="#…"` resolves to an `id` that exists. Hrefs containing `'` or `+` are skipped — those are built by JS string concatenation and are not real links.
- every local page-to-page link points at a file that exists. With three cross-linking pages, a wrong filename otherwise ships silently and 404s only for readers.

### 3 · Fact parity between editions

The core pass. `LangFilter` re-emits the document twice — once as the French reader sees it, once as the English reader does — then compares, chapter by chapter, everything that names **one reality**:

| Field | Why it must match |
|---|---|
| `sections` | `<h3>` numbers — a renumbering applied to one edition |
| `code` | identifiers in `<code>` — a path renamed in one edition only |
| `xref` | `href="#chN"` — a cross-reference added on one side |
| `tags` | `lv-tag` labels — a band marked in one edition |
| `stages` | exercise numbers |
| `figs` | figure numbers |

A regex cannot do this, because language wrappers nest and a subtree must leave with its parent. That is what `LangFilter` is for, and it is reusable:

```python
import sys; sys.path.insert(0, "tools")
from audit import view, sections
fr = sections(view(html, "fr"))
```

Deliberately **not** compared: prose, word counts, paragraph counts. Translations differ in length; that is correct.

### 4 · Structural parity between editions

Counts per chapter: tables, rows, callouts, copyable blocks, `h3`, `h4`, `li`, `pre`, exercises, FAQ entries. Any difference is reported as `rows FR12/EN11`.

This is what catches a table row added in French and forgotten in English — invisible to a reader of either edition alone, and the single most common real defect in a bilingual document.

Occasionally a difference is legitimate (a French-only note about French terminology). Then say so in a comment at the site, and either accept the failure knowingly or restructure so the count matches. Do not weaken the pass.

### 5 · Identifier sweep

Guarantees a renamed identifier never creeps back — after renaming a folder tree from French to English, for instance.

Two lists:

- `banned_anywhere` — strings distinctive enough that they cannot occur in prose: `02_ecriture`, `feuille_de_plans`, `nom-du-projet`.
- `banned_in_code` — words that **are** ordinary prose in the source language: `plan`, `son`, `lieu`, `duree`, `statut`. These are checked **only inside identifier-shaped strings**.

`literals()` defines identifier-shaped: a single-token `<code>` span, a single-token backticked Markdown token, a file-tree line, a CSV row. That scoping is what lets the pass hunt `son` as a column name while leaving thousands of French sentences containing *son* untouched.

Both lists start empty. Fill them the first time you rename something.

### 6 · Cross-file

The manual quotes things that also exist as real files the reader downloads. They drift.

`CONFIG["paired"]` — one entry per quoted value:

```python
"paired": [
    {"find": r"shot,block,duration[^\n<]*",
     "file": "templates/PROJECT/03_breakdown/shot_sheet.csv",
     "lines": "first"},
],
```

The manual's CSV header must be **byte-identical** to the file's. A student comparing them and finding a mismatch stops trusting both.

`CONFIG["tree"]` — a regex matching each copy of a file tree. Both editions' copies must list the same directories.

Add a check here whenever the manual starts asserting something a file can confirm: a row count, a duration total, a directory list.

### 7 · Level bands

For each band value, counts occurrences inside FR wrappers and inside EN wrappers, and fails if they differ. It resolves the wrapper by walking the `div` stack from the top of the file, so it handles nesting.

An unpaired band means the filter hides different content in each edition — a defect no reader of one language can see. Also fails if a band sits outside any language wrapper.

---

## Reading a failure

The message names the chapter and the field:

```
FAIL  [ch11] rows FR14/EN13
```

Read chapter 11's two blocks side by side, count rows, find the missing one. Not "reduce the French" — find what the English is missing, because the count difference means content is absent, not surplus.

```
FAIL  [ch5] sections  FR-only ['5.4']  EN-only ['5.5']
```

A renumbering applied to one edition only.

```
FAIL  manual.html: 'musique' used as an identifier — │   └── LICENCES/  le texte…
```

Look at what it matched before believing it. Here the "identifier" was a French comment inside a file-tree line — a tooling false positive, fixed by scoping `literals()` to ignore `<span class="cm">` comments, **not** by rewording the prose.

---

## When the audit is wrong

It happens, and the response matters.

**Fix the tool, not the document** — when the document is right and the check is too broad. Real examples from these repos:

- `<h4>` counted literally, missing `<h4 class="fr-only">` → phantom mismatch.
- A topic probe matching `figcaption` when searching for "caption", and matching inside words for short terms → false "already covered".
- The sweep reading a file tree's French comment as an identifier → false positive on `musique`.

Each was a tooling bug. Fixing the prose to satisfy a broken check makes the document worse and leaves the bug for next time.

**Then re-verify before reporting.** After changing the tool, re-run against tag-stripped text and confirm the finding actually disappears for the right reason.

**Never**: delete a failing pass, add an exception for one file, or lower a threshold to get green. A green audit that checks nothing is worse than no audit — it produces confidence without grounds.

---

## Beyond the seven

Ad-hoc checks worth writing when the question comes up. Keep them in the scratchpad, not the repo, unless they earn a permanent place.

**Coverage probe** — is topic X actually in the manual? Search the **tag-stripped** text of both editions, with word boundaries:

```python
import re
txt = re.sub(r"\s+", " ", re.sub(r"<[^>]+>", " ", html))
for name, pat in [("colour space", r"sRGB|Rec\.?\s?709|espace colorim")]:
    n = len(re.findall(pat, txt, re.I))
    print("%-20s %s" % (name, n or "ABSENT"))
```

Anchor patterns with `\b`. Matching `oral` inside *temporal* produces confident nonsense, and that nonsense will end up in a report.

**Table column consistency** — header cell count vs each row's cell count, per table. Catches a column added to `thead` and not to the rows.

**Language purity** — three source-language function words on one line inside the other edition. See `architecture.md` §1.

**External link check** — extract every `href="http…"` and HTTP-check it. Run before every commit that adds links:

```bash
grep -o 'href="https\?://[^"]*"' *.html | sed 's/.*href="//;s/"$//' | sort -u | \
while read u; do printf "  %s %s\n" \
  "$(curl -s -o /dev/null -w '%{http_code}' -L --max-time 15 "$u")" "$u"; done
```

**README drift** — word counts, file sizes, chapter counts and section counts in the README go stale after every substantial edit. Nothing catches this automatically; check it whenever you reframe.
